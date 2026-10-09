# Heap Basics

> **CLRS:** 6.1 (Heaps) · **Status:** ✅ Written · **Prerequisites:** [Asymptotic notation](../../00-Foundations/asymptotic-notation.md), [Trees](../../14-Mathematical-Foundations/trees.md) (helpful) · **Next:** [Maintaining the heap property](maintaining-heap-property.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

A **(binary) heap** is an array that represents a **nearly complete binary tree** and satisfies the **heap property**:

- **Max-heap:** every node's value is **≥** its children's values, so the **largest** element is at the root.
- **Min-heap:** every node's value is **≤** its children's values, so the **smallest** is at the root.

**What problem does it solve?** It gives **instant access to the largest (or smallest) element**, Θ(1), and lets you remove it or insert new elements in **Θ(log n)**, all without fully sorting the data.

**Where it's used**
- [Heapsort](heapsort.md): Θ(n log n) worst-case, in-place sorting.
- [Priority queues](priority-queues.md): Java `PriorityQueue`, OS process schedulers, event simulation.
- Graph algorithms: [Dijkstra](../../07-Graph-Algorithms/shortest-paths/dijkstra.md), [Prim](../../07-Graph-Algorithms/minimum-spanning-trees/prim.md).
- Top-k queries ("the 10 most frequent words"), streaming medians (two heaps), k-way merging, Huffman coding.

> **Don't confuse** a heap (this data structure) with "the heap" (the memory area where Java allocates objects). They're unrelated.

**Prerequisites:** binary trees (root, parent, child, leaf, height) and arrays.

---

## B. Intuition

**Analogy: a company org chart where every boss earns more than their direct reports.** Then:
- the CEO (the root) earns the most, so **finding the maximum is instant**;
- but two people in different departments aren't ordered relative to each other. A heap is **partially** ordered, not sorted.

That "partial order" is the point. Maintaining full sorted order costs Θ(n) per insertion in an array. Maintaining only "parent ≥ child" costs Θ(log n), because a change only affects one root-to-leaf path.

**Why store a tree in an array?** Because the tree is *nearly complete* (every level is full except possibly the last, which fills from the left), node positions follow a fixed pattern. You can compute parent and child positions with arithmetic, with **no pointers needed**. That makes heaps memory-efficient and cache-friendly.

---

## C. How it works internally

### Array ↔ tree mapping

The max-heap from CLRS Figure 6.1, A = [16, 14, 10, 8, 7, 9, 3, 2, 4, 1]:

```
Array (1-indexed, CLRS):   index:  1   2   3   4   5   6   7   8   9   10
                           value: 16  14  10   8   7   9   3   2   4   1

Tree:                              16 (1)
                               /            \
                         14 (2)              10 (3)
                        /      \            /      \
                    8 (4)      7 (5)     9 (6)     3 (7)
                   /    \      /
               2 (8)  4 (9)  1 (10)
```

**Index formulas**

| | 1-indexed (CLRS) | 0-indexed (Java) |
|---|---|---|
| PARENT(i) | ⌊i/2⌋ | (i − 1)/2 |
| LEFT(i) | 2i | 2i + 1 |
| RIGHT(i) | 2i + 1 | 2i + 2 |
| leaves | ⌊n/2⌋ + 1 … n | n/2 … n − 1 |
| last internal node | ⌊n/2⌋ | n/2 − 1 |

On real hardware these are a shift: 2i is `i << 1`, and i/2 is `i >> 1`.

**Check:** node 4 (value 8) has children at 8 and 9 (values 2 and 4), and 8 ≥ 2, 4 ✓. Its parent is ⌊4/2⌋ = 2 (value 14), and 14 ≥ 8 ✓.

### Key properties

| Property | Statement | Why |
|---|---|---|
| **Height** | A heap of n elements has height **⌊log₂ n⌋** | A nearly complete tree with height h has between 2ʰ and 2ʰ⁺¹ − 1 nodes |
| **Max at the root** | A[1] = max | By transitivity: every path from the root goes downhill |
| **Leaves** | About **half** the nodes (⌈n/2⌉) are leaves | Nodes ⌊n/2⌋ + 1 … n have no children |
| **Nodes at height h** | At most **⌈n/2ʰ⁺¹⌉** | Used to prove BUILD-HEAP is O(n) |
| **Minimum (in a max-heap)** | Is one of the leaves | A node with a child is ≥ that child, so it can't be the strict minimum unless they're equal |
| **Not sorted** | Siblings and cousins are unordered | e.g. 8 (index 4) < 9 (index 6), even though index 4 < 6 |

**Is a sorted array a heap?** A descending array is a valid max-heap (each element ≥ everything after it), and an ascending array is a min-heap. The converse is false: a heap usually isn't sorted.

### Edge cases

- Empty heap: n = 0. Operations on an empty heap must throw an error ("heap underflow").
- One element: it's the root and a leaf, and it's trivially a heap.
- Duplicates: allowed. The property uses ≥, not >.

---

## D. Algorithm and pseudocode

```
PARENT(i)  return ⌊i/2⌋
LEFT(i)    return 2i
RIGHT(i)   return 2i + 1

IS-MAX-HEAP(A, n)                      ▷ verify the heap property
1  for i ← 2 to n
2      if A[PARENT(i)] < A[i]
3          return FALSE
4  return TRUE
```

**Correctness of IS-MAX-HEAP:** the heap property says every *parent–child* pair is ordered. Every node except the root has exactly one parent, so checking each i ≥ 2 against its parent covers every edge of the tree exactly once. That takes n − 1 comparisons, Θ(n).

**Heap size vs array length:** CLRS distinguishes `length[A]` (the capacity) from `heap-size[A]` (how many elements are currently in the heap). Elements A[heap-size + 1 .. length] aren't part of the heap. That's exactly how heapsort shrinks the heap while the sorted part grows at the end of the array.

---

## E. Implementation

```java
import java.util.*;

/** Array representation of a binary heap: index arithmetic, property checks and the height formula. */
public class HeapBasics {

    // 0-indexed formulas used throughout this repository's Java code
    static int parent(int i) { return (i - 1) >>> 1; }
    static int left(int i)   { return 2 * i + 1; }
    static int right(int i)  { return 2 * i + 2; }

    /** True if a[0..n-1] satisfies the max-heap property: every node >= its parent's children. */
    static boolean isMaxHeap(int[] a, int n) {
        for (int i = 1; i < n; i++) if (a[parent(i)] < a[i]) return false;
        return true;
    }

    static boolean isMinHeap(int[] a, int n) {
        for (int i = 1; i < n; i++) if (a[parent(i)] > a[i]) return false;
        return true;
    }

    /** Height of a heap with n >= 1 elements = floor(log2 n). */
    static int height(int n) { return 31 - Integer.numberOfLeadingZeros(n); }

    /** Pretty-prints the heap level by level. */
    static String levels(int[] a, int n) {
        StringBuilder sb = new StringBuilder();
        for (int start = 0, size = 1, level = 0; start < n; start += size, size *= 2, level++) {
            sb.append("level ").append(level).append(": ");
            for (int i = start; i < Math.min(n, start + size); i++) sb.append(a[i]).append(' ');
            sb.append('\n');
        }
        return sb.toString();
    }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        int[] a = {16, 14, 10, 8, 7, 9, 3, 2, 4, 1};             // CLRS Figure 6.1 (0-indexed here)
        int n = a.length;
        System.out.print(levels(a, n));
        System.out.println("node at index 3 (value " + a[3] + "): parent index " + parent(3)
                + " (value " + a[parent(3)] + "), children " + left(3) + "," + right(3)
                + " (values " + a[left(3)] + "," + a[right(3)] + ")");
        System.out.println("height = " + height(n) + ", leaves are indices " + (n / 2) + ".." + (n - 1));

        check(isMaxHeap(a, n), "CLRS Figure 6.1 array is a max-heap");
        check(a[0] == Arrays.stream(a).max().getAsInt(), "root holds the maximum");
        check(height(n) == 3, "height of a 10-element heap is floor(log2 10) = 3");

        int[] notHeap = {23, 17, 14, 6, 13, 10, 1, 5, 7, 12};      // CLRS Exercise 6.1-6
        check(!isMaxHeap(notHeap, notHeap.length), "[23,17,14,6,13,10,1,5,7,12] is NOT a max-heap (7 > its parent 6)");

        int[] desc = {9, 8, 7, 6, 5, 4, 3, 2, 1};
        int[] asc  = {1, 2, 3, 4, 5, 6, 7, 8, 9};
        check(isMaxHeap(desc, desc.length) && isMinHeap(asc, asc.length), "a descending array is a max-heap; ascending is a min-heap");

        // The minimum of a max-heap is always among the leaves (indices n/2 .. n-1)
        int minIdx = 0; for (int i = 1; i < n; i++) if (a[i] < a[minIdx]) minIdx = i;
        check(minIdx >= n / 2, "minimum (" + a[minIdx] + ") is a leaf");

        // Height formula vs the definition, for many n
        boolean h = true;
        for (int m = 1; m <= 1 << 16; m++) {
            int levelsCount = 0; for (int last = m - 1; ; last = parent(last)) { levelsCount++; if (last == 0) break; }
            if (levelsCount - 1 != height(m)) h = false;               // depth of the last node = height
        }
        check(h, "height == floor(log2 n) for every n up to 65536");

        // Index formulas are mutual inverses
        boolean idx = true;
        for (int i = 0; i < 100000; i++) if (parent(left(i)) != i || parent(right(i)) != i) idx = false;
        check(idx, "parent(left(i)) == parent(right(i)) == i");
    }
}
```

**Output:**

```
level 0: 16 
level 1: 14 10 
level 2: 8 7 9 3 
level 3: 2 4 1 
node at index 3 (value 8): parent index 1 (value 14), children 7,8 (values 2,4)
height = 3, leaves are indices 5..9
ok   CLRS Figure 6.1 array is a max-heap
ok   root holds the maximum
ok   height of a 10-element heap is floor(log2 10) = 3
ok   [23,17,14,6,13,10,1,5,7,12] is NOT a max-heap (7 > its parent 6)
ok   a descending array is a max-heap; ascending is a min-heap
ok   minimum (1) is a leaf
ok   height == floor(log2 n) for every n up to 65536
ok   parent(left(i)) == parent(right(i)) == i
```

**Java-specific details**
- `(i - 1) >>> 1` is an unsigned shift, equivalent to `(i - 1) / 2` for i ≥ 1. It also gives a harmless value for i = 0, since the root has no parent.
- `31 - Integer.numberOfLeadingZeros(n)` computes ⌊log₂ n⌋ exactly, with no floating-point rounding problems.
- **Java's `PriorityQueue` is a min-heap** stored exactly like this, in an `Object[] queue` array with 0-indexed `2i + 1` and `2i + 2` children. For a max-heap use `new PriorityQueue<>(Comparator.reverseOrder())`.

**Common mistakes:** mixing 1-indexed and 0-indexed formulas (`left = 2i` with a 0-indexed array makes the root its own left child), and believing a heap is sorted.

---

## F. Time complexity

| Operation | Time | Why |
|---|---|---|
| Find max (root) | **Θ(1)** | A[0] |
| PARENT, LEFT, RIGHT | Θ(1) | arithmetic |
| IS-MAX-HEAP | Θ(n) | n − 1 parent–child comparisons |
| Height | Θ(log n) by formula | ⌊log₂ n⌋ |
| Find min in a max-heap | Θ(n) | it could be any leaf, and there are about n/2 leaves |
| Search for an arbitrary value | Θ(n) | the heap order doesn't guide the search (unlike a [BST](../../04-Trees/binary-search-trees/bst-basics.md)) |

**Height derivation:** a nearly complete binary tree of height h has full levels 0..h − 1 (holding 1 + 2 + … + 2ʰ⁻¹ = 2ʰ − 1 nodes) plus between 1 and 2ʰ nodes on level h. So 2ʰ ≤ n ≤ 2ʰ⁺¹ − 1, and taking logs gives h ≤ log₂ n < h + 1, so **h = ⌊log₂ n⌋**. Every heap operation that walks one path is therefore O(log n).

## G. Space complexity

**Θ(n)** for the array itself, and **zero pointer overhead**. A pointer-based tree would need 2–3 references per node (16–24 extra bytes each in Java). Auxiliary space for the operations above is Θ(1).

## H. Complexity summary

| Operation | Binary heap | Sorted array | Unsorted array | Balanced BST |
|---|---|---|---|---|
| Find max | **Θ(1)** | Θ(1) | Θ(n) | Θ(log n) |
| Insert | **Θ(log n)** | Θ(n) | Θ(1) | Θ(log n) |
| Extract max | **Θ(log n)** | Θ(1) (from the end) | Θ(n) | Θ(log n) |
| Search arbitrary | Θ(n) | Θ(log n) | Θ(n) | Θ(log n) |
| Build from n items | **Θ(n)** | Θ(n log n) | Θ(n) | Θ(n log n) |

The insert and extract operations are explained in [Priority queues](priority-queues.md), and build in [Building a heap](building-a-heap.md).

## I. Advantages, limitations, and comparisons

**Advantages:** Θ(1) max, Θ(log n) updates, Θ(n) build, compact (no pointers), cache-friendly.

**Limitations:** no efficient search or ordered iteration. Merging two heaps is Θ(n); use [binomial](../../04-Trees/binomial-heaps/binomial-trees.md) or [Fibonacci heaps](../../04-Trees/fibonacci-heaps/structure.md) for fast merging.

**Variants:** d-ary heaps (d children per node: shallower, with cheaper decrease-key, used in Dijkstra), min-max heaps, and pairing heaps.

**Interview follow-ups:** "What's the difference between a heap and a BST?" (a heap orders parents and children only; a BST orders left, node and right), "Where's the minimum of a max-heap?" (among the leaves), "Is an array sorted in descending order a heap?" (yes).

---

## J. Practice

**Beginner**
1. What are the minimum and maximum numbers of elements in a heap of height h? (CLRS Exercise 6.1-1)
2. Is [23, 17, 14, 6, 13, 10, 1, 5, 7, 12] a max-heap? (CLRS Exercise 6.1-6)
3. Give the 0-indexed children of index 4.

**Intermediate**
4. Show that an n-element heap has height ⌊lg n⌋ (CLRS Exercise 6.1-2).
5. Show that the leaves are exactly the nodes indexed ⌊n/2⌋ + 1, …, n (CLRS Exercise 6.1-7).
6. In a max-heap of distinct elements, where can the second-largest element be?

**Advanced**
7. Give the index formulas for a d-ary heap.
8. Prove there are at most ⌈n/2ʰ⁺¹⌉ nodes of height h in an n-element heap.

**Interview questions**

<details><summary>Q1. Why is a heap stored in an array rather than with pointers?</summary>

A heap is always a nearly complete binary tree, so node i's children are at 2i + 1 and 2i + 2 and its parent at (i − 1)/2. Arithmetic replaces pointers: no memory overhead, better cache locality, and Θ(1) navigation.
</details>

<details><summary>Q2. Can you search for an element in a heap in O(log n)?</summary>

No. The heap property only orders each parent relative to its children, so a value could be in either subtree. Searching is Θ(n). If you need search, use a balanced BST, or a hash map alongside the heap (an "indexed priority queue").
</details>

<details><summary>Q3. Where is the smallest element in a max-heap?</summary>

At one of the leaves (indices n/2 … n − 1 when 0-indexed), since any internal node is ≥ its children. Finding it requires scanning about n/2 leaves, Θ(n).
</details>

**Worked problem: check whether an array represents a max-heap.** For each i ≥ 1, check a[(i − 1)/2] ≥ a[i]. That's Θ(n) time and Θ(1) space, and it's exactly `isMaxHeap` above. There's no need to recurse.

**Coding problems**
- GeeksforGeeks · Check if an array represents a binary heap *(verify link)*
- LeetCode 1046 · Last Stone Weight (simple max-heap use) *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. A heap is a nearly complete binary tree stored in an array, with parent ≥ children (max-heap).
2. 0-indexed: parent (i − 1)/2, children 2i + 1 and 2i + 2.
3. Height ⌊log₂ n⌋, so path operations are O(log n).
4. The max is at the root. A heap is **not** sorted, and search is Θ(n).
5. About half the nodes are leaves (indices n/2 … n − 1).

**Formulas:** 2ʰ ≤ n ≤ 2ʰ⁺¹ − 1. At most ⌈n/2ʰ⁺¹⌉ nodes at height h.

**Common mistakes:** mixing index conventions, assuming sortedness, and expecting O(log n) search.

**Quiz**
1. 0-indexed parent of index 9?
2. Height of a heap with 100 elements?
3. Is [10, 9, 8, 7, 6] a max-heap?
4. How many leaves does a 15-node heap have?

**Answers**

<details><summary>Show answers</summary>

1. (9 − 1)/2 = 4.
2. ⌊log₂ 100⌋ = 6.
3. Yes. Any descending array is a max-heap.
4. 8 (indices 7..14 when 0-indexed, which is ⌈15/2⌉).
</details>

**Related topics:** [Maintaining the heap property](maintaining-heap-property.md) · [Building a heap](building-a-heap.md) · [Heapsort](heapsort.md) · [Priority queues](priority-queues.md) · [Binomial heaps](../../04-Trees/binomial-heaps/binomial-trees.md)
