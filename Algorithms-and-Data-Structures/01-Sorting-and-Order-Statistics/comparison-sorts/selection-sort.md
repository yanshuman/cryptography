# Selection Sort

> **CLRS:** Exercise 2.2-2 · **Status:** ✅ Written · **Prerequisites:** [Algorithm basics](../../00-Foundations/algorithm-basics.md), [Minimum and maximum](../order-statistics/minimum-and-maximum.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

**Selection sort** repeatedly **selects the smallest remaining element** and swaps it into the next position of the sorted prefix.

**What problem does it solve?** It sorts in place with the **minimum possible number of swaps**: at most n − 1. That matters when *writing* is far more expensive than *reading* or comparing (flash memory, EEPROM, or moving heavy physical objects).

**Why learn it?**
- It's the simplest "find the minimum, put it first" idea, and the ancestor of [heapsort](../heapsort/heapsort.md), which is selection sort with a faster way to find the minimum (a heap).
- It's a clean example of an algorithm whose running time **doesn't depend on the input order**: Θ(n²) always.
- It's a standard example of an **unstable** sort, and of how to fix that.

**Prerequisites:** finding the minimum of an array in n − 1 comparisons.

---

## B. Intuition

**Analogy: picking a sports team in order of height.** Scan the whole line and find the shortest person. Send them to position 1. Scan the remaining people, find the shortest, send them to position 2. And so on.

You always scan *everything* that's left, even if the line is already in order, because you can't know a person is the shortest without checking all the others. That's why it never gets faster on sorted input.

**Why it works:** after step i, the first i positions hold the i smallest elements in order. Those are final and never touched again.

---

## C. How it works internally

**Input:** A = [64, 25, 12, 22, 11]. The bar `|` separates the sorted prefix from the rest.

| Step i | Unsorted part scanned | Minimum (index) | Swap | Array after |
|---|---|---|---|---|
| 0 | [64, 25, 12, 22, 11] | 11 (4) | A[0] ↔ A[4] | [11 \| 25, 12, 22, 64] |
| 1 | [25, 12, 22, 64] | 12 (2) | A[1] ↔ A[2] | [11, 12 \| 25, 22, 64] |
| 2 | [25, 22, 64] | 22 (3) | A[2] ↔ A[3] | [11, 12, 22 \| 25, 64] |
| 3 | [25, 64] | 25 (3) | none (already in place) | [11, 12, 22, 25 \| 64] |
| done | | | | [11, 12, 22, 25, 64] |

Comparisons: 4 + 3 + 2 + 1 = **10 = n(n − 1)/2**. Swaps: **3** (at most n − 1 = 4).

### Why selection sort is NOT stable

Swapping across a long distance can jump an element over an equal one:

```
Input:   [2a, 2b, 1]          (2a and 2b are equal keys; a and b mark the original order)
i = 0:   min of all = 1 at index 2 → swap A[0] ↔ A[2]
         [1, 2b, 2a]          ← 2a jumped over 2b: their original order is reversed
i = 1:   min of [2b, 2a] = 2b (first occurrence) → no swap
Result:  [1, 2b, 2a]          ✗ not stable
```

**The stable variant:** instead of swapping, **shift** the elements between i and the minimum one place right and insert the minimum at i (as insertion sort does). It's stable, but now Θ(n²) **writes**, so it loses selection sort's one advantage.

**Edge cases:** empty or one element → nothing to do. All equal → n(n − 1)/2 comparisons and 0 swaps (when swapping is skipped for i = minIndex). Already sorted → still n(n − 1)/2 comparisons.

---

## D. Algorithm and pseudocode

```
SELECTION-SORT(A, n)
1  for i ← 1 to n − 1
2      min ← i
3      for j ← i + 1 to n
4          if A[j] < A[min]
5              min ← j
6      if min ≠ i
7          exchange A[i] ↔ A[min]
```

| Line | Meaning |
|---|---|
| 1 | Positions 1..n − 1 each receive their final element. The last one is then automatically correct. |
| 2–5 | Find the index of the minimum of A[i..n] (the [minimum-finding](../order-statistics/minimum-and-maximum.md) loop) |
| 6–7 | Swap it into position i. Skip the swap if it's already there. |

### Loop invariant (CLRS Exercise 2.2-2)

> **Invariant:** at the start of iteration i, A[1..i − 1] contains the i − 1 **smallest** elements of the array, in sorted order. (Equivalently, the prefix is sorted, and every element of A[1..i − 1] ≤ every element of A[i..n].)

- **Initialization:** i = 1, so the prefix is empty. ✓
- **Maintenance:** the inner loop finds the minimum of A[i..n], which is the smallest element not in the prefix. Every prefix element is ≤ it (by the invariant), so placing it at position i keeps the prefix sorted, and A[1..i] holds the i smallest. ✓
- **Termination:** the loop ends with i = n. A[1..n − 1] holds the n − 1 smallest elements, sorted, so A[n] must be the largest. **The whole array is sorted.** ✓

**Why only n − 1 iterations (CLRS asks this):** once the n − 1 smallest are placed, the remaining element is the maximum and is already in the last position.

---

## E. Implementation

```java
import java.util.*;

/** Selection sort: standard (unstable, <= n-1 swaps) and a stable shifting variant. */
public class SelectionSort {

    static long comparisons, swaps;

    /** In place, NOT stable. Always n(n-1)/2 comparisons, at most n-1 swaps. */
    public static void sort(int[] a) {
        int n = a.length;
        for (int i = 0; i < n - 1; i++) {
            int min = i;
            for (int j = i + 1; j < n; j++) {
                comparisons++;
                if (a[j] < a[min]) min = j;               // '<' picks the FIRST minimum
            }
            if (min != i) { int t = a[i]; a[i] = a[min]; a[min] = t; swaps++; }
        }
    }

    /** Generic unstable version (used to demonstrate instability). */
    public static <T> void sort(T[] a, Comparator<? super T> cmp) {
        for (int i = 0; i < a.length - 1; i++) {
            int min = i;
            for (int j = i + 1; j < a.length; j++) if (cmp.compare(a[j], a[min]) < 0) min = j;
            T t = a[i]; a[i] = a[min]; a[min] = t;
        }
    }

    /** Stable variant: rotate the minimum into place instead of swapping (Theta(n^2) writes). */
    public static <T> void stableSort(T[] a, Comparator<? super T> cmp) {
        for (int i = 0; i < a.length - 1; i++) {
            int min = i;
            for (int j = i + 1; j < a.length; j++) if (cmp.compare(a[j], a[min]) < 0) min = j;
            T key = a[min];
            System.arraycopy(a, i, a, i + 1, min - i);    // shift a[i..min-1] right by one
            a[i] = key;
        }
    }

    record Item(int key, String tag) {
        @Override public String toString() { return key + tag; }
    }

    static <T> boolean isStable(Item[] sorted, Item[] original) {
        // equal keys must appear in the same relative order as in the original array
        List<Item> orig = Arrays.asList(original);
        for (int i = 1; i < sorted.length; i++)
            if (sorted[i - 1].key() == sorted[i].key() && orig.indexOf(sorted[i - 1]) > orig.indexOf(sorted[i])) return false;
        return true;
    }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        int[] a = {64, 25, 12, 22, 11};
        comparisons = swaps = 0;
        sort(a);
        System.out.println("sorted: " + Arrays.toString(a) + "  comparisons=" + comparisons + " swaps=" + swaps);
        check(Arrays.equals(a, new int[]{11, 12, 22, 25, 64}), "example sorted");
        check(comparisons == 10 && swaps == 3, "10 comparisons, 3 swaps (as in the trace)");

        int n = 1000;
        int[] sorted = new int[n], reversed = new int[n];
        for (int i = 0; i < n; i++) { sorted[i] = i; reversed[i] = n - i; }
        comparisons = swaps = 0; sort(sorted);
        check(comparisons == (long) n * (n - 1) / 2 && swaps == 0, "sorted input: still n(n-1)/2 comparisons (no best-case speed-up), 0 swaps");
        comparisons = swaps = 0; sort(reversed);
        check(swaps <= n - 1, "reverse input: at most n-1 swaps (here " + swaps + ")");

        Random rnd = new Random(3);
        boolean ok = true, swapBound = true;
        for (int t = 0; t < 2000; t++) {
            int[] x = rnd.ints(rnd.nextInt(60), -40, 40).toArray();
            int[] exp = x.clone(); Arrays.sort(exp);
            swaps = 0; sort(x);
            if (!Arrays.equals(x, exp)) ok = false;
            if (swaps > Math.max(0, x.length - 1)) swapBound = false;
        }
        check(ok, "2000 random arrays match Arrays.sort");
        check(swapBound, "never more than n-1 swaps");

        // Instability demo from Section C: [2a, 2b, 1]
        Item[] demo = {new Item(2, "a"), new Item(2, "b"), new Item(1, "")};
        Item[] d1 = demo.clone(); sort(d1, Comparator.comparingInt(Item::key));
        Item[] d2 = demo.clone(); stableSort(d2, Comparator.comparingInt(Item::key));
        System.out.println("standard on [2a, 2b, 1] -> " + Arrays.toString(d1) + "   stable variant -> " + Arrays.toString(d2));
        check(!isStable(d1, demo), "standard selection sort is NOT stable (2b now before 2a)");
        check(isStable(d2, demo), "shifting variant IS stable");

        Item[] many = new Item[300];
        for (int i = 0; i < many.length; i++) many[i] = new Item(rnd.nextInt(6), "#" + i);
        Item[] m2 = many.clone(); stableSort(m2, Comparator.comparingInt(Item::key));
        check(isStable(m2, many), "stable variant keeps order on 300 random items");
    }
}
```

**Output:**

```
sorted: [11, 12, 22, 25, 64]  comparisons=10 swaps=3
ok   example sorted
ok   10 comparisons, 3 swaps (as in the trace)
ok   sorted input: still n(n-1)/2 comparisons (no best-case speed-up), 0 swaps
ok   reverse input: at most n-1 swaps (here 500)
ok   2000 random arrays match Arrays.sort
ok   never more than n-1 swaps
standard on [2a, 2b, 1] -> [1, 2b, 2a]   stable variant -> [1, 2a, 2b]
ok   standard selection sort is NOT stable (2b now before 2a)
ok   shifting variant IS stable
ok   stable variant keeps order on 300 random items
```

**Note on the reversed-input result (500 swaps, not 999):** the first swap puts both the smallest and the largest element in their final places at once. Each swap on a reversed array fixes two positions, so only n/2 swaps are needed.

**Java-specific details:** `System.arraycopy(a, i, a, i + 1, len)` handles overlapping ranges correctly (it behaves like `memmove`), which is exactly what the stable shift needs.

**Common mistakes**
1. Swapping inside the inner loop every time a smaller element is found. That's still correct but turns into Θ(n²) swaps, which defeats the purpose.
2. Using `<=` when finding the minimum. That picks the *last* equal minimum, which makes instability worse.
3. Claiming selection sort is stable.

---

## F. Time complexity

The inner loop runs n − i times for i = 1..n − 1 (1-indexed):

> **Comparisons = Σᵢ₌₁ⁿ⁻¹ (n − i) = (n − 1) + (n − 2) + … + 1 = n(n − 1)/2**

This number is **identical for every input**: the algorithm never checks whether the array is already in order.

| Case | Comparisons | Swaps | Time |
|---|---|---|---|
| Best (sorted) | n(n − 1)/2 | 0 | **Θ(n²)** |
| Average | n(n − 1)/2 | ≈ n − Hₙ, so Θ(n) | **Θ(n²)** |
| Worst | n(n − 1)/2 | n − 1 | **Θ(n²)** |

**Swaps: Θ(n) worst case.** That's the minimum possible for an in-place sort that moves elements by swapping (a permutation with c cycles needs exactly n − c swaps).

**Could a better minimum-finder help?** Yes, and that's exactly **heapsort**: replace the Θ(n) linear scan with a heap's Θ(log n) extract-min, and Σ log n = Θ(n log n). See [Heapsort](../heapsort/heapsort.md).

## G. Space complexity

Θ(1) auxiliary (min index, loop counters, swap temporary). **In place**, no recursion, the same in all cases. The stable variant is also Θ(1) auxiliary.

## H. Complexity summary

| Algorithm | Best time | Average time | Worst time | Auxiliary space | Stable? | In-place? |
|---|---|---|---|---|---|---|
| Selection sort | Θ(n²) | Θ(n²) | Θ(n²) | Θ(1) | ❌ No | ✅ Yes |
| Stable selection (shifting) | Θ(n²) | Θ(n²) | Θ(n²) | Θ(1) | ✅ Yes | ✅ Yes |

Comparison-based ✅ · adaptive ❌ · swaps ≤ n − 1 ✅

## I. Advantages, limitations, and comparisons

**Advantages**
- **At most n − 1 swaps** (Θ(n) writes), the fewest of any common sort.
- Simple, in place, with completely predictable running time.

**Limitations**
- Θ(n²) **even on sorted input** (not adaptive).
- Not stable (in the swap form).

| vs | Selection is better when… |
|---|---|
| [Insertion](insertion-sort.md) | Writes are expensive (insertion sort can do Θ(n²) writes) |
| [Bubble](bubble-sort.md) | Almost always (far fewer writes) |
| [Heapsort](../heapsort/heapsort.md) | n is tiny. Heapsort is "selection sort done right" for large n. |

**Real-world use:** situations where writes are costly, such as flash memory with limited write cycles, or physically moving large items. It's also the conceptual basis for heapsort and for partial sorting ("find the top k": run k passes, Θ(nk)).

**Interview follow-ups:** "Is selection sort stable? Make it stable.", "Minimum swaps to sort a permutation" (count the cycles: n − cycles), "Find the k smallest" (k passes, or better, a heap or quickselect).

---

## J. Practice

**Beginner**
1. Trace it on [29, 10, 14, 37, 13].
2. Modify it to select the **maximum** and place it at the end.
3. How many comparisons does it make for n = 7?

**Intermediate**
4. Implement double-ended selection sort (find the min and max per pass). How many passes does it need?
5. Compute the minimum number of swaps needed to sort a permutation of 0..n − 1 (cycle decomposition).
6. Prove that selection sort's swap count is at most n − 1.

**Advanced**
7. Show that the expected number of swaps on a random permutation is n − Hₙ.
8. Explain how replacing the linear minimum scan with a heap turns selection sort into heapsort, and derive its complexity.

**Interview questions**

<details><summary>Q1. Why is selection sort Θ(n²) even on sorted input?</summary>

To be sure an element is the minimum of the remaining part, it has to compare against all of them. The inner loop always runs n − i times regardless of the order, so it always does n(n − 1)/2 comparisons.
</details>

<details><summary>Q2. When is selection sort a good choice?</summary>

When writes or swaps are much more expensive than comparisons (for example flash memory), because it does at most n − 1 swaps. For small n it's also simple and predictable.
</details>

<details><summary>Q3. Give an example showing that selection sort is unstable.</summary>

[2a, 2b, 1]: the first swap exchanges 2a with 1, giving [1, 2b, 2a]. The two equal 2s have swapped their relative order.
</details>

**Worked problem: minimum swaps to sort a permutation.** For [4, 3, 1, 2] (values 1..4), follow the cycles: position 1 holds 4 → position 4 holds 2 → position 2 holds 3 → position 3 holds 1 → back to position 1. That's one cycle of length 4, so the answer is 4 − 1 = **3 swaps**. In general, the minimum number of swaps is n − (number of cycles). Selection sort achieves this minimum on permutations.

**Coding problems**
- GeeksforGeeks · Minimum swaps required to sort an array *(verify link)*
- LeetCode 912 · Sort an Array *(verify link)*
- LeetCode 215 · Kth Largest Element (contrast with selection-based approaches) *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. Repeatedly find the minimum of the unsorted part and swap it to the front.
2. Always n(n − 1)/2 comparisons, so Θ(n²) in every case.
3. At most n − 1 swaps, the fewest writes of the common sorts.
4. Not stable (long-distance swaps). The shifting variant is stable but costs Θ(n²) writes.
5. Heapsort = selection sort + a heap for Θ(log n) minimum-finding.

**Formulas:** comparisons = n(n − 1)/2. Swaps ≤ n − 1. Minimum swaps for a permutation = n − #cycles.

**Common mistakes:** swapping inside the inner loop, calling it stable, expecting a best-case speed-up.

**Quiz**
1. How many comparisons for n = 10?
2. Max swaps for n = 10?
3. Is selection sort adaptive?
4. Which famous algorithm improves selection sort's minimum-finding step?

**Answers**

<details><summary>Show answers</summary>

1. 45.
2. 9.
3. No: Θ(n²) regardless of the input order.
4. Heapsort, which uses a max-heap and finds the max in O(log n).
</details>

**Related topics:** [Insertion sort](insertion-sort.md) · [Bubble sort](bubble-sort.md) · [Heapsort](../heapsort/heapsort.md) · [Minimum and maximum](../order-statistics/minimum-and-maximum.md) · [Sorting overview](../README.md#stability)
