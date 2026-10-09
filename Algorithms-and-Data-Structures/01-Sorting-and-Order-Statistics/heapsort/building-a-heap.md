# Building a Heap (BUILD-MAX-HEAP)

> **CLRS:** 6.3 · **Status:** ✅ Written · **Prerequisites:** [Maintaining the heap property](maintaining-heap-property.md), [Summations](../../14-Mathematical-Foundations/summations.md) · **Next:** [Heapsort](heapsort.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

**BUILD-MAX-HEAP** turns an **arbitrary array** into a max-heap, in place, by calling [MAX-HEAPIFY](maintaining-heap-property.md) on every non-leaf node, **from the bottom up**.

**What problem does it solve?** Before you can use a heap (for [heapsort](heapsort.md), or a [priority queue](priority-queues.md) loaded with initial data) you need one. Inserting n items one at a time costs Θ(n log n). BUILD-MAX-HEAP does it in **Θ(n)**, a surprising and important result.

**Where it's used:** step 1 of heapsort. `new PriorityQueue<>(collection)` in Java (it calls `heapify()`, which is Θ(n)). Python's `heapq.heapify`. And top-k algorithms that start from a full array.

**Prerequisites:** MAX-HEAPIFY and its precondition (both subtrees must already be heaps). The O(n) proof uses the summation Σ h/2ʰ.

---

## B. Intuition

**Why bottom-up?** MAX-HEAPIFY only works when the node's subtrees are already heaps. **Leaves are trivially heaps**, and about half the array is leaves. So start at the last *internal* node and move backwards toward the root: by the time you reach a node, both of its subtrees have already been fixed.

**Analogy: organising a tournament bracket from the first round up.** Settle every small group first, then the semi-finals, and finally the final. Each stage relies on the stage below already being settled.

**Why it's only O(n) (the surprise):** most nodes are near the **bottom**, and nodes near the bottom can only sink a little. Half the nodes (the leaves) do **no** work. A quarter can sink at most 1 level, an eighth at most 2 levels, and so on. Only the single root can sink log n levels. The total work is dominated by the many cheap nodes, and it adds up to less than 2n.

---

## C. How it works internally

**CLRS Figure 6.3:** A = [4, 1, 3, 2, 16, 9, 10, 14, 8, 7] (1-indexed, n = 10). The non-leaf nodes are indices ⌊10/2⌋ = 5 down to 1.

```
Initial:                4
                     /     \
                   1         3
                 /   \     /   \
                2     16  9     10
               / \   /
             14   8 7
```

| Step | heapify(i) | Value at i | Action | Array after |
|---|---|---|---|---|
| 1 | i = 5 | 16 | child 7 is smaller, so no swap | [4, 1, 3, 2, 16, 9, 10, 14, 8, 7] |
| 2 | i = 4 | 2 | children 14 and 8, swap with 14 | [4, 1, 3, **14**, 16, 9, 10, **2**, 8, 7] |
| 3 | i = 3 | 3 | children 9 and 10, swap with 10 | [4, 1, **10**, 14, 16, 9, **3**, 2, 8, 7] |
| 4 | i = 2 | 1 | children 14 and 16, swap with 16; then at i = 5, child 7: swap | [4, **16**, 10, 14, **7**, 9, 3, 2, 8, **1**] |
| 5 | i = 1 | 4 | children 16 and 10, swap with 16; at i = 2, children 14 and 7: swap with 14; at i = 4, children 2 and 8: swap with 8 | [**16**, **14**, 10, **8**, 7, 9, 3, 2, **4**, 1] |

```
Final max-heap:        16
                     /     \
                   14        10
                 /   \     /   \
                8     7   9     3
               / \   /
              2   4 1
```

Total swaps: 0 + 1 + 1 + 2 + 3 = **7** (compared with a naive bound of n·log n ≈ 33).

**Edge cases:** n ≤ 1 (the loop runs 0 times). An already-valid heap (every heapify returns immediately, about 2·(n/2) comparisons). A sorted ascending array (the worst case for swaps, but still Θ(n)).

---

## D. Algorithm and pseudocode

```
BUILD-MAX-HEAP(A)
1  heap-size[A] ← length[A]
2  for i ← ⌊length[A]/2⌋ downto 1
3      MAX-HEAPIFY(A, i)
```

**Why start at ⌊n/2⌋?** Nodes ⌊n/2⌋ + 1 … n are leaves ([Heap basics](heap-basics.md)), and heapifying a leaf does nothing.
**Why go downward (from ⌊n/2⌋ to 1)?** So that every node's children are processed, and are already heap roots, before the node itself.

### Loop invariant (CLRS §6.3)

> At the start of each iteration of the `for` loop, each node i + 1, i + 2, …, n is the root of a max-heap.

- **Initialization:** i = ⌊n/2⌋, and nodes ⌊n/2⌋ + 1 … n are leaves, which are trivial max-heaps. ✓
- **Maintenance:** node i's children are 2i and 2i + 1, both greater than i, so by the invariant both are roots of max-heaps. That's exactly MAX-HEAPIFY's precondition, so after the call node i is a heap root. Nodes i + 1 … n remain heap roots (heapify only changes i's subtree, and leaves it a heap). Decrementing i re-establishes the invariant. ✓
- **Termination:** i = 0, so every node 1 … n is a heap root. In particular **node 1 is**, so the whole array is a max-heap. ✓

