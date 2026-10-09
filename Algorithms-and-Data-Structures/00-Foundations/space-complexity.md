# Space Complexity

> **CLRS:** discussed throughout Ch. 2–3 (e.g. "sorts in place") · **Status:** ✅ Written · **Prerequisites:** [Time complexity](time-complexity.md) · **Next:** [Asymptotic notation](asymptotic-notation.md)

---

## A. Introduction

**Space complexity** describes **how much memory an algorithm needs as a function of the input size n**.

**What problem does it solve?** Memory is limited: a phone app, an embedded device, a 4 GB container, or simply the JVM's default stack. Two algorithms with the same time complexity can differ hugely in memory. Merge sort needs Θ(n) extra memory and heapsort needs Θ(1), even though both run in Θ(n log n) time.

**Where it matters in real software**
- **Big data:** sorting a 500 GB file that doesn't fit in RAM needs *external* algorithms.
- **Recursion limits:** deep recursion causes Java's `StackOverflowError`.
- **Mobile and embedded:** in-place algorithms avoid allocating memory.
- **Caching and memoisation:** you deliberately spend memory to save time.

**Prerequisites:** time complexity, recursion, and a rough idea of the call stack.

---

## B. Intuition

Think of working on a desk:

- The **input** is the pile of papers you were given. You don't count it against yourself.
- **Auxiliary space** is the *extra* scratch paper you need while working.
- The **recursion stack** is a pile of sticky notes, one for each "I'll come back to this" (each pending function call).

An algorithm that only needs a pencil and a couple of sticky notes, no matter how big the pile is, uses **Θ(1) extra space**. That's **in place**. One that photocopies the whole pile needs **Θ(n)** extra space.

---

## C. How it works internally

### What counts as space

| Component | Description | Example |
|---|---|---|
| **Input space** | Memory holding the input itself | the array of n elements |
| **Auxiliary space** | Extra memory the algorithm allocates | merge sort's temporary array |
| **Recursion stack** | One **stack frame** per active call (parameters, locals, return address) | recursive binary search: ~log n frames |
| **Output space** | Memory for the result, if it's separate from the input | returning a new sorted copy |

> **Total space** = input + auxiliary (including the stack). When people say "space complexity" in interviews they almost always mean **auxiliary space**, and you should say so explicitly.

### How the recursion stack grows

Each call pushes a frame. A frame is popped when its call returns. The **maximum stack depth** at any moment determines stack space, **not** the total number of calls.

Example: recursive sum of [5, 3, 8]:

```
call sum(a, 0)           stack: [sum0]
  call sum(a, 1)         stack: [sum0, sum1]
    call sum(a, 2)       stack: [sum0, sum1, sum2]
      call sum(a, 3)     stack: [sum0, sum1, sum2, sum3]   ← max depth = n + 1
      return 0
    return 8 + 0
  return 3 + 8
return 5 + 11 = 16
```

Depth n + 1 means **Θ(n) stack space**, even though the iterative version needs Θ(1).

**Naive Fibonacci makes Θ(φⁿ) calls, but its stack depth is only n**, because only one path from root to leaf of the call tree is ever active at once. Its space is **Θ(n)** while its time is exponential.

### Stack depth of common recursive algorithms

| Algorithm | Max recursion depth | Why |
|---|---|---|
| Linear recursion (sum, factorial) | n | one call per element |
| Recursive binary search | ⌊log₂ n⌋ + 1 | range halves each call |
| Merge sort | ⌈log₂ n⌉ + 1 | both halves are size n/2, and they're processed one after the other |
| Quicksort, worst case (e.g. sorted input with last-element pivot) | n | partitions of size n − 1 and 0 |
| Quicksort, recursing on the **smaller** side + looping on the larger | ≤ log₂ n | the smaller side is ≤ n/2 |
| DFS on a graph | up to V | a long path graph |

### Memory in Java (64-bit JVM with compressed pointers, typical)

| Thing | Size |
|---|---|
| `int` | 4 bytes |
| `long`, `double` | 8 bytes |
| reference | 4 bytes (compressed) or 8 bytes |
| object header | 12–16 bytes |
| `int[n]` | ≈ 16 + 4n bytes |
| `Integer` object | ≈ 16 bytes (4 bytes of data + header + padding) |
| `ArrayList<Integer>` of n | ≈ 4n (refs) + 16n (`Integer` objects), **about 5× an `int[]`** |
| `HashMap` entry | ≈ 32–48 bytes per entry |
| default thread stack | typically 512 KB–1 MB (`-Xss` to change). Roughly 10⁴–10⁵ simple frames. |

Asymptotically `int[]` and `ArrayList<Integer>` are both Θ(n), but the **constant factor differs by about 5×**. That's why primitive arrays matter for large inputs.

---

## D. Algorithm and pseudocode

### Counting auxiliary space: a recipe

