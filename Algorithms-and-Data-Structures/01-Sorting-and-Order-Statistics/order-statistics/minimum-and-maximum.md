# Minimum and Maximum

> **CLRS:** 9.1 (Minimum and maximum), Exercise 9.1-1 (second smallest) · **Status:** ✅ Written · **Prerequisites:** [Algorithm basics](../../00-Foundations/algorithm-basics.md), [Lower bounds](../linear-time-sorting/lower-bounds.md) · **Next:** [Quickselect](quickselect.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

The **i-th order statistic** of a set of n elements is the **i-th smallest** element:

| Name | i |
|---|---|
| minimum | 1 |
| maximum | n |
| (lower) median | ⌊(n + 1)/2⌋ |
| k-th smallest | k |

This page covers the simplest cases (min, max, both at once, and the second smallest) and **exactly how many comparisons they need**. [Quickselect](quickselect.md) and [median of medians](median-of-medians.md) handle general i.

**What problem does it solve?** Finding extremes is everywhere: the highest score, the cheapest flight, the latest timestamp, bounding boxes (min and max of x and y), normalisation (x − min)/(max − min), and counting sort's range detection. When done millions of times, the difference between 2n and 1.5n comparisons matters.

**Prerequisites:** the FIND-MAX loop and loop invariants ([Algorithm basics](../../00-Foundations/algorithm-basics.md)).

---

## B. Intuition

**Minimum = a knockout tournament where the smallest wins.** Every element except the winner must **lose at least once** (otherwise how would you know it isn't the minimum?). Each comparison produces exactly one loser, so you need **at least n − 1 comparisons**, and the simple scan uses exactly that many.

**Min and max together: compare in pairs.** First compare the two elements of a pair with each other. The smaller one is the only candidate for the minimum and the larger one is the only candidate for the maximum. So each pair costs **3 comparisons** (one within the pair, the smaller against the current min, the larger against the current max) instead of 4. That gives about 3n/2 instead of 2n − 2.

**Second smallest: hold a tournament.** The second smallest must have **lost only to the champion** (anyone who lost to a non-champion has two smaller elements). The champion played about lg n matches, so the runner-up is the smallest of those lg n opponents.

---

## C. How it works internally

### Minimum of [4, 2, 7, 1, 9, 3]

best = 4 → compare 2 (best = 2) → 7 → 1 (best = 1) → 9 → 3. That's **5 = n − 1 comparisons**.

### Min and max of [4, 2, 7, 1, 9, 3, 8] (n = 7, odd)

| Step | Compare | Result | min, max | Comparisons so far |
|---|---|---|---|---|
| init | (odd n) min = max = 4 | | 4, 4 | 0 |
| pair (2, 7) | 2 < 7 | small 2, large 7 | | 1 |
| | 2 < min 4? yes | | min 2 | 2 |
| | 7 > max 4? yes | | max 7 | 3 |
| pair (1, 9) | 1 < 9 | small 1, large 9 | | 4 |
| | 1 < 2? yes | | min 1 | 5 |
| | 9 > 7? yes | | max 9 | 6 |
| pair (3, 8) | 3 < 8 | | | 7 |
| | 3 < 1? no | | | 8 |
| | 8 > 9? no | | min 1, max 9 | 9 |

**9 comparisons = 3⌊7/2⌋**, against 2(n − 1) = 12 for two separate scans.

### Second smallest by tournament: [5, 2, 8, 1, 7, 3, 6, 4]

```
Round 1:   (5 vs 2 → 2)   (8 vs 1 → 1)   (7 vs 3 → 3)   (6 vs 4 → 4)
Round 2:        (2 vs 1 → 1)                 (3 vs 4 → 3)
Final:                      (1 vs 3 → 1)        champion = 1 (7 = n − 1 comparisons)

Opponents of the champion: 8 (round 1), 2 (round 2), 3 (final): lg 8 = 3 of them
Second smallest = min(8, 2, 3) = 2       (2 more comparisons)
Total: 7 + 2 = 9 = n + ⌈lg n⌉ − 2
```

**Edge cases:** n = 1 (min = max = A[0], 0 comparisons). Duplicates (a tie, whichever is fine). An empty array (undefined, so throw). Odd n in the pairwise method (initialise both min and max with the first element).

---

## D. Algorithm and pseudocode

```
MINIMUM(A)                                     ▷ CLRS 9.1: n − 1 comparisons, optimal
1  min ← A[1]
2  for i ← 2 to length[A]
3      if min > A[i]
4          min ← A[i]
5  return min

MIN-MAX(A, n)                                  ▷ 3⌊n/2⌋ comparisons
1  if n is odd
2      min ← max ← A[1]; start ← 2
3  else
4      if A[1] < A[2] then (min, max) ← (A[1], A[2]) else (min, max) ← (A[2], A[1])
5      start ← 3
6  for i ← start to n − 1 step 2              ▷ process the pair A[i], A[i+1]
7      if A[i] < A[i + 1] then small ← A[i]; large ← A[i + 1]
8      else small ← A[i + 1]; large ← A[i]
9      if small < min then min ← small
10     if large > max then max ← large
11 return (min, max)
```

**Comparison count:** n odd: (n − 1)/2 pairs × 3 = 3⌊n/2⌋. n even: 1 initial + (n − 2)/2 pairs × 3 = 3n/2 − 2. Both are **at most 3⌊n/2⌋**.

**Correctness of MIN-MAX (invariant):** before each pair, min and max are the minimum and maximum of the elements processed so far. Within a pair, the larger element can't be a new minimum unless the smaller one also is (and the smaller one is ≤ it), and symmetrically for the maximum. So checking only the smaller against min and only the larger against max is enough. ✓

**Lower bounds** (these are about *all* algorithms):
- **Min alone:** n − 1. Every non-minimum must lose a comparison at least once.
- **Min and max:** ⌈3n/2⌉ − 2. An adversary argument: each element starts with "never compared" status and needs to lose at least once (not the min) and win at least once (not the max). The adversary makes most comparisons provide at most one new piece of that information. MIN-MAX's pairing is therefore optimal.
- **Second smallest:** n + ⌈lg n⌉ − 2 (CLRS Exercise 9.1-1). The tournament achieves it.

---

## E. Implementation

```java
import java.util.*;

/** Minimum (n-1 comparisons), simultaneous min-max (3*floor(n/2)), and second smallest via tournament (n + ceil(lg n) - 2). */
public class MinMax {

    static long comparisons;

    static boolean less(int a, int b) { comparisons++; return a < b; }

    static int minimum(int[] a) {
        int min = a[0];
        for (int i = 1; i < a.length; i++) if (less(a[i], min)) min = a[i];
        return min;
    }

    static int[] minMax(int[] a) {
        int n = a.length, min, max, start;
        if (n % 2 == 1) { min = max = a[0]; start = 1; }
        else { if (less(a[0], a[1])) { min = a[0]; max = a[1]; } else { min = a[1]; max = a[0]; } start = 2; }
        for (int i = start; i + 1 < n; i += 2) {
            int small, large;
            if (less(a[i], a[i + 1])) { small = a[i]; large = a[i + 1]; } else { small = a[i + 1]; large = a[i]; }
            if (less(small, min)) min = small;
            if (less(max, large)) max = large;
        }
        return new int[]{min, max};
    }

    /** Tournament: returns {smallest, second smallest}. Each match records the loser against the winner. */
    static int[] secondSmallest(int[] a) {
        int n = a.length;
        List<List<Integer>> beaten = new ArrayList<>();            // beaten.get(i) = values that index i defeated
        for (int i = 0; i < n; i++) beaten.add(new ArrayList<>());
        List<Integer> round = new ArrayList<>();
        for (int i = 0; i < n; i++) round.add(i);
        while (round.size() > 1) {
            List<Integer> next = new ArrayList<>();
            for (int i = 0; i + 1 < round.size(); i += 2) {
                int x = round.get(i), y = round.get(i + 1);
                int win = less(a[x], a[y]) ? x : y, lose = win == x ? y : x;
                beaten.get(win).add(a[lose]);
                next.add(win);
            }
            if (round.size() % 2 == 1) next.add(round.get(round.size() - 1));   // bye
            round = next;
        }
        int champ = round.get(0);
        List<Integer> cands = beaten.get(champ);                   // at most ceil(lg n) of them
        int second = cands.get(0);
        for (int i = 1; i < cands.size(); i++) if (less(cands.get(i), second)) second = cands.get(i);
        return new int[]{a[champ], second};
    }

    static int ceilLg(int n) { return 32 - Integer.numberOfLeadingZeros(n - 1); }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        int[] a = {4, 2, 7, 1, 9, 3, 8};
        comparisons = 0; int mn = minimum(a); long cMin = comparisons;
        comparisons = 0; int[] mm = minMax(a); long cMM = comparisons;
        System.out.printf("min = %d (%d comparisons);  min,max = %d,%d (%d comparisons, vs 2(n-1) = %d)%n",
                mn, cMin, mm[0], mm[1], cMM, 2 * (a.length - 1));
        check(cMin == a.length - 1 && cMM == 3 * (a.length / 2), "n-1 and 3*floor(n/2) comparisons");

        int[] t = {5, 2, 8, 1, 7, 3, 6, 4};
        comparisons = 0; int[] ss = secondSmallest(t);
        System.out.printf("tournament on %s: smallest %d, second %d, comparisons %d (n + ceil(lg n) - 2 = %d)%n",
                Arrays.toString(t), ss[0], ss[1], comparisons, t.length + ceilLg(t.length) - 2);
        check(ss[0] == 1 && ss[1] == 2 && comparisons == t.length + ceilLg(t.length) - 2, "tournament second-smallest is optimal here");

        Random rnd = new Random(14);
        boolean ok = true, bounds = true;
        for (int trial = 0; trial < 5000; trial++) {
            int n = 2 + rnd.nextInt(300);
            int[] x = rnd.ints(n, -1000, 1000).toArray();
            int[] s = x.clone(); Arrays.sort(s);
            comparisons = 0; int m = minimum(x); if (m != s[0] || comparisons != n - 1) ok = false;
            comparisons = 0; int[] r = minMax(x);
            if (r[0] != s[0] || r[1] != s[n - 1]) ok = false;
            if (comparisons > 3 * (n / 2)) bounds = false;
            comparisons = 0; int[] q = secondSmallest(x);
            if (q[0] != s[0] || q[1] != s[1]) ok = false;
            if (comparisons > n + ceilLg(n) - 2) bounds = false;
        }
        check(ok, "5000 random arrays: min, max and second smallest all correct");
        check(bounds, "min-max <= 3*floor(n/2) and second-smallest <= n + ceil(lg n) - 2 every time");
    }
}
```

**Output:**

```
min = 1 (6 comparisons);  min,max = 1,9 (9 comparisons, vs 2(n-1) = 12)
ok   n-1 and 3*floor(n/2) comparisons
tournament on [5, 2, 8, 1, 7, 3, 6, 4]: smallest 1, second 2, comparisons 9 (n + ceil(lg n) - 2 = 9)
ok   tournament second-smallest is optimal here
ok   5000 random arrays: min, max and second smallest all correct
ok   min-max <= 3*floor(n/2) and second-smallest <= n + ceil(lg n) - 2 every time
```

**Java-specific details:** `IntSummaryStatistics stats = Arrays.stream(a).summaryStatistics();` gives min, max, sum and average in one pass. It's convenient, though it doesn't use the 3n/2 trick. For `Collection`s, use `Collections.min` and `Collections.max` (2 passes).

**Common mistakes:** initialising min to 0 or max to 0 (wrong for negative data), comparing each pair element against both min and max (4 comparisons per pair, losing the benefit), and forgetting the odd n case.

---

## F. Time complexity

| Problem | Comparisons (exact) | Time | Optimal? |
|---|---|---|---|
| Minimum or maximum | n − 1 | Θ(n) | ✅ (every non-winner must lose once) |
| Both, separately | 2n − 2 | Θ(n) | ❌ |
| Both, pairwise | ≤ 3⌊n/2⌋ | Θ(n) | ✅ (lower bound ⌈3n/2⌉ − 2) |
| Second smallest (tournament) | n + ⌈lg n⌉ − 2 | Θ(n) | ✅ |
| k-th smallest, general | — | Θ(n) expected ([quickselect](quickselect.md)), Θ(n) worst case ([median of medians](median-of-medians.md)) | ✅ asymptotically |

All of these are Θ(n) time. The differences are in the constant, which matters in tight loops (game engines, real-time analytics) and in theory (exact lower bounds).

## G. Space complexity

- Minimum and min-max: **Θ(1)**.
- Tournament: Θ(n) to record who beat whom. (A heap-based tournament can reuse the array.)

## H. Complexity summary

| Operation | Time | Comparisons | Auxiliary space |
|---|---|---|---|
| MINIMUM | Θ(n) | n − 1 | Θ(1) |
| MIN-MAX (pairs) | Θ(n) | ≤ 3⌊n/2⌋ | Θ(1) |
| Second smallest (tournament) | Θ(n) | n + ⌈lg n⌉ − 2 | Θ(n) |

## I. Advantages, limitations, and comparisons

- For **one** extreme, the simple scan is optimal and branch-predictor friendly.
- The **pairwise** trick saves 25% of the comparisons when you need both min and max. With modern SIMD instructions, the practical gain varies.
- The **tournament** generalises to the **top-k** problem, and it's how *tournament sort* and the "selection tree" in external merge sort work.
- **Streaming top-2:** keep `first` and `second` in one pass, which takes up to 2n comparisons but uses O(1) memory. Simpler, though not comparison-optimal.

**Interview follow-ups:** "Find min and max with the fewest comparisons", "Find the second largest", "Find the min of a rotated sorted array" (binary search, O(log n)), "Min of a stack in O(1)" (an auxiliary stack).

---

## J. Practice

**Beginner**
1. Find the max of [3, 9, 2, 9, 1] and count the comparisons.
2. Count MIN-MAX's comparisons for n = 10.
3. Write a function returning the index of the minimum (the first occurrence).

**Intermediate**
4. Show that the second smallest of n elements can be found with n + ⌈lg n⌉ − 2 comparisons in the worst case (CLRS Exercise 9.1-1).
5. Show that ⌈3n/2⌉ − 2 comparisons are necessary in the worst case to find both min and max (CLRS Exercise 9.1-2). *(Hint: count how many "units of information" are needed.)*
6. Find the minimum of a rotated sorted array in O(log n) (LeetCode 153).

**Advanced**
7. Find the k smallest elements with a tournament, using n + (k − 1)⌈lg n⌉ comparisons.
8. Design a data structure that supports push, pop and getMin in O(1) (LeetCode 155).

**Interview questions**

<details><summary>Q1. How can you find both the min and max with fewer than 2n comparisons?</summary>

Process the elements in pairs. Compare the two elements of a pair, then compare only the smaller one with the running min and the larger one with the running max. That's 3 comparisons per 2 elements, so about 1.5n in total, which is optimal.
</details>

<details><summary>Q2. Why is n − 1 comparisons optimal for finding the minimum?</summary>

Every element that isn't the minimum must lose at least one comparison. Otherwise the algorithm can't rule it out as the minimum. Each comparison produces exactly one loser, so at least n − 1 comparisons are needed.
</details>

<details><summary>Q3. Find the second largest element efficiently.</summary>

Simple O(n) way: one pass tracking the largest and second largest (handling duplicates as required). Comparison-optimal way: a knockout tournament (n − 1 comparisons) and then the max among the about lg n elements the champion beat, which totals n + ⌈lg n⌉ − 2.
</details>

**Worked problem: bounding box of n points.** Run MIN-MAX on the x-coordinates and on the y-coordinates: 2 × 3⌊n/2⌋ ≈ 3n comparisons in total, instead of 4n − 4 for four separate scans.

**Coding problems**
- LeetCode 153 · Find Minimum in Rotated Sorted Array *(verify link)*
- LeetCode 414 · Third Maximum Number *(verify link)*
- LeetCode 155 · Min Stack *(verify link)*
- LeetCode 1460-style "second largest" problems; GeeksforGeeks · Maximum and minimum of an array using minimum number of comparisons *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. The i-th order statistic is the i-th smallest element. Min is i = 1, max is i = n.
2. Min alone: n − 1 comparisons, which is optimal (everyone else must lose once).
3. Min and max together: pairs, ≤ 3⌊n/2⌋ comparisons, which is optimal.
4. Second smallest: tournament, n + ⌈lg n⌉ − 2 comparisons.
5. All of these are Θ(n). General selection is in [quickselect](quickselect.md).

**Formulas:** n − 1. 3⌊n/2⌋ (min-max). n + ⌈lg n⌉ − 2 (second smallest). Median index ⌊(n + 1)/2⌋.

**Common mistakes:** initialising with 0, 4 comparisons per pair, forgetting the odd n case.

**Quiz**
1. Comparisons to find the max of 100 elements?
2. Pairwise min-max comparisons for n = 100?
3. Tournament second-smallest comparisons for n = 64?
4. Lower median index of 10 elements (1-indexed)?

**Answers**

<details><summary>Show answers</summary>

1. 99.
2. 1 + 49·3 = 148 (even n). The bound 3⌊n/2⌋ = 150 also holds.
3. 64 + 6 − 2 = 68.
4. ⌊11/2⌋ = 5.
</details>

**Related topics:** [Quickselect](quickselect.md) · [Median of medians](median-of-medians.md) · [Algorithm basics](../../00-Foundations/algorithm-basics.md) · [Priority queues](../heapsort/priority-queues.md) · [Lower bounds](../linear-time-sorting/lower-bounds.md)
