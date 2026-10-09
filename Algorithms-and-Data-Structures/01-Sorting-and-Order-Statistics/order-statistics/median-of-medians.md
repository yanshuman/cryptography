# Median of Medians (Selection in Worst-Case Linear Time)

> **CLRS:** 9.3 (SELECT) · **Status:** ✅ Written · **Prerequisites:** [Quickselect](quickselect.md), [Recurrence relations](../../00-Foundations/recurrence-relations.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

**Median of medians** (Blum, Floyd, Pratt, Rivest and Tarjan, 1973; CLRS's **SELECT**) finds the k-th smallest element in **Θ(n) worst-case time**. That's a guarantee, not just an expectation.

It's quickselect with a **carefully chosen pivot**: the median of the medians of groups of 5. That pivot is guaranteed to have **at least ~30% of the elements on each side**, so every round discards at least ~30% of the array, whatever the input.

**What problem does it solve?** [Quickselect](quickselect.md) is Θ(n) **expected** but Θ(n²) in the worst case. Median of medians gives a **deterministic** linear bound. That matters:
- as a **theoretical result**: selection is Θ(n), unlike sorting;
- as a **fallback** in introselect (C++ `std::nth_element` implementations guard quickselect this way);
- for building deterministic Θ(n log n) worst-case quicksort, using the true median as the pivot;
- in other worst-case-linear algorithms that need a good splitter (some geometry and LP algorithms).

**Prerequisites:** quickselect, recurrences with unequal subproblem sizes.

---

## B. Intuition

**Analogy: choosing a "typical" student fairly quickly.** Split the class into small teams of 5. Each team names its own median-height member, which is easy since sorting 5 people is trivial. Then take the median of those team medians. That person isn't necessarily the true median, but they're guaranteed to be **taller than a lot of people and shorter than a lot of people**:
- they're taller than half of the team medians,
- and each of those team medians is taller than 2 of their own teammates,
- so they're taller than about 3/5 × 1/2 = **3/10 of the whole class**. By symmetry, they're also shorter than about 3/10.

With a pivot like that, partitioning always throws away **at least 30%** of the array. The trick is that finding this pivot is itself a smaller selection problem (the median of n/5 medians), solved **recursively**.

---

## C. How it works internally

### The guarantee, pictured (CLRS Figure 9.1)

Arrange the groups of 5 as columns, each sorted top (small) to bottom (large), with the columns ordered by their medians (the middle row). Let x be the median of medians:

```
          ← medians smaller than x      x     medians larger than x →
small  ·      ·      ·      ·      ·    ·     ·      ·      ·      ·
       ·      ·      ·      ·      ·    ·     ·      ·      ·      ·
median ■      ■      ■      ■      ■   [x]    □      □      □      □      ← the middle row
       ·      ·      ·      ·      ·    ●     ●      ●      ●      ●
large  ·      ·      ·      ·      ·    ●     ●      ●      ●      ●

● = elements guaranteed to be ≥ x: the bottom 3 of each column from x's column rightwards
    (each column's median ≥ x, and the 2 elements below each median are larger still)
```

- **At least half** of the ⌈n/5⌉ groups have a median ≥ x. Each of those groups contributes **3 elements ≥ x** (its median and the two below it). Excluding x's own group and a possible partial last group:

  > number of elements > x ≥ 3·(⌈½⌈n/5⌉⌉ − 2) ≥ **3n/10 − 6**

- Symmetrically, at least 3n/10 − 6 elements are < x.
- So the recursive call after partitioning is on at most **n − (3n/10 − 6) = 7n/10 + 6** elements.

### A small trace

Find the median (k = 13) of 25 elements: 1 to 25 shuffled into 5 groups of 5.

```
Groups:                   [12 3 25 7 18] [9 21 1 15 6] [24 4 11 20 2] [8 16 23 5 13] [19 10 22 14 17]
Sorted, median marked:    3 7 [12] 18 25 | 1 6 [9] 15 21 | 2 4 [11] 20 24 | 5 8 [13] 16 23 | 10 14 [17] 19 22
Medians:                  12, 9, 11, 13, 17   → median of medians (recursive SELECT on 5 values) = 12
Partition around 12:      11 elements < 12, then 12, then 13 elements > 12
k = 13 > 12 (12 is rank 12), so recurse on the 13 larger elements for rank 13 − 12 = 1 → 13
```

12 has rank 12 in 1..25, close to the middle, exactly as the guarantee promises.

---

## D. Algorithm and pseudocode

CLRS 9.3, SELECT(A, i) returns the i-th smallest:

```
SELECT(A, i)
1  if n ≤ 5 (or any small constant): sort A directly and return A[i]
2  divide the n elements into ⌈n/5⌉ groups of 5 (the last group may be smaller)
3  find the median of each group by insertion-sorting the group
4  x ← SELECT(medians, ⌈⌈n/5⌉/2⌉)                ▷ recursive median of the medians
5  partition A around x; let k = rank of x       ▷ k − 1 elements on the low side
6  if i = k: return x
7  elseif i < k: return SELECT(low side, i)
8  else return SELECT(high side, i − k)
```

| Step | Cost |
|---|---|
| 2–3 | Θ(n): ⌈n/5⌉ groups × constant work |
| 4 | T(⌈n/5⌉) |
| 5 | Θ(n) |
| 7–8 | at most T(7n/10 + 6) |

**Correctness:** steps 5–8 are exactly [quickselect](quickselect.md)'s logic, which is correct for any pivot. The pivot choice only affects running time.

---

## E. Implementation

```java
import java.util.*;

/** Median-of-medians SELECT (CLRS 9.3): Theta(n) worst case. Measures the split guarantee and the group-size effect. */
public class MedianOfMedians {

    static long comparisons;
    static double worstSplitFraction;                 // smallest (side discarded)/(n) seen at large n

    /** Returns the k-th smallest (0-indexed) of a[lo..hi]; rearranges the array. */
    static int select(int[] a, int lo, int hi, int k, int g) {
        while (true) {
            int n = hi - lo + 1;
            if (n <= 2 * g) { insertionSort(a, lo, hi); return a[lo + k]; }
            int pivot = medianOfMedians(a, lo, hi, g);
            // 3-way partition so duplicates can't break the guarantee
            int lt = lo, i = lo, gt = hi;
            while (i <= gt) {
                comparisons++;
                if (a[i] < pivot) swap(a, lt++, i++);
                else if (a[i] > pivot) swap(a, i, gt--);
                else i++;
            }
            if (n >= 1000) worstSplitFraction = Math.min(worstSplitFraction,
                    Math.min(lt - lo + (gt - lt + 1), hi - gt + (gt - lt + 1)) / (double) n);
            int target = lo + k;
            if (target < lt) { hi = lt - 1; }
            else if (target > gt) { k = target - (gt + 1); lo = gt + 1; continue; }
            else return pivot;
            k = target - lo;
        }
    }

    /** Median of the medians of groups of size g, moved to the front of the range and selected recursively. */
    static int medianOfMedians(int[] a, int lo, int hi, int g) {
        int numMedians = 0;
        for (int s = lo; s <= hi; s += g) {
            int e = Math.min(s + g - 1, hi);
            insertionSort(a, s, e);
            swap(a, lo + numMedians++, s + (e - s) / 2);   // collect the group medians at the front
        }
        return select(a, lo, lo + numMedians - 1, (numMedians - 1) / 2, g);
    }

    static void insertionSort(int[] a, int lo, int hi) {
        for (int j = lo + 1; j <= hi; j++) {
            int key = a[j], i = j - 1;
            while (i >= lo) { comparisons++; if (a[i] <= key) break; a[i + 1] = a[i]; i--; }
            a[i + 1] = key;
        }
    }

    static void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }

    static int kth(int[] a, int k) { return select(a, 0, a.length - 1, k - 1, 5); }   // 1-indexed k

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        Random rnd = new Random(16);

        // Correctness, including adversarial patterns and duplicates
        boolean ok = true;
        for (int t = 0; t < 4000; t++) {
            int n = 1 + rnd.nextInt(400);
            int[] x = switch (t % 4) {
                case 0 -> rnd.ints(n).toArray();
                case 1 -> rnd.ints(n, 0, 5).toArray();                                   // heavy duplicates
                case 2 -> java.util.stream.IntStream.range(0, n).toArray();               // sorted
                default -> java.util.stream.IntStream.range(0, n).map(i -> n - i).toArray(); // reversed
            };
            int[] s = x.clone(); Arrays.sort(s);
            int k = 1 + rnd.nextInt(n);
            if (kth(x, k) != s[k - 1]) ok = false;
        }
        check(ok, "4000 arrays (random, duplicates, sorted, reversed) with random k match sorted[k-1]");

        // Linear worst-case behaviour: comparisons / n stays bounded on every input pattern
        System.out.printf("%n%10s %16s %16s %16s%n", "n", "random  comps/n", "sorted  comps/n", "organ   comps/n");
        double maxRatio = 0;
        worstSplitFraction = 1;
        for (int n : new int[]{10_000, 100_000, 1_000_000}) {
            int[] random = rnd.ints(n).toArray();
            int[] sorted = java.util.stream.IntStream.range(0, n).toArray();
            int[] organ = new int[n];
            for (int i = 0; i < n; i++) organ[i] = i < n / 2 ? i : n - i;
            double[] r = new double[3]; int c = 0;
            for (int[] arr : new int[][]{random, sorted, organ}) {
                comparisons = 0; kth(arr, (n + 1) / 2); r[c++] = comparisons / (double) n;
            }
            System.out.printf("%10d %16.2f %16.2f %16.2f%n", n, r[0], r[1], r[2]);
            for (double v : r) maxRatio = Math.max(maxRatio, v);
        }
        check(maxRatio < 30, "comparisons per element bounded by a constant on all patterns (linear worst case)");
        System.out.printf("smallest fraction kept on the discarded side, over all partitions with n >= 1000: %.3f (guarantee ~ 0.3)%n", worstSplitFraction);
        check(worstSplitFraction >= 0.29, "every partition discards at least ~30% of the elements");

        // Why groups of 5? Compare group sizes on the median of 2^18 random ints
        System.out.printf("%n%8s %18s%n", "groups", "comps/n at n=2^18");
        for (int g : new int[]{3, 5, 7, 9}) {
            int[] arr = rnd.ints(1 << 18).toArray();
            comparisons = 0; select(arr, 0, arr.length - 1, arr.length / 2, g);
            System.out.printf("%8d %18.2f%n", g, comparisons / (double) arr.length);
        }
    }
}
```

**Output:**

```
ok   4000 arrays (random, duplicates, sorted, reversed) with random k match sorted[k-1]

         n  random  comps/n  sorted  comps/n  organ   comps/n
     10000             7.81             6.74             7.46
    100000             8.31             7.08             7.92
   1000000             8.50             7.21             8.11
ok   comparisons per element bounded by a constant on all patterns (linear worst case)
smallest fraction kept on the discarded side, over all partitions with n >= 1000: 0.394 (guarantee ~ 0.3)
ok   every partition discards at least ~30% of the elements

  groups  comps/n at n=2^18
       3               9.78
       5               8.51
       7               8.68
       9               9.49
```

**What the output shows**
- **Comparisons per element stay flat** (about 7–8.5) on random, sorted and organ-pipe inputs as n grows 100×. That's a **linear worst case**. Compare quickselect's 3.4 on average, but about 2 × 10⁸ on sorted input with a fixed pivot.
- **The split guarantee holds:** the worst partition still discarded 39% of its elements, comfortably above the theoretical ~30% (which is a worst-case bound, and real data usually does better).
- **The price:** about 2.5× more comparisons than quickselect's ~3.4n, plus extra data movement for grouping. That's why it's used as a fallback rather than as the default.
- **Group sizes:** groups of 3 are the worst of the four, and 5–7 are the sweet spot. At this n the gap is modest, but asymptotically groups of 3 give Θ(n log n), so the gap keeps widening as n grows (see Section F).

**Java-specific details:** `switch` expressions (Java 14+) build the test inputs concisely. The algorithm works in place using index ranges. The recursion depth is O(log n), because each level shrinks by a constant factor.

**Common mistakes**
1. Using groups of 3: the recurrence becomes T(n/3) + T(2n/3) + n = Θ(n log n).
2. Taking the median of medians by **sorting** the medians (Θ(n log n)) instead of selecting recursively.
3. Two-way partitioning with many duplicates, which can break the 30% guarantee. Use 3-way partitioning, or make keys distinct.

---

## F. Time complexity

### The recurrence

> **T(n) ≤ T(⌈n/5⌉) + T(7n/10 + 6) + O(n)**

### Why it's linear (substitution, CLRS §9.3)

Guess T(n) ≤ cn. Then:

T(n) ≤ c⌈n/5⌉ + c(7n/10 + 6) + an
≤ cn/5 + c + 7cn/10 + 6c + an
= **(9/10)cn** + 7c + an
= cn + (−cn/10 + 7c + an)

That's ≤ cn when −cn/10 + 7c + an ≤ 0, i.e. **c ≥ 10a·n/(n − 70)**, which holds for n ≥ 140 with c ≥ 20a. So **T(n) = O(n)**, and therefore Θ(n). ∎

**The key fact:** 1/5 + 7/10 = **9/10 < 1**. The total size of the two recursive subproblems is a constant fraction smaller than n, so the recursion tree's level sums form a decreasing geometric series: n + 0.9n + 0.81n + … = 10n.

### Why groups of 5 and not 3?

| Group size g | Recursive sizes | Sum of fractions | Result |
|---|---|---|---|
| 3 | n/3 + 2n/3 | **= 1** | T(n) = Θ(n log n) ❌ (every level sums to n, and there are log n levels) |
| **5** | n/5 + 7n/10 | **9/10 < 1** | **Θ(n)** ✅ |
| 7 | n/7 + 5n/7 | 6/7 < 1 | Θ(n) ✅ (more work per group) |

| Case | Time |
|---|---|
| Best | Θ(n) |
| Average | Θ(n) |
| **Worst** | **Θ(n)**, with a large constant (the proof allows about 20n; about 7–9n comparisons measured) |

## G. Space complexity

- In place, apart from recursion.
- **Recursion stack: Θ(log n)**, because the sizes shrink by a constant factor (≤ 7/10) at each nested level.

## H. Complexity summary

| Algorithm | Best | Expected | Worst | Extra space | Practical speed |
|---|---|---|---|---|---|
| [Quickselect](quickselect.md) | Θ(n) | Θ(n) (~3.4n comparisons) | Θ(n²) | Θ(1) | fastest |
| **Median of medians** | Θ(n) | Θ(n) | **Θ(n)** | Θ(log n) | ~2–3× slower |
| Introselect | Θ(n) | Θ(n) | Θ(n) | Θ(log n) | ≈ quickselect |
| Sort, then index | Θ(n log n) | Θ(n log n) | Θ(n log n) | — | slow |

## I. Advantages, limitations, and comparisons

**Advantages:** a **deterministic linear worst case**, immune to adversarial inputs, and no randomness needed.

**Limitations:** large constant factors (slower than quickselect on real data), more complex code, and more data movement.

**Deterministic quicksort with a Θ(n log n) worst case:** use SELECT to find the true median as the pivot at every level: T(n) = 2T(n/2) + Θ(n) = Θ(n log n) worst case. It's theoretically elegant, but slower in practice than randomized quicksort or heapsort.

**Practical modern alternative:** *introselect* runs quickselect and switches to median of medians only if the recursion goes deeper than expected, giving average-case speed with a worst-case guarantee. Alexandrescu's "Fast Deterministic Selection" (2017) brings deterministic selection close to quickselect speed.

**Interview follow-ups:** "Can you find the median in guaranteed linear time?", "Why groups of 5?", "How would you get a guaranteed O(n log n) quicksort?" (median-of-medians pivot, or introsort's heapsort fallback).

---

## J. Practice

**Beginner**
1. Run SELECT on the 25-element trace in Section C, but with k = 5.
2. In groups of 5, how many elements are guaranteed to be ≥ the median of medians (ignoring the boundary terms)?
3. Why must the median of medians be found recursively and not by sorting?

**Intermediate**
4. Analyse SELECT with groups of 7, and with groups of 3 (CLRS Exercise 9.3-1).
5. Show that quicksort can run in O(n lg n) worst case using SELECT for the pivot (CLRS Exercise 9.3-3).
6. Given a black-box worst-case linear median subroutine, select any order statistic in linear time (CLRS Exercise 9.3-5).

**Advanced**
7. Find the k-th quantiles (k − 1 order statistics that split the set into k equal parts) in O(n lg k) (CLRS Exercise 9.3-6).
8. Find the k numbers closest to the median in O(n) (CLRS Exercise 9.3-7).

**Interview questions**

<details><summary>Q1. How does median of medians guarantee linear time?</summary>

It picks a pivot that at least ~30% of the elements are smaller than and at least ~30% are larger than, so each partition discards at least 30%. Finding the pivot costs T(n/5) and the remaining recursion costs at most T(7n/10), plus O(n) for grouping and partitioning. Since 1/5 + 7/10 < 1, T(n) = O(n).
</details>

<details><summary>Q2. Why isn't median of medians used by default?</summary>

Its constant factor is larger (about 2.5× quickselect's comparisons in the measurements above, plus extra data movement for grouping). Randomized quickselect is linear in expectation with a tiny constant, and its worst case is astronomically unlikely. Median of medians serves as a fallback (introselect) when guarantees are needed.
</details>

<details><summary>Q3. Why does using groups of 3 fail?</summary>

The guarantee becomes about n/3 elements on each side, so the recurrence is T(n) ≤ T(n/3) + T(2n/3) + O(n). The fractions sum to exactly 1, so every level of the recursion tree does about n work across log n levels, giving Θ(n log n).
</details>

**Worked problem: a guaranteed-linear "k closest to the median".** (1) Find the median m with SELECT, Θ(n). (2) Compute dᵢ = |aᵢ − m|, Θ(n). (3) Find the k-th smallest d with SELECT, Θ(n). (4) Output the elements with dᵢ ≤ that threshold, handling ties to output exactly k, Θ(n). The total is Θ(n) worst case.

**Coding problems**
- LeetCode 215 · Kth Largest Element (implement deterministic selection as an exercise) *(verify link)*
- LeetCode 462 · Minimum Moves to Equal Array Elements II (median) *(verify link)*
- GeeksforGeeks · K-th smallest element in worst-case linear time *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. Groups of 5 → group medians → the recursive median of those medians is the pivot.
2. That pivot has at least 3n/10 − 6 elements on each side, so every partition discards ≥ ~30%.
3. T(n) ≤ T(n/5) + T(7n/10 + 6) + O(n) = **Θ(n) worst case**, because 1/5 + 7/10 < 1.
4. Groups of 3 fail (they give Θ(n log n)). Groups of 5 or 7 work.
5. A large constant, so it's used as a fallback (introselect) rather than the default.

**Formulas:** elements on each side ≥ 3n/10 − 6. T(n) ≤ T(⌈n/5⌉) + T(7n/10 + 6) + O(n).

**Common mistakes:** groups of 3, sorting the medians, ignoring duplicates.

**Quiz**
1. What's the maximum size of the recursive call after partitioning (n large)?
2. What's the sum of the recursive fractions with groups of 5?
3. Is median of medians randomised?
4. Stack depth?

**Answers**

<details><summary>Show answers</summary>

1. 7n/10 + 6.
2. 1/5 + 7/10 = 9/10.
3. No, it's fully deterministic.
4. Θ(log n).
</details>

**Related topics:** [Quickselect](quickselect.md) · [Minimum and maximum](minimum-and-maximum.md) · [Recurrence relations](../../00-Foundations/recurrence-relations.md) · [Quicksort analysis](../quicksort/complexity-analysis.md) · [Sorting overview](../README.md)
