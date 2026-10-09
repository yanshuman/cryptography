# Divide and Conquer

> **CLRS:** Chapter 2.3 (Designing algorithms), Chapter 4 (maximum-subarray in the 3rd edition) · **Status:** ✅ Written · **Prerequisites:** [Recurrence relations](recurrence-relations.md) · **Next:** [Merge sort](../01-Sorting-and-Order-Statistics/comparison-sorts/merge-sort.md)

---

## A. Introduction

**Divide and conquer (D&C)** is an algorithm design technique with three steps:

1. **Divide** the problem into smaller subproblems of the same kind.
2. **Conquer** the subproblems by solving them **recursively**. Small enough subproblems (base cases) are solved directly.
3. **Combine** the subproblem solutions into the solution of the original problem.

**What problem does it solve?** Many problems that look as if they need Θ(n²) brute force can be solved in Θ(n log n) or better, by splitting the work so that the expensive "everyone compares with everyone" step is replaced by cheap combine steps.

**Where it's used**

| Algorithm | Divide | Combine | Complexity |
|---|---|---|---|
| [Merge sort](../01-Sorting-and-Order-Statistics/comparison-sorts/merge-sort.md) | split in half | merge | Θ(n log n) |
| [Quicksort](../01-Sorting-and-Order-Statistics/quicksort/quicksort-basics.md) | partition around a pivot | nothing | Θ(n log n) expected |
| Binary search | check the middle, keep one half | nothing | Θ(log n) |
| [Strassen](../08-Advanced-Algorithm-Topics/matrix-operations/strassens-algorithm.md) | split into quadrants | 7 products + additions | Θ(n^2.81) |
| [FFT](../08-Advanced-Algorithm-Topics/polynomials-and-fft/dft-and-fft.md) | even and odd coefficients | butterflies | Θ(n log n) |
| [Closest pair](../11-Computational-Geometry/closest-pair-of-points.md) | split by x | check a narrow strip | Θ(n log n) |
| Karatsuba multiplication | split the digits | 3 products | Θ(n^1.585) |

In real software: `Arrays.sort` and `Collections.sort` (TimSort, quicksort), external sorting in databases, MapReduce (map = divide, reduce = combine), parallel computing (subproblems can run on different cores), and signal processing (FFT).

**Prerequisites:** recursion, and solving recurrences.

---

## B. Intuition

**Analogy: counting the votes in a country.** One person counting 100 million ballots takes forever. Instead, split them by state, then by district, then by polling station. Each station counts its own pile (the small subproblem). Results are **added up** on the way back (combine). The work gets done in parallel, and no single person handles everything.

**Why it can be faster.** Take a brute-force algorithm that compares all pairs, Θ(n²). If you split the input in half:
- each half costs (n/2)² = n²/4, and both together cost n²/2, so splitting **once** already halves the work;
- splitting recursively (log n times) with a **linear** combine gives T(n) = 2T(n/2) + n = Θ(n log n).

The trick is always the same: **make the combine step cheap**, ideally Θ(n) or less.

**When D&C helps:** the subproblems are **independent** (non-overlapping). If they overlap (the same subproblem is solved many times, like fib(n − 2) appearing inside fib(n − 1)), use [dynamic programming](../06-Algorithm-Design-Techniques/dynamic-programming/fundamentals.md) instead.

---

## C. How it works internally

### Example 1: Maximum subarray

**Problem:** given an array (with negative numbers), find the contiguous subarray with the largest sum.

> A = [−2, 1, −3, 4, −1, 2, 1, −5, 4] → the best is [4, −1, 2, 1], with sum **6**.

**Brute force:** try all n(n + 1)/2 subarrays, Θ(n²).

**Divide and conquer:**
- **Divide:** split at mid = 4. Left = [−2, 1, −3, 4], right = [−1, 2, 1, −5, 4].
- **Conquer:** the best subarray lies (a) entirely in the left half, (b) entirely in the right half, or (c) **crosses the middle**. Solve (a) and (b) recursively.
- **Combine:** find (c) in Θ(n). The best crossing subarray = (best suffix of the left half ending at mid) + (best prefix of the right half starting at mid + 1).

