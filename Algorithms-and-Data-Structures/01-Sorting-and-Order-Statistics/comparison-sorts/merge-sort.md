# Merge Sort

> **CLRS:** Chapter 2.3 (Designing algorithms: the divide-and-conquer approach) · **Status:** ✅ Written · **Prerequisites:** [Divide and conquer](../../00-Foundations/divide-and-conquer.md), [Recurrence relations](../../00-Foundations/recurrence-relations.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

**Merge sort** sorts by **divide and conquer**: split the array into two halves, sort each half recursively, then **merge** the two sorted halves into one sorted array.

**What problem does it solve?** It sorts in **Θ(n log n) time in every case** (best, average and worst), and it's **stable**. It beats every Θ(n²) sort for large n and never degrades the way quicksort can.

**Where it's used in real software**
- **Java `Collections.sort`, `List.sort` and `Arrays.sort(Object[])`** use TimSort, a merge sort variant. Merge sort is chosen because object sorts must be **stable**.
- **External sorting:** databases sort data larger than RAM by sorting chunks, writing them to disk, and doing a k-way merge (`ORDER BY` on huge tables, `sort` on large files).
- **Sorting linked lists:** merge sort needs only O(1) extra space on lists, since no random access is required.
- **Parallel sorting** (`Arrays.parallelSort`, MapReduce shuffle): the halves are independent.
- **Counting inversions**, and other "merge-and-count" problems.

**Prerequisites:** recursion, divide and conquer, and the Master Theorem.

---

## B. Intuition

**Analogy: merging two sorted piles of exam papers.** Two teachers each hand you a pile sorted by roll number. To combine them, look at the top paper of each pile, take the smaller one, and repeat. You never need to look below the top cards, so merging two piles of total size n takes just n steps.

**Merge sort applies this recursively.** How do you get two sorted piles? Split your pile in half and ask two helpers to sort each half the same way. Eventually each helper holds a single paper, which is already sorted. Then the merges happen on the way back up.

**Why Θ(n log n):** there are log₂ n levels of splitting, and at each level the merges handle all n elements once. That's n work × log n levels.

---

## C. How it works internally

**Input:** A = [38, 27, 43, 3, 9, 82, 10].

### Divide phase (top-down splitting)

```
                     [38 27 43 3 9 82 10]
                    /                    \
           [38 27 43 3]                [9 82 10]
           /          \                /        \
       [38 27]      [43 3]        [9 82]      [10]
       /    \       /    \        /    \
    [38]   [27]  [43]   [3]     [9]   [82]
```

### Conquer and combine (merging back up)

```
    [38] [27] → [27 38]     [43] [3] → [3 43]     [9] [82] → [9 82]     [10]
         [27 38] + [3 43] → [3 27 38 43]          [9 82] + [10] → [9 10 82]
                  [3 27 38 43] + [9 10 82] → [3 9 10 27 38 43 82]
```

### One merge in detail: L = [3, 27, 38, 43], R = [9, 10, 82]

| Step | L pointer | R pointer | Compare | Take | Output so far |
|---|---|---|---|---|---|
| 1 | 3 | 9 | 3 ≤ 9 | 3 | [3] |
| 2 | 27 | 9 | 27 ≤ 9? no | 9 | [3, 9] |
| 3 | 27 | 10 | no | 10 | [3, 9, 10] |
| 4 | 27 | 82 | 27 ≤ 82 | 27 | [3, 9, 10, 27] |
| 5 | 38 | 82 | yes | 38 | [3, 9, 10, 27, 38] |
| 6 | 43 | 82 | yes | 43 | [3, 9, 10, 27, 38, 43] |
| 7 | (empty) | 82 | — | copy the rest: 82 | [3, 9, 10, 27, 38, 43, 82] |

6 comparisons for 7 elements. **A merge of n elements makes at most n − 1 comparisons.**

**Stability comes from one detail:** on a tie (L[i] = R[j]), take from **L** (use `≤`). Then equal elements from the left half, which came earlier in the input, stay first.

**Edge cases**

| Input | Behaviour |
|---|---|
| n = 0 or 1 | base case, return immediately |
| odd n | halves of size ⌈n/2⌉ and ⌊n/2⌋. Works the same. |
| already sorted | still Θ(n log n) in the textbook version. With the "skip if a[mid] ≤ a[mid+1]" check it becomes Θ(n). |
| all equal | Θ(n log n), and stable |
| very large n | the recursion depth is only ⌈log₂ n⌉ + 1, about 31 for n = 10⁹, so there's no stack-overflow risk |

---

## D. Algorithm and pseudocode

CLRS version (1-indexed, using **sentinels** ∞ so the merge loop needs no "is this half empty?" checks):

```
MERGE(A, p, q, r)                       ▷ merges sorted A[p..q] and A[q+1..r]
1   n₁ ← q − p + 1
2   n₂ ← r − q
3   create arrays L[1..n₁ + 1] and R[1..n₂ + 1]
4   for i ← 1 to n₁
5       L[i] ← A[p + i − 1]
6   for j ← 1 to n₂
7       R[j] ← A[q + j]
8   L[n₁ + 1] ← ∞                       ▷ sentinels: never chosen while real elements remain
9   R[n₂ + 1] ← ∞
10  i ← 1
11  j ← 1
12  for k ← p to r
13      if L[i] ≤ R[j]                  ▷ "≤" makes it stable
14          A[k] ← L[i]
15          i ← i + 1
16      else A[k] ← R[j]
17          j ← j + 1

MERGE-SORT(A, p, r)
1  if p < r                             ▷ at least two elements
2      q ← ⌊(p + r)/2⌋                  ▷ divide
3      MERGE-SORT(A, p, q)              ▷ conquer left
4      MERGE-SORT(A, q + 1, r)          ▷ conquer right
5      MERGE(A, p, q, r)                ▷ combine
```

**Initial call:** MERGE-SORT(A, 1, n).

### Loop invariant for MERGE (CLRS §2.3.1)

> At the start of each iteration of the `for` loop (lines 12–17), A[p..k − 1] contains the k − p **smallest** elements of L[1..n₁ + 1] and R[1..n₂ + 1], in sorted order. Moreover, L[i] and R[j] are the smallest elements of their arrays that haven't been copied back into A.

- **Initialization:** k = p, so A[p..p − 1] is empty, and i = j = 1 point at the smallest elements of L and R. ✓
- **Maintenance:** suppose L[i] ≤ R[j]. Then L[i] is the smallest element not yet copied (it's the smallest in L, and ≤ the smallest in R). Copying it to A[k] makes A[p..k] the k − p + 1 smallest, in sorted order. Incrementing i keeps L[i] as the smallest remaining in L. The case R[j] < L[i] is symmetric. ✓
- **Termination:** k = r + 1. A[p..r] holds the r − p + 1 smallest of all n₁ + n₂ + 2 elements, which is every element except the two sentinels, in sorted order. ✓

**Correctness of MERGE-SORT** follows by strong induction on n = r − p + 1: the base case n ≤ 1 is sorted. Otherwise both recursive calls sort strictly smaller ranges (by the inductive hypothesis), and MERGE combines two sorted ranges correctly (by the invariant).

### Bottom-up (iterative) merge sort

Merge runs of width 1, then 2, then 4, … with no recursion:

```
MERGE-SORT-BOTTOM-UP(A, n)
1  width ← 1
2  while width < n
3      for lo ← 1 to n step 2·width
4          MERGE(A, lo, min(lo + width − 1, n), min(lo + 2·width − 1, n))
5      width ← 2·width
```

---

## E. Implementation

```java
import java.util.*;

/** Merge sort: top-down (CLRS-style, but without sentinels), bottom-up, a generic stable version, and checks. */
public class MergeSort {

    static long comparisons;

    // ---------- Top-down, one reusable buffer ----------
    public static void sort(int[] a) {
        if (a.length < 2) return;
        int[] buf = new int[a.length];                   // allocated ONCE, reused by every merge
        sort(a, buf, 0, a.length - 1);
    }

    private static void sort(int[] a, int[] buf, int lo, int hi) {
        if (lo >= hi) return;                            // 0 or 1 element: already sorted
        int mid = lo + (hi - lo) / 2;                    // divide
        sort(a, buf, lo, mid);                           // conquer left
        sort(a, buf, mid + 1, hi);                       // conquer right
        merge(a, buf, lo, mid, hi);                      // combine
    }

    /** Merges sorted a[lo..mid] and a[mid+1..hi]. Java has no infinity for int, so we test bounds instead of using sentinels. */
    static void merge(int[] a, int[] buf, int lo, int mid, int hi) {
        System.arraycopy(a, lo, buf, lo, hi - lo + 1);
        int i = lo, j = mid + 1;
        for (int k = lo; k <= hi; k++) {
            if (i > mid)              a[k] = buf[j++];   // left exhausted
            else if (j > hi)          a[k] = buf[i++];   // right exhausted
            else {
                comparisons++;
                if (buf[i] <= buf[j]) a[k] = buf[i++];   // '<=' -> stable
                else                  a[k] = buf[j++];
            }
        }
    }

    // ---------- Optimised top-down: insertion-sort cutoff + skip already-ordered merges ----------
    static final int CUTOFF = 16;
    public static void sortOptimised(int[] a) {
        int[] buf = new int[a.length];
        sortOpt(a, buf, 0, a.length - 1);
    }
    private static void sortOpt(int[] a, int[] buf, int lo, int hi) {
        if (hi - lo + 1 <= CUTOFF) { insertion(a, lo, hi); return; }
        int mid = lo + (hi - lo) / 2;
        sortOpt(a, buf, lo, mid);
        sortOpt(a, buf, mid + 1, hi);
        comparisons++;
        if (a[mid] <= a[mid + 1]) return;               // halves already in order: no merge needed
        merge(a, buf, lo, mid, hi);
    }
    private static void insertion(int[] a, int lo, int hi) {
        for (int j = lo + 1; j <= hi; j++) {
            int key = a[j], i = j - 1;
            while (i >= lo) {
                comparisons++;
                if (a[i] <= key) break;
                a[i + 1] = a[i];
                i--;
            }
            a[i + 1] = key;
        }
    }

    // ---------- Bottom-up (no recursion) ----------
    public static void sortBottomUp(int[] a) {
        int n = a.length;
        int[] buf = new int[n];
        for (int width = 1; width < n; width *= 2)
            for (int lo = 0; lo < n - width; lo += 2 * width)
                merge(a, buf, lo, lo + width - 1, Math.min(lo + 2 * width - 1, n - 1));
    }

    // ---------- Generic stable version for objects ----------
    @SuppressWarnings("unchecked")
    public static <T> void sort(T[] a, Comparator<? super T> cmp) {
        T[] buf = (T[]) new Object[a.length];
        sortG(a, buf, 0, a.length - 1, cmp);
    }
    private static <T> void sortG(T[] a, T[] buf, int lo, int hi, Comparator<? super T> cmp) {
        if (lo >= hi) return;
        int mid = lo + (hi - lo) / 2;
        sortG(a, buf, lo, mid, cmp);
        sortG(a, buf, mid + 1, hi, cmp);
        System.arraycopy(a, lo, buf, lo, hi - lo + 1);
        int i = lo, j = mid + 1;
        for (int k = lo; k <= hi; k++) {
            if (i > mid) a[k] = buf[j++];
            else if (j > hi) a[k] = buf[i++];
            else if (cmp.compare(buf[i], buf[j]) <= 0) a[k] = buf[i++];
            else a[k] = buf[j++];
        }
    }

    /** Worst-case comparisons of top-down merge sort (CLRS Exercise 2.3-3 style): n*ceil(lg n) - 2^ceil(lg n) + 1. */
    static long worstCaseBound(int n) {
        if (n < 2) return 0;
        int c = 32 - Integer.numberOfLeadingZeros(n - 1);   // ceil(log2 n)
        return (long) n * c - (1L << c) + 1;
    }

    record Item(int key, int originalIndex) {}

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        int[] a = {38, 27, 43, 3, 9, 82, 10};
        comparisons = 0;
        sort(a);
        System.out.println("sorted: " + Arrays.toString(a) + "  comparisons=" + comparisons);
        check(Arrays.equals(a, new int[]{3, 9, 10, 27, 38, 43, 82}), "example sorted");

        Random rnd = new Random(5);
        boolean td = true, bu = true, opt = true, bound = true;
        for (int t = 0; t < 3000; t++) {
            int[] x = rnd.ints(rnd.nextInt(200), -100, 100).toArray();
            int[] exp = x.clone(); Arrays.sort(exp);
            int[] y = x.clone(); comparisons = 0; sort(y);
            if (!Arrays.equals(y, exp)) td = false;
            if (comparisons > worstCaseBound(x.length)) bound = false;
            int[] z = x.clone(); sortBottomUp(z); if (!Arrays.equals(z, exp)) bu = false;
            int[] w = x.clone(); sortOptimised(w); if (!Arrays.equals(w, exp)) opt = false;
        }
        check(td, "top-down: 3000 random arrays match Arrays.sort");
        check(bu, "bottom-up: 3000 random arrays match Arrays.sort");
        check(opt, "optimised (cutoff + skip): 3000 random arrays match");
        check(bound, "comparisons never exceed n*ceil(lg n) - 2^ceil(lg n) + 1");

        int n = 1 << 16;
        int[] sorted = new int[n]; for (int i = 0; i < n; i++) sorted[i] = i;
        comparisons = 0; sort(sorted.clone());
        long plain = comparisons;
        comparisons = 0; sortOptimised(sorted.clone());
        System.out.println("already-sorted n=65536: textbook comparisons=" + plain + ", optimised comparisons=" + comparisons);
        check(plain >= (long) n / 2 * 16, "textbook version still does ~ (n/2) lg n comparisons on sorted input");
        check(comparisons < 2L * n, "skip-merge optimisation makes sorted input linear");

        Item[] items = new Item[2000];
        for (int i = 0; i < items.length; i++) items[i] = new Item(rnd.nextInt(20), i);
        sort(items, Comparator.comparingInt(Item::key));
        boolean stable = true;
        for (int i = 1; i < items.length; i++)
            if (items[i - 1].key() > items[i].key()
                || (items[i - 1].key() == items[i].key() && items[i - 1].originalIndex() > items[i].originalIndex())) stable = false;
        check(stable, "generic merge sort is stable");

        int big = 1_000_000;
        int[] large = rnd.ints(big).toArray(); int[] exp = large.clone();
        sort(large);
        Arrays.sort(exp);
        check(Arrays.equals(large, exp), "1,000,000 random ints sorted correctly");
    }
}
```

**Output:**

```
sorted: [3, 9, 10, 27, 38, 43, 82]  comparisons=14
ok   example sorted
ok   top-down: 3000 random arrays match Arrays.sort
ok   bottom-up: 3000 random arrays match Arrays.sort
ok   optimised (cutoff + skip): 3000 random arrays match
ok   comparisons never exceed n*ceil(lg n) - 2^ceil(lg n) + 1
already-sorted n=65536: textbook comparisons=524288, optimised comparisons=65535
ok   textbook version still does ~ (n/2) lg n comparisons on sorted input
ok   skip-merge optimisation makes sorted input linear
ok   generic merge sort is stable
ok   1,000,000 random ints sorted correctly
```

**Dry run of the example:** the program splits exactly as in the tree in Section C. The merges cost [38]+[27] = 1 comparison, [43]+[3] = 1, [27 38]+[3 43] = 3 (3 vs 27, then 27 vs 43, then 38 vs 43, then copy 43), [9]+[82] = 1, [9 82]+[10] = 2, and the final merge = 6 (the table in Section C). The total is **14 comparisons**, which equals the worst-case bound n⌈lg n⌉ − 2^⌈lg n⌉ + 1 = 7·3 − 8 + 1 = 14 for n = 7. This input happens to be a worst case.

**Java-specific details**
- **No sentinels:** Java `int` has no ∞, and using `Integer.MAX_VALUE` breaks if the data contains MAX_VALUE. Explicit bounds checks (`i > mid`, `j > hi`) are the standard replacement.
- **One buffer, allocated once.** Allocating a new array inside every `merge` call creates about n log n garbage elements.
- **Generic arrays:** `(T[]) new Object[n]` is the usual unchecked cast. It's safe here because the buffer never escapes.
- `Arrays.sort(Object[])` is TimSort and **guarantees stability**. `Arrays.sort(int[])` is dual-pivot quicksort, which isn't stable, but that can't matter for primitives (equal ints are indistinguishable).

**Common mistakes**
1. `mid = (lo + hi) / 2` can overflow for very large indices. Use `lo + (hi - lo) / 2`.
2. Using `<` in the merge comparison, which breaks stability.
3. Forgetting to copy the leftover elements of one half.
4. Recursing on `(lo, mid - 1)` and `(mid, hi)` inconsistently, which causes infinite recursion when hi = lo + 1.

---

## F. Time complexity

### Recurrence

| Step | Cost |
|---|---|
| Divide (compute mid) | Θ(1) |
| Conquer (two halves) | 2T(n/2) |
| Combine (MERGE of n elements) | Θ(n): each element is copied in and written back once, with ≤ n − 1 comparisons |

> **T(n) = 2T(n/2) + Θ(n)**, T(1) = Θ(1)

**Master Theorem:** a = 2, b = 2, n^(log₂ 2) = n, f(n) = Θ(n), which is Case 2, so **Θ(n log n)**.

**Recursion tree:** log₂ n + 1 levels, each doing cn work in total, so cn log₂ n + cn = Θ(n log n). (The full picture is in [Recurrence relations](../../00-Foundations/recurrence-relations.md#method-1-recursion-tree-best-for-building-intuition).)

### Best, average and worst case are all Θ(n log n)

The algorithm always splits and always merges, whatever the input, and every merge of size m costs Θ(m) in copying regardless of its comparisons. The **number of comparisons** does vary with the input:
- **Worst case:** n⌈lg n⌉ − 2^⌈lg n⌉ + 1 comparisons (each merge of m elements uses m − 1).
- **Best case (sorted input):** about (n/2) lg n comparisons: each merge stops comparing once one half runs out, which happens after m/2 comparisons. The test above measured exactly 524,288 = (65536/2) × 16.
- With the **skip-merge** check (`a[mid] ≤ a[mid+1]`), sorted input takes Θ(n), measured as 65,535 = n − 1 comparisons. That's the idea behind TimSort's adaptivity.

**Comparison with the lower bound:** any comparison sort needs ⌈lg(n!)⌉ ≈ n lg n − 1.44n comparisons in the worst case ([Lower bounds](../linear-time-sorting/lower-bounds.md)). Merge sort uses at most n lg n − n + 1, so it's **within about 0.44n comparisons of optimal**. It's asymptotically optimal.

## G. Space complexity

| Component | Size |
|---|---|
| Merge buffer | **Θ(n)** (one array, reused) |
| Recursion stack | Θ(log n), depth ⌈lg n⌉ + 1 |
| Bottom-up version | Θ(n) buffer, Θ(1) stack |
| **Total auxiliary** | **Θ(n)**, so **not in place** |

**On linked lists:** merging just relinks nodes, so you only need Θ(log n) stack, or **Θ(1)** with bottom-up merging. That makes merge sort the standard choice for sorting linked lists.
**In-place array merge sort** exists (block merge sort, WikiSort), but it's complex and has larger constants.

## H. Complexity summary

| Algorithm | Best time | Average time | Worst time | Auxiliary space | Stable? | In-place? |
|---|---|---|---|---|---|---|
| Merge sort (top-down) | Θ(n log n) | Θ(n log n) | Θ(n log n) | Θ(n) | ✅ Yes | ❌ No |
| Merge sort + skip check | Θ(n) | Θ(n log n) | Θ(n log n) | Θ(n) | ✅ Yes | ❌ No |
| Merge sort on a linked list | Θ(n log n) | Θ(n log n) | Θ(n log n) | Θ(1) (bottom-up) | ✅ Yes | ✅ (relinking) |

Comparison-based ✅ · worst-case optimal ✅ · parallelisable ✅

## I. Advantages, limitations, and comparisons

**Advantages**
- **Guaranteed Θ(n log n)** with no bad inputs, which makes it predictable for real-time and adversarial settings.
- **Stable.**
- Excellent for **linked lists** and **external/disk** sorting (sequential access only).
- Easy to parallelise.

**Limitations**
- **Θ(n) extra memory** for arrays.
- Usually **slower than quicksort** on in-memory primitive arrays, because of the copying and somewhat worse cache behaviour.
- Not adaptive unless optimised (TimSort fixes this).

| vs | Merge sort wins | The other wins |
|---|---|---|
| [Quicksort](../quicksort/quicksort-basics.md) | Worst case, stability, linked lists, external sorting | Average speed, memory (in place) |
| [Heapsort](../heapsort/heapsort.md) | Stability, speed in practice (cache) | Memory (Θ(1)) |
| [Insertion sort](insertion-sort.md) | Large n | Small or nearly sorted n |

**Practical engineering:** cut off to insertion sort below about 16 elements, skip the merge when the halves are already ordered, and alternate the roles of input and buffer to avoid copying. **TimSort** combines all of these and detects existing runs, which gives O(n) on sorted data and O(n log n) worst case.

**Interview follow-ups:** "Sort a linked list in O(n log n)" (LeetCode 148), "Merge k sorted lists" (a heap, or pairwise merging), "Count inversions", "How would you sort 100 GB with 1 GB of RAM?" (external merge sort: about 100 sorted chunks, then a k-way merge with a heap).

---

## J. Practice

**Beginner**
1. Trace merge sort on [3, 41, 52, 26, 38, 57, 9, 49] (CLRS Figure 2.4).
2. Merge [1, 4, 7] and [2, 3, 8, 9] by hand and count the comparisons.
3. Why does merge sort's recursion depth equal ⌈log₂ n⌉ + 1?

**Intermediate**
4. Rewrite MERGE without sentinels (CLRS Exercise 2.3-2).
5. Implement merge sort on a singly linked list.
6. Prove T(n) = n lg n for n a power of 2, by induction (CLRS Exercise 2.3-3).

**Advanced**
7. Implement a k-way merge of k sorted arrays in O(N log k) using a min-heap.
8. Design external merge sort for a 100 GB file with 1 GB of memory. How many passes does it need?

**Interview questions**

<details><summary>Q1. Why is merge sort used for sorting objects in Java but not primitives?</summary>

Object sorts must be **stable**: equal objects can still be distinguishable, and users depend on multi-key sorting. Merge sort (TimSort) is stable. For primitives, stability is invisible, so Java uses the faster, in-place dual-pivot quicksort.
</details>

<details><summary>Q2. What's the space complexity of merge sort, and can it be reduced?</summary>

Θ(n) auxiliary for the merge buffer, plus a Θ(log n) stack. On linked lists it drops to Θ(1) (with bottom-up merging) because merging just relinks nodes. In-place array variants exist but are complicated and slower in practice.
</details>

<details><summary>Q3. Is merge sort's best case faster than its worst case?</summary>

Not asymptotically. The textbook version is Θ(n log n) on every input, although sorted input uses about half the comparisons. Adding the check "if a[mid] ≤ a[mid+1], skip the merge" makes sorted input Θ(n).
</details>

**Worked problem: sort a linked list (LeetCode 148).** Find the middle with slow and fast pointers, cut the list, recursively sort both halves, and merge them by relinking. T(n) = 2T(n/2) + Θ(n) gives Θ(n log n), with Θ(log n) stack. Bottom-up merging of sublists of size 1, 2, 4, … achieves Θ(1) extra space.

**Coding problems**
- LeetCode 912 · Sort an Array *(verify link)*
- LeetCode 148 · Sort List *(verify link)*
- LeetCode 88 · Merge Sorted Array *(verify link)*
- LeetCode 23 · Merge k Sorted Lists *(verify link)*
- LeetCode 315 · Count of Smaller Numbers After Self *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. Divide in half, sort each half recursively, merge in Θ(n).
2. T(n) = 2T(n/2) + Θ(n) = **Θ(n log n) in every case**.
3. **Stable** (take from the left on ties). **Θ(n) extra space** on arrays.
4. Within about 0.44n comparisons of the theoretical minimum, so worst-case optimal.
5. The basis of TimSort (Java object sorts) and external sorting.

**Formulas**
- Merge of m elements: ≤ m − 1 comparisons
- Worst-case total: n⌈lg n⌉ − 2^⌈lg n⌉ + 1
- Recursion depth: ⌈lg n⌉ + 1

**Common mistakes**
- Allocating a buffer in every merge call.
- `<` instead of `≤` (instability).
- Overflow in the mid calculation.
- Forgetting the leftover copy.

**Quiz**
1. What's the recurrence for merge sort?
2. Why is merge sort stable?
3. Space complexity on arrays? On linked lists?
4. How many levels does the recursion tree have for n = 1024?

**Answers**

<details><summary>Show answers</summary>

1. T(n) = 2T(n/2) + Θ(n).
2. On ties the merge takes the element from the left half first, preserving the original order of equal keys.
3. Θ(n) on arrays (plus a Θ(log n) stack). Θ(1) on linked lists with bottom-up merging (Θ(log n) if recursive).
4. lg 1024 + 1 = 11.
</details>

**Related topics:** [Divide and conquer](../../00-Foundations/divide-and-conquer.md) · [Counting inversions](../../00-Foundations/divide-and-conquer.md#example-2-counting-inversions) · [Quicksort](../quicksort/quicksort-basics.md) · [Heapsort](../heapsort/heapsort.md) · [Lower bounds](../linear-time-sorting/lower-bounds.md) · [Priority queues (k-way merge)](../heapsort/priority-queues.md)
