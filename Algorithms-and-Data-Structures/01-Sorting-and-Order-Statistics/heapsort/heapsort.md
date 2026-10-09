# Heapsort

> **CLRS:** 6.4 · **Status:** ✅ Written · **Prerequisites:** [Heap basics](heap-basics.md), [MAX-HEAPIFY](maintaining-heap-property.md), [Building a heap](building-a-heap.md) · **Next:** [Priority queues](priority-queues.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

**Heapsort** sorts an array by (1) turning it into a **max-heap**, then (2) repeatedly **moving the maximum (the root) to the end** of the array and shrinking the heap by one.

**What problem does it solve?** It's the only common comparison sort that is **both** Θ(n log n) in the **worst case** *and* **in place** (Θ(1) extra memory):

| | Worst-case Θ(n log n) | In place |
|---|---|---|
| [Merge sort](../comparison-sorts/merge-sort.md) | ✅ | ❌ (Θ(n) buffer) |
| [Quicksort](../quicksort/quicksort-basics.md) | ❌ (Θ(n²)) | ✅ |
| **Heapsort** | ✅ | ✅ |

**Where it's used**
- **Introsort** (C++ `std::sort`, .NET `Array.Sort`): quicksort that **falls back to heapsort** if the recursion gets too deep, which guarantees O(n log n).
- **Java's `Arrays.sort` for primitives** (JDK 14+) also falls back to heapsort in its dual-pivot quicksort when partitioning goes badly.
- **Embedded and real-time systems,** where memory is tiny and a worst-case guarantee is required (the Linux kernel's `lib/sort.c` is a heapsort).
- **Partial sorting:** extracting only the top k gives Θ(n + k log n).

**Prerequisites:** the three heap pages before this one.

---

## B. Intuition

**Analogy: a "king of the hill" competition.** Arrange everyone into a hierarchy where each person beats everyone below them (a max-heap). The person at the top is the overall winner. Send them to the end of the line (their final place). Promote someone from the bottom to the empty top spot. They're probably weak, so let them sink until the hierarchy is valid again (heapify). The new top person is the second-best. Repeat.

**Two zones in one array:**

```
[ ——— max-heap (shrinking) ——— | ——— sorted part (growing) ——— ]
  index 0 … size−1                 size … n−1
```

Every step moves the heap's root (the largest remaining element) into the first slot of the sorted zone. This is [selection sort](../comparison-sorts/selection-sort.md) with the Θ(n) "find the max" scan replaced by an O(log n) heap operation.

---

## C. How it works internally

**Input:** A = [4, 1, 3, 2, 16, 9, 10, 14, 8, 7] (the same as in [Building a heap](building-a-heap.md)).

**Phase 1: BUILD-MAX-HEAP** → [16, 14, 10, 8, 7, 9, 3, 2, 4, 1]

**Phase 2: extract repeatedly** (CLRS Figure 6.4). `|` marks the boundary between the heap and the sorted part.

| Step | Swap root ↔ last of heap | Then heapify the root | Array (heap \| sorted) |
|---|---|---|---|
| 1 | 16 ↔ 1 | 1 sinks: → 14 → 8 → 4 | [14, 8, 10, 4, 7, 9, 3, 2, 1 \| 16] |
| 2 | 14 ↔ 1 | 1 sinks: → 10 → 9 | [10, 8, 9, 4, 7, 1, 3, 2 \| 14, 16] |
| 3 | 10 ↔ 2 | 2 sinks: → 9 → 3 | [9, 8, 3, 4, 7, 1, 2 \| 10, 14, 16] |
| 4 | 9 ↔ 2 | 2 sinks: → 8 → 7 | [8, 7, 3, 4, 2, 1 \| 9, 10, 14, 16] |
| 5 | 8 ↔ 1 | 1 sinks: → 7 → 4 | [7, 4, 3, 1, 2 \| 8, 9, 10, 14, 16] |
| 6 | 7 ↔ 2 | 2 sinks: → 4 | [4, 2, 3, 1 \| 7, 8, 9, 10, 14, 16] |
| 7 | 4 ↔ 1 | 1 sinks: → 3 | [3, 2, 1 \| 4, 7, 8, 9, 10, 14, 16] |
| 8 | 3 ↔ 1 | 1 sinks: → 2 | [2, 1 \| 3, 4, 7, 8, 9, 10, 14, 16] |
| 9 | 2 ↔ 1 | heap of size 1, done | [1 \| 2, 3, 4, 7, 8, 9, 10, 14, 16] |

> Result: [1, 2, 3, 4, 7, 8, 9, 10, 14, 16]

### Why heapsort is NOT stable

The swap in phase 2 moves elements across long distances, and BUILD-HEAP reorders equal elements arbitrarily:

```
Input [1a, 1b] is already a max-heap (1a ≥ 1b).
Step 1: swap root 1a with the last element 1b → [1b, 1a] → sorted as [1b, 1a]   ✗ order of the equal 1s reversed
```

**Edge cases:** n ≤ 1 (nothing to do). All equal (heapify never swaps, so Θ(n) after the build). Sorted or reversed input (still Θ(n log n), so heapsort isn't adaptive).

---

## D. Algorithm and pseudocode

```
HEAPSORT(A)
1  BUILD-MAX-HEAP(A)
2  for i ← length[A] downto 2
3      exchange A[1] ↔ A[i]             ▷ move the current max to its final position
4      heap-size[A] ← heap-size[A] − 1  ▷ exclude it from the heap
5      MAX-HEAPIFY(A, 1)                ▷ restore the heap (the root may be too small)
```

| Line | Meaning |
|---|---|
| 1 | Θ(n): the whole array becomes a max-heap |
| 3 | A[1] is the largest element of A[1..i]. Put it at position i, its final sorted position. |
| 4 | Shrink the heap so the sorted suffix is never touched again |
| 5 | The new root (an old leaf) violates the property only at the root, and both subtrees are still heaps, so MAX-HEAPIFY's precondition holds |

### Loop invariant (CLRS Exercise 6.4-2)

> At the start of each iteration of the `for` loop: A[1..i] is a max-heap containing the **i smallest** elements of A, and A[i + 1..n] contains the **n − i largest** elements of A, **in sorted order**.

- **Initialization:** i = n. A[1..n] is a max-heap (from line 1) containing all elements, and the sorted suffix is empty. ✓
- **Maintenance:** A[1] is the maximum of A[1..i], which is the largest of the i smallest elements, so it's ≤ every element of A[i + 1..n]. Swapping it to position i makes A[i..n] the n − i + 1 largest, in sorted order. After shrinking the heap and heapifying, A[1..i − 1] is a max-heap of the i − 1 smallest elements. Decrementing i re-establishes the invariant. ✓
- **Termination:** i = 1. A[2..n] holds the n − 1 largest elements in sorted order, and A[1] is the smallest. **The whole array is sorted.** ✓

---

## E. Implementation

```java
import java.util.*;

/** Heapsort (CLRS 6.4): in place, Theta(n log n) worst case, not stable. Includes comparison counting. */
public class HeapSort {

    static long comparisons;

    public static void sort(int[] a) {
        int n = a.length;
        for (int i = n / 2 - 1; i >= 0; i--) siftDown(a, i, n);     // Phase 1: BUILD-MAX-HEAP, Theta(n)
        for (int end = n - 1; end > 0; end--) {                       // Phase 2: n - 1 extractions
            int t = a[0]; a[0] = a[end]; a[end] = t;                   // max -> its final position
            siftDown(a, 0, end);                                       // heap is now a[0..end-1]
        }
    }

    /** Iterative MAX-HEAPIFY on a[0..n-1] using the "hole" technique. */
    static void siftDown(int[] a, int i, int n) {
        int value = a[i];
        while (true) {
            int child = 2 * i + 1;
            if (child >= n) break;
            if (child + 1 < n) { comparisons++; if (a[child + 1] > a[child]) child++; }
            comparisons++;
            if (a[child] <= value) break;
            a[i] = a[child];
            i = child;
        }
        a[i] = value;
    }

    /** Generic heapsort for objects, used to demonstrate that heapsort is not stable. */
    public static <T> void sort(T[] a, Comparator<? super T> cmp) {
        int n = a.length;
        for (int i = n / 2 - 1; i >= 0; i--) siftDown(a, i, n, cmp);
        for (int end = n - 1; end > 0; end--) {
            T t = a[0]; a[0] = a[end]; a[end] = t;
            siftDown(a, 0, end, cmp);
        }
    }
    static <T> void siftDown(T[] a, int i, int n, Comparator<? super T> cmp) {
        T value = a[i];
        while (true) {
            int child = 2 * i + 1;
            if (child >= n) break;
            if (child + 1 < n && cmp.compare(a[child + 1], a[child]) > 0) child++;
            if (cmp.compare(a[child], value) <= 0) break;
            a[i] = a[child]; i = child;
        }
        a[i] = value;
    }

    record Item(int key, int originalIndex) {}

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        int[] a = {4, 1, 3, 2, 16, 9, 10, 14, 8, 7};
        sort(a);
        System.out.println("sorted: " + Arrays.toString(a));
        check(Arrays.equals(a, new int[]{1, 2, 3, 4, 7, 8, 9, 10, 14, 16}), "CLRS example sorted");

        Random rnd = new Random(2);
        boolean ok = true;
        for (int t = 0; t < 3000; t++) {
            int[] x = rnd.ints(rnd.nextInt(300), -1000, 1000).toArray();
            int[] exp = x.clone(); Arrays.sort(exp);
            sort(x);
            if (!Arrays.equals(x, exp)) ok = false;
        }
        int[][] edge = {{}, {7}, {5, 5, 5, 5}, {1, 2, 3, 4, 5}, {5, 4, 3, 2, 1}, {Integer.MAX_VALUE, Integer.MIN_VALUE, 0}};
        for (int[] e : edge) { int[] exp = e.clone(); Arrays.sort(exp); sort(e); if (!Arrays.equals(e, exp)) ok = false; }
        check(ok, "3000 random arrays + edge cases (empty, single, equal, sorted, reversed, extremes)");

        // Comparisons are ~ 2 n log2 n in the worst case, whatever the input order (not adaptive)
        int n = 1 << 16;
        int lg = 16;
        int[] sorted = new int[n], reversed = new int[n], random = rnd.ints(n).toArray();
        for (int i = 0; i < n; i++) { sorted[i] = i; reversed[i] = n - i; }
        System.out.printf("n = %d, n log2 n = %d%n", n, (long) n * lg);
        for (Object[] c : new Object[][]{{"sorted", sorted}, {"reversed", reversed}, {"random", random}}) {
            comparisons = 0; sort((int[]) c[1]);
            System.out.printf("  %-9s comparisons = %,9d  (%.2f n log2 n)%n", c[0], comparisons, comparisons / (double) ((long) n * lg));
            if (comparisons > 2L * n * lg) ok = false;
        }
        check(ok, "comparisons <= 2 n log2 n on every input");

        // Not stable: equal keys can come out in a different order
        Item[] items = new Item[200];
        for (int i = 0; i < items.length; i++) items[i] = new Item(rnd.nextInt(5), i);
        sort(items, Comparator.comparingInt(Item::key));
        boolean sortedByKey = true, stable = true;
        for (int i = 1; i < items.length; i++) {
            if (items[i - 1].key() > items[i].key()) sortedByKey = false;
            if (items[i - 1].key() == items[i].key() && items[i - 1].originalIndex() > items[i].originalIndex()) stable = false;
        }
        check(sortedByKey, "generic heapsort sorts by key");
        check(!stable, "heapsort is NOT stable (equal keys came out reordered)");
    }
}
```

**Output:**

```
sorted: [1, 2, 3, 4, 7, 8, 9, 10, 14, 16]
ok   CLRS example sorted
ok   3000 random arrays + edge cases (empty, single, equal, sorted, reversed, extremes)
n = 65536, n log2 n = 1048576
  sorted    comparisons = 1,948,406  (1.86 n log2 n)
  reversed  comparisons = 1,839,970  (1.75 n log2 n)
  random    comparisons = 1,894,757  (1.81 n log2 n)
ok   comparisons <= 2 n log2 n on every input
ok   generic heapsort sorts by key
ok   heapsort is NOT stable (equal keys came out reordered)
```

**What the measurements show:** sorted, reversed and random inputs all cost about 1.7–1.9 · n log₂ n comparisons. Heapsort **doesn't care about input order**, for better (no bad case) or worse (no good case either). Merge sort uses at most about 1 · n log₂ n comparisons. That roughly 2× comparison count, plus poor cache locality, is why heapsort is usually slower in practice.

**Java-specific details**
- The extremes test (`Integer.MAX_VALUE`, `Integer.MIN_VALUE`) confirms there are no overflow-prone tricks like using `a - b` as a comparator.
- In production, prefer `Arrays.sort`. Heapsort is a teaching and fallback algorithm.
- For a heapsort over objects with a top-k early exit, `PriorityQueue` with `poll()` k times works, but it isn't in place.

**Common mistakes**
1. Heapifying with the **old** heap size after the swap, which drags the sorted element back into the heap.
2. Building a **min-heap** for ascending output (that sorts descending in place).
3. Starting the extraction loop at `end = n` (out of bounds) or stopping at `end >= 0` (one wasted iteration).

---

## F. Time complexity

| Phase | Cost | Derivation |
|---|---|---|
| BUILD-MAX-HEAP | **Θ(n)** | Σ (nodes at height h) × h = O(n). See [Building a heap](building-a-heap.md). |
| n − 1 extractions | **O(n log n)** | each MAX-HEAPIFY on a heap of size i costs O(log i), so Σᵢ₌₂ⁿ O(log i) = O(log n!) = O(n log n) |
| **Total** | **O(n log n)** | |

**Worst case:** Θ(n log n). The extraction phase alone is Ω(n log n) for distinct keys (an old leaf placed at the root tends to sink almost all the way back down).

**Best case:** for **distinct** keys, it's still **Θ(n log n)**. This is a non-trivial result (Schaffer & Sedgewick, 1993): no input order makes heapsort substantially faster. With **all keys equal**, each heapify stops immediately, giving Θ(n).

**Average case:** Θ(n log n), about 2n log₂ n comparisons for the standard version.

**Bottom-up heapsort** (Floyd's or Wegener's variant): sink the new root straight down to a leaf along the larger children (1 comparison per level instead of 2), then sift it back up a little. That gives about n log₂ n + O(n) comparisons, halving the count.

**Assumptions:** comparisons are O(1) and elements are in a random-access array.

## G. Space complexity

- **Auxiliary: Θ(1).** One temporary variable, plus loop indices. Truly **in place**.
- **Recursion:** none with iterative sift-down. Θ(log n) if MAX-HEAPIFY is written recursively.
- It's the same for every input.

## H. Complexity summary

| Algorithm | Best time | Average time | Worst time | Auxiliary space | Stable? | In-place? |
|---|---|---|---|---|---|---|
| Heapsort | Θ(n log n) (distinct keys) | Θ(n log n) | Θ(n log n) | Θ(1) | ❌ No | ✅ Yes |

Comparison-based ✅ · adaptive ❌ · comparisons ≈ 2n log₂ n

## I. Advantages, limitations, and comparisons

**Advantages**
- **Guaranteed O(n log n) and O(1) memory**, a combination no other common sort offers.
- No bad inputs, so it's immune to adversarial "quicksort killer" inputs.
- Simple, non-recursive implementation.

**Limitations**
- **Slower in practice** than quicksort and merge sort: about 2× the comparisons, and **poor cache locality** (children at 2i + 1 jump far away in memory for large heaps).
- **Not stable.**
- **Not adaptive:** sorted input costs as much as random input.

| | Heapsort | Quicksort | Merge sort |
|---|---|---|---|
| Worst case | Θ(n log n) | Θ(n²) | Θ(n log n) |
| Typical speed | slowest of the three | fastest | middle |
| Extra memory | Θ(1) | Θ(log n) stack | Θ(n) |
| Stable | ❌ | ❌ | ✅ |
| Cache behaviour | poor | excellent | good |

**When to use it:** when you need a worst-case guarantee **and** constant memory (embedded systems), as a fallback inside introsort, or for partial sorting (the top k).

**Interview follow-ups:** "Why doesn't anyone use heapsort if it's O(n log n) worst case?" (constants and cache), "What does introsort do?", "Sort a k-sorted array in O(n log k)" (a heap of size k + 1).

---

## J. Practice

**Beginner**
1. Trace heapsort on [5, 13, 2, 25, 7, 17, 20, 8, 4] (CLRS Exercise 6.4-1).
2. What's the running time on an array that's already sorted in increasing order? Decreasing order? (CLRS Exercise 6.4-3)
3. Why does heapsort use a max-heap to sort in ascending order?

**Intermediate**
4. Prove the loop invariant above (CLRS Exercise 6.4-2).
5. Implement heapsort with a **min-heap** that produces ascending output without reversing. *(Hint: put the heap's root at the end of the array.)*
6. Show that heapsort's worst case is Ω(n lg n) (CLRS Exercise 6.4-4).

**Advanced**
7. Implement bottom-up (Floyd) heapsort and measure the comparison savings.
8. Implement introsort: quicksort with a depth limit of 2⌊log₂ n⌋, falling back to heapsort, with insertion sort for small parts.

**Interview questions**

<details><summary>Q1. Explain heapsort's two phases and their costs.</summary>

Phase 1 builds a max-heap in place in Θ(n). Phase 2 repeats n − 1 times: swap the root (the max) with the last heap element, shrink the heap, and sift the new root down in O(log n). The total is Θ(n log n), with Θ(1) extra space.
</details>

<details><summary>Q2. Heapsort and merge sort are both O(n log n). Why is quicksort usually preferred?</summary>

Quicksort has smaller constant factors and excellent cache locality (it scans contiguous memory), so it's typically 2–3× faster than heapsort. Its Θ(n²) worst case is made vanishingly unlikely by randomisation and is eliminated entirely by introsort's heapsort fallback.
</details>

<details><summary>Q3. Is heapsort stable? Can it be made stable?</summary>

No. Swapping the root with the last element moves equal keys past each other. You can make it stable by sorting (key, original index) pairs, which costs extra memory. Merge sort is the natural stable choice.
</details>

**Worked problem: sort a nearly sorted array** where each element is at most k positions away from its target. Use a min-heap of the first k + 1 elements. Repeatedly extract the minimum into the output and push the next element in. That's Θ(n log k) time and Θ(k) space, which beats full heapsort's Θ(n log n) when k ≪ n.

**Coding problems**
- LeetCode 912 · Sort an Array (implement heapsort) *(verify link)*
- LeetCode 1046 · Last Stone Weight *(verify link)*
- GeeksforGeeks · Sort a k-sorted array *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. Build a max-heap (Θ(n)), then swap the max to the end and sift down, n − 1 times (Θ(n log n)).
2. Θ(n log n) worst case **and** Θ(1) extra space, which is unique among the common sorts.
3. Not stable, not adaptive.
4. About 2n log₂ n comparisons and poor cache use, so it's slower than quicksort in practice.
5. Used as introsort's worst-case fallback, and in memory-constrained systems.

**Formulas:** Θ(n) build + Σ log i = Θ(n log n). About 2n log₂ n comparisons (n log₂ n with the bottom-up variant).

**Common mistakes:** heapifying with the wrong size, using a min-heap by mistake, claiming it's stable.

**Quiz**
1. What's the auxiliary space of heapsort?
2. After k extractions, where are the k largest elements?
3. Is heapsort faster on sorted input?
4. Which hybrid sort uses heapsort as a safety net?

**Answers**

<details><summary>Show answers</summary>

1. Θ(1).
2. In the last k positions of the array, in sorted order.
3. No: still Θ(n log n). The build reorders it, and every extraction costs about log n.
4. Introsort (C++ `std::sort`). Java's primitive sort also falls back to heapsort.
</details>

**Related topics:** [Building a heap](building-a-heap.md) · [Priority queues](priority-queues.md) · [Selection sort](../comparison-sorts/selection-sort.md) · [Quicksort](../quicksort/quicksort-basics.md) · [Merge sort](../comparison-sorts/merge-sort.md) · [Lower bounds](../linear-time-sorting/lower-bounds.md)
