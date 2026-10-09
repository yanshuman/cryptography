# Quicksort Complexity Analysis

> **CLRS:** 7.2 (Performance of quicksort), 7.4 (Analysis of quicksort) · **Status:** ✅ Written · **Prerequisites:** [Quicksort basics](quicksort-basics.md), [Randomized quicksort](randomized-quicksort.md), [Recurrence relations](../../00-Foundations/recurrence-relations.md), [Random variables](../../14-Mathematical-Foundations/random-variables.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

Quicksort's running time depends entirely on **how balanced the partitions are**. This page derives every case rigorously:

| Case | Result | Section |
|---|---|---|
| Worst case | Θ(n²) | [Worst case](#worst-case-θn²) |
| Best case | Θ(n log n) | [Best case](#best-case-θn-log-n) |
| Any constant-ratio split (even 99:1) | Θ(n log n) | [Balanced partitioning](#balanced-partitioning-why-even-91-is-fine) |
| Average or expected (random pivot) | Θ(n log n), about 1.39 n log₂ n comparisons | [Expected running time](#expected-running-time-randomized-quicksort) |
| Stack space | Θ(log n) to Θ(n) | [Space](#g-space-complexity) |

**Why it matters:** "Is quicksort O(n log n) or O(n²)?" is one of the most common interview questions, and the correct answer needs these distinctions. The expected-time proof is also the classic example of **indicator random variables**, a technique used throughout randomised algorithms.

**Prerequisites:** recurrences, harmonic numbers Hₙ = 1 + 1/2 + … + 1/n ≈ ln n, and linearity of expectation.

---

## B. Intuition

Picture the **recursion tree**. Every level does at most n work in total (the partitions at one level touch disjoint parts of the array). So:

> **total time ≈ n × (depth of the recursion tree)**

- If every split is **n − 1 / 0**, the depth is **n**, so the time is **n²**.
- If every split is **½ / ½**, the depth is **log₂ n**, so the time is **n log n**.
- If every split is **9/10 / 1/10**, the depth is **log_{10/9} n ≈ 6.6 log₂ n**, which is **still Θ(n log n)**, just with a bigger constant.

**The key insight:** as long as each split cuts off a **constant fraction**, the depth stays logarithmic. Only splits that peel off O(1) elements at a time, like the sorted-input disaster, cause quadratic time. And a random pivot almost never does that repeatedly.

---

## C. How it works internally: the analysis in detail

### Worst case: Θ(n²)

**When:** every PARTITION produces sizes n − 1 and 0. That happens on sorted, reverse-sorted or all-equal input with a fixed last-element pivot and Lomuto partitioning.

> T(n) = T(n − 1) + T(0) + Θ(n) = T(n − 1) + Θ(n)

Unrolling: T(n) = Θ(n) + Θ(n − 1) + … + Θ(1) = Θ(Σk) = **Θ(n²)**, exactly n(n − 1)/2 comparisons.

**Proving that no split is worse (CLRS §7.4.1).** The worst-case recurrence over all possible splits is

> T(n) = max_{0 ≤ q ≤ n−1} ( T(q) + T(n − q − 1) ) + Θ(n)

**Guess** T(n) ≤ cn². Substituting:

T(n) ≤ max_q ( cq² + c(n − q − 1)² ) + Θ(n) = c · max_q ( q² + (n − q − 1)² ) + Θ(n)

The expression q² + (n − q − 1)² is a convex parabola in q, so it's maximised at an **endpoint**, q = 0 or q = n − 1, where it equals (n − 1)² = n² − 2n + 1. So:

T(n) ≤ c(n² − 2n + 1) + Θ(n) ≤ cn²,  as long as c(2n − 1) dominates the Θ(n) term (choose c large enough). ∎

So the worst case is O(n²), and the sorted-input example shows it's Ω(n²). **Worst case = Θ(n²).**

### Best case: Θ(n log n)

**When:** every pivot is the median, so the sizes are ⌊(n − 1)/2⌋ and ⌈(n − 1)/2⌉.

> T(n) ≤ 2T(n/2) + Θ(n), so Θ(n log n) (Master Theorem, Case 2)

### Balanced partitioning: why even 9:1 is fine

> T(n) = T(9n/10) + T(n/10) + cn

```
Level 0:                       cn                              → cn
Level 1:          c(9n/10)            c(n/10)                  → cn
Level 2:   c(81n/100) c(9n/100)   c(9n/100) c(n/100)           → cn
  ...
Shortest path (always 1/10):  depth log₁₀ n       → levels up to here cost exactly cn each
Longest path (always 9/10):   depth log_{10/9} n  → below log₁₀ n, levels cost ≤ cn
Total ≤ cn · log_{10/9} n = Θ(n log n)
```

**For any constant split α : (1 − α)** with 0 < α ≤ ½, the depth is log_{1/(1−α)} n = Θ(log n). The exact constant comes from information theory: T(n) ≈ n log₂ n / H(α), where H(α) = −α log₂ α − (1 − α) log₂(1 − α).

| Split | H(α) | Comparisons ≈ |
|---|---|---|
| 50:50 | 1.000 | 1.00 n log₂ n |
| 75:25 | 0.811 | 1.23 n log₂ n |
| 90:10 | 0.469 | 2.13 n log₂ n |
| 99:1 | 0.081 | 12.4 n log₂ n |

Even a 99:1 split is Θ(n log n), about 12× slower than perfect, but still nowhere near n².

### Intuition for the average case: alternating good and bad splits

Suppose the pivots alternate between a terrible split (n − 1 / 0) and a perfect split. A bad level followed by a good level costs Θ(n) + Θ(n − 1) = Θ(n), and produces halves of size about (n − 1)/2: the **same result as one good split**, at only a constant factor more cost. So bad splits get "absorbed" by good ones, as long as good splits happen a constant fraction of the time.

### Expected running time (randomized quicksort)

**Setup (CLRS §7.4.2).** The running time is dominated by the comparisons in PARTITION (everything else is O(n) across all calls). Let X = the total number of comparisons. We'll compute **E[X]**.

Rename the elements by sorted order: **z₁ < z₂ < … < zₙ** (assume they're distinct), and let Zᵢⱼ = {zᵢ, zᵢ₊₁, …, zⱼ}.

**Fact 1:** two elements are compared **at most once**. Comparisons happen only between the pivot and the other elements of the current subarray, and after the partition the pivot is never used again.

So X = Σᵢ<ⱼ Xᵢⱼ, where **Xᵢⱼ = I{zᵢ is compared with zⱼ}**, an **indicator random variable** (1 if the event happens, 0 if not). By **linearity of expectation**:

> E[X] = Σᵢ₌₁ⁿ⁻¹ Σⱼ₌ᵢ₊₁ⁿ **Pr{zᵢ is compared with zⱼ}**

**Fact 2: when are zᵢ and zⱼ compared?** Look at the set Zᵢⱼ. Until some element of Zᵢⱼ is chosen as a pivot, all of Zᵢⱼ stays together in the same subarray (a pivot outside the range sends the whole range to the same side). The **first** pivot chosen from Zᵢⱼ decides everything:
- if it's **zᵢ or zⱼ**, then that pivot is compared with everything in its subarray, including the other one. **They are compared.**
- if it's some zₖ **strictly between** them, then zᵢ goes left and zⱼ goes right. **They're never compared.**

Every element of Zᵢⱼ (j − i + 1 elements) is equally likely to be the first pivot chosen from it (random pivots), so:

> **Pr{zᵢ compared with zⱼ} = 2 / (j − i + 1)**

**Summing.** Let k = j − i:

> E[X] = Σᵢ₌₁ⁿ⁻¹ Σₖ₌₁ⁿ⁻ⁱ 2/(k + 1) < Σᵢ₌₁ⁿ⁻¹ Σₖ₌₁ⁿ 2/k = Σᵢ₌₁ⁿ⁻¹ 2Hₙ < 2n·Hₙ = **O(n log n)**

because Hₙ = ln n + O(1). The exact value works out to:

> **E[X] = 2(n + 1)Hₙ − 4n ≈ 2n ln n ≈ 1.386 n log₂ n**

A matching lower bound (Ω(n log n) for any comparison sort, see [Lower bounds](../linear-time-sorting/lower-bounds.md)) gives **expected time Θ(n log n)**.

**Same result from a recurrence.** If the pivot's rank is uniform, then C(n) = (n − 1) + (1/n) Σ_{q=0}^{n−1} (C(q) + C(n − 1 − q)) = (n − 1) + (2/n) Σ_{q=0}^{n−1} C(q). Solving this (multiply by n and subtract the n − 1 version) gives the same closed form, 2(n + 1)Hₙ − 4n. The program below checks this exactly.

**Interpretation of the constant:** 1.39 n log₂ n is **39% more comparisons** than a perfect median pivot (n log₂ n), and about 39% above the information-theoretic minimum. That's the price of random pivots. Median-of-3 pivots reduce it to about 1.19 n log₂ n.

---

## D. Algorithm and pseudocode

The algorithms are unchanged from [Quicksort basics](quicksort-basics.md) and [Randomized quicksort](randomized-quicksort.md). For the analysis, it's convenient to count comparisons as:

```
COMPARISONS(n) for one partition of n elements = n − 1      ▷ the pivot is compared with every other element
```

### Proof checklist (how to present quicksort's complexity in an exam)

1. State that PARTITION is Θ(n), with n − 1 comparisons.
2. Worst case: write the max-recurrence, use convexity to show the endpoints are worst, giving Θ(n²), and give the sorted-input example.
3. Best case: 2T(n/2) + Θ(n) = Θ(n log n).
4. Constant-ratio splits: a recursion tree with logarithmic depth gives Θ(n log n).
5. Expected: indicator variables Xᵢⱼ, Pr = 2/(j − i + 1), the sum is ≤ 2nHₙ, so Θ(n log n).

---

## E. Implementation

The program verifies every formula on this page: the worst case n(n − 1)/2, the exact expected value via the recurrence vs the closed form, the measured average of randomized quicksort, and the n log n / H(α) cost of forced 9:1 splits.

```java
import java.util.*;

/** Verifies quicksort's complexity results: worst case, exact expectation, measured average, forced 9:1 splits. */
public class QuickSortAnalysis {

    static long comparisons;
    static final Random rng = new Random(77);

    static int partition(int[] a, int p, int r) {                       // Lomuto, n-1 comparisons
        int x = a[r], i = p - 1;
        for (int j = p; j < r; j++) { comparisons++; if (a[j] <= x) { i++; int t = a[i]; a[i] = a[j]; a[j] = t; } }
        int t = a[i + 1]; a[i + 1] = a[r]; a[r] = t;
        return i + 1;
    }

    static void randomized(int[] a, int p, int r) {
        while (p < r) {
            int k = p + rng.nextInt(r - p + 1); int t = a[k]; a[k] = a[r]; a[r] = t;
            int q = partition(a, p, r);
            if (q - p < r - q) { randomized(a, p, q - 1); p = q + 1; } else { randomized(a, q + 1, r); r = q - 1; }
        }
    }

    /** Quicksort whose pivot is forced to have rank floor(alpha * size) in its subarray. a holds a permutation. */
    static void forcedSplit(int[] a, int p, int r, double alpha) {
        while (p < r) {
            int[] copy = Arrays.copyOfRange(a, p, r + 1); Arrays.sort(copy);   // oracle, NOT counted
            int target = copy[(int) (alpha * (r - p))];
            for (int k = p; k <= r; k++) if (a[k] == target) { int t = a[k]; a[k] = a[r]; a[r] = t; break; }
            int q = partition(a, p, r);
            if (q - p < r - q) { forcedSplit(a, p, q - 1, alpha); p = q + 1; } else { forcedSplit(a, q + 1, r, alpha); r = q - 1; }
        }
    }

    static double harmonic(int n) { double h = 0; for (int k = 1; k <= n; k++) h += 1.0 / k; return h; }
    static double closedForm(int n) { return 2.0 * (n + 1) * harmonic(n) - 4.0 * n; }
    static double log2(double x) { return Math.log(x) / Math.log(2); }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        // 1. Exact expected comparisons: recurrence C(n) = n - 1 + (2/n) * sum_{q<n} C(q) vs the closed form
        int N = 2000;
        double[] C = new double[N + 1]; double prefix = 0;
        boolean match = true;
        for (int n = 1; n <= N; n++) {
            prefix += C[n - 1];
            C[n] = (n - 1) + 2.0 * prefix / n;
            if (Math.abs(C[n] - closedForm(n)) > 1e-6 * Math.max(1, C[n])) match = false;
        }
        System.out.printf("expected comparisons: recurrence C(1000) = %.3f, closed form 2(n+1)H_n - 4n = %.3f%n", C[1000], closedForm(1000));
        check(match, "recurrence equals 2(n+1)H_n - 4n for every n up to 2000");

        // 2. Measured average of randomized quicksort vs the formula
        System.out.printf("%n%8s %14s %14s %10s %16s%n", "n", "measured avg", "2(n+1)H_n-4n", "ratio", "/ (n log2 n)");
        boolean close = true;
        for (int n : new int[]{1000, 10_000, 100_000}) {
            int runs = n == 100_000 ? 10 : 50;
            long total = 0;
            for (int run = 0; run < runs; run++) {
                int[] a = rng.ints(n).toArray();
                comparisons = 0; randomized(a, 0, n - 1); total += comparisons;
            }
            double avg = (double) total / runs, exp = closedForm(n);
            System.out.printf("%8d %14.0f %14.0f %10.4f %16.4f%n", n, avg, exp, avg / exp, avg / (n * log2(n)));
            if (Math.abs(avg / exp - 1) > 0.02) close = false;
        }
        check(close, "measured average within 2% of the exact expectation (the last column approaches 2 ln 2 = 1.386)");

        // 3. Worst case: sorted input with a fixed pivot
        int n = 3000;
        int[] sorted = new int[n]; for (int i = 0; i < n; i++) sorted[i] = i;
        comparisons = 0;
        for (int p = 0, r = n - 1; p < r; ) { int q = partition(sorted, p, r); r = q - 1; }   // the pivot is always the max
        check(comparisons == (long) n * (n - 1) / 2, "worst case = n(n-1)/2 = " + (long) n * (n - 1) / 2 + " comparisons");

        // 4. Forced constant splits: cost ~ n log2 n / H(alpha)
        System.out.printf("%n%8s %10s %18s %18s%n", "split", "n", "comps/(n log2 n)", "theory 1/H(alpha)");
        boolean splitsOk = true;
        for (double alpha : new double[]{0.5, 0.25, 0.1}) {
            double H = -(alpha * log2(alpha) + (1 - alpha) * log2(1 - alpha));
            for (int m : new int[]{1 << 12, 1 << 14}) {
                int[] perm = new int[m]; for (int i = 0; i < m; i++) perm[i] = i;
                for (int i = m - 1; i > 0; i--) { int j = rng.nextInt(i + 1); int t = perm[i]; perm[i] = perm[j]; perm[j] = t; }
                comparisons = 0; forcedSplit(perm, 0, m - 1, alpha);
                double ratio = comparisons / (m * log2(m));
                System.out.printf("%5.0f:%-3.0f %10d %18.3f %18.3f%n", (1 - alpha) * 100, alpha * 100, m, ratio, 1 / H);
                if (ratio > 1.15 / H) splitsOk = false;
            }
        }
        check(splitsOk, "constant-ratio splits stay Theta(n log n), close to n log2 n / H(alpha)");
    }
}
```

**Output:**

```
expected comparisons: recurrence C(1000) = 10985.913, closed form 2(n+1)H_n - 4n = 10985.913
ok   recurrence equals 2(n+1)H_n - 4n for every n up to 2000

       n   measured avg   2(n+1)H_n-4n      ratio     / (n log2 n)
    1000          11003          10986     1.0016           1.1041
   10000         155697         155772     0.9995           1.1717
  100000        2009345        2018053     0.9957           1.2097
ok   measured average within 2% of the exact expectation (the last column approaches 2 ln 2 = 1.386)
ok   worst case = n(n-1)/2 = 4498500 comparisons

   split          n   comps/(n log2 n)  theory 1/H(alpha)
   50:50        4096              0.834              1.000
   50:50       16384              0.857              1.000
   75:25        4096              1.043              1.233
   75:25       16384              1.069              1.233
   90:10        4096              1.778              2.132
   90:10       16384              1.827              2.132
ok   constant-ratio splits stay Theta(n log n), close to n log2 n / H(alpha)
```

**Reading the output**
- The recurrence and the closed form agree **exactly** (to floating-point precision).
- The measured averages are within 1% of 2(n + 1)Hₙ − 4n. The last column climbs slowly toward 2 ln 2 ≈ 1.386 because of the −4n term. Asymptotic constants are approached slowly.
- Forced 90:10 splits cost about 1.8 · n log₂ n comparisons and rise slowly toward the theoretical 2.13 as n grows. That's still clearly Θ(n log n), not quadratic.

**Java-specific detail:** the forced-split experiment uses `Arrays.sort` on a copy as an "oracle" to find the pivot. Those comparisons aren't counted, because we only want PARTITION's work.

---

## F. Time complexity: final summary

| Case | Recurrence | Solution | Comparisons |
|---|---|---|---|
| Worst | T(n) = T(n − 1) + Θ(n) | **Θ(n²)** | n(n − 1)/2 |
| Best | T(n) = 2T(n/2) + Θ(n) | **Θ(n log n)** | ≈ n log₂ n |
| α : (1 − α) split | T(n) = T(αn) + T((1 − α)n) + Θ(n) | **Θ(n log n)** | ≈ n log₂ n / H(α) |
| Expected (random pivot) | E[X] = Σ 2/(j − i + 1) | **Θ(n log n)** | 2(n + 1)Hₙ − 4n ≈ 1.39 n log₂ n |
| Median-of-3 pivot (random) | | Θ(n log n) | ≈ 1.19 n log₂ n |

**Assumptions:** distinct keys for the expected-value formula. With many duplicates, use [3-way partitioning](partitioning.md).

## G. Space complexity

| Variant | Stack depth |
|---|---|
| Naive recursion, worst case | **Θ(n)** (one frame per element, which overflows the stack for large n) |
| Naive recursion, expected | Θ(log n) |
| **Recurse on the smaller side, loop on the larger** (CLRS Problem 7-4) | **O(log n) worst case**: each recursive call is on ≤ half the current size |

## H. Complexity summary

| Algorithm | Best time | Average time | Worst time | Auxiliary space | Stable? | In-place? |
|---|---|---|---|---|---|---|
| Quicksort (fixed pivot) | Θ(n log n) | Θ(n log n) (random inputs) | Θ(n²) | Θ(log n)–Θ(n) stack | ❌ | ✅ |
| Randomized quicksort | Θ(n log n) | Θ(n log n) expected (every input) | Θ(n²) w.p. → 0 | Θ(log n) | ❌ | ✅ |

## I. Advantages, limitations, and comparisons

- **Theory vs practice:** randomized quicksort makes about **39% more comparisons** than merge sort's ≈ n log₂ n, but it's still usually **faster**, because each comparison is cheaper (sequential access, no copying, a tight loop). Counting comparisons isn't the same as measuring time.
- **The analysis technique transfers:** indicator variables plus linearity of expectation also give quickselect's expected Θ(n), the expected height of random BSTs (O(log n)), and treap analysis.

**Interview follow-ups:** "Prove quicksort's expected time" (sketch the indicator-variable proof), "Why does a 90:10 split still give n log n?", "How deep can the recursion go, and how do you bound it?"

---

## J. Practice

**Beginner**
1. Show that quicksort's best-case running time is Ω(n lg n) (CLRS Exercise 7.4-2).
2. What's the running time when all elements are equal? (CLRS Exercise 7.2-2)
3. Show that the running time is Θ(n²) when the array is sorted in decreasing order (CLRS Exercise 7.2-3).

**Intermediate**
4. Show that q² + (n − q − 1)² is maximised at q = 0 or q = n − 1 (CLRS Exercise 7.4-3).
5. Show that randomized quicksort's expected running time is Ω(n lg n) (CLRS Exercise 7.4-4).
6. Solve the recurrence C(n) = (n − 1) + (2/n) Σ C(q) exactly.

**Advanced**
7. CLRS Problem 7-4: stack depth. Give a version with O(lg n) worst-case stack depth.
8. CLRS Exercise 7.4-5: with insertion sort for subarrays of size < k, the expected time is O(nk + n lg(n/k)). How should k be chosen?

**Interview questions**

<details><summary>Q1. Sketch the proof that randomized quicksort runs in expected O(n log n).</summary>

Count the comparisons. Elements zᵢ < zⱼ (by rank) are compared at most once, and only if one of them is the first pivot chosen from {zᵢ, …, zⱼ}. That happens with probability 2/(j − i + 1). By linearity of expectation, E[comparisons] = Σᵢ<ⱼ 2/(j − i + 1) ≤ 2n·Hₙ = O(n log n).
</details>

<details><summary>Q2. Why is a 90/10 split still O(n log n)?</summary>

Each level of the recursion tree does at most cn work in total, and the depth is log_{10/9} n = O(log n), because the largest subproblem shrinks by a constant factor at each level. Total O(n log n). The constant is larger (about 2.1× the balanced case), but the growth rate is the same.
</details>

<details><summary>Q3. What's the expected number of comparisons, numerically?</summary>

2(n + 1)Hₙ − 4n ≈ 2n ln n ≈ 1.39 n log₂ n. For n = 1000 that's 10,986. For n = 10⁶ it's about 2.6 × 10⁷.
</details>

**Worked problem: is quicksort with "the pivot is the 10th smallest element" O(n log n)?** No. A split of 10 / (n − 11) peels off a **constant number** of elements, not a constant fraction. The recurrence is T(n) = T(n − 11) + T(10) + Θ(n) = Θ(n²). The split must cut off a constant **fraction**.

**Coding problems:** this is a theory page. Apply it on LeetCode 912 · Sort an Array *(verify link)* (watch the time limit with sorted tests) and LeetCode 215 · Kth Largest *(verify link)* (the same analysis gives quickselect's expected Θ(n)).

---

## K. Revision notes

**Five key takeaways**
1. Time ≈ n × the recursion depth. Depth n gives n², depth log n gives n log n.
2. Worst case Θ(n²), proved via the max-recurrence and convexity (the endpoints are worst).
3. Any constant-fraction split gives Θ(n log n), with the constant 1/H(α).
4. Expected comparisons: Pr[zᵢ, zⱼ compared] = 2/(j − i + 1), so E = 2(n + 1)Hₙ − 4n ≈ 1.39 n log₂ n.
5. Recurse on the smaller side to guarantee O(log n) stack.

**Formulas**
- Worst: n(n − 1)/2
- Expected: 2(n + 1)Hₙ − 4n
- α-split: ≈ n log₂ n / H(α)
- Hₙ ≈ ln n + 0.5772

**Common mistakes:** "quicksort is O(n log n)" without naming the case, thinking unbalanced constant splits are quadratic, and forgetting the stack.

**Quiz**
1. Pr that z₃ and z₇ are compared?
2. Expected comparisons for n = 1000?
3. Depth of the recursion tree for a 99:1 split with n = 10⁶?
4. Worst-case stack depth with smaller-side recursion for n = 10⁶?

**Answers**

<details><summary>Show answers</summary>

1. 2/(7 − 3 + 1) = 2/5.
2. About 10,986.
3. log_{100/99} 10⁶ = ln(10⁶)/ln(1.0101) ≈ 1375 levels (vs about 20 for perfect splits).
4. At most log₂ 10⁶ ≈ 20.
</details>

**Related topics:** [Quicksort basics](quicksort-basics.md) · [Randomized quicksort](randomized-quicksort.md) · [Recurrence relations](../../00-Foundations/recurrence-relations.md) · [Lower bounds](../linear-time-sorting/lower-bounds.md) · [Quickselect](../order-statistics/quickselect.md) · [Random variables](../../14-Mathematical-Foundations/random-variables.md)
