# Radix Sort

> **CLRS:** 8.3 · **Status:** ✅ Written · **Prerequisites:** [Counting sort](counting-sort.md), [Stability](../README.md#stability) · **Next:** [Bucket sort](bucket-sort.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

**Radix sort** sorts keys **digit by digit**. The common **LSD (least-significant-digit-first)** version sorts on the last digit, then the second-to-last, and so on, using a **stable** sort (usually [counting sort](counting-sort.md)) for each pass. After the pass on the most significant digit, the array is fully sorted.

**What problem does it solve?** Counting sort is linear only when the range k is small. Radix sort handles **large ranges** by splitting each key into d small digits: for example, a 32-bit integer is 4 bytes, each in [0, 255]. That gives **Θ(d(n + k))** time, linear when d is constant and k = O(n).

**Where it's used**
- Sorting large arrays of **fixed-width integers**: IDs, IP addresses, timestamps, 64-bit hashes. It's often 2–3× faster than comparison sorts for n ≥ 10⁵.
- **Strings** of bounded length: MSD radix sort, and suffix-array construction (prefix doubling).
- **GPU sorting** (CUB and Thrust use radix sort: it's highly parallel).
- **Historical:** punched-card sorting machines (Hollerith, 1890 US census) used LSD radix sort, one column per pass.

**Prerequisites:** counting sort and why stability matters.

---

## B. Intuition

**Analogy: sorting dates written as YYYY-MM-DD.** Stable-sort by **day** first, then stable-sort by **month**, then stable-sort by **year**. When you sort by year, ties (the same year) keep their month-and-day order from the earlier passes. So within a year, dates are already ordered by month, and within a month by day.

**Why least significant first?** Each later pass becomes the **primary** key, and stability preserves the earlier, less significant, ordering as the **tie-breaker**. Going most-significant-first with a single global stable sort per digit doesn't work: the less significant passes would scramble the order of the more significant digits.

**Why stability is non-negotiable:** if a pass reorders equal digits arbitrarily, it destroys the order produced by all the previous passes. The test program below shows exactly this failure with an unstable inner sort.

---

## C. How it works internally

**CLRS Figure 8.3:** seven 3-digit numbers.

```
 input      after sorting     after sorting     after sorting
            on digit 1        on digit 2        on digit 3
            (ones)            (tens)            (hundreds)
  329          720               720               329
  457          355               329               355
  657          436               436               436
  839          457               839               457
  436          657               355               657
  720          329               457               720
  355          839               657               839
```

**Look at pass 2 (tens):** 720 and 329 both have tens digit 2. They appear in the order **720, 329**, because that's their order after pass 1 (720 ends in 0, 329 ends in 9). Stability preserved it. In pass 3, 329 and 355 (both hundreds digit 3) stay in the order 329, 355, decided by pass 2, which is correct.

### Choosing the digit size (radix)

Write a b-bit key as d = ⌈b/r⌉ digits of r bits each, so each digit is in [0, 2ʳ − 1]. Each counting-sort pass is Θ(n + 2ʳ). The total is **Θ((b/r)(n + 2ʳ))** (CLRS Lemma 8.4).

| Digit size r | Passes for b = 32 | Count array size | Notes |
|---|---|---|---|
| 1 bit | 32 | 2 | far too many passes |
| 4 bits | 8 | 16 | |
| **8 bits (a byte)** | **4** | **256** | **the standard choice**: the count array fits in L1 cache |
| 16 bits | 2 | 65,536 | fewer passes, but the count array thrashes the cache for moderate n |
| 32 bits | 1 | 4 × 10⁹ | that's just counting sort, and impossible |

**Theory (CLRS):** choosing r ≈ lg n balances the terms, giving Θ(bn/lg n), so for b = O(lg n) it's linear. **Practice:** r = 8 or 11 bits, because of cache effects.

### Negative integers

In two's complement, negative numbers have the top bit set, so they'd sort **after** positives by unsigned bytes. Fix: flip the sign bit (`x ^ 0x80000000`) to map signed order onto unsigned order, sort, and flip back. Equivalently, treat the top byte's counting as signed.

### LSD vs MSD

| | LSD (least significant digit first) | MSD (most significant digit first) |
|---|---|---|
| Order | rightmost digit first | leftmost digit first, then recurse into each bucket |
| Needs a stable inner sort | yes | no (recursion separates the buckets) |
| Natural for | fixed-width integers | strings of varying length, lexicographic order |
| Can stop early | no, always all d passes | yes, when a bucket has 1 element or the strings differ early |

---

## D. Algorithm and pseudocode

CLRS 8.3:

```
RADIX-SORT(A, d)
1  for i ← 1 to d                       ▷ digit 1 is the lowest-order digit
2      use a stable sort to sort array A on digit i
```

That's the whole algorithm. The stable sort is COUNTING-SORT on the digit value.

### Correctness (CLRS Lemma 8.3, by induction on the number of passes)

> **Claim:** after pass i, the array is sorted with respect to the **last i digits** (the number formed by digits 1..i).

- **Base (i = 1):** a stable sort on digit 1 sorts by the last digit. ✓
- **Step:** assume the array is sorted by the last i − 1 digits, and run a stable sort on digit i. Take two elements x and y with x's last-i-digit number < y's.
  - If digitᵢ(x) < digitᵢ(y), the pass puts x before y. ✓
  - If digitᵢ(x) = digitᵢ(y), then x's last i − 1 digits are smaller. By the hypothesis x was already before y, and **stability keeps it that way**. ✓
- **Termination:** after d passes the array is sorted by all d digits, so it's fully sorted. ∎

The proof uses stability exactly once, in the tie case. Without it the argument (and the algorithm) fails.

---

## E. Implementation

```java
import java.util.*;

/** LSD radix sort: base-256 for signed ints, digit-wise for fixed-length strings, and the stability requirement. */
public class RadixSort {

    /** LSD radix sort on 32-bit signed ints, one byte per pass (4 passes of counting sort). */
    static void sort(int[] a) {
        int n = a.length;
        int[] src = a, dst = new int[n];
        for (int shift = 0; shift < 32; shift += 8) {
            int[] count = new int[257];
            for (int x : src) count[digit(x, shift) + 1]++;                     // histogram (offset by 1)
            for (int v = 0; v < 256; v++) count[v + 1] += count[v];              // count[v] = start index of digit v
            for (int x : src) dst[count[digit(x, shift)]++] = x;                 // stable: forward scan into start slots
            int[] t = src; src = dst; dst = t;                                   // ping-pong buffers
        }
        // after 4 passes (an even number) the result is back in the original array 'a'
    }

    /** Byte of x at the given shift; the top byte's sign bit is flipped so negatives sort first. */
    static int digit(int x, int shift) {
        int d = (x >>> shift) & 0xFF;
        return shift == 24 ? d ^ 0x80 : d;
    }

    /** CLRS-style: sort decimal numbers with d digits, using a stable counting sort per digit. */
    static int[] sortDecimal(int[] a, int d, boolean stableInnerSort) {
        int[] cur = a.clone();
        for (int i = 0, pow = 1; i < d; i++, pow *= 10) {
            final int p = pow;
            int[] count = new int[10];
            for (int x : cur) count[(x / p) % 10]++;
            for (int v = 1; v < 10; v++) count[v] += count[v - 1];
            int[] out = new int[cur.length];
            if (stableInnerSort)
                for (int j = cur.length - 1; j >= 0; j--) out[--count[(cur[j] / p) % 10]] = cur[j];   // backwards: stable
            else
                for (int j = 0; j < cur.length; j++) out[--count[(cur[j] / p) % 10]] = cur[j];        // forwards: UNSTABLE
            cur = out;
        }
        return cur;
    }

    /** LSD radix sort for strings of equal length w over 8-bit chars (e.g. plate numbers, fixed IDs). */
    static void sortStrings(String[] a, int w) {
        int n = a.length;
        String[] aux = new String[n];
        for (int pos = w - 1; pos >= 0; pos--) {
            int[] count = new int[257];
            for (String s : a) count[s.charAt(pos) + 1]++;
            for (int r = 0; r < 256; r++) count[r + 1] += count[r];
            for (String s : a) aux[count[s.charAt(pos)]++] = s;
            System.arraycopy(aux, 0, a, 0, n);
        }
    }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        int[] clrs = {329, 457, 657, 839, 436, 720, 355};                        // CLRS Figure 8.3
        int[] r = sortDecimal(clrs, 3, true);
        System.out.println("CLRS example, stable digits:   " + Arrays.toString(r));
        check(Arrays.equals(r, new int[]{329, 355, 436, 457, 657, 720, 839}), "matches CLRS Figure 8.3");

        int[] bad = sortDecimal(clrs, 3, false);
        System.out.println("same, UNSTABLE inner sort:     " + Arrays.toString(bad));
        int[] exp = clrs.clone(); Arrays.sort(exp);
        check(!Arrays.equals(bad, exp), "an unstable per-digit sort breaks radix sort");

        Random rnd = new Random(10);
        boolean ok = true;
        for (int t = 0; t < 2000; t++) {
            int[] x = rnd.ints(rnd.nextInt(500)).toArray();                     // full 32-bit range incl. negatives
            int[] e = x.clone(); Arrays.sort(e);
            sort(x);
            if (!Arrays.equals(x, e)) ok = false;
        }
        int[] edge = {Integer.MIN_VALUE, -1, 0, 1, Integer.MAX_VALUE, -256, 255, 256, Integer.MIN_VALUE};
        int[] ee = edge.clone(); Arrays.sort(ee); sort(edge);
        check(ok && Arrays.equals(edge, ee), "2000 random full-range int arrays + extremes (MIN_VALUE, -1, MAX_VALUE) match Arrays.sort");

        String[] plates = {"KA05MN", "DL01AB", "KA01ZZ", "MH12AA", "DL01AA", "KA05AB"};
        String[] expS = plates.clone(); Arrays.sort(expS);
        sortStrings(plates, 6);
        System.out.println("plates: " + Arrays.toString(plates));
        check(Arrays.equals(plates, expS), "fixed-length strings sorted lexicographically");

        int n = 2_000_000;
        int[] big = rnd.ints(n).toArray(), bigExp = big.clone();
        sort(big); Arrays.sort(bigExp);
        check(Arrays.equals(big, bigExp), "2,000,000 random ints sorted with 4 passes of counting sort");
    }
}
```

**Output:**

```
CLRS example, stable digits:   [329, 355, 436, 457, 657, 720, 839]
ok   matches CLRS Figure 8.3
same, UNSTABLE inner sort:     [355, 329, 457, 436, 657, 720, 839]
ok   an unstable per-digit sort breaks radix sort
ok   2000 random full-range int arrays + extremes (MIN_VALUE, -1, MAX_VALUE) match Arrays.sort
plates: [DL01AA, DL01AB, KA01ZZ, KA05AB, KA05MN, MH12AA]
ok   fixed-length strings sorted lexicographically
ok   2,000,000 random ints sorted with 4 passes of counting sort
```

**The failure, explained:** with a forward (unstable) final pass, 329 and 355, which share the hundreds digit 3, come out as **355, 329**. The last pass reversed the order that the tens-digit pass had established. Every pass's work was undone by the next.

**Java-specific details**
- `x >>> shift` is an **unsigned** shift, required so that negative numbers' bytes extract correctly. (`>>` would smear the sign bit.)
- **Ping-pong buffers:** alternate between `src` and `dst` instead of copying back after each pass. Four passes is even, so the result ends in `a`.
- The `count[digit + 1]` offset turns the prefix sum into **start indices**, so a forward scan is stable (the classic Sedgewick formulation). This is equivalent to CLRS's end indices with a backward scan.

**Common mistakes**
1. An unstable inner sort, as shown above.
2. Processing the most significant digit first in an LSD loop.
3. Ignoring negatives (MIN_VALUE ends up last), or using `>>` instead of `>>>`.
4. Using radix sort for small n, where the setup overhead (4 count arrays plus a buffer) loses to insertion sort or `Arrays.sort`.

---

## F. Time complexity

**CLRS Lemma 8.3:** for n d-digit numbers whose digits are in [0, k − 1], RADIX-SORT with a Θ(n + k) stable sort takes **Θ(d(n + k))**.

Derivation: d passes × Θ(n + k) per pass.

**CLRS Lemma 8.4 (choosing r):** b-bit keys, r-bit digits: **Θ((b/r)(n + 2ʳ))**.
- If b < lg n: choose r = b, giving Θ(n) (one pass of counting sort).
- If b ≥ lg n: choose r = lg n, giving **Θ(bn/lg n)**.

**Examples**

| Keys | n | d (passes) | k | Work |
|---|---|---|---|---|
| 32-bit ints, bytes | 10⁶ | 4 | 256 | 4(10⁶ + 256) ≈ 4 × 10⁶ |
| 64-bit longs, bytes | 10⁶ | 8 | 256 | ≈ 8 × 10⁶ |
| Comparison sort, for contrast | 10⁶ | — | — | n log₂ n ≈ 2 × 10⁷ comparisons |
| 6-char plates | 10⁶ | 6 | 256 | ≈ 6 × 10⁶ |

**Best = average = worst:** radix sort does the same work regardless of the input order.

**Is it really "linear"?** Only if d is constant. To sort n **distinct** numbers you need keys of at least log₂ n bits, so d ≥ log n / r. With r fixed that gives Θ(n log n) in the most general setting. Radix sort's advantage is in constants and in using fixed machine words, not in beating log n for arbitrary key sizes.

## G. Space complexity

- LSD radix sort: an **output buffer of Θ(n)** plus a **count array of Θ(k)**, so **Θ(n + k)** auxiliary. **Not in place.**
- MSD radix sort: Θ(n + k·depth) with an auxiliary array. In-place MSD variants ("American flag sort") exist but aren't stable.

## H. Complexity summary

| Algorithm | Best time | Average time | Worst time | Auxiliary space | Stable? | In-place? |
|---|---|---|---|---|---|---|
| LSD radix sort | Θ(d(n + k)) | Θ(d(n + k)) | Θ(d(n + k)) | Θ(n + k) | ✅ Yes | ❌ No |
| MSD radix sort | Ω(n) (can stop early) | Θ(d(n + k)) | Θ(d(n + k)) | Θ(n + dk) | ✅ (with an auxiliary array) | ❌ |

Comparison-based ❌ · requires fixed-length digit keys

## I. Advantages, limitations, and comparisons

**Advantages:** linear in n for fixed-width keys, stable, predictable, very fast in practice for large integer arrays, and parallelisable (it's the GPU sort of choice).

**Limitations:** it needs keys with digits (integers or fixed-length strings), not general comparators. It uses Θ(n) extra memory, has overhead for small n, and has poor cache behaviour when the radix is large. Floating-point keys need a bit-twiddling transform (flip the bits of negatives).

| vs | Radix wins | The other wins |
|---|---|---|
| Quicksort or `Arrays.sort` | large n, fixed-width integer keys | general objects, small n, memory-constrained settings |
| [Counting sort](counting-sort.md) | large key range (32/64-bit) | tiny range (k ≤ n): one pass is enough |
| Merge sort (stable) | integer keys | arbitrary comparators |

**Interview follow-ups:** "Sort a million 32-bit integers as fast as possible", "Why must the digit sort be stable?", "Maximum Gap (LeetCode 164)", "Sort strings of equal length".

---

## J. Practice

**Beginner**
1. Trace radix sort on COW, DOG, SEA, RUG, ROW, MOB, BOX, TAB, BAR, EAR, TAR, DIG, BIG, TEA, NOW, FOX (CLRS Exercise 8.3-1).
2. Which of insertion sort, merge sort, heapsort and quicksort are stable, and so usable inside radix sort? (CLRS Exercise 8.3-2)
3. How many passes does radix sort need for 64-bit keys with 16-bit digits?

**Intermediate**
4. Prove the correctness lemma using induction (CLRS Exercise 8.3-3).
5. Sort n integers in [0, n³ − 1] in O(n) time (CLRS Exercise 8.3-4). *(Hint: treat them as 3-digit base-n numbers.)*
6. Implement radix sort for signed `long` values.

**Advanced**
7. Implement MSD radix sort for variable-length strings (with an end-of-string marker).
8. Maximum Gap (LeetCode 164): find the largest gap between consecutive sorted values in O(n) time, using radix sort or the pigeonhole bucket method.

**Interview questions**

<details><summary>Q1. Why does LSD radix sort need a stable sort per digit?</summary>

After sorting by digit i, elements with equal digit i must stay in the order produced by the less significant digits, because that order is the correct tie-breaker. An unstable pass would scramble it. The correctness proof uses stability exactly in the case of equal digits.
</details>

<details><summary>Q2. What is radix sort's time complexity, and is it really linear?</summary>

Θ(d(n + k)) for d digits of range k. With fixed-width keys (for example 32-bit ints, d = 4 bytes, k = 256) that's linear in n. In general, distinct keys need about log n bits, so d grows with log n. It's "linear" only for bounded key width.
</details>

<details><summary>Q3. How do you radix-sort negative integers?</summary>

Flip the sign bit of the most significant byte (or of the whole key), which maps two's-complement signed order onto unsigned order. Sort as unsigned, using unsigned shifts (`>>>` in Java) to extract digits.
</details>

**Worked problem: sort n integers in [0, n² − 1] in O(n).** Write each number in base n, which gives 2 digits, each in [0, n − 1]. Two passes of counting sort with k = n take Θ(2(n + n)) = Θ(n). A single counting sort would need k = n², so Θ(n²).

**Coding problems**
- LeetCode 164 · Maximum Gap *(verify link)*
- LeetCode 912 · Sort an Array (radix sort solution) *(verify link)*
- LeetCode 2343 · Query Kth Smallest Trimmed Number *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. Sort digit by digit from the least significant end, with a **stable** sort each time.
2. Θ(d(n + k)) time and Θ(n + k) space. Linear for fixed-width keys.
3. Stability is what makes the induction (and the algorithm) work.
4. Practical choice: 8-bit digits (4 passes for int, 8 for long). Flip the sign bit for negatives.
5. LSD suits fixed-width integers. MSD suits variable-length strings.

**Formulas:** Θ(d(n + k)). With b-bit keys and r-bit digits: Θ((b/r)(n + 2ʳ)), with r ≈ lg n in theory.

**Common mistakes:** an unstable inner sort, the wrong digit order, using `>>` instead of `>>>`, forgetting negatives.

**Quiz**
1. How many counting-sort passes for 32-bit ints with 8-bit digits?
2. In LSD radix sort, which digit is processed last?
3. Is radix sort comparison-based?
4. Time to sort n numbers in [0, n² − 1] with base-n radix sort?

**Answers**

<details><summary>Show answers</summary>

1. 4.
2. The most significant digit.
3. No.
4. Θ(n): 2 passes with k = n.
</details>

**Related topics:** [Counting sort](counting-sort.md) · [Bucket sort](bucket-sort.md) · [Lower bounds](lower-bounds.md) · [String matching](../../10-String-Matching/naive-string-matching.md) (string keys) · [Stability](../README.md#stability)