### The alternative: build by repeated insertion (top-down)

```
BUILD-MAX-HEAP′(A)                     ▷ CLRS Problem 6-1
1  heap-size[A] ← 1
2  for i ← 2 to length[A]
3      MAX-HEAP-INSERT(A, A[i])        ▷ sift-up, O(log i)
```

It's correct, but **Θ(n log n) in the worst case** (ascending input: each new element sifts all the way up to the root). It may also produce a *different* heap from BUILD-MAX-HEAP, since heaps aren't unique. Its advantage is that it works **online**, as elements arrive.

---

## E. Implementation

```java
import java.util.*;

/** BUILD-MAX-HEAP (bottom-up, Theta(n)) vs building by repeated insertion (Theta(n log n) worst). */
public class BuildHeap {

    static long swaps;

    static void heapify(int[] a, int i, int n) {               // iterative MAX-HEAPIFY
        while (true) {
            int l = 2 * i + 1, r = l + 1, largest = i;
            if (l < n && a[l] > a[largest]) largest = l;
            if (r < n && a[r] > a[largest]) largest = r;
            if (largest == i) return;
            int t = a[i]; a[i] = a[largest]; a[largest] = t;
            swaps++;
            i = largest;
        }
    }

    /** Bottom-up: heapify every internal node, last to first. Theta(n). */
    static void buildMaxHeap(int[] a) {
        int n = a.length;
        for (int i = n / 2 - 1; i >= 0; i--) heapify(a, i, n);  // n/2 - 1 = last internal node (0-indexed)
    }

    /** Top-down: insert elements one by one with sift-up. Theta(n log n) worst case. */
    static void buildByInsertion(int[] a) {
        for (int size = 1; size < a.length; size++) {
            int i = size;                                       // the new element is a[size]; sift it up
            while (i > 0 && a[(i - 1) / 2] < a[i]) {
                int p = (i - 1) / 2;
                int t = a[i]; a[i] = a[p]; a[p] = t;
                swaps++;
                i = p;
            }
        }
    }

    static boolean isMaxHeap(int[] a) {
        for (int i = 1; i < a.length; i++) if (a[(i - 1) / 2] < a[i]) return false;
        return true;
    }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        int[] a = {4, 1, 3, 2, 16, 9, 10, 14, 8, 7};              // CLRS Figure 6.3
        swaps = 0;
        buildMaxHeap(a);
        System.out.println("BUILD-MAX-HEAP: " + Arrays.toString(a) + "  swaps=" + swaps);
        check(Arrays.equals(a, new int[]{16, 14, 10, 8, 7, 9, 3, 2, 4, 1}), "matches CLRS Figure 6.3");
        check(swaps == 7, "7 swaps, as in the trace");

        Random rnd = new Random(1);
        boolean ok = true, bound = true;
        for (int t = 0; t < 3000; t++) {
            int[] x = rnd.ints(rnd.nextInt(400), -500, 500).toArray();
            swaps = 0; buildMaxHeap(x);
            if (!isMaxHeap(x)) ok = false;
            if (swaps > x.length) bound = false;                  // proof gives swaps <= n - (number of 1 bits) < n
        }
        check(ok, "3000 random arrays become valid max-heaps");
        check(bound, "swaps never exceed n");

        System.out.printf("%n%10s %22s %22s %14s%n", "n", "bottom-up swaps", "insertion swaps", "n log2 n");
        for (int n : new int[]{1 << 10, 1 << 14, 1 << 18, 1 << 20}) {
            int[] asc = new int[n];
            for (int i = 0; i < n; i++) asc[i] = i;               // ascending: worst case for both methods
            int[] b = asc.clone(), c = asc.clone();
            swaps = 0; buildMaxHeap(b); long bu = swaps;
            swaps = 0; buildByInsertion(c); long ins = swaps;
            System.out.printf("%10d %15d (%.2f n) %15d (%.2f n) %14d%n", n, bu, (double) bu / n, ins, (double) ins / n,
                    (long) n * (31 - Integer.numberOfLeadingZeros(n)));
            if (!(isMaxHeap(b) && isMaxHeap(c))) ok = false;
            if (bu >= n) bound = false;
        }
        check(ok && bound, "bottom-up stays below n swaps; insertion grows like n log n");
    }
}
```

