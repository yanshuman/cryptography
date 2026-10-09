# Counting Sort

> **CLRS:** 8.2 · **Status:** ✅ Written · **Prerequisites:** [Lower bounds for sorting](lower-bounds.md), [Stability](../README.md#stability) · **Next:** [Radix sort](radix-sort.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

**Counting sort** sorts n **integers in a known small range [0, k]** by **counting** how many times each value occurs, then using those counts to place every element directly into its final position. It never compares two elements.

**What problem does it solve?** It sorts in **Θ(n + k)** time. When k = O(n) that's **linear**, beating the [Ω(n log n) comparison lower bound](lower-bounds.md), which doesn't apply because no comparisons are made.

**Where it's used**
- As the **stable inner sort of [radix sort](radix-sort.md)**: one counting-sort pass per digit.
- Sorting by small integer fields: ages (0–150), exam marks (0–100), grades, months, byte values (0–255), DNA bases (A, C, G, T).
- **Histogram problems:** "sort by frequency", "the k most frequent elements" (bucket by count), "relative sort array".
- Image processing (histogram equalisation), suffix-array construction (rank sorting), and database GROUP BY on low-cardinality columns.

**Prerequisites:** arrays, prefix sums, and the idea of stability.

---

## B. Intuition

**Analogy: exam papers marked 0–10.** Instead of comparing papers, make 11 labelled piles and drop each paper on the pile for its mark. Then pick up the piles in order 0, 1, …, 10. You never compare two papers. You just look at each paper's mark once.

**Why prefix sums?** Knowing that there are, say, 2 papers with mark 0 and 3 with mark 1 tells you that the **mark-1 papers go in positions 2, 3 and 4**. The running total ("how many are ≤ v") is exactly the **last position** for value v. This lets us place satellite data (whole records, not just numbers) directly into an output array.

**Why go backwards in the final pass?** To keep it **stable**. Scanning the input from the end and filling each value's slots from right to left means the *last* occurrence of a value gets the *last* slot. Equal keys therefore keep their original order.

---

## C. How it works internally

**CLRS Figure 8.2:** A = [2, 5, 3, 0, 2, 3, 0, 3], with k = 5.

**Step 1: count the occurrences.** C[v] = the number of elements equal to v.

| v | 0 | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|---|
| C[v] | 2 | 0 | 2 | 3 | 0 | 1 |

**Step 2: prefix sums.** C[v] = the number of elements ≤ v, which is the **last output position** (1-indexed) for value v.

| v | 0 | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|---|
| C[v] | 2 | 2 | 4 | 7 | 7 | 8 |

**Step 3: place the elements, scanning A from right to left.**

| j | A[j] | C[A[j]] → output position | Output B (1-indexed) | C after decrement |
|---|---|---|---|---|
| 8 | 3 | 7 | B[7] = 3 | C[3] = 6 |
| 7 | 0 | 2 | B[2] = 0 | C[0] = 1 |
| 6 | 3 | 6 | B[6] = 3 | C[3] = 5 |
| 5 | 2 | 4 | B[4] = 2 | C[2] = 3 |
| 4 | 0 | 1 | B[1] = 0 | C[0] = 0 |
| 3 | 3 | 5 | B[5] = 3 | C[3] = 4 |
| 2 | 5 | 8 | B[8] = 5 | C[5] = 7 |
| 1 | 2 | 3 | B[3] = 2 | C[2] = 2 |

> B = [0, 0, 2, 2, 3, 3, 3, 5]

**Stability check:** the three 3s were at input positions 3, 6 and 8. They're placed at 5, 6 and 7, in the **same** relative order. ✓

**Edge cases**

| Situation | Handling |
|---|---|
| Negative numbers | Shift by the minimum: use index (x − min), where k = max − min |
| Unknown range | Scan once to find min and max (Θ(n)) |
| Huge range (k ≫ n), e.g. values up to 10⁹ | **Don't use it.** Θ(k) time and memory (a 4 GB count array). Use radix sort or a comparison sort. |
| Empty input | return immediately |
| Non-integer keys | Map them to integers first (char codes, enum ordinals), or use a comparison sort |

---

## D. Algorithm and pseudocode

CLRS 8.2 (A is the input, B the output, and keys are in 0..k):

```
COUNTING-SORT(A, B, k)
1   for i ← 0 to k
2       C[i] ← 0
3   for j ← 1 to length[A]
4       C[A[j]] ← C[A[j]] + 1           ▷ C[i] = number of elements equal to i
5   for i ← 1 to k
6       C[i] ← C[i] + C[i − 1]          ▷ C[i] = number of elements ≤ i
7   for j ← length[A] downto 1          ▷ backwards → stable
8       B[C[A[j]]] ← A[j]
9       C[A[j]] ← C[A[j]] − 1
```

| Lines | Cost | Purpose |
|---|---|---|
| 1–2 | Θ(k) | clear the counts |
| 3–4 | Θ(n) | histogram |
| 5–6 | Θ(k) | prefix sums give the final positions |
| 7–9 | Θ(n) | place each element at its position, then move that value's slot left |

### Correctness

**Invariant for lines 7–9:** before processing A[j], for each value v, C[v] = (the number of elements ≤ v) − (the number of elements equal to v already placed, which are the ones from A[j + 1..n]).

So C[v] is the last unfilled slot in v's block [first(v), last(v)]. Placing A[j] there and decrementing C[v] keeps the invariant. Every element lands in its own value's block, so B is sorted. Elements of value v are placed from right to left as j decreases, so the **later** input occurrences take the **later** slots, which means the sort is **stable**. ✓

### The simpler "histogram" version (keys only, not stable or record-preserving)

```
for v ← 0 to k:  write v into the output C[v] times
```

It's fine for plain integers, but it can't carry satellite data (records), and the stability question disappears because equal integers are indistinguishable.

---

## E. Implementation

```java
import java.util.*;

/** Counting sort (CLRS 8.2): stable version for records, histogram version, negative-key handling, and checks. */
public class CountingSort {

    /** Stable counting sort of ints in [0, k]. Returns a new sorted array. Theta(n + k). */
    static int[] sort(int[] a, int k) {
        int[] count = new int[k + 1];
        for (int x : a) count[x]++;                                   // histogram
        for (int v = 1; v <= k; v++) count[v] += count[v - 1];       // count[v] = #elements <= v
        int[] out = new int[a.length];
        for (int j = a.length - 1; j >= 0; j--)                       // backwards -> stable
            out[--count[a[j]]] = a[j];                                // 0-indexed: decrement first
        return out;
    }

    /** Handles any int range by shifting with the minimum; k = max - min. */
    static int[] sortAnyRange(int[] a) {
        if (a.length == 0) return a.clone();
        int min = Arrays.stream(a).min().getAsInt(), max = Arrays.stream(a).max().getAsInt();
        long range = (long) max - min;
        if (range > 50_000_000) throw new IllegalArgumentException("range too large for counting sort: " + range);
        int[] count = new int[(int) range + 1];
        for (int x : a) count[x - min]++;
        for (int v = 1; v < count.length; v++) count[v] += count[v - 1];
        int[] out = new int[a.length];
        for (int j = a.length - 1; j >= 0; j--) out[--count[a[j] - min]] = a[j];
        return out;
    }

    /** Generic stable counting sort by an integer key in [0, k], so it carries whole records. */
    static <T> List<T> sortBy(List<T> items, java.util.function.ToIntFunction<T> key, int k) {
        int[] count = new int[k + 1];
        for (T t : items) count[key.applyAsInt(t)]++;
        for (int v = 1; v <= k; v++) count[v] += count[v - 1];
        Object[] out = new Object[items.size()];
        for (int j = items.size() - 1; j >= 0; j--) out[--count[key.applyAsInt(items.get(j))]] = items.get(j);
        @SuppressWarnings("unchecked") List<T> res = (List<T>) Arrays.asList(out);
        return res;
    }

    /** Histogram-only version (not record-preserving). */
    static void sortInPlaceHistogram(int[] a, int k) {
        int[] count = new int[k + 1];
        for (int x : a) count[x]++;
        int i = 0;
        for (int v = 0; v <= k; v++) while (count[v]-- > 0) a[i++] = v;
    }

    record Student(String name, int marks) {}

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        int[] a = {2, 5, 3, 0, 2, 3, 0, 3};                            // CLRS Figure 8.2
        int[] b = sort(a, 5);
        System.out.println("A = " + Arrays.toString(a) + "  ->  B = " + Arrays.toString(b));
        check(Arrays.equals(b, new int[]{0, 0, 2, 2, 3, 3, 3, 5}), "matches CLRS Figure 8.2");

        Random rnd = new Random(6);
        boolean ok = true, okAny = true, okHist = true;
        for (int t = 0; t < 3000; t++) {
            int k = rnd.nextInt(100);
            int[] x = rnd.ints(rnd.nextInt(300), 0, k + 1).toArray();
            int[] exp = x.clone(); Arrays.sort(exp);
            if (!Arrays.equals(sort(x, k), exp)) ok = false;
            int[] neg = rnd.ints(rnd.nextInt(300), -1000, 1000).toArray();
            int[] expNeg = neg.clone(); Arrays.sort(expNeg);
            if (!Arrays.equals(sortAnyRange(neg), expNeg)) okAny = false;
            int[] h = x.clone(); sortInPlaceHistogram(h, k);
            if (!Arrays.equals(h, exp)) okHist = false;
        }
        check(ok, "3000 random arrays in [0, k] match Arrays.sort");
        check(okAny, "negative numbers handled by shifting with min");
        check(okHist, "histogram version also correct (for plain ints)");

        // Stability on records: sort students by marks; equal marks keep input (alphabetical) order
        List<Student> students = List.of(new Student("Asha", 85), new Student("Bilal", 72), new Student("Chen", 85),
                new Student("Deepa", 90), new Student("Eli", 72), new Student("Farah", 85));
        List<Student> byMarks = sortBy(students, Student::marks, 100);
        System.out.println("by marks: " + byMarks);
        boolean stable = true;
        for (int i = 1; i < byMarks.size(); i++)
            if (byMarks.get(i - 1).marks() == byMarks.get(i).marks()
                && byMarks.get(i - 1).name().compareTo(byMarks.get(i).name()) > 0) stable = false;
        check(stable, "stable: Asha, Chen, Farah (all 85) stay in their original order");

        // Linear growth when k = O(n): work = n + k
        int n = 1_000_000;
        int[] big = rnd.ints(n, 0, 256).toArray();                    // e.g. byte values
        int[] exp = big.clone(); Arrays.sort(exp);
        check(Arrays.equals(sort(big, 255), exp), "1,000,000 byte values sorted in Theta(n + 256)");

        try { sortAnyRange(new int[]{0, 2_000_000_000}); check(false, "should refuse"); }
        catch (IllegalArgumentException e) { check(true, "refuses a range of 2e9 (would need an 8 GB count array)"); }
    }
}
```

**Output:**

```
A = [2, 5, 3, 0, 2, 3, 0, 3]  ->  B = [0, 0, 2, 2, 3, 3, 3, 5]
ok   matches CLRS Figure 8.2
ok   3000 random arrays in [0, k] match Arrays.sort
ok   negative numbers handled by shifting with min
ok   histogram version also correct (for plain ints)
by marks: [Student[name=Bilal, marks=72], Student[name=Eli, marks=72], Student[name=Asha, marks=85], Student[name=Chen, marks=85], Student[name=Farah, marks=85], Student[name=Deepa, marks=90]]
ok   stable: Asha, Chen, Farah (all 85) stay in their original order
ok   1,000,000 byte values sorted in Theta(n + 256)
ok   refuses a range of 2e9 (would need an 8 GB count array)
```

**Dry run (0-indexed Java):** `out[--count[a[j]]]` decrements first and then writes, which turns CLRS's 1-indexed "write at C, then decrement" into 0-indexed positions.

**Java-specific details**
- `long range = (long) max - min` avoids `int` overflow when max − min exceeds 2³¹ (for example min = −2×10⁹ and max = 2×10⁹).
- `x - min` is safe once the range is known to fit.
- A guard against huge ranges is good engineering: `new int[k + 1]` with k = 2×10⁹ would throw `OutOfMemoryError` or `NegativeArraySizeException`.

**Common mistakes**
1. Iterating forwards in the final pass. It still sorts correctly, but it's **not stable**, which **breaks radix sort**.
2. Off-by-one between 1-indexed CLRS and 0-indexed Java (writing to `count[x]` and then decrementing → `ArrayIndexOutOfBoundsException` at position n).
3. Using it when k ≫ n.
4. Forgetting negative values.

---

## F. Time complexity

| Phase | Work |
|---|---|
| Clear counts (lines 1–2) | Θ(k) |
| Histogram (lines 3–4) | Θ(n) |
| Prefix sums (lines 5–6) | Θ(k) |
| Placement (lines 7–9) | Θ(n) |
| **Total** | **Θ(n + k)** |

- **Best = average = worst = Θ(n + k).** The work doesn't depend on the order of the input at all.
- **When k = O(n), this is Θ(n): linear time.**
- **When k = n²:** it's Θ(n²), which is worse than comparison sorts. Radix sort fixes this by splitting the key into digits.

**Why the Ω(n log n) bound doesn't apply:** the step `count[A[j]]++` uses the key's **value as an address**. One array access extracts about log₂ k bits of information about the key, while a comparison yields only 1 bit. The decision-tree model doesn't capture this.

**Assumption:** keys are integers (or map to integers) in a range of size k, and indexing an array costs O(1).

## G. Space complexity

| Component | Size |
|---|---|
| Count array | Θ(k) |
| Output array | Θ(n) |
| **Total auxiliary** | **Θ(n + k)**, so **not in place** |

The histogram version needs only Θ(k) extra and writes back into the input, but it can't carry records.

## H. Complexity summary

| Algorithm | Best time | Average time | Worst time | Auxiliary space | Stable? | In-place? |
|---|---|---|---|---|---|---|
| Counting sort | Θ(n + k) | Θ(n + k) | Θ(n + k) | Θ(n + k) | ✅ Yes | ❌ No |

Comparison-based ❌ · requires integer keys in [0, k]

## I. Advantages, limitations, and comparisons

**Advantages:** linear time for small ranges, stable, simple, and fully predictable (no bad inputs).

**Limitations:** it needs **integer keys** and a **small range**. It uses Θ(n + k) memory, and it's useless for k ≫ n (like 64-bit IDs).

| Situation | Best choice |
|---|---|
| n = 10⁶, keys in [0, 255] | **Counting sort** |
| n = 10⁶ ages (0–150) of records, stability needed | **Counting sort** |
| n = 10⁶, 32-bit integers | [Radix sort](radix-sort.md) (4 passes of counting sort on bytes) or `Arrays.sort` |
| n = 1000, keys up to 10⁹ | a comparison sort (k is far too big) |
| Real numbers | [bucket sort](bucket-sort.md) or a comparison sort |

**Interview follow-ups:** "Sort an array where values are in [0, 100]", "Why is counting sort stable, and why does it matter?", "Sort characters by frequency", "Maximum gap" (bucket or pigeonhole idea).

---

## J. Practice

**Beginner**
1. Trace COUNTING-SORT on A = [6, 0, 2, 0, 1, 3, 4, 6, 1, 3, 2] (CLRS Exercise 8.2-1).
2. Prove counting sort is stable (CLRS Exercise 8.2-2).
3. Sort [−3, 2, −1, 0, 2] with counting sort using a min offset.

**Intermediate**
4. If the last loop ran forwards (j = 1 to n), would the algorithm still work? Would it be stable? (CLRS Exercise 8.2-3)
5. Preprocess n integers in [0, k] in Θ(n + k) so that "how many fall in [a, b]?" can be answered in O(1) (CLRS Exercise 8.2-4).
6. Sort a string's characters by frequency (LeetCode 451) using counts.

**Advanced**
7. Implement in-place counting sort ("American flag sort", unstable) for integer keys.
8. Use counting sort to compute the H-index in Θ(n) (LeetCode 274).

**Interview questions**

<details><summary>Q1. When is counting sort better than quicksort?</summary>

When keys are integers in a small range k = O(n), such as ages, grades or bytes. Counting sort is Θ(n + k), linear, and stable, while quicksort is Θ(n log n). When k ≫ n (arbitrary 32-bit values), counting sort's Θ(k) memory and time make it worse.
</details>

<details><summary>Q2. Why does counting sort traverse the input backwards in the final step?</summary>

The prefix count C[v] points to the last slot for value v. Placing elements from the end of the input into slots from right to left keeps equal keys in their original relative order, which makes the sort stable. Stability is required when counting sort is used inside radix sort.
</details>

<details><summary>Q3. How does counting sort beat the Ω(n log n) lower bound?</summary>

The bound only applies to comparison sorts. Counting sort never compares elements. It uses each key directly as an array index, which extracts more than one bit of information per step.
</details>

**Worked problem: Relative Sort Array (LeetCode 1122).** Values are in [0, 1000]. Count every value of arr1. Then, for each value in arr2's order, output it count[v] times and zero its count. Finally output the remaining values in increasing order, scanning v = 0..1000. That's Θ(n + m + k) with k = 1000.

**Coding problems**
- LeetCode 1122 · Relative Sort Array *(verify link)*
- LeetCode 451 · Sort Characters By Frequency *(verify link)*
- LeetCode 274 · H-Index *(verify link)*
- LeetCode 75 · Sort Colors (k = 2) *(verify link)*
- LeetCode 1365 · How Many Numbers Are Smaller Than the Current Number *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. Count each key, take prefix sums (giving final positions), then place elements backwards.
2. Θ(n + k) time and space. Linear when k = O(n).
3. **Stable** because of the backwards final pass. This is essential for radix sort.
4. Not a comparison sort, so the Ω(n log n) bound doesn't apply.
5. Needs integer keys in a small, known range. Shift by min for negatives.

**Formulas:** time Θ(n + k), space Θ(n + k). C[v] after the prefix sums = #{elements ≤ v}.

**Common mistakes:** a forward final pass (unstable), huge k, off-by-one when converting from 1-indexed.

**Quiz**
1. Complexity for n = 10⁶ keys in [0, 10⁶]?
2. What does C[v] mean after the prefix-sum step?
3. Is counting sort in place?
4. Which algorithm uses counting sort as a subroutine?

**Answers**

<details><summary>Show answers</summary>

1. Θ(n + k) = Θ(2 × 10⁶), which is linear.
2. The number of elements ≤ v, which is the last output position for value v.
3. No: it needs a Θ(n) output array and a Θ(k) count array.
4. Radix sort (LSD), which needs a stable per-digit sort.
</details>

**Related topics:** [Lower bounds](lower-bounds.md) · [Radix sort](radix-sort.md) · [Bucket sort](bucket-sort.md) · [Stability](../README.md#stability)
