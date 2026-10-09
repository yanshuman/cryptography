# Maintaining the Heap Property (MAX-HEAPIFY)

> **CLRS:** 6.2 · **Status:** ✅ Written · **Prerequisites:** [Heap basics](heap-basics.md), [Recurrence relations](../../00-Foundations/recurrence-relations.md) · **Next:** [Building a heap](building-a-heap.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

**MAX-HEAPIFY(A, i)** fixes a heap in which **only node i** might violate the max-heap property: A[i] might be smaller than one of its children, **but both of i's subtrees are already valid max-heaps**. It lets A[i] "**sift down**" (also called *bubble down* or *percolate down*) until the property holds again.

**What problem does it solve?** Whenever the root of a heap gets replaced by a smaller value (after extracting the max in heapsort or a priority queue), the heap breaks at that one spot. MAX-HEAPIFY repairs it in **O(log n)**, without rebuilding everything.

**Where it's used:** it's the core subroutine of [BUILD-MAX-HEAP](building-a-heap.md), [HEAPSORT](heapsort.md) and [HEAP-EXTRACT-MAX](priority-queues.md). Java's `PriorityQueue.poll()` calls its own version, `siftDown`.

**Prerequisites:** heap index formulas and the heap property.

---

## B. Intuition

**Analogy: a new, under-qualified manager at the top of an org chart.** The rule is that every boss outranks their reports. The new person at node i is outranked by one of their reports, so they **swap places with their best report** (the larger child). Now that report is the boss, which is fine, since they outrank their old colleague, the other child. The new person keeps sinking until they're above everyone below them, or they reach the bottom.

**Why swap with the *larger* child?** After the swap, the promoted child becomes the parent of the other child. Only the larger child is guaranteed to be ≥ its sibling. Swapping with the smaller child would create a new violation.

**Why only one path?** Each swap fixes the current spot, and the only possible new problem is one level lower, at the position the value moved into. So the work follows a single root-to-leaf path: **O(height) = O(log n)**.

---

## C. How it works internally

**CLRS Figure 6.2:** A = [16, **4**, 10, 14, 7, 9, 3, 2, 8, 1] (1-indexed). Call MAX-HEAPIFY(A, 2). Node 2 (value 4) violates the property, and both of its subtrees are heaps.

**Step 1:** at i = 2 (value 4), the children are 14 (index 4) and 7 (index 5). The largest is 14, so swap.

```
            16                                16
         /      \                          /      \
      [4]        10           →         14         10
     /   \      /  \                   /   \      /  \
   14     7    9    3               [4]     7    9    3
  /  \   /                          /  \   /
 2    8 1                          2    8 1
```

**Step 2:** at i = 4 (value 4), the children are 2 (index 8) and 8 (index 9). The largest is 8, so swap.

```
            16
         /      \
       14        10
      /   \     /  \
     8     7   9    3
    / \   /
   2  [4] 1
```

**Step 3:** at i = 9 (value 4), it's a leaf with no children, so stop.

> Result: A = [16, 14, 10, 8, 7, 9, 3, 2, 4, 1], a valid max-heap. Two swaps, which is at most the height (3).

**Edge cases**

| Situation | Behaviour |
|---|---|
| i is a leaf | no children, so return immediately |
| i has only a left child | compare with the left child only. The right index is ≥ heap-size. |
| A[i] ≥ both children | no swap, so it's Θ(1) |
| Equal values | no swap needed (≥ holds). Using `>` avoids pointless swaps. |
| Subtrees aren't heaps | **the precondition is violated**, and MAX-HEAPIFY won't fix them. That's why BUILD-HEAP works bottom-up. |

---

## D. Algorithm and pseudocode

```
MAX-HEAPIFY(A, i)
1   l ← LEFT(i)
2   r ← RIGHT(i)
3   if l ≤ heap-size[A] and A[l] > A[i]
4       largest ← l
5   else largest ← i
6   if r ≤ heap-size[A] and A[r] > A[largest]
7       largest ← r
8   if largest ≠ i
9       exchange A[i] ↔ A[largest]
10      MAX-HEAPIFY(A, largest)
```

| Line | Meaning |
|---|---|
| 1–2 | Child indices |
| 3–7 | Find the largest among A[i], A[l] and A[r], making sure each child exists (l, r ≤ heap-size) |
| 8 | If i already holds the largest, the subtree rooted at i is a heap, so we're done |
| 9 | Promote the larger child. Position i is now correct. |
| 10 | The old A[i] now sits at `largest` and might violate the property *there*, so recurse. The subtrees below it are still heaps. |

### Correctness (by induction on the height of node i)

**Precondition:** the subtrees rooted at LEFT(i) and RIGHT(i) are max-heaps.
**Postcondition:** the subtree rooted at i is a max-heap.

- **Height 0 (a leaf):** a single node is a heap. ✓
- **Height h:** if A[i] ≥ both children, then since both subtrees are heaps, the whole subtree is a heap. ✓ Otherwise we swap A[i] with the larger child, say A[l]. Now:
  - the new A[i] (the old A[l]) is ≥ the old A[i] (now at l) and ≥ A[r] (because it was the larger child), so position i is fine;
  - the right subtree is untouched, so it's still a heap;
  - the left subtree is a heap except possibly at its root l. Its own subtrees are unchanged heaps, so the precondition holds there. MAX-HEAPIFY(A, l) is called on a node of height h − 1, and by induction it makes the subtree at l a heap. ✓

**Termination:** each recursive call moves to a child, which strictly increases the index, and the recursion stops at a leaf or when no swap is needed. That's at most h calls.

### Iterative version (no recursion: preferred in practice)

```
MAX-HEAPIFY-ITERATIVE(A, i, n)
1  while true
2      largest ← index of max among i, LEFT(i), RIGHT(i) that are ≤ n
3      if largest = i then return
4      exchange A[i] ↔ A[largest]
5      i ← largest
```

**Optimisation: "hole" sifting.** Instead of swapping (3 writes per level), hold the value in a temporary variable, move the larger child up into the hole at each level, and write the held value once at the end. That's 1 write per level plus 1. Java's `PriorityQueue.siftDown` does this.

---

## E. Implementation

```java
import java.util.*;

/** MAX-HEAPIFY: recursive (CLRS), iterative with "hole" sifting, and checks of the O(log n) bound. */
public class MaxHeapify {

    static int swaps;

    static int left(int i)  { return 2 * i + 1; }
    static int right(int i) { return 2 * i + 2; }

    /** CLRS MAX-HEAPIFY, 0-indexed. Precondition: subtrees of i are max-heaps within a[0..n-1]. */
    static void heapifyRecursive(int[] a, int i, int n) {
        int l = left(i), r = right(i), largest = i;
        if (l < n && a[l] > a[largest]) largest = l;
        if (r < n && a[r] > a[largest]) largest = r;
        if (largest != i) {
            int t = a[i]; a[i] = a[largest]; a[largest] = t;
            swaps++;
            heapifyRecursive(a, largest, n);
        }
    }

    /** Iterative sift-down that moves a "hole" instead of swapping: 1 write per level. */
    static void heapify(int[] a, int i, int n) {
        int value = a[i];
        while (true) {
            int child = left(i);
            if (child >= n) break;                                  // i is a leaf
            if (child + 1 < n && a[child + 1] > a[child]) child++;  // pick the larger child
            if (a[child] <= value) break;                           // value fits here
            a[i] = a[child];                                        // move the child up into the hole
            i = child;
        }
        a[i] = value;
    }

    static boolean isMaxHeap(int[] a, int n) {
        for (int i = 1; i < n; i++) if (a[(i - 1) / 2] < a[i]) return false;
        return true;
    }

    static int height(int n) { return 31 - Integer.numberOfLeadingZeros(n); }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        int[] a = {16, 4, 10, 14, 7, 9, 3, 2, 8, 1};              // CLRS Figure 6.2, node 2 = index 1 here
        System.out.println("before: " + Arrays.toString(a));
        swaps = 0;
        heapifyRecursive(a, 1, a.length);
        System.out.println("after:  " + Arrays.toString(a) + "  swaps=" + swaps);
        check(Arrays.equals(a, new int[]{16, 14, 10, 8, 7, 9, 3, 2, 4, 1}), "matches CLRS Figure 6.2");
        check(swaps == 2, "two swaps (4 -> 14, then 4 -> 8)");

        int[] b = {16, 4, 10, 14, 7, 9, 3, 2, 8, 1};
        heapify(b, 1, b.length);
        check(Arrays.equals(a, b), "iterative hole-sifting gives the same result");

        // Random test: build a valid heap, overwrite the root with a small value, heapify, check.
        Random rnd = new Random(9);
        boolean ok = true, bound = true;
        for (int t = 0; t < 5000; t++) {
            int n = 1 + rnd.nextInt(500);
            Integer[] boxed = rnd.ints(n, 0, 1000).boxed().toArray(Integer[]::new);
            Arrays.sort(boxed, Collections.reverseOrder());          // descending array = valid max-heap
            int[] h = Arrays.stream(boxed).mapToInt(Integer::intValue).toArray();
            int[] h2 = h.clone();
            h[0] = h2[0] = -rnd.nextInt(1000);                      // break the property at the root only
            swaps = 0;
            heapifyRecursive(h, 0, n);
            heapify(h2, 0, n);
            if (!isMaxHeap(h, n) || !isMaxHeap(h2, n)) ok = false;
            if (swaps > height(n)) bound = false;
        }
        check(ok, "5000 random heaps repaired by both versions");
        check(bound, "swaps never exceed the height floor(log2 n)");

        // Worst case: a tiny value at the root of a full heap sinks all the way to a leaf.
        int n = (1 << 15) - 1;                                       // perfect tree, height 14
        int[] full = new int[n];
        for (int i = 0; i < n; i++) full[i] = n - i;                 // descending = heap
        full[0] = -1;
        swaps = 0; heapifyRecursive(full, 0, n);
        System.out.println("n=" + n + " (height " + height(n) + "): worst-case swaps = " + swaps);
        check(swaps == height(n), "worst case = height = log2 n swaps");
    }
}
```

**Output:**

```
before: [16, 4, 10, 14, 7, 9, 3, 2, 8, 1]
after:  [16, 14, 10, 8, 7, 9, 3, 2, 4, 1]  swaps=2
ok   matches CLRS Figure 6.2
ok   two swaps (4 -> 14, then 4 -> 8)
ok   iterative hole-sifting gives the same result
ok   5000 random heaps repaired by both versions
ok   swaps never exceed the height floor(log2 n)
n=32767 (height 14): worst-case swaps = 14
ok   worst case = height = log2 n swaps
```

**Java-specific details**
- **Prefer the iterative version.** Recursion depth is only O(log n), so it can't overflow, but the loop avoids call overhead and is what libraries use.
- **Bounds checks matter:** `child + 1 < n` must be tested *before* reading `a[child + 1]`.
- Pass the **heap size n** explicitly instead of using `a.length`, because heapsort shrinks the heap inside the same array.

**Common mistakes**
1. Swapping with the *left* child whenever it's larger than A[i], without checking that the right child is larger still.
2. Using `a.length` instead of the current heap size.
3. Calling heapify on a node whose subtrees *aren't* heaps and expecting the whole tree to be fixed. It only repairs a single violation at node i.

---

## F. Time complexity

**Work per level:** Θ(1) (two comparisons and at most one swap). **Number of levels:** at most the height of node i.

> **T = O(h) = O(log n)**, where h is the height of node i.

### The CLRS recurrence: why "2n/3"?

CLRS writes **T(n) ≤ T(2n/3) + Θ(1)**. The recursive call works on one child's subtree. How big can a child's subtree be? It's largest when the **bottom level is exactly half full**, so the left subtree is full one level deeper than the right:

```
Left subtree has height h with 2^(h+1) − 1 nodes; right subtree has height h − 1 with 2^h − 1 nodes.
Total n = 1 + (2^(h+1) − 1) + (2^h − 1) = 3·2^h − 1
Left / n = (2^(h+1) − 1) / (3·2^h − 1) ≤ 2/3
```

So each recursive call is on at most 2n/3 nodes. Master Theorem: a = 1, b = 3/2, n⁰ = 1, f = Θ(1), which is Case 2, so **T(n) = O(log n)**.

| Case | Time | When |
|---|---|---|
| Best | **Θ(1)** | A[i] already ≥ both children |
| Worst | **Θ(log n)** | the value sinks to a leaf (measured above: exactly 14 swaps for height 14) |
| On a node of height h | Θ(h) worst case | this sharper bound is what makes [BUILD-HEAP O(n)](building-a-heap.md) |

## G. Space complexity

- **Recursive (CLRS):** Θ(log n) stack in the worst case. It's tail-recursive, but Java doesn't optimise that.
- **Iterative:** **Θ(1)** auxiliary. It works **in place** in the array.

## H. Complexity summary

| Operation | Best | Worst | Auxiliary space |
|---|---|---|---|
| MAX-HEAPIFY on node i of height h | Θ(1) | Θ(h) | Θ(1) iterative, Θ(h) recursive |
| MAX-HEAPIFY at the root | Θ(1) | Θ(log n) | Θ(1) iterative |

## I. Advantages, limitations, and comparisons

- **Sift-down vs sift-up:** sift-*down* (this page) fixes a node that may be **too small**, by moving it toward the leaves, comparing with 2 children per level. Sift-*up* fixes a node that may be **too large**, by moving it toward the root, comparing with 1 parent per level. Sift-up is used by INSERT and INCREASE-KEY. See [Priority queues](priority-queues.md).
- **Floyd's trick (used in heapsort):** sink the value straight to a leaf, choosing the larger child at each level *without* comparing against the value, then sift it back up. On average the value belongs near the bottom, so this saves about half the comparisons.

**Interview follow-ups:** "Write siftDown for a min-heap", "Why compare with both children?", "Why does BUILD-HEAP go bottom-up?" (because heapify needs both subtrees to already be heaps).

---

## J. Practice

**Beginner**
1. Trace MAX-HEAPIFY(A, 3) on A = [27, 17, 3, 16, 13, 10, 1, 5, 7, 12, 4, 8, 9, 0] (CLRS Exercise 6.2-1).
2. Write MIN-HEAPIFY (CLRS Exercise 6.2-2).
3. What happens if A[i] is larger than both children? (CLRS Exercise 6.2-3)

**Intermediate**
4. What happens when i > heap-size/2? (CLRS Exercise 6.2-4)
5. Rewrite MAX-HEAPIFY iteratively (CLRS Exercise 6.2-5).
6. Show that the worst-case running time on a heap of size n is Ω(lg n) (CLRS Exercise 6.2-6).

**Advanced**
7. Implement Floyd's "sink to the bottom, then sift up" variant and count the comparisons saved.
8. Write sift-down for a d-ary heap. How does the cost per level depend on d?

**Interview questions**

<details><summary>Q1. What does heapify assume, and what does it guarantee?</summary>

It assumes both subtrees of node i are already heaps, so only node i may violate the property. It guarantees the subtree rooted at i becomes a heap, in O(height of i) time.
</details>

<details><summary>Q2. Why must you swap with the larger child?</summary>

The child that moves up becomes the parent of the other child. Only the larger child is guaranteed to be ≥ its sibling. Promoting the smaller one would create a new violation.
</details>

<details><summary>Q3. What's the time complexity of heapify, and why?</summary>

O(log n). Each step is constant work and moves one level down, and the height of a heap is ⌊log₂ n⌋. More precisely it's O(h) for a node of height h.
</details>

**Worked problem: replace the max in a heap.** To replace the root with a new value x (the "replace-top" or `heapreplace` operation): set A[0] = x and call heapify(0). That's O(log n), and cheaper than doing a separate extract-max and insert. It's used in top-k streaming: keep a min-heap of size k, and if a new item beats the root, replace the root.

**Coding problems**
- LeetCode 215 · Kth Largest Element in an Array (heap of size k) *(verify link)*
- LeetCode 703 · Kth Largest Element in a Stream *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. MAX-HEAPIFY sifts node i down, swapping with the larger child until it's ≥ both children.
2. Precondition: both subtrees are already heaps.
3. Cost: O(height of i) = O(log n). The recurrence is T(n) ≤ T(2n/3) + Θ(1).
4. The iterative "hole" version uses Θ(1) space and fewer writes.
5. It's the building block of BUILD-HEAP, HEAPSORT and EXTRACT-MAX.

**Formulas:** the largest child subtree is ≤ 2n/3 nodes. Swaps ≤ ⌊log₂ n⌋.

**Common mistakes:** not comparing with both children, using the array length instead of the heap size, and using it on non-heap subtrees.

**Quiz**
1. What's the max number of swaps for n = 1000?
2. Why 2n/3 in the recurrence?
3. Sift-down or sift-up for EXTRACT-MAX?
4. What's the best-case time?

**Answers**

<details><summary>Show answers</summary>

1. ⌊log₂ 1000⌋ = 9.
2. The largest a child's subtree can be (when the bottom level is half full) is 2n/3 of the nodes.
3. Sift-down (the last leaf moves to the root and sinks).
4. Θ(1), when the node is already ≥ its children.
</details>

**Related topics:** [Heap basics](heap-basics.md) · [Building a heap](building-a-heap.md) · [Heapsort](heapsort.md) · [Priority queues](priority-queues.md)