**Output:**

```
BUILD-MAX-HEAP: [16, 14, 10, 8, 7, 9, 3, 2, 4, 1]  swaps=7
ok   matches CLRS Figure 6.3
ok   7 swaps, as in the trace
ok   3000 random arrays become valid max-heaps
ok   swaps never exceed n

         n        bottom-up swaps        insertion swaps       n log2 n
      1024            1015 (0.99 n)            8204 (8.01 n)          10240
     16384           16371 (1.00 n)          196624 (12.00 n)         229376
    262144          262127 (1.00 n)         4194324 (16.00 n)        4718592
   1048576         1048557 (1.00 n)        18874390 (18.00 n)       20971520
ok   bottom-up stays below n swaps; insertion grows like n log n
```

**What the table shows:** bottom-up building uses about **1.00·n** swaps at every size (linear). Repeated insertion uses 8n, 12n, 16n, 18n, which keeps growing with log n (about (log₂ n − 2)·n), so it's Θ(n log n).

**Java-specific details:** `new PriorityQueue<>(list)` uses the Θ(n) bottom-up `heapify()`. Calling `pq.addAll(list)` or `add` in a loop is Θ(n log n). For a large initial load, use the constructor.

**Common mistakes:** looping *upward* (from 0 to n/2), which breaks the invariant because children aren't heaps yet; and starting at n/2 instead of n/2 − 1 in 0-indexed code. (That one is harmless but wasteful: it heapifies a leaf.)

---

## F. Time complexity

### The easy (loose) bound: O(n log n)

There are n/2 calls to MAX-HEAPIFY, each O(log n), so O(n log n). That's true, but not tight.

### The tight bound: O(n)

A call on a node of **height h** costs O(h). An n-element heap has **at most ⌈n/2ʰ⁺¹⌉ nodes of height h** ([Heap basics](heap-basics.md)). So:

> T(n) = Σ_{h=0}^{⌊lg n⌋} ⌈n/2ʰ⁺¹⌉ · O(h) = O( n · Σ_{h=0}^{∞} h/2ʰ )

The series Σ h·xʰ = x/(1 − x)² (differentiate the geometric series). At x = ½:

> Σ_{h=0}^{∞} h/2ʰ = (½)/(½)² = **2**

So **T(n) = O(n · 2) = O(n)**. Since every element must be looked at, it's also Ω(n), and therefore **Θ(n)**.

**Intuition table** (n = 15, a perfect tree):

| Height h | Nodes at this height | Max sink distance | Max work |
|---|---|---|---|
| 0 (leaves) | 8 | 0 | 0 |
| 1 | 4 | 1 | 4 |
| 2 | 2 | 2 | 4 |
| 3 (root) | 1 | 3 | 3 |
| **Total** | 15 | | **11 < n** |

**Exact worst case:** the maximum number of swaps is n − (number of 1 bits in the binary representation of n), which is always < n. For n = 1024 the bound is 1023 (1024 has one 1 bit), and the measured ascending input used 1015, just under it.

| Method | Best | Worst | Online? |
|---|---|---|---|
| BUILD-MAX-HEAP (bottom-up) | Θ(n) | **Θ(n)** | ❌ |
| Repeated insertion (top-down) | Θ(n) (descending input) | **Θ(n log n)** | ✅ |

## G. Space complexity

**Θ(1) auxiliary** with iterative heapify (Θ(log n) stack with the recursive version). It's in place: the heap is built inside the input array.

## H. Complexity summary

| Operation | Time | Auxiliary space |
|---|---|---|
| BUILD-MAX-HEAP | Θ(n) | Θ(1) |
| Build by n insertions | Θ(n log n) worst | Θ(1) |
| Java `new PriorityQueue<>(coll)` | Θ(n) | Θ(n) (copies into its own array) |

