# Bucket Sort

> **CLRS:** 8.4 · **Status:** ✅ Written · **Prerequisites:** [Insertion sort](../comparison-sorts/insertion-sort.md), [Counting sort](counting-sort.md), [Random variables](../../14-Mathematical-Foundations/random-variables.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

**Bucket sort** splits the range of the input into **n equal-sized intervals ("buckets")**, distributes the elements into them by value, sorts each (small) bucket, usually with insertion sort, and concatenates the buckets in order.

**What problem does it solve?** It sorts **real numbers** (not just integers) in **Θ(n) expected time**, provided the input is **drawn uniformly at random** from a known range. Counting and radix sort need integer keys. Bucket sort works on floating-point values.

**Where it's used**
- Sorting **uniformly distributed floats**: random samples, hashed values, normalised scores in [0, 1).
- **Histogramming and binning** in data analysis, as the first step of approximate quantiles.
- The **"Maximum Gap"** trick (pigeonhole buckets) and **top-k frequent** elements (buckets indexed by frequency).
- External sorting and parallel sorting (sample sort is a bucket sort with buckets chosen by sampling).

**Prerequisites:** insertion sort, linked lists or dynamic arrays, and expected value (for the proof).

---

## B. Intuition

**Analogy: a post office with 10 pigeonholes labelled by the first digit of the PIN code.** Letters arrive in random order. Drop each letter into its pigeonhole (one step per letter, no comparisons), sort the few letters inside each hole by hand, then collect the holes in order.

**Why it's fast on uniform data:** with n buckets and n uniformly random values, each bucket gets **about 1 element on average**. Sorting a handful of elements takes constant time, so the total is about n.

**Why it can be slow:** if the data **isn't** uniform, for example if everything falls between 0.50 and 0.51, then all n elements land in **one bucket**, and insertion sort on it costs Θ(n²). Bucket sort trusts its assumption about the distribution.

---

## C. How it works internally

**CLRS Figure 8.4:** A = [.78, .17, .39, .26, .72, .94, .21, .12, .23, .68], n = 10, bucket index = ⌊10 · x⌋.

| Bucket | Range | Elements (in arrival order) | After insertion sort |
|---|---|---|---|
| 0 | [0, .1) | — | — |
| 1 | [.1, .2) | .17, .12 | .12, .17 |
| 2 | [.2, .3) | .26, .21, .23 | .21, .23, .26 |
| 3 | [.3, .4) | .39 | .39 |
| 4 | [.4, .5) | — | — |
| 5 | [.5, .6) | — | — |
| 6 | [.6, .7) | .68 | .68 |
| 7 | [.7, .8) | .78, .72 | .72, .78 |
| 8 | [.8, .9) | — | — |
| 9 | [.9, 1) | .94 | .94 |

**Concatenate:** [.12, .17, .21, .23, .26, .39, .68, .72, .78, .94]

**Why concatenation gives a sorted array:** if x < y and they're in different buckets, then ⌊nx⌋ < ⌊ny⌋, so x's bucket comes first. If they're in the same bucket, the per-bucket sort orders them.

**Edge cases**

| Situation | Handling |
|---|---|
| Values in [a, b) rather than [0, 1) | index = ⌊n · (x − a)/(b − a)⌋ |
| x = b exactly (the maximum value) | clamp the index to n − 1 |
| Skewed or clustered data | degrades toward Θ(n²). Use more adaptive bucket boundaries (sample sort) or a comparison sort. |
| Integer keys | it works too, and with buckets of width 1 it becomes counting sort |
| NaN / infinities | must be handled separately |

---

## D. Algorithm and pseudocode

CLRS 8.4 (input values in [0, 1)):

```
BUCKET-SORT(A)
1  n ← length[A]
2  for i ← 1 to n
3      insert A[i] into list B[⌊n · A[i]⌋]
4  for i ← 0 to n − 1
5      sort list B[i] with insertion sort
6  concatenate the lists B[0], B[1], …, B[n − 1] together in order
```

**Correctness:** shown in Section C (different buckets are ordered by index, and each bucket is sorted internally).
**Stability:** appending to buckets in input order and using a stable per-bucket sort (insertion sort) makes bucket sort **stable**.

---

## E. Implementation

```java
import java.util.*;

/** Bucket sort (CLRS 8.4) for doubles in [0,1) and [lo,hi), with the sum of n_i^2 measured on uniform vs skewed data. */
public class BucketSort {

    static long insertionWork;                      // total element moves inside buckets, ~ sum of n_i^2 / 4

    /** Sorts values in [0, 1). Theta(n) expected for uniform input. */
    static void sort(double[] a) { sort(a, 0.0, 1.0); }

    /** Sorts values in [lo, hi). Each bucket covers (hi - lo) / n. */
    static void sort(double[] a, double lo, double hi) {
        int n = a.length;
        if (n < 2) return;
        List<List<Double>> buckets = new ArrayList<>(n);
        for (int i = 0; i < n; i++) buckets.add(new ArrayList<>());
        for (double x : a) {
            int idx = (int) ((x - lo) / (hi - lo) * n);
            if (idx >= n) idx = n - 1;                 // x == hi (or rounding): clamp to the last bucket
            if (idx < 0) idx = 0;
            buckets.get(idx).add(x);
        }
        int k = 0;
        for (List<Double> b : buckets) {
            insertionSort(b);
            for (double x : b) a[k++] = x;             // concatenate
        }
    }

    static void insertionSort(List<Double> b) {
        for (int j = 1; j < b.size(); j++) {
            double key = b.get(j); int i = j - 1;
            while (i >= 0 && b.get(i) > key) { b.set(i + 1, b.get(i)); i--; insertionWork++; }
            b.set(i + 1, key);
        }
    }

    /** Sum of squared bucket sizes: the quantity the expected-time proof bounds by 2n - 1. */
    static long sumSquares(double[] a) {
        int n = a.length; int[] size = new int[n];
        for (double x : a) size[Math.min(n - 1, (int) (x * n))]++;
        long s = 0; for (int c : size) s += (long) c * c;
        return s;
    }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        double[] a = {.78, .17, .39, .26, .72, .94, .21, .12, .23, .68};          // CLRS Figure 8.4
        sort(a);
        System.out.println("sorted: " + Arrays.toString(a));
        check(Arrays.equals(a, new double[]{.12, .17, .21, .23, .26, .39, .68, .72, .78, .94}), "matches CLRS Figure 8.4");

        Random rnd = new Random(13);
        boolean ok = true;
        for (int t = 0; t < 2000; t++) {
            double[] x = rnd.doubles(rnd.nextInt(400)).toArray();
            double[] e = x.clone(); Arrays.sort(e);
            sort(x);
            if (!Arrays.equals(x, e)) ok = false;
            double[] y = rnd.doubles(rnd.nextInt(200), -50, 50).toArray();
            double[] ey = y.clone(); Arrays.sort(ey);
            sort(y, -50, 50);
            if (!Arrays.equals(y, ey)) ok = false;
        }
        check(ok, "2000 random arrays in [0,1) and [-50,50) match Arrays.sort");

        // Expected-time proof: E[sum n_i^2] = 2n - 1 for uniform input
        int n = 100_000, trials = 20;
        double avg = 0;
        for (int t = 0; t < trials; t++) avg += sumSquares(rnd.doubles(n).toArray());
        avg /= trials;
        System.out.printf("uniform, n=%,d: average sum of n_i^2 = %,.0f   (theory 2n - 1 = %,d)%n", n, avg, 2 * n - 1);
        check(Math.abs(avg - (2 * n - 1)) / (2.0 * n) < 0.01, "E[sum n_i^2] = 2n - 1 confirmed (within 1%)");

        double[] uniform = rnd.doubles(20_000).toArray();
        double[] skewed = new double[20_000];
        for (int i = 0; i < skewed.length; i++) skewed[i] = 0.5 + rnd.nextDouble() * 0.00004;  // width 0.8/n: all in ONE bucket
        insertionWork = 0; sort(uniform);  long wu = insertionWork;
        insertionWork = 0; sort(skewed);   long ws = insertionWork;
        System.out.printf("n=20,000 insertion moves: uniform %,d  vs  skewed %,d (about n^2/4 = %,d)%n", wu, ws, 20_000L * 20_000 / 4);
        check(wu < 20_000 && ws > 50_000_000L, "uniform input is linear; skewed input collapses into one bucket and goes quadratic");
    }
}
```

**Output:**

```
sorted: [0.12, 0.17, 0.21, 0.23, 0.26, 0.39, 0.68, 0.72, 0.78, 0.94]
ok   matches CLRS Figure 8.4
ok   2000 random arrays in [0,1) and [-50,50) match Arrays.sort
uniform, n=100,000: average sum of n_i^2 = 200,024   (theory 2n - 1 = 199,999)
ok   E[sum n_i^2] = 2n - 1 confirmed (within 1%)
n=20,000 insertion moves: uniform 5,022  vs  skewed 100,338,117 (about n^2/4 = 100,000,000)
ok   uniform input is linear; skewed input collapses into one bucket and goes quadratic
```

**What the output shows**
- The measured Σ nᵢ² averages about **200,000**, matching the theoretical **2n − 1 = 199,999** to within a fraction of a percent. This is the heart of the expected-time proof (Section F).
- On uniform input, insertion sort inside the buckets did only about **5,000 moves** for 20,000 elements. On skewed input, where every value lies within one bucket's width, it did about **10⁸ moves** (n²/4, the expected inversions of one big bucket): roughly a 20,000× slowdown on the same n.

**Java-specific details**
- `List<List<Double>>` boxes every value (`Double` objects). For performance, use primitive arrays: count the bucket sizes first, then place into one `double[]` with offsets, which is counting-sort style.
- `(int) (x * n)` truncates toward zero, which is correct for x ≥ 0. Clamp for x = hi and for floating-point rounding.
- `Arrays.equals(double[], double[])` compares with `Double.equals` semantics (it distinguishes −0.0 from 0.0, and NaN equals NaN).

**Common mistakes:** not clamping the index for the maximum value, using too few buckets (n/10 buckets means about 10 per bucket, which is still fine), applying it to non-uniform data, and expecting a Θ(n) worst case.

---

## F. Time complexity

Let nᵢ be the number of elements in bucket i. Distribution and concatenation take Θ(n). Insertion sort on bucket i costs O(nᵢ²). So:

> T(n) = Θ(n) + Σᵢ₌₀ⁿ⁻¹ O(nᵢ²)

**Expected time (CLRS §8.4).** Take expectations over the random input:

> E[T(n)] = Θ(n) + Σᵢ O(E[nᵢ²])

**Claim: E[nᵢ²] = 2 − 1/n.** Define indicator variables Xᵢⱼ = I{A[j] falls in bucket i}, so nᵢ = Σⱼ Xᵢⱼ. Each element lands in bucket i with probability 1/n, independently. Then:

E[nᵢ²] = E[(Σⱼ Xᵢⱼ)²] = Σⱼ E[Xᵢⱼ²] + Σⱼ≠ₖ E[Xᵢⱼ Xᵢₖ]
= n · (1/n) + n(n − 1) · (1/n²)       (Xᵢⱼ² = Xᵢⱼ, and independence)
= 1 + (n − 1)/n = **2 − 1/n**

Therefore E[Σ nᵢ²] = n(2 − 1/n) = **2n − 1** (confirmed empirically above), and:

> **E[T(n)] = Θ(n) + n · O(2 − 1/n) = Θ(n)**

**Worst case:** all elements in one bucket, so Σ nᵢ² = n², and the cost is **Θ(n²)** with insertion sort. (Using a Θ(n log n) sort inside each bucket bounds the worst case at Θ(n log n).)

**Weaker condition (CLRS):** linear expected time holds whenever the **sum of the squared bucket sizes is linear** in expectation. Uniformity is sufficient, not necessary.

| Case | Time |
|---|---|
| Best (one element per bucket) | Θ(n) |
| Expected (uniform input) | **Θ(n)** |
| Worst (everything in one bucket) | Θ(n²) (Θ(n log n) with merge sort inside the buckets) |

## G. Space complexity

**Θ(n)** auxiliary: n bucket headers plus n element slots (and, in Java, the boxing overhead). **Not in place.**

## H. Complexity summary

| Algorithm | Best time | Average time | Worst time | Auxiliary space | Stable? | In-place? |
|---|---|---|---|---|---|---|
| Bucket sort | Θ(n) | Θ(n) expected (uniform input) | Θ(n²) | Θ(n) | ✅ Yes (stable bucket sort) | ❌ No |

Comparison-based: partly (distribution by arithmetic, then comparisons within buckets)

## I. Advantages, limitations, and comparisons

**Advantages:** linear expected time for floats, stable, simple, and parallelisable (the buckets are independent).

**Limitations:** it depends on the distribution, which makes it vulnerable to skew and to adversarial input. It needs Θ(n) memory and must know the range.

| Linear-time sort | Keys | Assumption | Guarantee |
|---|---|---|---|
| [Counting](counting-sort.md) | integers in [0, k] | k = O(n) | worst case Θ(n + k) |
| [Radix](radix-sort.md) | fixed-width digits | d constant | worst case Θ(d(n + k)) |
| **Bucket** | reals in a known range | **uniform distribution** | **expected** Θ(n) |

**Related ideas:** *sample sort* chooses bucket boundaries from a random sample, so the buckets stay balanced for any distribution. It's widely used in parallel and distributed sorting. *Pigeonhole sort* is bucket sort with width-1 buckets on integers.

**Interview follow-ups:** "Sort a million random floats in [0, 1)", "Maximum Gap in O(n)" (n − 1 buckets: the maximum gap can't lie inside a bucket), "Top K frequent elements" (buckets indexed by frequency, O(n)), "What happens with non-uniform data?"

---

## J. Practice

**Beginner**
1. Trace BUCKET-SORT on [.79, .13, .16, .64, .39, .20, .89, .53, .71, .42] (CLRS Exercise 8.4-1).
2. What's the worst-case running time, and how could you reduce it to O(n log n) while keeping linear expected time? (CLRS Exercise 8.4-2)
3. Adapt bucket sort to integers in [0, 999] using 10 buckets.

**Intermediate**
4. Points uniformly distributed in the unit disc: design buckets for sorting by distance from the origin (CLRS Exercise 8.4-4).
5. Top K Frequent Elements in O(n) using buckets indexed by count (LeetCode 347).
6. Show E[nᵢ²] = 2 − 1/n step by step.

**Advanced**
7. Maximum Gap (LeetCode 164) in O(n) using n − 1 buckets with min and max per bucket.
8. Given a known non-uniform distribution with CDF P, sort in expected linear time (CLRS Exercise 8.4-5). *(Hint: bucket by P(x), which is uniform.)*

**Interview questions**

<details><summary>Q1. When does bucket sort run in linear time?</summary>

When the input is spread evenly over the buckets, typically uniformly distributed over a known range. Then each bucket holds O(1) elements in expectation (E[Σ nᵢ²] = 2n − 1), so the per-bucket sorts cost O(n) in total.
</details>

<details><summary>Q2. What's bucket sort's worst case, and when does it happen?</summary>

Θ(n²) with insertion sort, when all (or most) elements fall into the same bucket because the data is clustered or skewed. Using merge sort inside each bucket caps it at Θ(n log n).
</details>

<details><summary>Q3. How does Maximum Gap use buckets?</summary>

With n numbers spanning [min, max], the maximum gap is at least (max − min)/(n − 1). Make n − 1 buckets of that width and record only each bucket's min and max. A gap of maximal size can't fall inside a bucket, so the answer is the largest of (next non-empty bucket's min − previous non-empty bucket's max). That's O(n).
</details>

**Worked problem: Top K Frequent (LeetCode 347) in O(n).** Count frequencies with a HashMap. Create buckets[0..n], where bucket[f] lists the elements with frequency f. Walk the buckets from f = n downwards, collecting elements until you have k. That's Θ(n) time (no sorting), compared with Θ(n log k) using a heap.

**Coding problems**
- LeetCode 164 · Maximum Gap *(verify link)*
- LeetCode 347 · Top K Frequent Elements *(verify link)*
- LeetCode 451 · Sort Characters By Frequency *(verify link)*
- LeetCode 220 · Contains Duplicate III (bucketing by value window) *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. Distribute into n range buckets, sort each one (insertion sort), and concatenate.
2. Θ(n) **expected** for uniformly distributed input, because E[Σ nᵢ²] = 2n − 1.
3. Θ(n²) worst case when everything lands in one bucket.
4. Stable if the bucket appends and the per-bucket sort are stable. Θ(n) extra space.
5. It's an assumption about the distribution. Sample sort makes it robust.

**Formulas:** bucket index = ⌊n(x − lo)/(hi − lo)⌋. E[nᵢ²] = 2 − 1/n. E[T] = Θ(n).

**Common mistakes:** not clamping the index, applying it to skewed data, claiming a Θ(n) worst case.

**Quiz**
1. Expected elements per bucket with n buckets and n uniform values?
2. E[Σ nᵢ²] for n = 1000?
3. What happens if all values are in [0.5, 0.501)?
4. Is bucket sort comparison-based?

**Answers**

<details><summary>Show answers</summary>

1. 1.
2. 2n − 1 = 1999.
3. They all go into one bucket, so insertion sort on n elements costs Θ(n²).
4. Partly. Distribution uses arithmetic on the values (not comparisons), but sorting within each bucket compares.
</details>

**Related topics:** [Counting sort](counting-sort.md) · [Radix sort](radix-sort.md) · [Insertion sort](../comparison-sorts/insertion-sort.md) · [Lower bounds](lower-bounds.md) · [Random variables](../../14-Mathematical-Foundations/random-variables.md) · [Distributions guide (Uniform)](../../../distribution/uniform.md)
