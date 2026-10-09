# Randomized Quicksort

> **CLRS:** 7.3 (A randomized version of quicksort), and Chapter 5 (probabilistic analysis) · **Status:** ✅ Written · **Prerequisites:** [Quicksort basics](quicksort-basics.md), [Partitioning](partitioning.md), [Probability](../../14-Mathematical-Foundations/probability.md) · **Next:** [Complexity analysis](complexity-analysis.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

**Randomized quicksort** chooses the pivot **uniformly at random** from the current subarray, instead of always taking the first or last element.

**What problem does it solve?** Deterministic quicksort has *bad inputs*: sorted, reverse-sorted or specially crafted arrays make it Θ(n²). Real data is often sorted or nearly sorted, and attackers can craft inputs deliberately. With a random pivot, **no particular input is bad**: the **expected** running time is Θ(n log n) for **every** input, and the chance of quadratic behaviour is astronomically small.

**This is the key conceptual shift:**

| | Deterministic quicksort | Randomized quicksort |
|---|---|---|
| Randomness comes from | the **input** (we *assume* it's random) | the **algorithm's coin flips** |
| "Average Θ(n log n)" means | averaged over all possible inputs | expected over the coin flips, **for any fixed input** |
| Sorted input | Θ(n²) every time | Θ(n log n) expected |
| An adversary who knows the code | can always force Θ(n²) | can't (they don't know the coin flips) |

**Where it's used:** most production quicksorts randomise in some way, or use deterministic sampling (median-of-three, Tukey's "ninther"). **Introsort** adds a hard guarantee by switching to heapsort. Randomisation also underlies [quickselect](../order-statistics/quickselect.md), randomised hashing, and treaps.

**Prerequisites:** quicksort, partitioning, and the idea of expected value.

---

## B. Intuition

**Analogy: a referee who flips a coin to decide who kicks off.** No team can prepare a strategy that exploits a fixed rule, because there's no fixed rule.

**Why a random pivot is usually good:** a pivot is "good enough" if it lands in the **middle half** of the sorted order, between the 25th and 75th percentile. Then the larger side is at most 3/4 of the array. A random pivot is good enough **with probability ½**. So, on average, every other partition shrinks the problem by a constant factor, and you only need O(log n) levels of good pivots. Bad pivots just waste some levels in between.

**Why bad luck doesn't add up:** to get Θ(n²) you'd need a bad pivot almost **every single time**, which has probability about (small)ⁿ. It's like flipping a fair coin and getting heads a thousand times in a row.

---

## C. How it works internally

### Effect on sorted input

```
Deterministic (last-element pivot) on [1, 2, 3, 4, 5, 6, 7, 8]:
pivot 8 → [1..7] | 8 | []      pivot 7 → [1..6] | 7 | []     ...   n levels, n²/2 comparisons

Randomized on the same array (one possible run):
random pivot = 5 → [1,2,3,4] | 5 | [6,7,8]
  random pivot = 2 → [1] | 2 | [3,4]        random pivot = 7 → [6] | 7 | [8]
  ...   about log n levels, about 1.39 n log₂ n comparisons
```

### How likely is a "bad" run?