## I. Advantages, limitations, and comparisons

- **Bottom-up is strictly better** when all the data is available up front.
- **Insertion-based building** is needed when the data **streams in** and you must answer max queries along the way.
- Building a heap (Θ(n)) is much cheaper than sorting (Θ(n log n)). If you only need the top k elements, build a heap and extract k times, for **Θ(n + k log n)**.

**Interview follow-ups:** "Why is build-heap O(n) and not O(n log n)?" (most nodes are near the bottom, and Σ h/2ʰ = 2), "Find the k largest of n numbers" (Θ(n + k log n) with build-heap, or Θ(n log k) with a size-k min-heap).

---

## J. Practice

**Beginner**
1. Trace BUILD-MAX-HEAP on [5, 3, 17, 10, 84, 19, 6, 22, 9] (CLRS Exercise 6.3-1).
2. Why does the loop go from ⌊n/2⌋ down to 1, and not up? (CLRS Exercise 6.3-2)
3. Which indices are leaves when n = 12 (0-indexed)?

**Intermediate**
4. Prove there are at most ⌈n/2ʰ⁺¹⌉ nodes of height h (CLRS Exercise 6.3-3).
5. Show that Σ h/2ʰ = 2.
6. Give an input where building by insertion takes Θ(n log n).

**Advanced**
7. Prove the exact bound: BUILD-MAX-HEAP performs at most n − popcount(n) swaps.
8. Find the k largest elements in Θ(n + k log k) using a heap of candidates.

**Interview questions**

<details><summary>Q1. Why is building a heap O(n)?</summary>

Heapify costs O(height of the node), and most nodes have small height: n/2 leaves cost 0, n/4 nodes cost at most 1, n/8 cost at most 2, and so on. The total is n·Σ h/2ʰ⁺¹ = O(n), because the series converges (to 2 when written as Σ h/2ʰ).
</details>

<details><summary>Q2. Why must build-heap process nodes from the bottom up?</summary>

MAX-HEAPIFY requires both subtrees of a node to already be heaps. Going from the last internal node back to the root guarantees each node's children were processed first. Leaves need no processing.
</details>

<details><summary>Q3. How would you get the top 10 of 10 million numbers?</summary>

Keep a min-heap of size 10. For each number, if it's larger than the heap's minimum, replace the root and sift down. That's Θ(n log k) time and Θ(k) memory, with one streaming pass. If the data is all in memory, quickselect is Θ(n) expected.
</details>

**Worked problem: top-k with build-heap.** For n = 10⁶ and k = 100: build-heap is about 10⁶ operations, and 100 extract-max calls are about 100 × 20 = 2000. That's roughly 1.002 × 10⁶ steps, compared with about 2 × 10⁷ for sorting everything.

**Coding problems**
- LeetCode 215 · Kth Largest Element in an Array *(verify link)*
- LeetCode 347 · Top K Frequent Elements *(verify link)*
- LeetCode 973 · K Closest Points to Origin *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. BUILD-MAX-HEAP calls heapify on nodes ⌊n/2⌋ … 1, bottom-up.
2. Invariant: the nodes after i are already heap roots.
3. Θ(n), not Θ(n log n), because Σ h/2ʰ = 2.
4. Building by insertion is Θ(n log n) worst case, but works online.
5. Java's `new PriorityQueue<>(collection)` is the Θ(n) build.

**Formulas:** at most ⌈n/2ʰ⁺¹⌉ nodes at height h. Σ h/2ʰ = 2. Worst-case swaps = n − popcount(n).

**Common mistakes:** iterating upward, and quoting O(n log n) as tight.

**Quiz**
1. What's the first index heapified for n = 20 (1-indexed)?
2. Is BUILD-MAX-HEAP's result unique for a given input?
3. Exact upper bound on the swaps for n = 16?
4. Bottom-up or insertion: which is better for streaming input?

**Answers**

<details><summary>Show answers</summary>

1. ⌊20/2⌋ = 10.
2. Yes, for a given input BUILD-MAX-HEAP is deterministic. But different algorithms (insertion vs bottom-up) can produce different valid heaps.
3. 16 − popcount(16) = 16 − 1 = 15.
4. Insertion: it handles elements as they arrive.
</details>

**Related topics:** [MAX-HEAPIFY](maintaining-heap-property.md) · [Heapsort](heapsort.md) · [Priority queues](priority-queues.md) · [Summations](../../14-Mathematical-Foundations/summations.md) · [Quickselect](../order-statistics/quickselect.md)
