# 01 · Sorting and Order Statistics

> **CLRS Part II** (Chapters 6–9), plus the foundational sorts from Chapter 2 and its problems · **Prerequisites:** [00-Foundations](../00-Foundations/) · **Status:** ✅ all 20 pages written

Sorting means rearranging items into order. It's the most-studied problem in computer science because **so much else depends on it**: binary search, duplicate detection, merging datasets, Kruskal's MST, sweep-line geometry, database indexes and ORDER BY queries. This section covers 10 sorting algorithms in depth, the theoretical limit on how fast comparison sorting can be, and how to find the k-th smallest element *without* fully sorting.

---

## Contents

### Comparison sorts: the foundations
| Page | One-line summary |
|---|---|
| [Bubble sort](comparison-sorts/bubble-sort.md) | Repeatedly swap adjacent out-of-order pairs. Θ(n²). Teaching only. |
| [Selection sort](comparison-sorts/selection-sort.md) | Repeatedly select the minimum. Θ(n²) comparisons but only n − 1 swaps. |
| [Insertion sort](comparison-sorts/insertion-sort.md) | Insert each element into the sorted prefix. Θ(n²) worst case, **Θ(n) on nearly sorted input**. |
| [Merge sort](comparison-sorts/merge-sort.md) | Divide and conquer: sort the halves, merge them. Θ(n log n) always, stable, Θ(n) extra space. |

### Heapsort (CLRS Ch. 6)
| Page | One-line summary |
|---|---|
| [Heap basics](heapsort/heap-basics.md) | A complete binary tree stored in an array, with parent ≥ children |
| [Maintaining the heap property](heapsort/maintaining-heap-property.md) | MAX-HEAPIFY: sift a value down, O(log n) |
| [Building a heap](heapsort/building-a-heap.md) | BUILD-MAX-HEAP in **O(n)**, not O(n log n) |
| [Heapsort](heapsort/heapsort.md) | Θ(n log n) worst case, in place, not stable |
| [Priority queues](heapsort/priority-queues.md) | Insert, extract-max, increase-key in O(log n). Java's `PriorityQueue`. |

### Quicksort (CLRS Ch. 7)
| Page | One-line summary |
|---|---|
| [Quicksort basics](quicksort/quicksort-basics.md) | Partition around a pivot, recurse on both sides |
| [Partitioning](quicksort/partitioning.md) | Lomuto vs Hoare vs 3-way (Dutch national flag) |
| [Randomized quicksort](quicksort/randomized-quicksort.md) | A random pivot gives Θ(n log n) **expected** time on every input |
| [Complexity analysis](quicksort/complexity-analysis.md) | Worst Θ(n²), best Θ(n log n), and the expected-case proof (~1.39 n log₂ n comparisons) |

### Sorting in linear time (CLRS Ch. 8)
| Page | One-line summary |
|---|---|
| [Lower bounds](linear-time-sorting/lower-bounds.md) | Every comparison sort needs Ω(n log n) comparisons (decision-tree proof) |
| [Counting sort](linear-time-sorting/counting-sort.md) | Θ(n + k) for integer keys in [0, k], stable |
| [Radix sort](linear-time-sorting/radix-sort.md) | Digit by digit with a stable sort, Θ(d(n + k)) |
| [Bucket sort](linear-time-sorting/bucket-sort.md) | Θ(n) **expected** for uniformly distributed input |

### Medians and order statistics (CLRS Ch. 9)
| Page | One-line summary |
|---|---|
| [Minimum and maximum](order-statistics/minimum-and-maximum.md) | n − 1 comparisons for one, and 3⌊n/2⌋ for both at once |
| [Quickselect](order-statistics/quickselect.md) | k-th smallest in Θ(n) **expected** |
| [Median of medians](order-statistics/median-of-medians.md) | k-th smallest in Θ(n) **worst case** |

---

## Master comparison table

n = number of elements, k = range of key values, d = number of digits.

| Algorithm | Best | Average | Worst | Auxiliary space | Stable? | In place? | Comparison-based? |
|---|---|---|---|---|---|---|---|
| [Bubble sort](comparison-sorts/bubble-sort.md) (with early exit) | Θ(n) | Θ(n²) | Θ(n²) | Θ(1) | ✅ | ✅ | ✅ |
| [Selection sort](comparison-sorts/selection-sort.md) | Θ(n²) | Θ(n²) | Θ(n²) | Θ(1) | ❌ | ✅ | ✅ |
| [Insertion sort](comparison-sorts/insertion-sort.md) | Θ(n) | Θ(n²) | Θ(n²) | Θ(1) | ✅ | ✅ | ✅ |
| [Merge sort](comparison-sorts/merge-sort.md) | Θ(n log n) | Θ(n log n) | Θ(n log n) | Θ(n) | ✅ | ❌ | ✅ |
| [Heapsort](heapsort/heapsort.md) | Θ(n log n)* | Θ(n log n) | Θ(n log n) | Θ(1) | ❌ | ✅ | ✅ |
| [Quicksort](quicksort/quicksort-basics.md) (CLRS, last-element pivot) | Θ(n log n) | Θ(n log n) | Θ(n²) | Θ(log n) stack avg, Θ(n) worst | ❌ | ✅ | ✅ |
| [Randomized quicksort](quicksort/randomized-quicksort.md) | Θ(n log n) | Θ(n log n) expected | Θ(n²) (probability → 0) | Θ(log n) expected stack | ❌ | ✅ | ✅ |
| [Counting sort](linear-time-sorting/counting-sort.md) | Θ(n + k) | Θ(n + k) | Θ(n + k) | Θ(n + k) | ✅ | ❌ | ❌ |
| [Radix sort](linear-time-sorting/radix-sort.md) (LSD) | Θ(d(n + k)) | Θ(d(n + k)) | Θ(d(n + k)) | Θ(n + k) | ✅ | ❌ | ❌ |
| [Bucket sort](linear-time-sorting/bucket-sort.md) | Θ(n) | Θ(n) expected (uniform input) | Θ(n²) (all in one bucket) | Θ(n) | ✅ (if the bucket sort is stable) | ❌ | partly† |

