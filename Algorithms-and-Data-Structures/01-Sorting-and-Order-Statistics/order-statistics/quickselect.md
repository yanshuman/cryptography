# Quickselect (Selection in Expected Linear Time)

> **CLRS:** 9.2 (RANDOMIZED-SELECT) · **Status:** ✅ Written · **Prerequisites:** [Partitioning](../quicksort/partitioning.md), [Randomized quicksort](../quicksort/randomized-quicksort.md), [Minimum and maximum](minimum-and-maximum.md) · **Next:** [Median of medians](median-of-medians.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

**Quickselect** (Hoare's FIND, CLRS's **RANDOMIZED-SELECT**) finds the **k-th smallest element** of an unsorted array in **Θ(n) expected time**, without sorting it.

It's [quicksort](../quicksort/quicksort-basics.md) with one crucial change: after partitioning, **recurse into only the side that contains the k-th element**.

**What problem does it solve?** "Find the median", "the 90th percentile", "the k-th largest" or "the top k elements". Sorting costs Θ(n log n). Quickselect does it in Θ(n) on average.

**Where it's used**
- **C++ `std::nth_element`** (introselect: quickselect with a fallback).
- Statistics and data analysis: medians, percentiles (p95 latency), and outlier trimming.
- **Top-k queries:** after selecting the k-th element, the partition leaves the top k on one side, unsorted, in O(n) total.
- Computational geometry (median splits in k-d trees), and a weighted-median subroutine in some algorithms.

**Prerequisites:** partitioning, and the idea of expected running time.

---

## B. Intuition

**Analogy: a "hot or cold" search for the 50th-tallest student among 100.** Pick a random student as a reference and split everyone into "shorter" and "taller" groups. Say 30 are shorter. Then the 50th tallest is **in the taller group**, and is the 19th shortest there (50 − 30 − 1). **Throw away the other group completely** and repeat inside the remaining one.

**Why that's linear and not n log n:** quicksort has to sort *both* sides, so every level of the recursion does about n work, and there are log n levels. Quickselect keeps only *one* side, so the work shrinks geometrically: n + n/2 + n/4 + … ≈ **2n**. A random pivot gives roughly that on average.

---

## C. How it works internally

**Find the 4th smallest (k = 4) of A = [7, 2, 9, 4, 1, 8, 3, 6, 5].** The sorted order is 1 2 3 **4** 5 6 7 8 9, so the answer is 4.

| Step | Subarray (ranks to search) | Pivot | After partition | Pivot's rank | Decision |
|---|---|---|---|---|---|
| 1 | whole array, k = 4 | 6 | [2, 4, 1, 3, 5 \| **6** \| 9, 8, 7] | 6 | 4 < 6, so go **left**, keep k = 4 |
| 2 | [2, 4, 1, 3, 5], k = 4 | 3 | [2, 1 \| **3** \| 4, 5] | 3 | 4 > 3, so go **right**, k = 4 − 3 = 1 |
| 3 | [4, 5], k = 1 | 5 | [4 \| **5**] | 2 | 1 < 2, so go **left**, k = 1 |
| 4 | [4], k = 1 | – | single element | – | return **4** ✓ |

(The pivots here are illustrative. The real algorithm picks them at random.)

Each step **discards** part of the array. Only one subarray is ever processed.

**Edge cases**

| Situation | Handling |
|---|---|
| k out of range (k < 1 or k > n) | throw an error |
| k = 1 or k = n | still works, but a [linear scan](minimum-and-maximum.md) is simpler (n − 1 comparisons) |
| Many duplicates | Lomuto degrades. Use **3-way partitioning**: if k falls in the "= pivot" block, return immediately. |
| "k-th largest" | equals the (n − k + 1)-th smallest |
| Sorted input with a fixed pivot | Θ(n²). That's why the pivot is random. |

---

## D. Algorithm and pseudocode

CLRS 9.2 (returns the i-th smallest element of A[p..r]):

```
RANDOMIZED-SELECT(A, p, r, i)
1  if p = r
2      return A[p]                              ▷ one element: it must be the answer
3  q ← RANDOMIZED-PARTITION(A, p, r)
4  k ← q − p + 1                                ▷ the pivot's rank within A[p..r]
5  if i = k                                     ▷ the pivot is the answer
6      return A[q]
7  elseif i < k
8      return RANDOMIZED-SELECT(A, p, q − 1, i)  ▷ in the low side, same rank
9  else return RANDOMIZED-SELECT(A, q + 1, r, i − k)   ▷ in the high side, rank shifted by k
```

| Line | Meaning |
|---|---|
| 3 | Partition around a random pivot. Everything in A[p..q − 1] is ≤ A[q] < everything in A[q + 1..r]. |
| 4 | k − 1 elements are smaller, so the pivot is the k-th smallest in this subarray |
| 7–8 | The answer is among the k − 1 smaller elements |
| 9 | The answer is among the larger elements, and we've skipped k of them, so look for rank i − k there |

**Correctness (strong induction on the size):** PARTITION puts the pivot at rank k. If i = k we're done. If i < k, the i-th smallest overall is the i-th smallest of the low side, since all of the low side is ≤ the pivot. If i > k, it's the (i − k)-th smallest of the high side. Each recursive call is on a strictly smaller subarray (the pivot is excluded). ∎

**Iterative form** (it's a single tail call, so it turns into a loop and uses Θ(1) stack):

```
SELECT-ITERATIVE(A, i)                         ▷ 0-indexed target rank i
1  lo ← 0; hi ← n − 1
2  while lo < hi
3      q ← RANDOMIZED-PARTITION(A, lo, hi)
4      if q = i: return A[q]
5      elseif i < q: hi ← q − 1
6      else lo ← q + 1
7  return A[lo]
```

---

## E. Implementation

```java
import java.util.*;

/** Quickselect: iterative randomized select with 3-way partitioning, top-k, and comparison counting. */
public class QuickSelect {

    static long comparisons;
    static final Random rng = new Random(15);

    /** Returns the k-th smallest (k = 1..n). Rearranges a. Theta(n) expected. */
    static int select(int[] a, int k) {
        if (k < 1 || k > a.length) throw new IllegalArgumentException("k out of range: " + k);
        int target = k - 1, lo = 0, hi = a.length - 1;      // 0-indexed rank
        while (lo < hi) {
            int pivot = a[lo + rng.nextInt(hi - lo + 1)];
            // 3-way partition of a[lo..hi]: < pivot | == pivot | > pivot
            int lt = lo, i = lo, gt = hi;
            while (i <= gt) {
                comparisons++;
                if (a[i] < pivot) swap(a, lt++, i++);
                else if (a[i] > pivot) swap(a, i, gt--);
                else i++;
            }
            if (target < lt) hi = lt - 1;                   // in the "< pivot" block
            else if (target > gt) lo = gt + 1;              // in the "> pivot" block
            else return pivot;                              // inside the "== pivot" block: done
        }
        return a[lo];
    }

    /** k-th largest = (n - k + 1)-th smallest. */
    static int kthLargest(int[] a, int k) { return select(a, a.length - k + 1); }

    /** The k largest values (unsorted) in Theta(n) expected: select, then the last k positions hold them. */
    static int[] topK(int[] a, int k) {
        int[] b = a.clone();
        select(b, b.length - k + 1);                        // partitions b around the (n-k+1)-th smallest
        return Arrays.copyOfRange(b, b.length - k, b.length);
    }

    /** CLRS-style Lomuto select with a FIXED last-element pivot: quadratic on sorted input. */
    static int selectFixedPivot(int[] a, int k) {
        int lo = 0, hi = a.length - 1, target = k - 1;
        while (lo < hi) {
            int x = a[hi], s = lo - 1;
            for (int j = lo; j < hi; j++) { comparisons++; if (a[j] <= x) swap(a, ++s, j); }
            swap(a, s + 1, hi);
            int q = s + 1;
            if (q == target) return a[q];
            if (target < q) hi = q - 1; else lo = q + 1;
        }
        return a[lo];
    }

    static void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        int[] a = {7, 2, 9, 4, 1, 8, 3, 6, 5};
        System.out.println("4th smallest of [7,2,9,4,1,8,3,6,5] = " + select(a.clone(), 4));
        System.out.println("2nd largest = " + kthLargest(a.clone(), 2) + ",  median (5th) = " + select(a.clone(), 5));
        check(select(a.clone(), 4) == 4 && kthLargest(a.clone(), 2) == 8, "examples correct");

        boolean ok = true;
        for (int t = 0; t < 5000; t++) {
            int n = 1 + rng.nextInt(300);
            int[] x = rng.ints(n, -50, 50).toArray();        // lots of duplicates
            int[] s = x.clone(); Arrays.sort(s);
            int k = 1 + rng.nextInt(n);
            if (select(x.clone(), k) != s[k - 1]) ok = false;
        }
        check(ok, "5000 random arrays (with duplicates), random k: matches sorted[k-1]");

        int[] equal = new int[100_000]; Arrays.fill(equal, 42);
        comparisons = 0; int v = select(equal, 50_000);
        check(v == 42 && comparisons == 100_000, "all-equal input: one 3-way pass (n comparisons) and done");

        int[] data = rng.ints(1000, 0, 1_000_000).toArray();
        int[] top = topK(data, 5); Arrays.sort(top);
        int[] s = data.clone(); Arrays.sort(s);
        check(Arrays.equals(top, Arrays.copyOfRange(s, 995, 1000)), "top-5 of 1000 via one select");

        // Expected comparisons for the median ~ (2 + 2 ln 2) n = 3.386 n
        System.out.printf("%n%10s %24s %12s%n", "n", "avg comparisons (median)", "/ n");
        for (int n : new int[]{1_000, 10_000, 100_000, 1_000_000}) {
            long total = 0; int runs = 20;
            for (int r = 0; r < runs; r++) {
                int[] x = rng.ints(n).toArray();
                comparisons = 0; select(x, (n + 1) / 2); total += comparisons;
            }
            System.out.printf("%10d %24.0f %12.3f%n", n, total / (double) runs, total / (double) runs / n);
        }

        int n = 20_000;
        int[] sorted = new int[n]; for (int i = 0; i < n; i++) sorted[i] = i;
        comparisons = 0; selectFixedPivot(sorted.clone(), 1);
        long fixed = comparisons;
        comparisons = 0; select(sorted.clone(), 1);
        System.out.printf("%nsorted input, k=1: fixed last-element pivot %,d comparisons; random pivot %,d%n", fixed, comparisons);
        check(fixed == (long) n * (n - 1) / 2, "fixed pivot on sorted input is quadratic (n(n-1)/2)");
        check(comparisons < 10L * n, "random pivot stays linear");
    }
}
```

**Output:**

```
4th smallest of [7,2,9,4,1,8,3,6,5] = 4
2nd largest = 8,  median (5th) = 5
ok   examples correct
ok   5000 random arrays (with duplicates), random k: matches sorted[k-1]
ok   all-equal input: one 3-way pass (n comparisons) and done
ok   top-5 of 1000 via one select

         n avg comparisons (median)          / n
      1000                     3283        3.283
     10000                    34137        3.414
    100000                   330667        3.307
   1000000                  3253294        3.253

sorted input, k=1: fixed last-element pivot 199,990,000 comparisons; random pivot 38,522
ok   fixed pivot on sorted input is quadratic (n(n-1)/2)
ok   random pivot stays linear
```

**What the output shows**
- Comparisons per element stay flat at about **3.3–3.4 per element** as n grows 1000×. That's linear growth, scattered around the theoretical constant 2 + 2 ln 2 ≈ 3.386 for finding the median. (Each value is an average of 20 random runs, and 3-way partitioning counts comparisons slightly differently from the textbook model.)
- A fixed pivot on sorted input is a disaster: about 2 × 10⁸ comparisons, against about 4 × 10⁴ with a random pivot.

**Java-specific details**
- Java has **no built-in quickselect** (no `nth_element`). Options: write one as above, use a `PriorityQueue` of size k (Θ(n log k)), or sort (Θ(n log n)).
- Quickselect **rearranges** the array. Pass a copy if the caller needs the original order.
- **3-way partitioning** makes duplicate-heavy input fast, and returns immediately when the target rank falls inside the block equal to the pivot.

**Common mistakes**
1. Recursing on **both** sides. That's quicksort, Θ(n log n).
2. Forgetting to adjust k when going right (i − k).
3. A 0-indexed vs 1-indexed rank mismatch.
4. A fixed pivot on possibly sorted data.

---

## F. Time complexity

### Worst case: Θ(n²)
Every pivot is the min or max: T(n) = T(n − 1) + Θ(n) = Θ(n²). (Measured above: 199,990,000 comparisons for n = 20,000.)

### Best case: Θ(n)
The first pivot has rank i, so one partition, Θ(n).

### Expected case: Θ(n) (CLRS §9.2)

Let T(n) be the time on n elements. The random pivot has each rank k with probability 1/n. In the worst case we recurse into the **larger** side, of size max(k − 1, n − k):

> E[T(n)] ≤ (1/n) Σₖ₌₁ⁿ E[T(max(k − 1, n − k))] + O(n)

Each value in ⌈n/2⌉ … n − 1 appears at most twice as max(k − 1, n − k), so:

> E[T(n)] ≤ (2/n) Σ_{k=⌊n/2⌋}^{n−1} E[T(k)] + an

**Substitution:** guess E[T(n)] ≤ cn. Then

E[T(n)] ≤ (2c/n) · Σ_{k=⌊n/2⌋}^{n−1} k + an ≤ (2c/n) · (3n²/8 + n/2) + an = (3/4)cn + c + an

which is ≤ cn when cn/4 − c − an ≥ 0, i.e. c > 4a and n ≥ 2c/(c − 4a). So **E[T(n)] = O(n)**, and since it's also Ω(n) (it must read the input), **Θ(n)**.

**The simpler intuition (geometric series):** a pivot is "good" (in the middle half) with probability ½, and a good pivot shrinks the problem to ≤ ¾ of its size. On average every 2 partitions include a good one, so the expected work is ≤ 2n(1 + ¾ + (¾)² + …) = 2n · 4 = **8n**, a crude but linear bound.

**Exact constants** (Knuth): expected comparisons to find the median ≈ **(2 + 2 ln 2)n ≈ 3.386n** (the measurements above scatter around this value). For the minimum or maximum it's ≈ 2n.

| Case | Time |
|---|---|
| Best | Θ(n) |
| Expected (random pivot, any input) | **Θ(n)**, about 3.39n comparisons for the median |
| Worst | Θ(n²) (vanishing probability). For a guaranteed Θ(n), see [median of medians](median-of-medians.md). |

## G. Space complexity

- **Iterative version:** **Θ(1)** auxiliary. It works **in place** (it rearranges the input).
- Recursive CLRS version: Θ(log n) expected stack, Θ(n) worst case. It's tail-recursive, so convert it to a loop.

## H. Complexity summary

| Method for the k-th smallest | Time | Extra space | Modifies input? |
|---|---|---|---|
| Sort, then index | Θ(n log n) | Θ(1)–Θ(n) | yes, or copy |
| Size-k heap | Θ(n log k) | Θ(k) | no |
| **Quickselect** | **Θ(n) expected**, Θ(n²) worst | Θ(1) | yes |
| [Median of medians](median-of-medians.md) | **Θ(n) worst case** | Θ(log n) | yes |
| Introselect (quickselect + median-of-medians fallback) | Θ(n) worst case | Θ(log n) | yes |

## I. Advantages, limitations, and comparisons

**Advantages:** linear expected time, in place, simple, and much faster than sorting for single queries. Its constant (about 3.4) beats median-of-medians (about 20+).

**Limitations:** Θ(n²) worst case without safeguards. It modifies the array, isn't stable, and isn't suitable for streams (it needs all the data in memory).

| Situation | Best choice |
|---|---|
| One k-th query, data in memory | **Quickselect** |
| Streaming data, small k | size-k heap (Θ(n log k), Θ(k) memory) |
| Many queries on static data | sort once (Θ(n log n)), then Θ(1) per query |
| Running median of a stream | two heaps ([Priority queues](../heapsort/priority-queues.md)) |
| A hard real-time guarantee | median of medians or introselect |

**Interview follow-ups:** "Kth Largest Element" (the canonical question: discuss sort vs heap vs quickselect), "K closest points to origin", "Top K frequent", "Wiggle Sort II" (median + 3-way partition).

---

## J. Practice

**Beginner**
1. Find the 3rd smallest of [12, 3, 5, 7, 4, 19, 26] by hand with quickselect.
2. Convert "k-th largest" to "k-th smallest".
3. Why does quickselect only recurse on one side?

**Intermediate**
4. Show that RANDOMIZED-SELECT never makes a recursive call to a 0-length array (CLRS Exercise 9.2-1).
5. Write an iterative version of RANDOMIZED-SELECT (CLRS Exercise 9.2-3).
6. Describe a sequence of partitions that makes RANDOMIZED-SELECT take Θ(n²) to find the minimum of [3, 2, 9, 0, 7, 5, 4, 8, 6, 1] (CLRS Exercise 9.2-4).

**Advanced**
7. K Closest Points to Origin in Θ(n) expected time (LeetCode 973).
8. Wiggle Sort II (LeetCode 324): median via quickselect, then 3-way partition with virtual indexing.

**Interview questions**

<details><summary>Q1. How would you find the k-th largest element, and what are the trade-offs?</summary>

(1) Sort: O(n log n), simple. (2) A min-heap of size k: O(n log k) time, O(k) space, works on streams and doesn't modify the input. (3) Quickselect: O(n) expected, O(1) space, in place, but O(n²) worst case (made unlikely by a random pivot) and it modifies the array. For a one-off query on an array in memory, quickselect is fastest.
</details>

<details><summary>Q2. Why is quickselect O(n) on average when quicksort is O(n log n)?</summary>

Quicksort recurses into both sides, so each level does about n work over log n levels. Quickselect recurses into only one side, so the work forms a decreasing series: n + 3n/4 + (3/4)²n + … = O(n) in expectation.
</details>

<details><summary>Q3. How do you make quickselect's worst case linear?</summary>

Choose the pivot with the median-of-medians algorithm, which guarantees a 30/70 or better split, giving T(n) ≤ T(n/5) + T(7n/10) + O(n) = O(n). In practice, introselect uses fast quickselect and switches to median of medians only if progress stalls.
</details>

**Worked problem: K Closest Points to Origin (LeetCode 973).** Compute the squared distances (avoiding `Math.sqrt`) and quickselect on distance to find the k-th smallest. The first k positions are then the answer, in any order. That's Θ(n) expected, compared with Θ(n log k) using a max-heap of size k.

**Coding problems**
- LeetCode 215 · Kth Largest Element in an Array *(verify link)*
- LeetCode 973 · K Closest Points to Origin *(verify link)*
- LeetCode 347 · Top K Frequent Elements (quickselect on frequencies) *(verify link)*
- LeetCode 324 · Wiggle Sort II *(verify link)*
- LeetCode 462 · Minimum Moves to Equal Array Elements II (median) *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. Partition, then recurse into **only** the side holding rank k.
2. Θ(n) expected (about 3.39n comparisons for the median), Θ(n²) worst case.
3. Iterative and in place: Θ(1) extra space. It rearranges the array.
4. Use a random pivot and 3-way partitioning (for duplicates).
5. Median of medians and introselect give a Θ(n) worst case.

**Formulas:** E[T(n)] ≤ (2/n) Σ_{k ≥ n/2} E[T(k)] + O(n) = O(n). Median comparisons ≈ (2 + 2 ln 2)n.

**Common mistakes:** recursing on both sides, forgetting to adjust the rank, 0- vs 1-indexing.

**Quiz**
1. After partitioning, the pivot has rank 7 and you want rank 10. Where do you search, and for which rank?
2. Expected comparisons to find the median of 10⁶ elements?
3. Space complexity of iterative quickselect?
4. Which is better for the k-th largest in a stream: quickselect or a heap?

**Answers**

<details><summary>Show answers</summary>

1. The right side, for rank 10 − 7 = 3.
2. About 3.39 × 10⁶.
3. Θ(1).
4. The heap (size k). Quickselect needs all the data in memory at once.
</details>

**Related topics:** [Minimum and maximum](minimum-and-maximum.md) · [Median of medians](median-of-medians.md) · [Partitioning](../quicksort/partitioning.md) · [Randomized quicksort](../quicksort/randomized-quicksort.md) · [Priority queues](../heapsort/priority-queues.md)
