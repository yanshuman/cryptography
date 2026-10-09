# Quicksort Basics

> **CLRS:** 7.1 (Description of quicksort) · **Status:** ✅ Written · **Prerequisites:** [Divide and conquer](../../00-Foundations/divide-and-conquer.md) · **Next:** [Partitioning](partitioning.md) → [Randomized quicksort](randomized-quicksort.md) → [Complexity analysis](complexity-analysis.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

**Quicksort** is a divide-and-conquer sort that **partitions** the array around a chosen element (the **pivot**) so that everything smaller goes left and everything larger goes right, then **recursively sorts the two sides**. Once partitioned, the pivot is already in its **final position**.

**What problem does it solve?** It sorts **in place** and is, on average, the **fastest general-purpose comparison sort in practice**: Θ(n log n) expected, with very small constants and excellent cache behaviour.

**Where it's used**
- **Java `Arrays.sort` for primitive arrays:** dual-pivot quicksort.
- **C++ `std::sort`:** introsort (quicksort + heapsort fallback + insertion sort).
- **C `qsort`** (in many C libraries), Go's `sort` (pdqsort), and Rust's `sort_unstable` (pattern-defeating quicksort).
- Its partition step powers **quickselect** ([k-th smallest in Θ(n) expected](../order-statistics/quickselect.md)).

**Prerequisites:** recursion and divide and conquer.

---

## B. Intuition

**Analogy: sorting students by height using a "reference" student.** Pick one student as the pivot. Everyone shorter goes to the left of them and everyone taller goes to the right. The pivot is now standing **exactly where they'll be in the final line-up**. Now do the same for the left group and the right group, separately and independently.

**Compared with merge sort**

| | Merge sort | Quicksort |
|---|---|---|
| Hard work is done… | **after** recursion (merging) | **before** recursion (partitioning) |
| Divide step | trivial (split in the middle) | partition: Θ(n) |
| Combine step | merge: Θ(n) | trivial (nothing to do) |
| Subproblem sizes | always n/2 | depends on the pivot |
| Extra memory | Θ(n) | Θ(log n) stack |

**Why the pivot matters:** a pivot near the median splits the array in half, giving log n levels and Θ(n log n). A pivot that's always the smallest or largest splits n into 0 and n − 1, giving n levels and Θ(n²). See [Complexity analysis](complexity-analysis.md).

---

## C. How it works internally

**CLRS Figure 7.1:** A = [2, 8, 7, 1, 3, 5, 6, **4**]. The pivot is x = A[r] = **4** (the last element, as in CLRS's Lomuto partition).

The array is kept in four regions during partitioning:

```
[ ≤ pivot (lo..i) | > pivot (i+1..j−1) | not yet examined (j..r−1) | pivot (r) ]
```

| j | A[j] | A[j] ≤ 4? | Action | Array after (i marked by ^) |
|---|---|---|---|---|
| start | | | i = −1 | [2, 8, 7, 1, 3, 5, 6, 4] |
| 0 | 2 | yes | i = 0, swap A[0] ↔ A[0] | [**2**, 8, 7, 1, 3, 5, 6, 4]  i = 0 |
| 1 | 8 | no | — | [2, 8, 7, 1, 3, 5, 6, 4] |
| 2 | 7 | no | — | [2, 8, 7, 1, 3, 5, 6, 4] |
| 3 | 1 | yes | i = 1, swap A[1] ↔ A[3] | [2, **1**, 7, 8, 3, 5, 6, 4]  i = 1 |
| 4 | 3 | yes | i = 2, swap A[2] ↔ A[4] | [2, 1, **3**, 8, 7, 5, 6, 4]  i = 2 |
| 5 | 5 | no | — | |
| 6 | 6 | no | — | [2, 1, 3, 8, 7, 5, 6, 4] |
| end | | | swap A[i+1] ↔ A[r], i.e. A[3] ↔ A[7] | [2, 1, 3, **4**, 7, 5, 6, 8] |

> The pivot 4 is at index 3, its **final sorted position**. Left: [2, 1, 3] (all ≤ 4). Right: [7, 5, 6, 8] (all > 4).

**Recursion continues:**

```
quicksort([2,1,3])    → pivot 3 → [2,1] 3 []        → quicksort([2,1]) → pivot 1 → [] 1 [2]
quicksort([7,5,6,8])  → pivot 8 → [7,5,6] 8 []      → pivot 6 → [5] 6 [7]
Final: [1, 2, 3, 4, 5, 6, 7, 8]
```

**Edge cases**

| Input | Behaviour with the last-element pivot (CLRS) |
|---|---|
| empty or one element | base case (p ≥ r) |
| already sorted | the pivot is always the max, so the splits are n − 1 and 0: **Θ(n²)**, and recursion depth n |
| reverse sorted | the pivot is always the min: **Θ(n²)** |
| all equal | Lomuto puts everything on the left (A[j] ≤ x is always true): **Θ(n²)**. 3-way partitioning fixes this ([Partitioning](partitioning.md)). |
| random | Θ(n log n) expected |

The first three rows are why real implementations **randomise** the pivot or use **median-of-three**. See [Randomized quicksort](randomized-quicksort.md).

---

## D. Algorithm and pseudocode

CLRS 7.1 (1-indexed):

```
QUICKSORT(A, p, r)
1  if p < r
2      q ← PARTITION(A, p, r)        ▷ divide: A[p..q−1] ≤ A[q] < A[q+1..r]
3      QUICKSORT(A, p, q − 1)        ▷ conquer left
4      QUICKSORT(A, q + 1, r)        ▷ conquer right
                                     ▷ combine: nothing to do, the array is sorted in place

PARTITION(A, p, r)                   ▷ Lomuto partition scheme
1  x ← A[r]                          ▷ pivot
2  i ← p − 1                         ▷ end of the "≤ x" region
3  for j ← p to r − 1
4      if A[j] ≤ x
5          i ← i + 1
6          exchange A[i] ↔ A[j]      ▷ grow the "≤ x" region
7  exchange A[i + 1] ↔ A[r]          ▷ put the pivot between the two regions
8  return i + 1
```

**Initial call:** QUICKSORT(A, 1, n).

**PARTITION's loop invariant** (fully proved in [Partitioning](partitioning.md#lomuto-partition-clrs)): at the start of each iteration, for every index k:
1. if p ≤ k ≤ i, then A[k] ≤ x;
2. if i + 1 ≤ k ≤ j − 1, then A[k] > x;
3. if k = r, then A[k] = x.

At termination (j = r), every element has been classified, and line 7 places the pivot between the regions.

**QUICKSORT's correctness**, by strong induction on n = r − p + 1:
- **Base:** n ≤ 1 is already sorted. ✓
- **Step:** after PARTITION, A[q] is in its final position (everything left is ≤ it, everything right is > it). By induction the two recursive calls sort A[p..q − 1] and A[q + 1..r], both strictly smaller since the pivot is excluded. Sorted left + pivot + sorted right = sorted. ✓
- **Termination:** each recursive call excludes at least the pivot, so the sizes strictly decrease.

---

## E. Implementation

```java
import java.util.*;

/** Quicksort (CLRS 7.1) with Lomuto partitioning, plus an Integer-overflow-safe generic version and tests. */
public class QuickSortBasics {

    static long comparisons;

    public static void sort(int[] a) { quicksort(a, 0, a.length - 1); }

    static void quicksort(int[] a, int p, int r) {
        if (p < r) {
            int q = partition(a, p, r);
            quicksort(a, p, q - 1);
            quicksort(a, q + 1, r);
        }
    }

    /** CLRS PARTITION (Lomuto): pivot = a[r]. Returns the pivot's final index. */
    static int partition(int[] a, int p, int r) {
        int x = a[r];
        int i = p - 1;
        for (int j = p; j < r; j++) {
            comparisons++;
            if (a[j] <= x) {
                i++;
                swap(a, i, j);
            }
        }
        swap(a, i + 1, r);
        return i + 1;
    }

    static void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        int[] a = {2, 8, 7, 1, 3, 5, 6, 4};                        // CLRS Figure 7.1
        int q = partition(a, 0, a.length - 1);
        System.out.println("after one PARTITION: " + Arrays.toString(a) + "  pivot index = " + q);
        check(Arrays.equals(a, new int[]{2, 1, 3, 4, 7, 5, 6, 8}) && q == 3, "matches CLRS Figure 7.1");

        boolean leftOk = true, rightOk = true;
        for (int k = 0; k < q; k++) if (a[k] > a[q]) leftOk = false;
        for (int k = q + 1; k < a.length; k++) if (a[k] <= a[q]) rightOk = false;
        check(leftOk && rightOk, "everything left of the pivot is <= 4, everything right is > 4");

        sort(a);
        System.out.println("fully sorted:        " + Arrays.toString(a));
        check(Arrays.equals(a, new int[]{1, 2, 3, 4, 5, 6, 7, 8}), "sorted");

        Random rnd = new Random(8);
        boolean ok = true;
        for (int t = 0; t < 3000; t++) {
            int[] x = rnd.ints(rnd.nextInt(300), -200, 200).toArray();
            int[] exp = x.clone(); Arrays.sort(exp);
            sort(x);
            if (!Arrays.equals(x, exp)) ok = false;
        }
        check(ok, "3000 random arrays match Arrays.sort");

        // The weakness of a fixed pivot: sorted input is the worst case.
        int n = 2000;
        int[] sorted = new int[n]; for (int i = 0; i < n; i++) sorted[i] = i;
        int[] random = rnd.ints(n).toArray();
        comparisons = 0; sort(sorted);  long cs = comparisons;
        comparisons = 0; sort(random);  long cr = comparisons;
        System.out.printf("n = %d: sorted input %,d comparisons (= n(n-1)/2 = %,d); random input %,d (~1.39 n log2 n = %,.0f)%n",
                n, cs, (long) n * (n - 1) / 2, cr, 1.39 * n * (Math.log(n) / Math.log(2)));
        check(cs == (long) n * (n - 1) / 2, "sorted input with last-element pivot is quadratic");
        check(cr < 3L * n * 11, "random input is about n log n");
    }
}
```

**Output:**

```
after one PARTITION: [2, 1, 3, 4, 7, 5, 6, 8]  pivot index = 3
ok   matches CLRS Figure 7.1
ok   everything left of the pivot is <= 4, everything right is > 4
fully sorted:        [1, 2, 3, 4, 5, 6, 7, 8]
ok   sorted
ok   3000 random arrays match Arrays.sort
n = 2000: sorted input 1,999,000 comparisons (= n(n-1)/2 = 1,999,000); random input 23,663 (~1.39 n log2 n = 30,485)
ok   sorted input with last-element pivot is quadratic
ok   random input is about n log n
```

The measured random-input count (23,663) is close to the exact expected value 2(n + 1)Hₙ − 4n ≈ 24,730 for n = 2000, where Hₙ is the n-th harmonic number. A single random input varies around this average. The 1.39 n log₂ n figure (≈ 30,485) is only the leading term: the −4n and other lower-order terms still matter at this size. See [Complexity analysis](complexity-analysis.md).

**Java-specific details**
- **Recursion depth:** this naive version recurses n deep on sorted input. For n around 10⁵ that's a real risk of `StackOverflowError`. Fixes: randomise the pivot, and recurse on the **smaller** side while looping on the larger, which bounds the depth at log₂ n ([Space complexity](../../00-Foundations/space-complexity.md#e-implementation) shows this).
- **Don't use quicksort for `Object[]` when stability matters.** Java deliberately uses TimSort there.
- **Comparator overflow:** never compare with `a - b`. Use `Integer.compare(a, b)`.

**Common mistakes**
1. Recursing on `(p, q)` instead of `(p, q − 1)` with Lomuto partition, which recurses forever when the pivot is the max.
2. Forgetting the final swap that places the pivot.
3. Using `<` instead of `≤` in the partition test. That still works, but changes how equal elements are distributed.
4. A fixed first or last pivot on input that might be sorted (like real data).

---

## F. Time complexity

Summary (derived in full in [Complexity analysis](complexity-analysis.md)):

| Case | Recurrence | Result | When |
|---|---|---|---|
| Worst | T(n) = T(n − 1) + T(0) + Θ(n) | **Θ(n²)** | the pivot is always the min or max (sorted input with a fixed pivot) |
| Best | T(n) = 2T(n/2) + Θ(n) | **Θ(n log n)** | the pivot is always the median |
| Any constant split, e.g. 9:1 | T(n) = T(9n/10) + T(n/10) + Θ(n) | **Θ(n log n)** | even "unbalanced" constant splits are fine |
| Average or expected (random input or random pivot) | about 2n ln n ≈ 1.39 n log₂ n comparisons | **Θ(n log n)** | the typical case |

**PARTITION is Θ(n):** the `for` loop runs r − p times, with O(1) work each.

## G. Space complexity

- **Auxiliary arrays:** none. Quicksort is **in place**: Θ(1) besides the stack.
- **Recursion stack:** Θ(log n) for balanced splits, but **Θ(n) in the worst case**. The smaller-side-first trick guarantees **O(log n)**.

## H. Complexity summary

| Algorithm | Best time | Average time | Worst time | Auxiliary space | Stable? | In-place? |
|---|---|---|---|---|---|---|
| Quicksort (CLRS, last-element pivot) | Θ(n log n) | Θ(n log n) | Θ(n²) | Θ(log n) avg / Θ(n) worst (stack) | ❌ No | ✅ Yes |

## I. Advantages, limitations, and comparisons

**Advantages:** the fastest in practice (sequential memory access, few swaps, a tight inner loop), in place, and easy to parallelise (the two sides are independent).

**Limitations:** Θ(n²) worst case (fixed by randomisation or introsort), not stable, and naive versions degrade on many duplicates (fixed by 3-way partitioning).

| | Quicksort | Merge sort | Heapsort |
|---|---|---|---|
| Typical speed | **fastest** | ~1.5× slower | ~2–3× slower |
| Worst case | Θ(n²) | Θ(n log n) | Θ(n log n) |
| Memory | Θ(log n) | Θ(n) | Θ(1) |
| Stable | ❌ | ✅ | ❌ |

**Interview follow-ups:** "What's quicksort's worst case, and how do you avoid it?", "Quicksort vs merge sort: when would you use each?", "Why is quicksort faster in practice despite the same Θ(n log n)?", "Implement partition."

---

## J. Practice

**Beginner**
1. Trace PARTITION on [13, 19, 9, 5, 12, 8, 7, 4, 21, 2, 6, 11] (CLRS Exercise 7.1-1).
2. What value of q does PARTITION return when all elements are equal? (CLRS Exercise 7.1-2)
3. Modify QUICKSORT to sort in descending order (CLRS Exercise 7.1-4).

**Intermediate**
4. Argue that PARTITION runs in Θ(n) on a subarray of size n (CLRS Exercise 7.1-3).
5. Make QUICKSORT use O(log n) stack space in the worst case (CLRS Problem 7-4).
6. Implement quicksort on a doubly linked list.

**Advanced**
7. Implement introsort (depth limit 2⌊lg n⌋ → heapsort, cutoff 16 → insertion sort).
8. Construct an input that makes median-of-three quicksort quadratic (a "median-of-3 killer").

**Interview questions**

<details><summary>Q1. How does quicksort work?</summary>

Pick a pivot. Partition the array so that elements ≤ pivot are on its left and elements > pivot on its right, which puts the pivot in its final position. Recursively sort both sides. There's no combine step. It's Θ(n log n) on average and Θ(n²) in the worst case, and it sorts in place.
</details>

<details><summary>Q2. What's the worst case, and how do you avoid it?</summary>

Θ(n²), when the pivot is always the smallest or largest element (for example, sorted input with a first or last pivot). Avoid it with a random pivot or median-of-three, guarantee O(n log n) with introsort's heapsort fallback, and handle duplicates with 3-way partitioning.
</details>

<details><summary>Q3. Why is quicksort usually faster than merge sort?</summary>

It partitions in place with sequential scans (cache-friendly), does fewer data moves (merge sort copies every element at every level), and has a tight inner loop. Both are Θ(n log n) on average, but quicksort's constants are smaller.
</details>

**Worked problem: sort colours (0s, 1s and 2s) in one pass (LeetCode 75).** This is quicksort's partition step with pivot 1 and **three** regions (the Dutch national flag). Keep lo, mid and hi pointers: a 0 is swapped to lo, a 2 is swapped to hi, and a 1 is skipped. That's Θ(n) time and Θ(1) space. See [Partitioning](partitioning.md#3-way-partition-dutch-national-flag).

**Coding problems**
- LeetCode 912 · Sort an Array *(verify link)*
- LeetCode 75 · Sort Colors *(verify link)*
- LeetCode 215 · Kth Largest Element in an Array (quickselect) *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. Partition around a pivot, so the pivot lands in its final place. Recurse on both sides. No combine step.
2. PARTITION is Θ(n). Lomuto keeps the regions ≤ x | > x | unknown | pivot.
3. Θ(n log n) best and average, Θ(n²) worst (bad pivots).
4. In place (Θ(log n) stack if done carefully), not stable.
5. The fastest sort in practice, used in Java's primitive sort (dual-pivot) and C++'s `std::sort` (introsort).

**Formulas:** worst-case comparisons n(n − 1)/2. Expected comparisons about 1.39 n log₂ n.

**Common mistakes:** wrong recursion bounds, a fixed pivot on sorted data, unbounded stack depth.

**Quiz**
1. Where is the pivot after PARTITION?
2. Quicksort's combine step?
3. Worst-case input for the CLRS version?
4. Is quicksort stable?

**Answers**

<details><summary>Show answers</summary>

1. In its final sorted position.
2. None. The array is already sorted once both recursive calls return.
3. An already sorted (or reverse-sorted) array, or one where all elements are equal.
4. No.
</details>

**Related topics:** [Partitioning](partitioning.md) · [Randomized quicksort](randomized-quicksort.md) · [Complexity analysis](complexity-analysis.md) · [Merge sort](../comparison-sorts/merge-sort.md) · [Heapsort](../heapsort/heapsort.md) · [Quickselect](../order-statistics/quickselect.md)