1. List every variable, array and data structure the algorithm **allocates** (not the input).
2. Express each one's size in terms of n.
3. Add the **maximum recursion depth × frame size**.
4. Take the maximum at any one moment (memory freed earlier can be reused).

### Example: three ways to reverse an array

```
REVERSE-COPY(A, n)                 ▷ Θ(n) auxiliary
1  B ← new array[1..n]
2  for i ← 1 to n
3      B[i] ← A[n − i + 1]
4  return B

REVERSE-IN-PLACE(A, n)             ▷ Θ(1) auxiliary: in place
1  i ← 1, j ← n
2  while i < j
3      exchange A[i] ↔ A[j]
4      i ← i + 1, j ← j − 1

REVERSE-RECURSIVE(A, i, j)         ▷ Θ(n) stack: n/2 frames
1  if i ≥ j then return
2  exchange A[i] ↔ A[j]
3  REVERSE-RECURSIVE(A, i + 1, j − 1)
```

All three take **Θ(n) time** but differ in space: Θ(n), Θ(1) and Θ(n) (stack).

**In-place** (CLRS definition for sorting): only a **constant number** of elements are stored outside the input array at any time. Insertion sort and heapsort are in place. Merge sort isn't (it needs Θ(n) auxiliary space). Quicksort is usually called in place: Θ(1) auxiliary *array* space plus a recursion stack of Θ(log n) expected and Θ(n) worst case.

---

## E. Implementation

The program measures **maximum recursion depth** for merge sort and naive quicksort, checks the formulas above, and shows a `StackOverflowError` from deep recursion compared with an iterative version that has no such limit.

```java
import java.util.*;

/** Measures recursion depth (stack space) and compares recursive vs iterative space use. */
public class SpaceComplexityDemo {

    static int depth, maxDepth;

    static void enter() { depth++; maxDepth = Math.max(maxDepth, depth); }
    static void leave() { depth--; }

    // Merge sort: depth should be ceil(log2 n) + 1
    static void mergeSort(int[] a, int[] tmp, int lo, int hi) {
        enter();
        if (lo < hi) {
            int mid = (lo + hi) >>> 1;
            mergeSort(a, tmp, lo, mid);
            mergeSort(a, tmp, mid + 1, hi);
            int i = lo, j = mid + 1, k = lo;
            while (i <= mid && j <= hi) tmp[k++] = a[i] <= a[j] ? a[i++] : a[j++];
            while (i <= mid) tmp[k++] = a[i++];
            while (j <= hi) tmp[k++] = a[j++];
            System.arraycopy(tmp, lo, a, lo, hi - lo + 1);
        }
        leave();
    }

    // Quicksort with last-element pivot: on sorted input depth grows to n
    static void quickSortNaive(int[] a, int lo, int hi) {
        enter();
        if (lo < hi) {
            int p = partition(a, lo, hi);
            quickSortNaive(a, lo, p - 1);
            quickSortNaive(a, p + 1, hi);
        }
        leave();
    }

    // Same quicksort, but recurse on the smaller side and loop on the larger: depth <= log2 n
    static void quickSortSmallFirst(int[] a, int lo, int hi) {
        enter();
        while (lo < hi) {
            int p = partition(a, lo, hi);
            if (p - lo < hi - p) { quickSortSmallFirst(a, lo, p - 1); lo = p + 1; }
            else                 { quickSortSmallFirst(a, p + 1, hi); hi = p - 1; }
        }
        leave();
    }

    static int partition(int[] a, int lo, int hi) {           // CLRS Lomuto partition
        int x = a[hi], i = lo - 1;
        for (int j = lo; j < hi; j++) if (a[j] <= x) { i++; int t = a[i]; a[i] = a[j]; a[j] = t; }
        int t = a[i + 1]; a[i + 1] = a[hi]; a[hi] = t;
        return i + 1;
    }

    static long sumRecursive(int[] a, int i) { return i == a.length ? 0 : a[i] + sumRecursive(a, i + 1); }
    static long sumIterative(int[] a) { long s = 0; for (int x : a) s += x; return s; }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) throws Exception {
        int n = 4096;                                      // 2^12
        int[] sorted = new int[n];
        for (int i = 0; i < n; i++) sorted[i] = i;

        maxDepth = 0; mergeSort(sorted.clone(), new int[n], 0, n - 1);
        System.out.println("merge sort          n=" + n + "  max depth = " + maxDepth + "  (log2 n + 1 = 13)");
        check(maxDepth == 13, "merge sort depth = log2 n + 1");

        maxDepth = 0; quickSortNaive(sorted.clone(), 0, n - 1);
        System.out.println("naive quicksort     sorted input  max depth = " + maxDepth + "  (= n)");
        check(maxDepth == n, "naive quicksort on sorted input recurses n deep");

        maxDepth = 0; quickSortSmallFirst(sorted.clone(), 0, n - 1);
        System.out.println("smaller-side-first  sorted input  max depth = " + maxDepth + "  (<= log2 n + 1 = 13)");
        check(maxDepth <= 13, "recursing on the smaller side keeps depth <= log2 n + 1");

        // Deep recursion: run in a thread with a small stack so the overflow is guaranteed.
        int[] big = new int[1_000_000];
        Arrays.fill(big, 1);
        boolean[] overflowed = {false};
        Thread t = new Thread(null, () -> {
            try { sumRecursive(big, 0); }
            catch (StackOverflowError e) { overflowed[0] = true; }
        }, "small-stack", 256 * 1024);
        t.start(); t.join();
        System.out.println("recursive sum of 1,000,000 elements -> StackOverflowError: " + overflowed[0]);
        System.out.println("iterative sum of 1,000,000 elements -> " + sumIterative(big));
        check(overflowed[0], "1,000,000-deep recursion overflows a 256 KB stack");
        check(sumIterative(big) == 1_000_000, "iterative version needs only O(1) extra space");
    }
}
```

