# Bubble Sort

> **CLRS:** Problem 2-2 ("Correctness of bubblesort") · **Status:** ✅ Written · **Prerequisites:** [Algorithm basics](../../00-Foundations/algorithm-basics.md), [Time complexity](../../00-Foundations/time-complexity.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

**Bubble sort** repeatedly walks through the array, **compares each pair of adjacent elements, and swaps them if they're in the wrong order**. After each pass the largest remaining element has "bubbled up" to its final position at the end.

**What problem does it solve?** It sorts using only adjacent swaps. It's the simplest sort to describe and prove correct.

**Why learn it?** It's almost never used in production. Insertion sort is better in every practical way. But it's:
- the classic first sorting algorithm and a common interview warm-up;
- a clean example of an **invariant on the suffix** ("the last i elements are final");
- a way to understand **inversions** (each adjacent swap removes exactly one) and **early-exit optimisation** (adaptivity);
- historically used in hardware and parallel contexts, where only neighbouring cells can swap (see odd-even transposition sort and [sorting networks](../../08-Advanced-Algorithm-Topics/sorting-networks/comparison-networks.md)).

**Prerequisites:** loops, swaps, and inversions.

---

## B. Intuition

**Analogy: bubbles in water.** Heavy elements sink, light ones rise. In each pass, the biggest element you're carrying keeps beating its neighbour and gets pushed to the right end, like the biggest bubble rising to the surface.

**A second analogy: lining people up by height** with the rule that you can only swap two people *standing next to each other*. Walk down the line, swapping any adjacent pair where the taller person is on the left. After one walk, the tallest person is at the end. Repeat for the rest.

**Why it works:** each pass guarantees one more element reaches its final place, so n − 1 passes are enough. If a pass makes **no swaps**, no adjacent pair is out of order, which means the array is sorted, and we can stop early.

---

## C. How it works internally

**Input:** A = [5, 1, 4, 2, 8]. Elements marked **bold** are in their final positions.

**Pass 1** (compare indices 0–1, 1–2, 2–3, 3–4):

| Compare | Swap? | Array |
|---|---|---|
| 5, 1 | yes | [1, 5, 4, 2, 8] |
| 5, 4 | yes | [1, 4, 5, 2, 8] |
| 5, 2 | yes | [1, 4, 2, 5, 8] |
| 5, 8 | no | [1, 4, 2, 5, **8**] |

**Pass 2** (only up to index 3, because 8 is final):

| Compare | Swap? | Array |
|---|---|---|
| 1, 4 | no | [1, 4, 2, 5, 8] |
| 4, 2 | yes | [1, 2, 4, 5, 8] |
| 4, 5 | no | [1, 2, 4, **5**, **8**] |

**Pass 3:** compare (1, 2) and (2, 4). **No swaps**, so the array is sorted. Stop early.

> Result: [1, 2, 4, 5, 8] after 4 + 3 + 2 = **9 comparisons** and **4 swaps**. The input has exactly 4 inversions: (5,1), (5,4), (5,2), (4,2).

**Edge cases**

| Input | Behaviour |
|---|---|
| empty or one element | no passes |
| already sorted | one pass, 0 swaps, early exit → **n − 1 comparisons (best case)** |
| reverse sorted | n − 1 passes, n(n − 1)/2 swaps (the worst case) |
| small element at the end ("turtle"), e.g. [2, 3, 4, 5, 1] | the 1 moves only **one** step left per pass, so it needs n − 1 passes. Big elements ("rabbits") move fast; small ones crawl. Cocktail-shaker sort fixes this by alternating directions. |
| equal elements | never swapped (strict `>`), so the sort is **stable** |

---

## D. Algorithm and pseudocode

CLRS Problem 2-2 version:

```
BUBBLESORT(A)
1  for i ← 1 to length[A]
2      for j ← length[A] downto i + 1
3          if A[j] < A[j − 1]
4              exchange A[j] ↔ A[j − 1]
```

(CLRS's version bubbles the **smallest** element to the **front**. That's symmetric to the "largest to the end" version used in the Java code.)

**Optimised version** (early exit, and shrinking the range to the last swap position):

```
BUBBLE-SORT-OPTIMISED(A, n)
1  last ← n                            ▷ elements at positions ≥ last are final
2  while last > 1
3      newLast ← 0
4      for j ← 2 to last
5          if A[j − 1] > A[j]
6              exchange A[j − 1] ↔ A[j]
7              newLast ← j − 1         ▷ everything after the last swap is already in order
8      last ← newLast                  ▷ if no swap happened, newLast = 0, so we stop
```

### Loop invariant and correctness (CLRS Problem 2-2)

**Outer invariant** (for the largest-to-end version): at the start of pass i, the last i − 1 positions hold the i − 1 **largest** elements, **in sorted order**, in their final positions.

- **Initialization:** i = 1, so there are zero final elements. Trivially true. ✓
- **Maintenance:** inner invariant: during a pass, after comparing positions j − 1 and j, A[j] is the maximum of A[1 .. j]. (Each comparison carries the larger value forward.) At the end of the pass, A[n − i + 1] = the maximum of the unsorted part, which is the i-th largest element overall, now in its final place. ✓
- **Termination:** after n − 1 passes, the last n − 1 elements are final and sorted, and the remaining first element must be the smallest. **Sorted.** ✓

**Early exit:** if a pass makes no swaps, then A[j − 1] ≤ A[j] for every adjacent pair, so the array is sorted. ✓

**Why it terminates:** the passes are bounded by n − 1, and each pass is bounded by n − 1 comparisons.

---

## E. Implementation

```java
import java.util.*;

/** Bubble sort: plain, optimised (early exit + shrinking bound), and generic for stability tests. */
public class BubbleSort {

    static long comparisons, swaps;

    /** Plain version: always n(n-1)/2 comparisons. */
    public static void sortPlain(int[] a) {
        int n = a.length;
        for (int i = 0; i < n - 1; i++) {
            for (int j = 0; j < n - 1 - i; j++) {         // the last i elements are already final
                comparisons++;
                if (a[j] > a[j + 1]) { swap(a, j, j + 1); swaps++; }
            }
        }
    }

    /** Optimised: stop when a pass makes no swaps; shrink the range to the last swap position. */
    public static void sort(int[] a) {
        int last = a.length - 1;                           // compare pairs (j, j+1) for j < last
        while (last > 0) {
            int newLast = 0;
            for (int j = 0; j < last; j++) {
                comparisons++;
                if (a[j] > a[j + 1]) {                    // strict '>' keeps it stable
                    swap(a, j, j + 1);
                    swaps++;
                    newLast = j;                          // everything after j is now in order
                }
            }
            last = newLast;                               // 0 if no swaps, so the loop ends
        }
    }

    public static <T> void sort(T[] a, Comparator<? super T> cmp) {
        int last = a.length - 1;
        while (last > 0) {
            int newLast = 0;
            for (int j = 0; j < last; j++) {
                if (cmp.compare(a[j], a[j + 1]) > 0) {
                    T t = a[j]; a[j] = a[j + 1]; a[j + 1] = t;
                    newLast = j;
                }
            }
            last = newLast;
        }
    }

    static void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }

    static long inversions(int[] a) {
        long c = 0;
        for (int i = 0; i < a.length; i++) for (int j = i + 1; j < a.length; j++) if (a[i] > a[j]) c++;
        return c;
    }

    record Item(int key, int originalIndex) {}

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        int[] a = {5, 1, 4, 2, 8};
        comparisons = swaps = 0;
        sort(a);
        System.out.println("optimised on [5,1,4,2,8] -> " + Arrays.toString(a) + "  comparisons=" + comparisons + " swaps=" + swaps);
        check(Arrays.equals(a, new int[]{1, 2, 4, 5, 8}), "example sorted");

        int[] b = {5, 1, 4, 2, 8};
        comparisons = swaps = 0;
        sortPlain(b);
        System.out.println("plain     on [5,1,4,2,8] -> " + Arrays.toString(b) + "  comparisons=" + comparisons + " swaps=" + swaps);
        check(comparisons == 10, "plain version always makes n(n-1)/2 = 10 comparisons");

        int n = 1000;
        int[] sorted = new int[n], reversed = new int[n];
        for (int i = 0; i < n; i++) { sorted[i] = i; reversed[i] = n - i; }
        comparisons = swaps = 0; sort(sorted);
        check(comparisons == n - 1 && swaps == 0, "optimised on sorted input: one pass, n-1 comparisons (best case Theta(n))");
        comparisons = swaps = 0; sort(reversed);
        check(swaps == (long) n * (n - 1) / 2, "reverse sorted: n(n-1)/2 swaps (worst case)");

        Random rnd = new Random(11);
        boolean ok = true, invOk = true;
        for (int t = 0; t < 2000; t++) {
            int[] x = rnd.ints(rnd.nextInt(60), -30, 30).toArray();
            int[] exp = x.clone(); Arrays.sort(exp);
            long inv = inversions(x);
            int[] y = x.clone(); swaps = 0; sort(y);
            long optimisedSwaps = swaps;                   // record before running the plain version
            int[] z = x.clone(); sortPlain(z);
            if (!Arrays.equals(y, exp) || !Arrays.equals(z, exp)) ok = false;
            if (optimisedSwaps != inv) invOk = false;
        }
        check(ok, "2000 random arrays match Arrays.sort (both versions)");
        check(invOk, "number of swaps == number of inversions");

        Item[] items = new Item[300];
        for (int i = 0; i < items.length; i++) items[i] = new Item(rnd.nextInt(8), i);
        sort(items, Comparator.comparingInt(Item::key));
        boolean stable = true;
        for (int i = 1; i < items.length; i++)
            if (items[i - 1].key() > items[i].key()
                || (items[i - 1].key() == items[i].key() && items[i - 1].originalIndex() > items[i].originalIndex())) stable = false;
        check(stable, "stable");
    }
}
```

**Output:**

```
optimised on [5,1,4,2,8] -> [1, 2, 4, 5, 8]  comparisons=7 swaps=4
ok   example sorted
plain     on [5,1,4,2,8] -> [1, 2, 4, 5, 8]  comparisons=10 swaps=4
ok   plain version always makes n(n-1)/2 = 10 comparisons
ok   optimised on sorted input: one pass, n-1 comparisons (best case Theta(n))
ok   reverse sorted: n(n-1)/2 swaps (worst case)
ok   2000 random arrays match Arrays.sort (both versions)
ok   number of swaps == number of inversions
ok   stable
```

**Dry run (optimised):** pass 1 compares 4 pairs, and the last swap is at j = 2, so last = 2. Pass 2 compares pairs j = 0, 1: one swap at j = 1, so last = 1. Pass 3 compares j = 0: no swap, so last = 0, stop. That's 4 + 2 + 1 = **7 comparisons**, fewer than the 9 in the hand trace in Section C, which only used the early-exit flag. The "last swap position" trick skips the already-ordered tail.

**Java-specific details:** use a helper `swap` for primitives. For generic arrays, swap the references with a temporary variable. Always keep the comparison strict (`> 0`) for stability.

**Common mistakes**
1. Inner loop `j < n - i` instead of `j < n - 1 - i`, which reads `a[j + 1]` out of bounds.
2. Forgetting to reset the `swapped`/`newLast` flag at the start of each pass.
3. Claiming bubble sort's best case is Θ(n) **without** the early-exit optimisation. Plain bubble sort is always Θ(n²).

---

## F. Time complexity

**Plain version:** pass i compares n − i pairs, so the total is Σᵢ₌₁ⁿ⁻¹ (n − i) = **n(n − 1)/2** comparisons on **every** input, which is Θ(n²) in the best, average and worst cases.

**Optimised version**
- **Best case (sorted):** one pass, n − 1 comparisons, 0 swaps → **Θ(n)**.
- **Worst case (reverse sorted):** every pass needed, n(n − 1)/2 comparisons and n(n − 1)/2 swaps → **Θ(n²)**.
- **Average (random permutation):** swaps = the number of inversions, which averages **n(n − 1)/4**. Comparisons are still Θ(n²): the expected number of passes is n − O(√n), because small elements near the end move left only one position per pass. → **Θ(n²)**.

**Why swaps = inversions:** swapping two *adjacent* out-of-order elements fixes exactly that pair and leaves every other pair's order unchanged. So each swap removes exactly one inversion, and the array is sorted when none remain. (The test checks this on 2000 arrays.)

**Bubble vs insertion, same Θ(n²):** both perform exactly I (number of inversions) element moves, but a bubble *swap* costs 3 writes while an insertion *shift* costs 1. Bubble sort also often does more comparisons, because a pass can't stop early. Insertion sort is typically 2–3× faster.

## G. Space complexity

Θ(1) auxiliary (a temporary for swaps and a few indices). **In place**, no recursion, the same in all cases.

## H. Complexity summary

| Algorithm | Best time | Average time | Worst time | Auxiliary space | Stable? | In-place? |
|---|---|---|---|---|---|---|
| Bubble sort (plain) | Θ(n²) | Θ(n²) | Θ(n²) | Θ(1) | ✅ Yes | ✅ Yes |
| Bubble sort (early exit) | Θ(n) | Θ(n²) | Θ(n²) | Θ(1) | ✅ Yes | ✅ Yes |

Comparison-based ✅ · adaptive (with early exit) ✅

## I. Advantages, limitations, and comparisons

**Advantages:** extremely simple. Stable, in place, and detects an already sorted array in Θ(n) (with early exit). It only swaps neighbours, which suits parallel hardware (odd-even transposition sort runs in n parallel rounds).

**Limitations:** the slowest of the Θ(n²) sorts in practice (most writes, slow "turtles"). Never use it for real data of meaningful size.

| Sort | Comparisons (worst) | Writes (worst) | Best case | Notes |
|---|---|---|---|---|
| Bubble | n²/2 | 3 × n²/2 (swaps) | Θ(n) with early exit | slowest in practice |
| [Insertion](insertion-sort.md) | n²/2 | n²/2 (shifts) | Θ(n) | the best simple sort |
| [Selection](selection-sort.md) | n²/2 always | 3(n − 1) | Θ(n²) | fewest writes |

**Variants:** *cocktail-shaker sort* (alternates direction to move turtles quickly), *comb sort* (compares elements a shrinking gap apart, roughly O(n log n) in practice, similar in spirit to Shell sort), and *odd-even transposition sort* (parallel).

**Interview follow-ups:** "Optimise bubble sort" (early exit plus the last-swap bound), "Why is insertion sort preferred?", "Count the swaps bubble sort would make" (count the inversions in O(n log n) with merge sort).

---

## J. Practice

**Beginner**
1. Trace bubble sort on [3, 2, 1] and count the swaps.
2. Modify it to sort in descending order.
3. How many passes does the optimised version need on [1, 2, 3, 5, 4]?

**Intermediate**
4. Prove that swaps = inversions.
5. Implement cocktail-shaker sort and test it on a "turtle" input [2, 3, 4, 5, 1].
6. Given an array, compute how many swaps bubble sort *would* make, in O(n log n). *(Hint: [count inversions](../../00-Foundations/divide-and-conquer.md#example-2-counting-inversions).)*

**Advanced**
7. Prove that a random permutation needs n − Θ(√n) passes on average.
8. Implement odd-even transposition sort and argue that n rounds suffice.

**Interview questions**

<details><summary>Q1. What is the best-case complexity of bubble sort?</summary>

Θ(n), but only with the early-exit optimisation (stop when a pass makes no swaps). Plain bubble sort always does n(n − 1)/2 comparisons, Θ(n²), even on sorted input.
</details>

<details><summary>Q2. Why is insertion sort usually faster than bubble sort despite the same Θ(n²)?</summary>

Both move each inversion once, but insertion sort *shifts* (1 write per step) and stops each insertion early, while bubble sort *swaps* (3 writes) and runs whole passes. Insertion sort typically does about half the comparisons and a third of the writes.
</details>

<details><summary>Q3. Is bubble sort stable?</summary>

Yes. It only swaps adjacent elements when the left one is strictly greater, so equal elements never pass each other.
</details>

**Worked problem: minimum adjacent swaps to sort.** "What is the minimum number of adjacent swaps needed to sort an array?" The answer is the **number of inversions**: each adjacent swap removes at most one inversion, and bubble sort achieves exactly one per swap. Compute it in O(n log n) with merge sort, or with a Fenwick tree.

**Coding problems**
- LeetCode 912 · Sort an Array *(verify link)*
- GeeksforGeeks · Minimum adjacent swaps to sort / count inversions *(verify link)*
- HackerRank · Bubble Sort (count swaps) *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. Compare and swap adjacent pairs, and the largest element bubbles to the end each pass.
2. Invariant: after pass i, the last i elements are final and sorted.
3. With early exit: best Θ(n). Otherwise Θ(n²) in every case.
4. Swaps = inversions. Stable, in place.
5. The slowest of the simple sorts in practice. Prefer insertion sort.

**Formulas**
- Plain comparisons: n(n − 1)/2 (every input)
- Swaps: number of inversions (worst n(n − 1)/2, average n(n − 1)/4)

**Common mistakes**
- An off-by-one inner bound.
- Claiming Θ(n) best case for the plain version.
- Using `>=`, which breaks stability.

**Quiz**
1. How many comparisons does plain bubble sort make for n = 6?
2. What property does the array have after one pass?
3. What's a "turtle"?
4. Swaps needed for [2, 1, 4, 3]?

**Answers**

<details><summary>Show answers</summary>

1. 6·5/2 = 15.
2. The largest element is in the last position.
3. A small element near the end, which moves only one position left per pass and so slows bubble sort down.
4. 2: the inversions are (2,1) and (4,3).
</details>

**Related topics:** [Insertion sort](insertion-sort.md) · [Selection sort](selection-sort.md) · [Sorting networks](../../08-Advanced-Algorithm-Topics/sorting-networks/comparison-networks.md) · [Sorting overview](../README.md)