\* Heapsort with all keys equal is Θ(n), but for distinct keys even the best case is Θ(n log n).
† Bucket sort assigns elements to buckets by arithmetic on the key (not by comparison) and then comparison-sorts each small bucket.

**Real-world sorts**

| Library | Algorithm | Notes |
|---|---|---|
| Java `Arrays.sort(int[])`, and other primitive arrays | **Dual-pivot quicksort** (Yaroslavskiy) | Not stable (that doesn't matter for primitives). O(n log n) expected. Falls back to heapsort if recursion gets too deep (JDK 14+). |
| Java `Arrays.sort(Object[])`, `Collections.sort`, `List.sort` | **TimSort** | **Stable.** O(n) on already-sorted runs, O(n log n) worst case, O(n) extra space |
| C++ `std::sort` | **Introsort** (quicksort → heapsort if too deep → insertion sort for small parts) | O(n log n) worst case, not stable |
| C++ `std::stable_sort` | Merge sort | Stable |
| Python `sorted`, `list.sort` | TimSort (Powersort since 3.11) | Stable |

---

## Key concepts used throughout this section

These are explained **once** here, and every algorithm page links back.

### Stability

A sort is **stable** if elements with **equal keys keep their original relative order**.

```
Input  (sort by grade):  (Asha, B) (Ravi, A) (Meena, B) (Kiran, A)
Stable output:           (Ravi, A) (Kiran, A) (Asha, B) (Meena, B)    ← Ravi before Kiran, Asha before Meena, as in the input
Unstable (possible):     (Kiran, A) (Ravi, A) (Meena, B) (Asha, B)
```

**Why it matters:** *multi-key sorting*. To sort by (grade, then name), sort by name first, then **stable**-sort by grade. Radix sort depends on this: each digit pass must be stable. Java guarantees stability for object sorts (TimSort) for this reason.

**How to test stability:** sort (key, originalIndex) pairs by key only, then check that the originalIndex values are increasing within each group of equal keys. Every page's test harness does exactly this.

### In place

A sort is **in place** if it uses only O(1) extra memory beyond the input array (CLRS: "only a constant number of elements are stored outside the array at any time"). Quicksort is usually called in place even though its recursion stack is O(log n). See [Space complexity](../00-Foundations/space-complexity.md).

### Comparison-based sorting

A **comparison sort** learns about the order of elements *only* by comparing pairs (a[i] < a[j]?). It never looks inside a key (its digits or bits). Every comparison sort needs **Ω(n log n)** comparisons in the worst case. See [Lower bounds](linear-time-sorting/lower-bounds.md). Counting, radix and bucket sort beat this bound by **not** being comparison sorts: they use the key's value directly, which requires extra assumptions such as small integer keys or a uniform distribution.

### Adaptive sorting

A sort is **adaptive** if it runs faster on input that is already partly sorted. Insertion sort (Θ(n + inversions)), bubble sort with an early exit, and TimSort are adaptive. Heapsort and selection sort are not.

---

## Which sort should I use?

```
Need a stable sort of objects?                    → Merge sort / TimSort (Collections.sort)
Need O(1) extra space and a guaranteed n log n?   → Heapsort
General-purpose, fastest on average, in memory?   → (Randomized / dual-pivot) quicksort
Nearly sorted input, or n < ~20?                  → Insertion sort
Integer keys in a small range [0, k], k = O(n)?   → Counting sort
Fixed-width integers or strings (d digits)?       → Radix sort
Real numbers spread uniformly over [0, 1)?        → Bucket sort
Writes are very expensive (flash memory)?         → Selection sort (only n − 1 swaps)
Only need the k-th smallest or the top k?         → Quickselect / a heap of size k (no full sort)
Data doesn't fit in memory?                       → External merge sort
```

## Recommended reading order

1. [Insertion sort](comparison-sorts/insertion-sort.md), the CLRS starting point, with loop invariants
2. [Bubble](comparison-sorts/bubble-sort.md) and [selection](comparison-sorts/selection-sort.md), for contrast
3. [Merge sort](comparison-sorts/merge-sort.md), your first divide-and-conquer sort
4. Heaps → [heapsort](heapsort/heapsort.md) → [priority queues](heapsort/priority-queues.md)
5. [Quicksort](quicksort/quicksort-basics.md) → [partitioning](quicksort/partitioning.md) → [randomized](quicksort/randomized-quicksort.md) → [analysis](quicksort/complexity-analysis.md)
6. [Lower bounds](linear-time-sorting/lower-bounds.md), which explains why the next three are special
7. [Counting](linear-time-sorting/counting-sort.md) → [radix](linear-time-sorting/radix-sort.md) → [bucket](linear-time-sorting/bucket-sort.md)
8. [Min & max](order-statistics/minimum-and-maximum.md) → [quickselect](order-statistics/quickselect.md) → [median of medians](order-statistics/median-of-medians.md)

**How the code is tested:** every sorting implementation in this section is checked in its own `main()` against `Arrays.sort` on thousands of random arrays, including edge cases (empty, single element, all equal, sorted, reverse-sorted, duplicates). Stable sorts are also checked for stability. Every Java program was compiled and run with JDK 21.