**Output:**

```
merge sort          n=4096  max depth = 13  (log2 n + 1 = 13)
ok   merge sort depth = log2 n + 1
naive quicksort     sorted input  max depth = 4096  (= n)
ok   naive quicksort on sorted input recurses n deep
smaller-side-first  sorted input  max depth = 2  (<= log2 n + 1 = 13)
ok   recursing on the smaller side keeps depth <= log2 n + 1
recursive sum of 1,000,000 elements -> StackOverflowError: true
iterative sum of 1,000,000 elements -> 1000000
ok   1,000,000-deep recursion overflows a 256 KB stack
ok   iterative version needs only O(1) extra space
```

**Dry run (smaller-side-first on sorted input).** Each partition puts the pivot at the end, so the left side has p − lo = n − 1 elements and the right side is empty. The code recurses into the **empty** right side (depth 2) and *loops* on the large left side, so the stack never grows. This is the classic fix that bounds quicksort's stack to O(log n) even when its *time* is Θ(n²). See [Quicksort complexity analysis](../01-Sorting-and-Order-Statistics/quicksort/complexity-analysis.md).

**Java-specific notes**
- **Java doesn't do tail-call optimisation.** A recursive call in the last position still uses a frame, so converting to a loop is the only way to save the stack.
- **`StackOverflowError` is an `Error`, not an `Exception`.** Catching it is fine for a demo but not as normal control flow.
- You can enlarge the stack with `java -Xss64m` or `new Thread(group, task, name, stackSize)`. That's a workaround, not a fix.
- **Garbage collection:** temporary objects (like merge sort's arrays) count toward peak memory until they're collected. Allocate one buffer once and reuse it, as in `mergeSort(a, tmp, …)` above.

**Common mistakes**
- Saying "recursive binary search is O(1) space". It's O(log n) because of the stack.
- Allocating a new temporary array in **every** merge call. That's still Θ(n) peak space, but much slower, and it creates garbage.
- Counting the total number of calls instead of the maximum depth.

---

## F. Time complexity

Not the focus of this page. The examples above are: merge sort Θ(n log n), naive quicksort on sorted input Θ(n²), and both sums Θ(n). See [Time complexity](time-complexity.md).

**Time–space trade-offs** (spending memory to save time):

| Technique | Extra space | Time saved |
|---|---|---|
| Memoisation (DP) | Θ(#states) | exponential → polynomial |
| Hash set for lookups | Θ(n) | Θ(n) per lookup → Θ(1) expected |
| Prefix sums | Θ(n) | Θ(n) per range-sum query → Θ(1) |
| Counting sort | Θ(k) for the counts | beats the Θ(n log n) comparison bound |
| Precomputed tables (e.g. KMP prefix function) | Θ(m) | avoids re-scanning |

And the reverse (saving memory at the cost of time): in-place heapsort instead of merge sort, recomputing instead of caching, and streaming algorithms.

## G. Space complexity

| Example | Auxiliary space | Stack | Total extra |
|---|---|---|---|
| REVERSE-COPY | Θ(n) | Θ(1) | Θ(n) |
| REVERSE-IN-PLACE | Θ(1) | Θ(1) | Θ(1) |
| REVERSE-RECURSIVE | Θ(1) | Θ(n) | Θ(n) |
| Recursive binary search | Θ(1) | Θ(log n) | Θ(log n) |
| Merge sort | Θ(n) | Θ(log n) | Θ(n) |
| Quicksort (naive) | Θ(1) | Θ(log n) expected, Θ(n) worst | Θ(n) worst |
| Quicksort (smaller side first) | Θ(1) | Θ(log n) worst | Θ(log n) |
| Heapsort | Θ(1) | Θ(1) (iterative heapify) | Θ(1) |

## H. Complexity summary

See the sorting table in [complexity-comparison.md](../15-Practice-and-Revision/complexity-comparison.md#1-sorting-algorithms), which includes an auxiliary-space column for every algorithm.

## I. Advantages, limitations, and comparisons

- **In place** is good for memory and often for cache behaviour, but it usually costs **stability** (heapsort, quicksort) or simplicity.
- **Recursion** is elegant but carries a hidden O(depth) cost, and the stack is far smaller than the heap. For depth above about 10⁴ in Java, prefer iteration or an explicit `ArrayDeque` stack.
- **Asymptotics vs constants:** `ArrayList<Integer>` vs `int[]` is a 5× memory difference with the same Big-O.
- **Interview follow-ups:** "Can you do it in O(1) space?" (two pointers, in-place swaps, or reusing the input array as storage), and "What about the recursion stack?"

---

## J. Practice

**Beginner**
1. What is the auxiliary space of recursive factorial? Iterative factorial?
2. Is `Arrays.sort(int[])` in place? *(Mostly: dual-pivot quicksort uses O(log n) stack.)*
3. Space of checking whether a string is a palindrome with two pointers?

**Intermediate**
4. Give the stack depth of recursive DFS on a path graph of V vertices, and how to avoid overflow.
5. Rotate an array by k positions in O(1) extra space. *(Hint: reverse three times.)*
6. What's the peak memory of naive Fibonacci fib(40)? Time?

**Advanced**
7. Prove that quicksort that recurses on the smaller part uses O(log n) stack in the worst case.
8. Merge two sorted halves of an array **in place** in O(n log n) time. *(Hint: rotations, or block merge.)*

**Interview questions**

<details><summary>Q1. What's the difference between auxiliary space and space complexity?</summary>

Auxiliary space is the extra memory beyond the input. Total space complexity includes the input too. For sorting, "O(1) space" means O(1) **auxiliary** space, because the input array itself is Θ(n).
</details>

<details><summary>Q2. Why is recursive binary search O(log n) space but iterative O(1)?</summary>

Each recursive call adds a stack frame, and there are up to log₂ n + 1 nested calls before reaching the base case. The iterative version reuses the same three variables (lo, hi, mid).
</details>

<details><summary>Q3. Does a recursive algorithm that makes 2ⁿ calls use 2ⁿ space?</summary>

No. Stack space depends on the **maximum depth**, not the total number of calls. Calls finish and pop their frames. Naive Fibonacci makes about φⁿ calls but has depth n, so its space is Θ(n).
</details>

**Worked problem: move zeros to the end in O(1) space.** Keep a write pointer w = 0. For each element that isn't zero, copy it to a[w++]. Then fill a[w..n−1] with zeros. That's Θ(n) time and Θ(1) auxiliary space, and it keeps the relative order of the non-zero elements.

**Coding problems**
- LeetCode 283 · Move Zeroes *(verify link)*
- LeetCode 189 · Rotate Array *(verify link)*
- LeetCode 344 · Reverse String *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. Space complexity usually means **auxiliary** space: extra memory beyond the input.
2. Recursion costs stack space proportional to the **maximum depth**, not the number of calls.
3. "In place" means O(1) extra elements outside the input (CLRS definition).
4. Time ≥ space: you can't use memory without spending time touching it.
5. Java has no tail-call optimisation. For deep recursion, convert it to iteration.

**Formulas**
- Merge sort: Θ(n) auxiliary + Θ(log n) stack
- Quicksort stack: Θ(log n) best and expected, Θ(n) worst (Θ(log n) with smaller-side-first)
- Recursive binary search: Θ(log n) stack

**Common mistakes**
- Forgetting the recursion stack.
- Counting the input as auxiliary space.
- Calling quicksort "O(1) space".

**Quiz**
1. Auxiliary space of merge sort?
2. Stack depth of merge sort on n = 1024?
3. Which uses less memory for 10⁶ integers, `int[]` or `ArrayList<Integer>`, and roughly by how much?
4. Does naive Fibonacci's space grow exponentially?

**Answers**

<details><summary>Show answers</summary>

1. Θ(n) for the temporary array (plus Θ(log n) stack).
2. log₂ 1024 + 1 = 11 levels.
3. `int[]`, about 4 MB vs about 20 MB (roughly 5×).
4. No: Θ(n) stack space, because only one path of the call tree is active at a time.
</details>

**Related topics:** [Time complexity](time-complexity.md) · [Recurrence relations](recurrence-relations.md) · [Merge sort](../01-Sorting-and-Order-Statistics/comparison-sorts/merge-sort.md) · [Quicksort analysis](../01-Sorting-and-Order-Statistics/quicksort/complexity-analysis.md) · [Heapsort](../01-Sorting-and-Order-Statistics/heapsort/heapsort.md)