**Crossing computation at the top level:**

| Left suffixes ending at index 3 | sum | Right prefixes starting at index 4 | sum |
|---|---|---|---|
| [4] | 4 ✓ best | [−1] | −1 |
| [−3, 4] | 1 | [−1, 2] | 1 |
| [1, −3, 4] | 2 | [−1, 2, 1] | **2** ✓ best |
| [−2, 1, −3, 4] | 0 | [−1, 2, 1, −5] | −3 |
| | | [−1, 2, 1, −5, 4] | 1 |

Crossing best = 4 + 2 = **6**: the subarray [4, −1, 2, 1].
Left half best (recursively) = 4 ([4]). Right half best = 4 ([4] at the end, or [2, 1, …]). The answer is max(4, 4, 6) = **6**. ✓

Recurrence: T(n) = 2T(n/2) + Θ(n), which is **Θ(n log n)**.
(Kadane's algorithm, a DP/greedy idea, solves it in Θ(n). D&C isn't always the optimum, but it's a great teaching example, and it parallelises.)

### Example 2: Counting inversions

An **inversion** is a pair (i, j) with i < j but A[i] > A[j]. Counting them measures how unsorted an array is (it's used in ranking similarity, such as comparing two people's movie rankings).

> A = [2, 4, 1, 3, 5] → the inversions are (2,1), (4,1), (4,3), so **3**.

Brute force is Θ(n²). **D&C:** inversions = left-internal + right-internal + **split inversions** (one element in each half). Piggyback on merge sort: while merging, whenever we take an element from the **right** half, it's smaller than **every remaining element of the left half**, so add (number of remaining left elements). That gives Θ(n log n).

```
merge([2, 4], [1, 3, 5]):
take 1 (right): left still has [2, 4] → +2 inversions   (2,1), (4,1)
take 2 (left)
take 3 (right): left still has [4]    → +1 inversion    (4,3)
take 4 (left)
take 5 (right): left is empty         → +0
split inversions = 3, merged = [1, 2, 3, 4, 5]
```

### Example 3: Fast power

x^n = (x^(n/2))² if n is even, or x · (x^(n/2))² if n is odd. That's **one** recursive call of half size: T(n) = T(n/2) + Θ(1), which is **Θ(log n)**.

### Edge cases in D&C

- **Base case size:** n = 0 and n = 1 must be handled, or the recursion never stops.
- **Odd sizes:** split as ⌊n/2⌋ and ⌈n/2⌉. Use `mid = lo + (hi - lo) / 2`.
- **Infinite recursion:** if a split can produce a subproblem of the *same* size (for example a bad partition), the recursion never ends.
- **Small n:** recursion overhead dominates, so practical code switches to a simple algorithm below a cutoff (insertion sort below about 16–32 elements).

---

## D. Algorithm and pseudocode

### Generic template

```
DIVIDE-AND-CONQUER(P)
1  if P is small enough
2      return SOLVE-DIRECTLY(P)
3  split P into subproblems P₁, …, P_a, each of size about n/b      ▷ divide
4  for i ← 1 to a
5      Sᵢ ← DIVIDE-AND-CONQUER(Pᵢ)                                   ▷ conquer
6  return COMBINE(S₁, …, S_a)                                        ▷ combine
```

**Cost:** T(n) = a·T(n/b) + D(n) + C(n), where D = divide cost and C = combine cost. Solve it with the [Master Theorem](recurrence-relations.md#method-3-the-master-theorem-fastest-when-it-applies).

### Maximum subarray (CLRS 3rd edition, §4.1)

```
FIND-MAX-CROSSING-SUBARRAY(A, low, mid, high)
1  left-sum ← −∞; sum ← 0
2  for i ← mid downto low
3      sum ← sum + A[i]
4      if sum > left-sum then left-sum ← sum; max-left ← i
5  right-sum ← −∞; sum ← 0
6  for j ← mid + 1 to high
7      sum ← sum + A[j]
8      if sum > right-sum then right-sum ← sum; max-right ← j
9  return (max-left, max-right, left-sum + right-sum)

FIND-MAXIMUM-SUBARRAY(A, low, high)
1  if high = low then return (low, high, A[low])                  ▷ base case: one element
2  mid ← ⌊(low + high)/2⌋
3  (ll, lh, ls) ← FIND-MAXIMUM-SUBARRAY(A, low, mid)
4  (rl, rh, rs) ← FIND-MAXIMUM-SUBARRAY(A, mid + 1, high)
5  (cl, ch, cs) ← FIND-MAX-CROSSING-SUBARRAY(A, low, mid, high)
6  return whichever of the three has the largest sum
```

**Correctness (by strong induction on the size n = high − low + 1):**
- *Base:* n = 1. The only subarray is the element itself. ✓
- *Step:* any subarray of A[low..high] lies entirely left of mid, entirely right of it, or crosses it. These three cases are **exhaustive**. By the inductive hypothesis, lines 3–4 find the best in the first two cases. Line 5 finds the best crossing subarray, because a crossing subarray is a suffix of the left part joined to a prefix of the right part, and the two halves can be maximised independently. Taking the max of all three gives the overall best. ✓
- *Termination:* each call works on a strictly smaller range (for n ≥ 2, both halves have size < n), down to size 1.

---

## E. Implementation

```java
import java.util.*;

/** Divide-and-conquer examples: maximum subarray (vs Kadane), inversion counting, fast power. */
public class DivideAndConquer {

    // ---------- Maximum subarray: Theta(n log n) ----------
    record Result(int lo, int hi, long sum) {}

    static Result maxSubarray(int[] a, int lo, int hi) {
        if (lo == hi) return new Result(lo, hi, a[lo]);              // base case
        int mid = lo + (hi - lo) / 2;
        Result left = maxSubarray(a, lo, mid);                        // conquer left
        Result right = maxSubarray(a, mid + 1, hi);                   // conquer right
        Result cross = maxCrossing(a, lo, mid, hi);                   // combine
        if (left.sum() >= right.sum() && left.sum() >= cross.sum()) return left;
        if (right.sum() >= cross.sum()) return right;
        return cross;
    }

    static Result maxCrossing(int[] a, int lo, int mid, int hi) {
        long best = Long.MIN_VALUE, sum = 0; int bestL = mid;
        for (int i = mid; i >= lo; i--) { sum += a[i]; if (sum > best) { best = sum; bestL = i; } }
        long leftSum = best;
        best = Long.MIN_VALUE; sum = 0; int bestR = mid + 1;
        for (int j = mid + 1; j <= hi; j++) { sum += a[j]; if (sum > best) { best = sum; bestR = j; } }
        return new Result(bestL, bestR, leftSum + best);
    }

    /** Kadane's algorithm, Theta(n): used to cross-check the D&C answer. */
    static long kadane(int[] a) {
        long best = a[0], cur = a[0];
        for (int i = 1; i < a.length; i++) { cur = Math.max(a[i], cur + a[i]); best = Math.max(best, cur); }
        return best;
    }

    // ---------- Counting inversions with merge sort: Theta(n log n) ----------
    static long countInversions(int[] a) {
        return sortCount(a.clone(), new int[a.length], 0, a.length - 1);
    }

    static long sortCount(int[] a, int[] tmp, int lo, int hi) {
        if (lo >= hi) return 0;
        int mid = lo + (hi - lo) / 2;
        long inv = sortCount(a, tmp, lo, mid) + sortCount(a, tmp, mid + 1, hi);
        int i = lo, j = mid + 1, k = lo;
        while (i <= mid && j <= hi) {
            if (a[i] <= a[j]) tmp[k++] = a[i++];
            else { tmp[k++] = a[j++]; inv += mid - i + 1; }          // a[j] < every remaining left element
        }
        while (i <= mid) tmp[k++] = a[i++];
        while (j <= hi) tmp[k++] = a[j++];
        System.arraycopy(tmp, lo, a, lo, hi - lo + 1);
        return inv;
    }

    static long bruteInversions(int[] a) {
        long c = 0;
        for (int i = 0; i < a.length; i++) for (int j = i + 1; j < a.length; j++) if (a[i] > a[j]) c++;
        return c;
    }

    // ---------- Fast power: Theta(log n) multiplications ----------
    static int multiplications;
    static long power(long x, int n) {
        if (n == 0) return 1;
        long half = power(x, n / 2);          // ONE recursive call, reused
        multiplications++;
        long sq = half * half;
        if (n % 2 == 1) { multiplications++; return sq * x; }
        return sq;
    }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        int[] a = {-2, 1, -3, 4, -1, 2, 1, -5, 4};
        Result r = maxSubarray(a, 0, a.length - 1);
        System.out.println("A = " + Arrays.toString(a));
        System.out.println("max subarray = A[" + r.lo() + ".." + r.hi() + "] = "
                + Arrays.toString(Arrays.copyOfRange(a, r.lo(), r.hi() + 1)) + ", sum " + r.sum());
        check(r.sum() == 6 && r.lo() == 3 && r.hi() == 6, "D&C finds [4, -1, 2, 1] with sum 6");
        check(maxSubarray(new int[]{-3, -1, -2}, 0, 2).sum() == -1, "all-negative array: best is the single largest element");

        Random rnd = new Random(42);
        boolean agree = true;
        for (int t = 0; t < 1000; t++) {
            int[] b = rnd.ints(1 + rnd.nextInt(50), -20, 21).toArray();
            if (maxSubarray(b, 0, b.length - 1).sum() != kadane(b)) agree = false;
        }
        check(agree, "D&C matches Kadane on 1000 random arrays");

        int[] inv = {2, 4, 1, 3, 5};
        System.out.println("inversions in " + Arrays.toString(inv) + " = " + countInversions(inv));
        check(countInversions(inv) == 3, "[2,4,1,3,5] has 3 inversions");
        int[] rev = new int[100]; for (int i = 0; i < 100; i++) rev[i] = 100 - i;
        check(countInversions(rev) == 100L * 99 / 2, "reversed array of 100 has n(n-1)/2 = 4950 inversions");
        boolean invAgree = true;
        for (int t = 0; t < 500; t++) {
            int[] b = rnd.ints(rnd.nextInt(60), 0, 30).toArray();
            if (countInversions(b) != bruteInversions(b)) invAgree = false;
        }
        check(invAgree, "merge-sort count matches brute force on 500 random arrays");

        multiplications = 0;
        long p = power(3, 39);
        System.out.println("3^39 = " + p + " using " + multiplications + " multiplications (naive: 38)");
        check(p == 4052555153018976267L, "3^39 correct (fits in a long; 3^40 would overflow)");
        check(multiplications <= 2 * 5 + 2, "at most 2*floor(log2 39)+2 = 12 multiplications");
    }
}
```

**Output:**

```
A = [-2, 1, -3, 4, -1, 2, 1, -5, 4]
max subarray = A[3..6] = [4, -1, 2, 1], sum 6
ok   D&C finds [4, -1, 2, 1] with sum 6
ok   all-negative array: best is the single largest element
ok   D&C matches Kadane on 1000 random arrays
inversions in [2, 4, 1, 3, 5] = 3
ok   [2,4,1,3,5] has 3 inversions
ok   reversed array of 100 has n(n-1)/2 = 4950 inversions
ok   merge-sort count matches brute force on 500 random arrays
3^39 = 4052555153018976267 using 10 multiplications (naive: 38)
ok   3^39 correct (fits in a long; 3^40 would overflow)
ok   at most 2*floor(log2 39)+2 = 12 multiplications
```

**Dry run of `power(3, 39)`:** 39 → 19 → 9 → 4 → 2 → 1 → 0. Squarings happen at n = 1, 2, 4, 9, 19, 39 (6 of them), plus extra multiplications by x at the odd values n = 39, 19, 9 and 1 (4 of them). That's 10 multiplications instead of 38.

**Java-specific notes**
- `record Result(...)` (Java 16+) is a compact immutable tuple. Before Java 16, use a small class.
- **Overflow:** 3³⁹ ≈ 4.05 × 10¹⁸ fits in a `long` (max ≈ 9.22 × 10¹⁸), but 3⁴⁰ ≈ 1.2 × 10¹⁹ doesn't, and Java would silently wrap around. For real use, apply `Math.multiplyExact` (throws on overflow), `BigInteger.pow`, or work modulo m (as in [modular exponentiation](../09-Number-Theoretic-Algorithms/modular-exponentiation.md)).
- **Reuse buffers:** pass one `tmp` array down the recursion instead of allocating one per call.

**Common mistakes**
1. Calling `power(x, n/2)` **twice**, which turns Θ(log n) into Θ(n).
2. Forgetting the crossing case in max subarray.
3. Using `inv += 1` instead of `inv += mid - i + 1` when counting split inversions.
4. A missing or wrong base case, which leads to `StackOverflowError`.

---

## F. Time complexity

| Algorithm | Recurrence | Master case | Result |
|---|---|---|---|
| Max subarray (D&C) | T(n) = 2T(n/2) + Θ(n) | 2 | **Θ(n log n)** in every case |
| Count inversions | T(n) = 2T(n/2) + Θ(n) | 2 | **Θ(n log n)** |
| Fast power | T(n) = T(n/2) + Θ(1) | 2 | **Θ(log n)** |
| Binary search | T(n) = T(n/2) + Θ(1) | 2 | Θ(log n) worst, Θ(1) best |

**Derivation for max subarray:** the divide step is Θ(1) (compute mid). Conquer is 2T(n/2). Combine is FIND-MAX-CROSSING, which scans each element of the range once, so Θ(n). Then T(n) = 2T(n/2) + Θ(n), where a = b = 2, n^(log₂ 2) = n and f(n) = Θ(n), so this is Case 2 and the answer is Θ(n log n). There's no best or worst case difference: the work doesn't depend on the values.

**Assumptions:** constant-time arithmetic (true for `long` sums of `int`s up to about 2³¹ elements).

## G. Space complexity

| Algorithm | Auxiliary | Recursion stack | Total |
|---|---|---|---|
| Max subarray | Θ(1) per frame | Θ(log n) | **Θ(log n)** |
| Count inversions | Θ(n) temporary array (plus a copy of the input, so the input isn't modified) | Θ(log n) | **Θ(n)** |
| Fast power | Θ(1) | Θ(log n) | **Θ(log n)** (Θ(1) if iterative) |

## H. Complexity summary

| Problem | Brute force | Divide and conquer | Best known |
|---|---|---|---|
| Maximum subarray | Θ(n²) | Θ(n log n) | Θ(n) (Kadane) |
| Counting inversions | Θ(n²) | Θ(n log n) | Θ(n log n) (comparison model) |
| xⁿ | Θ(n) multiplications | Θ(log n) | Θ(log n) |
| Sorting | Θ(n²) (simple sorts) | Θ(n log n) | Θ(n log n) (comparison lower bound) |

## I. Advantages, limitations, and comparisons

**Advantages**
- Turns many Θ(n²) problems into Θ(n log n).
- **Naturally parallel:** independent subproblems can run on different cores (Java's `ForkJoinPool`, `Arrays.parallelSort`).
- **Cache-friendly:** small subproblems fit in cache.
- Correctness proofs follow the structure (strong induction).

**Limitations**
- **Recursion overhead:** function calls and the stack, which hurts for small n. Use a cutoff to an iterative algorithm.
- **Overlapping subproblems** make D&C exponential (naive Fibonacci). Use DP instead.
- **The combine step can be hard to design** (closest pair, FFT).
- **Not always optimal** (max subarray has a Θ(n) solution).

| Technique | Subproblems | Strategy | Example |
|---|---|---|---|
| Divide and conquer | independent | solve all, combine | merge sort |
| Dynamic programming | **overlapping** | solve each once, store it | LCS, knapsack |
| Greedy | one (after a choice) | make the locally best choice, never reconsider | activity selection, Huffman |

**Interview follow-ups:** "Can you do max subarray in O(n)?" (Kadane), "Parallelise merge sort", "Count inversions for an array of 10⁹ elements on disk" (external merge sort).

---

## J. Practice

**Beginner**
1. Write recursive binary search and state its recurrence.
2. Find the max of an array by D&C (split in half). What's the recurrence, and the number of comparisons?
3. Trace the inversion count on [3, 1, 2].

**Intermediate**
4. Modify max subarray to return an empty subarray (sum 0) if every number is negative.
5. Count "important reverse pairs" i < j with A[i] > 2·A[j] in Θ(n log n). *(LeetCode 493.)*
6. Find the majority element (appearing more than n/2 times) by D&C, in Θ(n log n).

**Advanced**
7. Karatsuba: multiply two n-digit numbers with 3 recursive multiplications of n/2 digits. Derive Θ(n^log₂3).
8. Prove that any D&C max-subarray algorithm with a linear combine and an even split is Θ(n log n), and explain how Kadane avoids the log factor.

**Interview questions**

<details><summary>Q1. What are the three steps of divide and conquer?</summary>

Divide the problem into smaller independent subproblems, conquer them recursively (solving base cases directly), and combine their solutions. Its cost is T(n) = aT(n/b) + (divide + combine cost).
</details>

<details><summary>Q2. When should you use dynamic programming instead of divide and conquer?</summary>

When the subproblems **overlap**, that is, the same subproblem is reached by many recursion paths. Plain D&C recomputes them, which is often exponential. DP solves each one once and stores the result.
</details>

<details><summary>Q3. How does counting inversions reuse merge sort?</summary>

During the merge, when you take an element from the right half, it's smaller than every element still waiting in the left half (both halves are sorted), so it forms (number remaining on the left) inversions at once. Adding these up over all merges counts every split inversion in Θ(n log n) total.
</details>

**Worked problem: count the elements smaller than each element to its right (LeetCode 315).** Use merge sort on (value, original index) pairs. When a left element is placed during the merge, the number of right elements already placed before it is exactly how many smaller elements lie to its right. Add that to its counter. The result is Θ(n log n).

**Coding problems**
- LeetCode 53 · Maximum Subarray *(verify link)*
- LeetCode 493 · Reverse Pairs *(verify link)*
- LeetCode 315 · Count of Smaller Numbers After Self *(verify link)*
- LeetCode 169 · Majority Element *(verify link)*
- LeetCode 50 · Pow(x, n) *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. D&C = divide, conquer recursively, combine.
2. Cost: T(n) = aT(n/b) + (divide + combine), solved with the Master Theorem.
3. The power of D&C comes from a **cheap combine** (usually linear).
4. It needs **independent** subproblems. If they overlap, use DP.
5. In practice, add a small-n cutoff and reuse buffers.

**Formulas**
- 2T(n/2) + Θ(n) = Θ(n log n)
- T(n/2) + Θ(1) = Θ(log n)
- 3T(n/2) + Θ(n) = Θ(n^1.585) (Karatsuba)

**Common mistakes**
- Duplicate recursive calls.
- A missing crossing or split case.
- A base case that never triggers.

**Quiz**
1. What is the combine step of merge sort? Of quicksort?
2. What's the recurrence for D&C max subarray?
3. Why does `power(x, n/2) * power(x, n/2)` take Θ(n)?
4. Which technique fits Fibonacci better, D&C or DP? Why?

**Answers**

<details><summary>Show answers</summary>

1. Merge sort: merging the two sorted halves, Θ(n). Quicksort: nothing, because the partition step (divide) does the work.
2. T(n) = 2T(n/2) + Θ(n), which is Θ(n log n).
3. It's T(n) = 2T(n/2) + Θ(1), which is Θ(n) by Master Case 1.
4. DP. fib(n − 1) and fib(n − 2) share subproblems, so plain D&C is exponential.
</details>

**Related topics:** [Recurrence relations](recurrence-relations.md) · [Merge sort](../01-Sorting-and-Order-Statistics/comparison-sorts/merge-sort.md) · [Quicksort](../01-Sorting-and-Order-Statistics/quicksort/quicksort-basics.md) · [Dynamic programming](../06-Algorithm-Design-Techniques/dynamic-programming/fundamentals.md) · [Closest pair](../11-Computational-Geometry/closest-pair-of-points.md)