For n = 1000, the expected number of comparisons is about 11,000. Over many runs on the **same** input, the comparison count clusters tightly around that mean. The program below shows the spread over 200 runs: every run stays within a few percent of the mean. The probability of exceeding the mean by a constant factor drops **exponentially** fast as n grows (it's a known "high-probability" bound: O(n log n) with probability 1 − 1/n^c).

### Pivot-selection strategies

| Strategy | Expected time | Worst case | Notes |
|---|---|---|---|
| First or last element | Θ(n log n) on *random* inputs only | Θ(n²) on sorted input | Never use it on real data |
| **Random element** | **Θ(n log n) expected on every input** | Θ(n²) with negligible probability | CLRS 7.3 |
| Median of three (first, middle, last) | Θ(n log n), and fast on sorted input | Θ(n²) on crafted "median-of-3 killer" inputs | Deterministic, cheap, common |
| Random median of three | ~1.19 n log₂ n comparisons | negligible | Fewer comparisons than plain random |
| Tukey's ninther (median of 3 medians of 3) | even better pivots for large n | crafted inputs exist | Used in Bentley–McIlroy `qsort` |
| **Introsort** (any of the above + a depth limit) | Θ(n log n) | **Θ(n log n) guaranteed** | Switches to heapsort at depth 2⌊log₂ n⌋ |
| Median of medians | Θ(n log n) | Θ(n log n) guaranteed | Too slow in practice. See [Median of medians](../order-statistics/median-of-medians.md). |

---

## D. Algorithm and pseudocode

CLRS 7.3:

```
RANDOMIZED-PARTITION(A, p, r)
1  i ← RANDOM(p, r)                    ▷ uniform over p..r
2  exchange A[r] ↔ A[i]                ▷ move the random pivot to the end
3  return PARTITION(A, p, r)           ▷ then the ordinary Lomuto partition

RANDOMIZED-QUICKSORT(A, p, r)
1  if p < r
2      q ← RANDOMIZED-PARTITION(A, p, r)
3      RANDOMIZED-QUICKSORT(A, p, q − 1)
4      RANDOMIZED-QUICKSORT(A, q + 1, r)
```

**Why swap the random element to the end?** It reuses the existing PARTITION unchanged. The rest of the algorithm is identical.

**Correctness** is identical to ordinary quicksort: PARTITION works for *any* pivot value. Randomness affects only the running time, never the result. (Algorithms like this are called **Las Vegas** algorithms: always correct, with random running time. By contrast, **Monte Carlo** algorithms are always fast but sometimes wrong, like Miller–Rabin [primality testing](../../09-Number-Theoretic-Algorithms/primality-testing.md).)

### Introsort (the production design)

```
INTROSORT(A, p, r, depthLimit)
1  while r − p + 1 > 16
2      if depthLimit = 0
3          HEAPSORT(A[p..r]); return    ▷ too many bad pivots: guarantee O(n log n)
4      depthLimit ← depthLimit − 1
5      q ← PARTITION with median-of-3 or random pivot
6      recurse on the smaller side; loop on the larger
7  INSERTION-SORT(A, p, r)              ▷ small pieces
▷ initial call: depthLimit = 2⌊lg n⌋
```

---

## E. Implementation

```java
import java.util.*;
import java.util.concurrent.ThreadLocalRandom;

/** Randomized quicksort (CLRS 7.3), its concentration around the mean, and a compact introsort. */
public class RandomizedQuickSort {

    static long comparisons;
    static Random rng = new Random(2024);                     // seeded for reproducible output

    // ---------- CLRS randomized quicksort ----------
    static void randomizedQuicksort(int[] a, int p, int r) {
        while (p < r) {                                       // smaller side recursively: O(log n) stack
            int q = randomizedPartition(a, p, r);
            if (q - p < r - q) { randomizedQuicksort(a, p, q - 1); p = q + 1; }
            else               { randomizedQuicksort(a, q + 1, r); r = q - 1; }
        }
    }
    static int randomizedPartition(int[] a, int p, int r) {
        int i = p + rng.nextInt(r - p + 1);                   // RANDOM(p, r)
        swap(a, i, r);
        return partition(a, p, r);
    }
    static int partition(int[] a, int p, int r) {             // Lomuto
        int x = a[r], i = p - 1;
        for (int j = p; j < r; j++) { comparisons++; if (a[j] <= x) swap(a, ++i, j); }
        swap(a, i + 1, r);
        return i + 1;
    }
    static void deterministicQuicksort(int[] a, int p, int r) {   // last-element pivot, for comparison
        while (p < r) {
            int q = partition(a, p, r);
            if (q - p < r - q) { deterministicQuicksort(a, p, q - 1); p = q + 1; }
            else               { deterministicQuicksort(a, q + 1, r); r = q - 1; }
        }
    }
    static void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }

    // ---------- Introsort: median-of-3 quicksort + heapsort fallback + insertion sort ----------
    static int heapsortFallbacks;
    static void introsort(int[] a) {
        int depth = 2 * (31 - Integer.numberOfLeadingZeros(Math.max(1, a.length)));
        intro(a, 0, a.length - 1, depth);
    }
    static void intro(int[] a, int lo, int hi, int depth) {
        while (hi - lo + 1 > 16) {
            if (depth-- == 0) { heapsortFallbacks++; heapsort(a, lo, hi); return; }
            int mid = lo + (hi - lo) / 2;                     // median-of-3 -> a[hi]
            if (a[mid] < a[lo]) swap(a, mid, lo);
            if (a[hi] < a[lo]) swap(a, hi, lo);
            if (a[mid] < a[hi]) swap(a, mid, hi);             // now a[lo] <= a[hi] <= a[mid]
            int q = partition(a, lo, hi);
            if (q - lo < hi - q) { intro(a, lo, q - 1, depth); lo = q + 1; }
            else                 { intro(a, q + 1, hi, depth); hi = q - 1; }
        }
        for (int j = lo + 1; j <= hi; j++) {                  // insertion sort for small pieces
            int key = a[j], i = j - 1;
            while (i >= lo && a[i] > key) { a[i + 1] = a[i]; i--; }
            a[i + 1] = key;
        }
    }
    static void heapsort(int[] a, int lo, int hi) {
        int n = hi - lo + 1;
        for (int i = n / 2 - 1; i >= 0; i--) sift(a, lo, i, n);
        for (int end = n - 1; end > 0; end--) { swap(a, lo, lo + end); sift(a, lo, 0, end); }
    }
    static void sift(int[] a, int lo, int i, int n) {
        while (2 * i + 1 < n) {
            int c = 2 * i + 1;
            if (c + 1 < n && a[lo + c + 1] > a[lo + c]) c++;
            if (a[lo + i] >= a[lo + c]) return;
            swap(a, lo + i, lo + c); i = c;
        }
    }

    static double expected(int n) { double h = 0; for (int k = 1; k <= n; k++) h += 1.0 / k; return 2 * (n + 1) * h - 4 * n; }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        // Correctness
        boolean ok = true, okIntro = true;
        for (int t = 0; t < 3000; t++) {
            int[] x = rng.ints(rng.nextInt(400), -100, 100).toArray();
            int[] exp = x.clone(); Arrays.sort(exp);
            int[] y = x.clone(); randomizedQuicksort(y, 0, y.length - 1);
            int[] z = x.clone(); introsort(z);
            if (!Arrays.equals(y, exp)) ok = false;
            if (!Arrays.equals(z, exp)) okIntro = false;
        }
        check(ok, "randomized quicksort: 3000 random arrays match Arrays.sort");
        check(okIntro, "introsort: 3000 random arrays match Arrays.sort");

        // Sorted input: deterministic vs randomized
        int n = 5000;
        int[] sorted = new int[n]; for (int i = 0; i < n; i++) sorted[i] = i;
        comparisons = 0; deterministicQuicksort(sorted.clone(), 0, n - 1); long det = comparisons;
        comparisons = 0; randomizedQuicksort(sorted.clone(), 0, n - 1);    long rnd = comparisons;
        System.out.printf("sorted input, n=%d: deterministic %,d comparisons, randomized %,d (expected %,.0f)%n",
                n, det, rnd, expected(n));
        check(det == (long) n * (n - 1) / 2, "deterministic pivot is quadratic on sorted input");
        check(rnd < 1.5 * expected(n), "random pivot is ~n log n on the very same input");

        // Concentration: 200 runs on the SAME input
        int m = 1000;
        int[] input = new int[m]; for (int i = 0; i < m; i++) input[i] = i;     // sorted (the adversarial case)
        long min = Long.MAX_VALUE, max = 0, sum = 0;
        for (int run = 0; run < 200; run++) {
            comparisons = 0; randomizedQuicksort(input.clone(), 0, m - 1);
            min = Math.min(min, comparisons); max = Math.max(max, comparisons); sum += comparisons;
        }
        double mean = sum / 200.0, e = expected(m);
        System.out.printf("200 runs on the same sorted array (n=%d): min %,d  mean %,.0f  max %,d   theory 2(n+1)H_n - 4n = %,.0f%n",
                m, min, mean, max, e);
        check(Math.abs(mean - e) / e < 0.03, "empirical mean within 3% of the exact expected value");
        check(max < 1.35 * e, "even the worst of 200 runs stays close to the mean (no run near n^2/2 = 499,500)");

        // Introsort guarantee on an organ-pipe input (hard for median-of-3)
        int big = 1 << 16;
        int[] organ = new int[big];
        for (int i = 0; i < big / 2; i++) { organ[i] = i; organ[big - 1 - i] = i; }
        int[] exp = organ.clone(); Arrays.sort(exp);
        heapsortFallbacks = 0; introsort(organ);
        check(Arrays.equals(organ, exp), "introsort sorts a 65,536-element organ-pipe array (heapsort fallbacks used: " + heapsortFallbacks + ")");

        // ThreadLocalRandom is the idiomatic generator in concurrent code
        int pick = ThreadLocalRandom.current().nextInt(0, 10);
        check(pick >= 0 && pick < 10, "ThreadLocalRandom.nextInt(lo, hi) gives a pivot index in [lo, hi)");
    }
}
```

**Output:**

```
ok   randomized quicksort: 3000 random arrays match Arrays.sort
ok   introsort: 3000 random arrays match Arrays.sort
sorted input, n=5000: deterministic 12,497,500 comparisons, randomized 73,207 (expected 70,963)
ok   deterministic pivot is quadratic on sorted input
ok   random pivot is ~n log n on the very same input
200 runs on the same sorted array (n=1000): min 9,783  mean 10,992  max 13,735   theory 2(n+1)H_n - 4n = 10,986
ok   empirical mean within 3% of the exact expected value
ok   even the worst of 200 runs stays close to the mean (no run near n^2/2 = 499,500)
ok   introsort sorts a 65,536-element organ-pipe array (heapsort fallbacks used: 9)
ok   ThreadLocalRandom.nextInt(lo, hi) gives a pivot index in [lo, hi)
```

**What the output shows**
- On **the same sorted array**, the deterministic pivot needs about 12.5 million comparisons, while the random pivot needs about 73 thousand, roughly **170× fewer**.
- Over 200 runs, the comparison counts range only from about 9.8k to 13.7k (−11% to +25%) around a mean of about 11.0k. That mean matches the exact formula 2(n + 1)Hₙ − 4n = 10,986 to within a fraction of a percent. Randomized quicksort's running time is **very predictable**, even though it's random. The quadratic worst case, 499,500, never comes close.
- **Introsort earned its keep:** the "organ-pipe" input (0, 1, …, n/2, …, 1, 0) is hard for deterministic median-of-three pivots, and introsort hit its depth limit **9 times**, switching those subarrays to heapsort. The result is still correct and still O(n log n). That's exactly the safety net it exists for.

**Java-specific details**
- Use **one shared `Random`** (or `ThreadLocalRandom.current()`), never `new Random()` per call, which is slow and can correlate between calls.
- Seeding (`new Random(2024)`) makes runs reproducible for testing. In production, leave it unseeded so attackers can't predict the pivots.
- `Math.random()` works but is slower (it's synchronised internally) and returns a double.

**Common mistakes**
1. Choosing `RANDOM(p, r)` with an off-by-one (`nextInt(r - p)` never picks r).
2. Randomising once at the top level only. The pivot must be random **at every partition**.
3. Shuffling the whole array first and then using a fixed pivot. That's actually **equivalent** in expectation and is a valid alternative (Sedgewick's recommendation), but it costs an extra Θ(n) pass.

---

## F. Time complexity

| Measure | Value | Holds for |
|---|---|---|
| **Expected** running time | **Θ(n log n)** | every input (expectation over the pivot choices) |
| Expected comparisons | 2(n + 1)Hₙ − 4n ≈ **2n ln n ≈ 1.39 n log₂ n** | every input with distinct keys |
| Worst-case running time | Θ(n²) | an unlucky sequence of pivots, with probability ≤ e^(−Ω(n)) of being quadratic |
| Best case | Θ(n log n) | the pivots happen to be medians |
| With high probability | O(n log n) | with probability ≥ 1 − 1/n^c |
| Introsort worst case | **Θ(n log n)** | guaranteed by the heapsort fallback |

The full derivation of the expected comparison count (indicator random variables: elements zᵢ and zⱼ are compared with probability 2/(j − i + 1)) is in [Complexity analysis](complexity-analysis.md#expected-running-time-randomized-quicksort).

**Duplicates:** with many equal keys, randomisation alone doesn't help Lomuto (all-equal input is still Θ(n²)). Combine randomisation with [3-way partitioning](partitioning.md).

## G. Space complexity

- In place, Θ(1) auxiliary.
- Stack: **Θ(log n) expected** depth. With smaller-side-first recursion (as in the code), it's **O(log n) guaranteed**.
- The generator's state is O(1).

## H. Complexity summary

| Algorithm | Best time | Average time | Worst time | Auxiliary space | Stable? | In-place? |
|---|---|---|---|---|---|---|
| Randomized quicksort | Θ(n log n) | Θ(n log n) **expected** (every input) | Θ(n²) (vanishing probability) | Θ(log n) stack | ❌ No | ✅ Yes |
| Introsort | Θ(n log n) | Θ(n log n) | **Θ(n log n)** | Θ(log n) | ❌ No | ✅ Yes |

## I. Advantages, limitations, and comparisons

**Advantages:** it removes bad inputs, keeps quicksort's speed, and the expected-time guarantee holds for any data distribution. It's also simple: one extra line.

**Limitations:** it's still Θ(n²) in the theoretical worst case (introsort fixes that), it needs a random number generator, and it's slightly slower per partition. It doesn't fix the duplicate problem by itself.

**Comparison with the alternatives**
- **Median-of-3:** cheaper, and great on sorted data, but deterministic, so crafted inputs exist.
- **Shuffling first:** the same expectation, plus one extra pass.
- **Introsort:** what libraries actually ship, because it gives the best of both.

**Interview follow-ups:** "What does expected O(n log n) mean, and how is it different from average-case?", "Can randomized quicksort still be O(n²)?", "How do production libraries avoid the worst case?"

---

## J. Practice

**Beginner**
1. Why do we analyse the expected running time of a randomized algorithm, rather than its worst case? (CLRS Exercise 7.3-1)
2. How many calls to RANDOM does RANDOMIZED-QUICKSORT make in the worst case? In the best case? (CLRS Exercise 7.3-2)
3. Modify randomized quicksort to use 3-way partitioning.

**Intermediate**
4. Show that the probability of a "good" pivot (in the middle half) is ½.
5. Implement random median-of-3 and measure the comparisons against plain random.
6. Prove that shuffling the input first and then using a fixed pivot has the same expected cost.

**Advanced**
7. Show that randomized quicksort runs in O(n log n) with high probability (use Chernoff bounds on the number of good pivots along each root-to-leaf path).
8. Construct a median-of-3 killer sequence for n = 16.

**Interview questions**

<details><summary>Q1. What's the difference between average-case and expected running time?</summary>

Average-case averages over a probability distribution of **inputs**, so it assumes the data is random, and a specific bad input can still be slow. Expected running time averages over the **algorithm's own random choices** for a fixed input, so it holds for every input, including adversarial ones.
</details>

<details><summary>Q2. Can randomized quicksort still take O(n²)?</summary>

Yes, in principle, if it repeatedly picks extreme pivots. But the probability is exponentially small, and the expected time is Θ(n log n) for every input. Introsort removes even that tiny risk with a heapsort fallback after 2⌊log₂ n⌋ levels.
</details>

<details><summary>Q3. How does Java's Arrays.sort avoid quicksort's worst case?</summary>

For primitives it uses dual-pivot quicksort with pivots chosen from 5 sampled elements, and (since JDK 14) it falls back to heapsort if the recursion gets too deep, giving O(n log n). For objects it uses TimSort, a merge sort, which has no quadratic case at all.
</details>

**Worked problem: why is a random pivot "good enough" half the time?** Rank the elements 1..n. A pivot of rank between n/4 and 3n/4 leaves both sides ≤ 3n/4. That range holds n/2 of the n ranks, so the probability is ½. Each good pivot multiplies the subproblem size by at most ¾, so after log_{4/3} n ≈ 2.41 log₂ n good pivots on any path, the size is 1. On average, half the pivots are good, so the expected depth is about 4.8 log₂ n, and the expected work is O(n log n).

**Coding problems**
- LeetCode 912 · Sort an Array (with random pivot; deterministic pivots time out on sorted tests) *(verify link)*
- LeetCode 215 · Kth Largest Element in an Array (randomized quickselect) *(verify link)*
- LeetCode 384 · Shuffle an Array (Fisher–Yates) *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. A random pivot means no input is bad: Θ(n log n) **expected on every input**.
2. Expected comparisons are 2(n + 1)Hₙ − 4n ≈ 1.39 n log₂ n.
3. The running time concentrates tightly around its mean (the measured spread over 200 runs was −11% to +25%).
4. Las Vegas: always correct, with a random running time.
5. Production sorts use introsort: quicksort + heapsort fallback + insertion sort.

**Formulas:** P(good pivot) = ½. Expected depth O(log n). Introsort depth limit 2⌊log₂ n⌋.

**Common mistakes:** confusing expected with average, an off-by-one in the random range, creating a new `Random` per call.

**Quiz**
1. Is randomized quicksort's output random?
2. What's the expected number of comparisons for n = 1000?
3. What does introsort switch to when the depth limit is hit?
4. Does randomisation fix the all-equal-keys problem for Lomuto?

**Answers**

<details><summary>Show answers</summary>

1. No. It's always the correctly sorted array. Only the running time is random.
2. About 10,986.
3. Heapsort, and insertion sort for small pieces.
4. No. Every element is ≤ the pivot whatever the pivot is. Use 3-way partitioning.
</details>

**Related topics:** [Quicksort basics](quicksort-basics.md) · [Partitioning](partitioning.md) · [Complexity analysis](complexity-analysis.md) · [Heapsort](../heapsort/heapsort.md) · [Quickselect](../order-statistics/quickselect.md) · [Randomized algorithms](../../13-Approximation-and-Randomization/randomized-algorithms.md)
