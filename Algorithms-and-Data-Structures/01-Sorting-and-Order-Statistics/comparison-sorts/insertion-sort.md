# Insertion Sort

> **CLRS:** Chapter 2.1 (the book's first algorithm) and 2.2 (its analysis) · **Status:** ✅ Written · **Prerequisites:** [Algorithm basics](../../00-Foundations/algorithm-basics.md), [Time complexity](../../00-Foundations/time-complexity.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

**Insertion sort** builds the sorted array **one element at a time**. Each new element is inserted into its correct position among the elements already sorted to its left.

**What problem does it solve?** It sorts n items using only comparisons, in place, and stably. It's also **very fast when the input is already nearly sorted**.

**Why we need it**
- It's the simplest sort that is actually *used* in practice: for small arrays (n < ~16–32), and as the finishing step of hybrid sorts.
- **Java's TimSort** (`Collections.sort`, `Arrays.sort(Object[])`) uses binary insertion sort for short runs. C++ introsort finishes with insertion sort.
- **Online:** it can sort data as it arrives, one item at a time.
- It's the CLRS example for **loop invariants** and **best/worst-case analysis**.

**Prerequisites:** arrays, loops, and loop invariants.

---

## B. Intuition

**Analogy: sorting a hand of playing cards.** You pick up cards one at a time. Your hand is always sorted. For each new card, you slide it leftwards past every card bigger than it, and drop it into the gap.

```
hand: [2 5 7]   new card: 4
         ← slide past 7, past 5, stop at 2
hand: [2 4 5 7]
```

**Why it works:** the left part (your hand) is *always sorted*. Inserting one card into a sorted hand keeps it sorted. After all the cards, the whole array is sorted.

**Why it's fast on nearly sorted input:** if every card is already almost in place, each one slides only a step or two. The work is proportional to **how out of order** the input is, not to n².

---

## C. How it works internally

**Input:** A = [5, 2, 4, 6, 1, 3] (the CLRS Figure 2.2 example).

The bar `|` separates the **sorted prefix** (left) from the unsorted part (right). The `key` is the element being inserted.

| Step | key | Action | Array after the step |
|---|---|---|---|
| start | – | the prefix [5] is trivially sorted | [5 \| 2, 4, 6, 1, 3] |
| j = 1 | 2 | 5 > 2 → shift 5 right, place 2 at index 0 | [2, 5 \| 4, 6, 1, 3] |
| j = 2 | 4 | 5 > 4 → shift 5. 2 < 4 → stop, place 4 at index 1 | [2, 4, 5 \| 6, 1, 3] |
| j = 3 | 6 | 5 < 6 → no shift, 6 stays | [2, 4, 5, 6 \| 1, 3] |
| j = 4 | 1 | shift 6, 5, 4, 2 → place 1 at index 0 | [1, 2, 4, 5, 6 \| 3] |
| j = 5 | 3 | shift 6, 5, 4. 2 < 3 → stop, place 3 at index 2 | [1, 2, 3, 4, 5, 6] |

**Inside one insertion (j = 4, key = 1)**, showing the "hole" moving left:

```
[2, 4, 5, 6, _ ]   key = 1 lifted out, the hole is at index 4
[2, 4, 5, _, 6 ]   6 > 1 → shift right
[2, 4, _, 5, 6 ]   5 > 1 → shift
[2, _, 4, 5, 6 ]   4 > 1 → shift
[_, 2, 4, 5, 6 ]   2 > 1 → shift; i = −1, stop
[1, 2, 4, 5, 6 ]   drop the key into the hole
```

**Edge cases**

| Input | Behaviour |
|---|---|
| empty or one element | loop runs 0 times, already sorted |
| already sorted | each key is compared once and never shifted: **n − 1 comparisons** |
| reverse sorted | each key shifts past the whole prefix: n(n − 1)/2 shifts, the **worst case** |
| all equal | the comparison `A[i] > key` is false immediately, so Θ(n), and **stability** is preserved |
| duplicates | equal elements are never shifted past each other (`>` not `≥`), which is why it's **stable** |

---

## D. Algorithm and pseudocode

CLRS version (1-indexed):

```
INSERTION-SORT(A)
1  for j ← 2 to length[A]
2      key ← A[j]
3      ▷ Insert A[j] into the sorted sequence A[1 .. j − 1].
4      i ← j − 1
5      while i > 0 and A[i] > key
6          A[i + 1] ← A[i]
7          i ← i − 1
8      A[i + 1] ← key
```

| Line | Meaning |
|---|---|
| 1 | Process each element from the second one onwards. The first element alone is a sorted prefix. |
| 2 | Lift out the key, which leaves a "hole" at position j |
| 4 | Start comparing with the element just left of the hole |
| 5 | Keep going while elements are **strictly greater** than the key (`>` keeps the sort stable) and we haven't run off the left end |
| 6 | Shift the bigger element right, moving the hole left |
| 7 | Move one position left |
| 8 | Drop the key into the hole |

### Loop invariant and correctness proof

> **Invariant:** at the start of each iteration of the `for` loop (line 1), the subarray A[1 .. j − 1] consists of the elements **originally** in A[1 .. j − 1], but in **sorted order**.

- **Initialization:** before the first iteration, j = 2, so A[1 .. 1] is a single element, which is trivially sorted and is the original element. ✓
- **Maintenance:** the body shifts A[j − 1], A[j − 2], … one position right until it finds the spot where the key fits (A[i] ≤ key, or i = 0), then inserts the key. So A[1 .. j] contains the original elements of A[1 .. j], in sorted order. Incrementing j re-establishes the invariant. ✓ (A fully formal proof would also give an invariant for the inner `while` loop: A[i + 2 .. j] holds the elements greater than the key, shifted right by one.)
- **Termination:** the loop ends when j = n + 1. Substituting: A[1 .. n] holds the original elements in sorted order, so **the whole array is sorted**. ✓

**Why it terminates:** the inner loop decreases i by 1 each time and stops at i = 0 at the latest. The outer loop runs exactly n − 1 times.

---

## E. Implementation

```java
import java.util.*;

/** Insertion sort (CLRS 2.1), with operation counters, a stable generic version and a self-test. */
public class InsertionSort {

    static long comparisons, shifts;

    /** Sorts a in ascending order, in place, stable. Theta(n^2) worst case, Theta(n) best. */
    public static void sort(int[] a) {
        for (int j = 1; j < a.length; j++) {            // CLRS j = 2..n  ->  Java j = 1..n-1
            int key = a[j];
            int i = j - 1;
            while (i >= 0) {
                comparisons++;
                if (a[i] <= key) break;                 // strict ">" for shifting keeps equal keys in order (stable)
                a[i + 1] = a[i];                        // shift the larger element right
                shifts++;
                i--;
            }
            a[i + 1] = key;                             // drop the key into the hole
        }
    }

    /** Generic version for any Comparable or Comparator, used to test stability on records. */
    public static <T> void sort(T[] a, Comparator<? super T> cmp) {
        for (int j = 1; j < a.length; j++) {
            T key = a[j];
            int i = j - 1;
            while (i >= 0 && cmp.compare(a[i], key) > 0) {
                a[i + 1] = a[i];
                i--;
            }
            a[i + 1] = key;
        }
    }

    /** Binary insertion sort: O(n log n) comparisons, but still Theta(n^2) element moves. */
    public static void binaryInsertionSort(int[] a) {
        for (int j = 1; j < a.length; j++) {
            int key = a[j], lo = 0, hi = j;             // find the first position whose value is > key (stable)
            while (lo < hi) {
                int mid = (lo + hi) >>> 1;
                if (a[mid] <= key) lo = mid + 1; else hi = mid;
            }
            System.arraycopy(a, lo, a, lo + 1, j - lo);  // shift the block right in one call
            a[lo] = key;
        }
    }

    static long inversions(int[] a) {
        long c = 0;
        for (int i = 0; i < a.length; i++) for (int j = i + 1; j < a.length; j++) if (a[i] > a[j]) c++;
        return c;
    }

    record Item(int key, int originalIndex) {}

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        int[] a = {5, 2, 4, 6, 1, 3};
        System.out.println("before: " + Arrays.toString(a));
        comparisons = shifts = 0;
        sort(a);
        System.out.println("after:  " + Arrays.toString(a) + "   comparisons=" + comparisons + " shifts=" + shifts);
        check(Arrays.equals(a, new int[]{1, 2, 3, 4, 5, 6}), "CLRS example sorted");

        // Best, worst and the shifts == inversions identity
        int n = 1000;
        int[] sorted = new int[n], reversed = new int[n];
        for (int i = 0; i < n; i++) { sorted[i] = i; reversed[i] = n - i; }
        comparisons = shifts = 0; sort(sorted);
        check(comparisons == n - 1 && shifts == 0, "already sorted: n-1 = 999 comparisons, 0 shifts (best case)");
        comparisons = shifts = 0; sort(reversed);
        check(shifts == (long) n * (n - 1) / 2, "reverse sorted: n(n-1)/2 = 499500 shifts (worst case)");

        Random rnd = new Random(7);
        boolean allOk = true, invOk = true, binOk = true;
        for (int t = 0; t < 2000; t++) {
            int[] x = rnd.ints(rnd.nextInt(60), -50, 50).toArray();
            int[] expected = x.clone(); Arrays.sort(expected);
            long inv = inversions(x);
            int[] y = x.clone(); shifts = 0; sort(y);
            if (!Arrays.equals(y, expected)) allOk = false;
            if (shifts != inv) invOk = false;
            int[] z = x.clone(); binaryInsertionSort(z);
            if (!Arrays.equals(z, expected)) binOk = false;
        }
        check(allOk, "2000 random arrays (incl. empty, duplicates, negatives) match Arrays.sort");
        check(invOk, "number of shifts == number of inversions on every array");
        check(binOk, "binary insertion sort also correct");

        // Stability: sort by key only; equal keys must keep increasing originalIndex
        Item[] items = new Item[500];
        for (int i = 0; i < items.length; i++) items[i] = new Item(rnd.nextInt(10), i);
        sort(items, Comparator.comparingInt(Item::key));
        boolean stable = true;
        for (int i = 1; i < items.length; i++) {
            if (items[i - 1].key() > items[i].key()) stable = false;
            if (items[i - 1].key() == items[i].key() && items[i - 1].originalIndex() > items[i].originalIndex()) stable = false;
        }
        check(stable, "stable: equal keys keep their original order");
    }
}
```

**Output:**

```
before: [5, 2, 4, 6, 1, 3]
after:  [1, 2, 3, 4, 5, 6]   comparisons=12 shifts=9
ok   CLRS example sorted
ok   already sorted: n-1 = 999 comparisons, 0 shifts (best case)
ok   reverse sorted: n(n-1)/2 = 499500 shifts (worst case)
ok   2000 random arrays (incl. empty, duplicates, negatives) match Arrays.sort
ok   number of shifts == number of inversions on every array
ok   binary insertion sort also correct
ok   stable: equal keys keep their original order
```

**Dry run check:** [5, 2, 4, 6, 1, 3] has 9 inversions: (5,2), (5,4), (5,1), (5,3), (2,1), (4,1), (4,3), (6,1), (6,3). The program made exactly **9 shifts**, matching the trace in Section C (1 + 1 + 0 + 4 + 3).

**Java-specific details**
- `a[i] <= key → break` is the Java form of CLRS's `A[i] > key` loop condition. Using `<` instead would shift equal elements and **break stability**.
- `System.arraycopy` moves a block in one native call, much faster than a manual loop. It's used in binary insertion sort. The number of moves is unchanged, but the constant factor is much smaller.
- For objects, use the generic version with a `Comparator`. `Comparator.comparingInt(Item::key)` avoids boxing.

**Common mistakes**
1. Checking `a[i] > key` *before* `i >= 0`, which gives `ArrayIndexOutOfBoundsException` at i = −1. (Java's `&&` short-circuits, so put `i >= 0` first.)
2. Writing the key at `a[i]` instead of `a[i + 1]`.
3. Swapping on every step instead of shifting (three writes instead of one; correct but slower).
4. Starting the outer loop at j = 0.

---

## F. Time complexity

Let tⱼ = the number of times the `while` test on line 5 runs for a given j (shifts + 1, or exactly the shifts if it stops at i = 0).

| Line | Cost | Times |
|---|---|---|
| 1 | c₁ | n |
| 2 | c₂ | n − 1 |
| 4 | c₄ | n − 1 |
| 5 | c₅ | Σⱼ₌₂ⁿ tⱼ |
| 6 | c₆ | Σⱼ₌₂ⁿ (tⱼ − 1) |
| 7 | c₇ | Σⱼ₌₂ⁿ (tⱼ − 1) |
| 8 | c₈ | n − 1 |

**T(n) = c₁n + (c₂ + c₄ + c₈)(n − 1) + c₅Σtⱼ + (c₆ + c₇)Σ(tⱼ − 1)**

**Best case: already sorted.** Each `while` test fails immediately, so tⱼ = 1:
T(n) = (c₁ + c₂ + c₄ + c₅ + c₈)n − (c₂ + c₄ + c₅ + c₈) = an + b, which is **Θ(n)**. Exactly n − 1 comparisons.

**Worst case: reverse sorted.** Each key moves all the way to the front, so tⱼ = j:
Σⱼ₌₂ⁿ j = n(n + 1)/2 − 1 and Σⱼ₌₂ⁿ (j − 1) = n(n − 1)/2.
That gives T(n) = an² + bn + c, which is **Θ(n²)**. Exactly n(n − 1)/2 shifts.

**Average case: random permutation.** A random pair (i, j) is inverted with probability ½, so the expected number of inversions is ½ · n(n − 1)/2 = **n(n − 1)/4**. On average each key shifts past half of the sorted prefix, giving tⱼ ≈ j/2. That's still **Θ(n²)**, about half the worst case.

**The exact characterisation:** insertion sort runs in **Θ(n + I)**, where I is the number of **inversions**. Each shift removes exactly one inversion, which the test above verifies. So for *nearly sorted* input with I = O(n), it's linear. That's why it's **adaptive**, and why hybrid sorts use it.

**Binary insertion sort** finds the position in O(log j) comparisons, giving Θ(n log n) comparisons in total, but **moves are still Θ(n²)**, so the time stays Θ(n²). It's useful when comparisons are expensive (long strings) and moves are cheap.

## G. Space complexity

- **Auxiliary:** one `key` and one index i, so **Θ(1)**. It's **in place**.
- **Recursion:** none (iterative).
- It's the same in every case.

## H. Complexity summary

| Algorithm | Best time | Average time | Worst time | Auxiliary space | Stable? | In-place? |
|---|---|---|---|---|---|---|
| Insertion sort | Θ(n) | Θ(n²) | Θ(n²) | Θ(1) | ✅ Yes | ✅ Yes |
| Binary insertion sort | Θ(n log n) comparisons, Θ(n) moves | Θ(n²) | Θ(n²) | Θ(1) | ✅ Yes | ✅ Yes |

Comparison-based ✅ · adaptive ✅ · online ✅

## I. Advantages, limitations, and comparisons

**Advantages**
- Simple, with tiny constant factors and no recursion. **The fastest sort for n ≲ 20**.
- **Adaptive:** Θ(n) on sorted or nearly sorted data.
- **Stable, in place and online.**

**Limitations**
- Θ(n²) on random or reversed data, which is useless above about 10⁴ elements.

**Comparison**

| vs | Insertion wins when… | The other wins when… |
|---|---|---|
| [Bubble sort](bubble-sort.md) | Always in practice: fewer writes (shifts instead of swaps), same Θ | — |
| [Selection sort](selection-sort.md) | The data is nearly sorted (Θ(n) vs always Θ(n²)) | Writes are expensive (selection does only n − 1 swaps) |
| [Merge sort](merge-sort.md) / [quicksort](../quicksort/quicksort-basics.md) | n is small or the data is nearly sorted | n is large |

**Practical use:** TimSort finds natural runs and extends short ones to about 32–64 elements with binary insertion sort. Introsort and Java's dual-pivot quicksort switch to insertion sort for subarrays below a small threshold.

**Interview follow-ups:** "Why is it used in hybrid sorts?", "Sort a linked list with insertion sort" (LeetCode 147), "What's its complexity on a k-sorted array, where every element is at most k positions from its place?" (Θ(nk)).

---

## J. Practice

**Beginner**
1. Trace insertion sort on [31, 41, 59, 26, 41, 58] (CLRS Exercise 2.1-1).
2. Rewrite it to sort in **descending** order (CLRS Exercise 2.1-2).
3. Count the comparisons on [1, 3, 2, 4, 5].

**Intermediate**
4. Prove that the number of shifts equals the number of inversions.
5. Implement insertion sort on a singly linked list.
6. Write the recursive version: sort A[1 .. n − 1], then insert A[n]. Give its recurrence (CLRS Exercise 2.3-4).

**Advanced**
7. Show that insertion sort on a k-sorted array runs in O(nk), and give a heap-based O(n log k) alternative.
8. Explain why binary insertion sort doesn't improve the worst-case *time*, and when it still helps.

**Interview questions**

<details><summary>Q1. When would you choose insertion sort over quicksort?</summary>

For small arrays (about n < 20), where its low overhead beats quicksort's recursion. For nearly sorted data, where it runs in Θ(n + inversions). And when you need a stable, in-place, online sort. That's why production sorts (TimSort, introsort) use it internally.
</details>

<details><summary>Q2. Is insertion sort stable? Why?</summary>

Yes. An element only moves left past elements that are strictly **greater** than it. Equal elements are never passed, so their relative order is preserved. Changing the comparison to `≥` would break stability.
</details>

<details><summary>Q3. What's the best case, and what input achieves it?</summary>

An already sorted array. Each key is compared once with its left neighbour and never shifted: n − 1 comparisons, Θ(n) time.
</details>

**Worked problem: sort a k-sorted array (each element at most k positions from its sorted place).** Insertion sort: each element shifts at most k times, so O(nk). Better: keep a min-heap of the next k + 1 elements and repeatedly extract the minimum into the output, giving O(n log k). See [Priority queues](../heapsort/priority-queues.md).

**Coding problems**
- LeetCode 147 · Insertion Sort List *(verify link)*
- LeetCode 912 · Sort an Array (it times out with insertion sort for large n, which is a useful lesson) *(verify link)*
- GeeksforGeeks · Sort a nearly sorted (k-sorted) array *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. Insert each element into the sorted prefix by shifting larger elements right.
2. Invariant: A[1..j−1] is the original first j − 1 elements, sorted.
3. Best Θ(n) (sorted input), worst and average Θ(n²). Precisely Θ(n + inversions).
4. Stable, in place, adaptive and online.
5. Used inside TimSort and introsort for small subarrays.

**Formulas**
- Worst-case shifts: n(n − 1)/2
- Average shifts: n(n − 1)/4
- Best-case comparisons: n − 1

**Common mistakes**
- Checking `a[i] > key` before `i >= 0`.
- Using `>=`, which breaks stability.
- Thinking binary insertion sort is O(n log n) overall.

**Quiz**
1. How many shifts does insertion sort make on [4, 3, 2, 1]?
2. What is the running time on an array with only 5 inversions, n = 10⁶?
3. Why is the outer loop index started at 1 in Java but 2 in CLRS?
4. Does binary search make insertion sort Θ(n log n)?

**Answers**

<details><summary>Show answers</summary>

1. 6, which is n(n − 1)/2 for n = 4 (every pair is inverted).
2. Θ(n + 5) = Θ(n), about 10⁶ steps.
3. CLRS arrays are 1-indexed and Java arrays are 0-indexed. Both start at the **second** element.
4. No. Comparisons drop to Θ(n log n), but shifting is still Θ(n²) moves.
</details>

**Related topics:** [Bubble sort](bubble-sort.md) · [Selection sort](selection-sort.md) · [Merge sort](merge-sort.md) · [Inversions in divide and conquer](../../00-Foundations/divide-and-conquer.md#example-2-counting-inversions) · [Sorting overview](../README.md)
