# Recurrence Relations

> **CLRS:** Chapter 4 (Recurrences: substitution, recursion-tree, master method) · **Status:** ✅ Written · **Prerequisites:** [Asymptotic notation](asymptotic-notation.md), [Summations](../14-Mathematical-Foundations/summations.md) · **Next:** [Divide and conquer](divide-and-conquer.md)

---

## A. Introduction

A **recurrence** is an equation that describes a function in terms of its value on **smaller inputs**. Recursive algorithms naturally produce them:

> Merge sort: **T(n) = 2T(n/2) + Θ(n)**, with T(1) = Θ(1)
> "To sort n items: sort two halves (2 × T(n/2)), then merge (Θ(n))."

**What problem does it solve?** For loops we add up iterations. For recursion we can't count directly, because the cost is defined in terms of itself. **Solving** the recurrence gives a closed form, such as T(n) = Θ(n log n).

**Where it's used:** analysing every divide-and-conquer algorithm (merge sort, quicksort, binary search, Strassen, FFT, closest pair, Karatsuba multiplication) and many recursive DP and backtracking algorithms.

**Prerequisites:** asymptotic notation, geometric series (1 + r + r² + … = 1/(1 − r) for r < 1), and logarithms.

---

## B. Intuition

Think of a recurrence as a **company hierarchy**:

- The CEO (problem size n) does some work themselves (the **combine cost** f(n)) and splits the rest among **a** managers, each with a problem of size n/b.
- Each manager does the same, recursively, down to interns (base cases).
- **Total work = the sum of everyone's own work.**

The question is **where most of the work happens**:

1. **At the top:** the CEO's own work dominates, so T(n) = Θ(f(n)).
2. **Spread evenly:** every level of the hierarchy does the same amount, so T(n) = f(n) × (number of levels) = Θ(f(n) log n).
3. **At the bottom:** there are so many interns that their combined work dominates, so T(n) = Θ(number of leaves) = Θ(n^(log_b a)).

That's exactly what the **Master Theorem**'s three cases say.

---

## C. How it works internally: three solution methods

### Method 1: Recursion tree (best for building intuition)

**Example:** T(n) = 2T(n/2) + cn (merge sort).

```
Level 0:                  cn                          → cn
                        /    \
Level 1:            c(n/2)  c(n/2)                    → cn
                    /  \      /  \
Level 2:        c(n/4) c(n/4) c(n/4) c(n/4)           → cn
                 ...                ...
Level log₂n:   c  c  c  c  ...  c  c   (n leaves)     → cn
                                                     ─────────
                         (log₂ n + 1) levels × cn  =  cn log₂ n + cn
```

- Level i has 2ⁱ nodes, each of size n/2ⁱ, each costing c·n/2ⁱ. The level total is 2ⁱ · c·n/2ⁱ = **cn**.
- The depth is log₂ n, because the size halves each level until it reaches 1.
- **T(n) = cn(log₂ n + 1) = Θ(n log n).**

**An uneven example:** T(n) = T(n/3) + T(2n/3) + cn.
Each full level still sums to cn (because n/3 + 2n/3 = n). The **longest** path follows the 2/3 branch: n → (2/3)n → (2/3)²n → … → 1, which has log_{3/2} n levels. The shortest path has log₃ n levels. Both are Θ(log n), so **T(n) = Θ(n log n)**. This is why quicksort with a constant-proportion split is still Θ(n log n). See [Quicksort complexity analysis](../01-Sorting-and-Order-Statistics/quicksort/complexity-analysis.md).

### Method 2: Substitution (guess, then prove by induction)

**Example:** prove T(n) = 2T(⌊n/2⌋) + n is O(n log n).

1. **Guess:** T(n) ≤ c·n log₂ n for some constant c > 0 (from the recursion tree).
2. **Inductive step:** assume it holds for ⌊n/2⌋:
   T(n) ≤ 2·c⌊n/2⌋ log₂(⌊n/2⌋) + n
   ≤ c·n log₂(n/2) + n
   = c·n log₂ n − c·n + n
   ≤ **c·n log₂ n** as long as **c ≥ 1**. ✓
3. **Base case:** take T(1) = 1. The bound c·1·log₂ 1 = 0 fails at n = 1, so start the induction at n = 2 and n = 3 (T(2) = 4, T(3) = 5) and choose c ≥ 2 so that 4 ≤ 2c and 5 ≤ 3c log₂ 3. Asymptotic notation only needs n ≥ n₀, so this is allowed.

**A common pitfall: the "inductive hypothesis must match exactly" trap.**
Trying to prove T(n) = 2T(n/2) + n is O(n) with the guess T(n) ≤ cn gives T(n) ≤ cn + n = (c + 1)n. That is **not** ≤ cn, so the proof fails, even though (c + 1)n is "O(n)". You must prove the **exact** form of the hypothesis. (And indeed T(n) = Θ(n log n), not O(n).)

**Subtracting a lower-order term:** sometimes the guess is right but the induction fails by a constant. For T(n) = 2T(n/2) + 1, the guess T(n) ≤ cn gives cn + 1, which fails. A **stronger** guess, T(n) ≤ cn − d, works: 2(c·n/2 − d) + 1 = cn − 2d + 1 ≤ cn − d when d ≥ 1. ✓

### Method 3: The Master Theorem (fastest, when it applies)

For recurrences of the form

> **T(n) = a·T(n/b) + f(n)**, with constants a ≥ 1 and b > 1,

compare f(n) with **n^(log_b a)**, the **watershed function**, which equals the number of leaves in the recursion tree:

| Case | Condition | Result | Where the work is |
|---|---|---|---|
| **1** | f(n) = O(n^(log_b a − ε)) for some ε > 0 (f is **polynomially smaller**) | T(n) = Θ(n^(log_b a)) | leaves |
| **2** | f(n) = Θ(n^(log_b a)) | T(n) = Θ(n^(log_b a) · log n) | every level equally |
| **3** | f(n) = Ω(n^(log_b a + ε)) for some ε > 0 (**polynomially larger**) **and** a·f(n/b) ≤ c·f(n) for some c < 1 (the *regularity condition*) | T(n) = Θ(f(n)) | root |

**Extended case 2** (widely used, from CLRS Exercise 4.4-2 and the 4th edition):
if f(n) = Θ(n^(log_b a) · logᵏ n) with k ≥ 0, then **T(n) = Θ(n^(log_b a) · log^(k+1) n)**.

### Worked Master Theorem examples

| Recurrence | a, b | n^(log_b a) | f(n) | Case | Solution | Algorithm |
|---|---|---|---|---|---|---|
| T(n) = 2T(n/2) + n | 2, 2 | n | n | 2 | **Θ(n log n)** | Merge sort |
| T(n) = T(n/2) + 1 | 1, 2 | n⁰ = 1 | 1 | 2 | **Θ(log n)** | Binary search |
| T(n) = 2T(n/2) + 1 | 2, 2 | n | 1 = O(n^(1−ε)) | 1 | **Θ(n)** | Tree traversal, finding max recursively |
| T(n) = 8T(n/2) + n² | 8, 2 | n³ | n² | 1 | **Θ(n³)** | Naive recursive matrix multiplication |
| T(n) = 7T(n/2) + n² | 7, 2 | n^2.807 | n² | 1 | **Θ(n^2.807)** | [Strassen](../08-Advanced-Algorithm-Topics/matrix-operations/strassens-algorithm.md) |
| T(n) = 3T(n/2) + n | 3, 2 | n^1.585 | n | 1 | **Θ(n^1.585)** | Karatsuba multiplication |
| T(n) = 3T(n/4) + n log n | 3, 4 | n^0.793 | n log n | 3 (regularity: 3(n/4)log(n/4) ≤ (3/4)n log n ✓) | **Θ(n log n)** | — |
| T(n) = T(n/2) + n | 1, 2 | 1 | n | 3 | **Θ(n)** | Quickselect, lucky case |
| T(n) = 2T(n/2) + n log n | 2, 2 | n | n log n | extended 2 (k = 1) | **Θ(n log² n)** | — |
| T(n) = 4T(n/2) + n² log n | 4, 2 | n² | n² log n | extended 2 (k = 1) | **Θ(n² log² n)** | — |

### When the Master Theorem does NOT apply

| Recurrence | Why not | Solve with |
|---|---|---|
| T(n) = T(n − 1) + n | Subtracts rather than divides | Unrolling: n + (n − 1) + … = **Θ(n²)** (insertion sort, worst-case quicksort) |
| T(n) = T(n − 1) + 1 | Subtracts | **Θ(n)** (linear recursion) |
| T(n) = 2T(n − 1) + 1 | Subtracts and branches | **Θ(2ⁿ)** (Towers of Hanoi: exactly 2ⁿ − 1 moves) |
| T(n) = T(n/3) + T(2n/3) + n | Unequal subproblem sizes | Recursion tree: **Θ(n log n)**, or the Akra–Bazzi theorem |
| T(n) = 2T(n/2) + n/log n | f is smaller than n, but not *polynomially* smaller (a gap between cases 1 and 2) | Recursion tree: **Θ(n log log n)** |
| T(n) = 2T(√n) + log n | Not of the n/b form | Substitute m = log n → S(m) = 2S(m/2) + m → **Θ(log n · log log n)** |

**Floors and ceilings:** T(n) = T(⌈n/2⌉) + T(⌊n/2⌋) + n behaves like 2T(n/2) + n. They don't affect the asymptotic solution, which is why we usually ignore them.

---

## D. Algorithm and pseudocode: a procedure for solving any recurrence

```
SOLVE-RECURRENCE(T)
1  if T has the form a·T(n/b) + f(n):
2      compute w = n^(log_b a)
3      compare f(n) with w:
4         f polynomially smaller           → Case 1: Θ(w)
5         f = Θ(w · logᵏ n)                → Case 2: Θ(w · log^(k+1) n)
6         f polynomially larger + regular  → Case 3: Θ(f(n))
7         otherwise (gap)                  → go to line 9
8  if T has the form T(n − c) + f(n): unroll it into a sum and evaluate the sum
9  otherwise: draw the recursion tree, sum each level, sum over all levels
10 prove the guess formally with substitution (induction) if rigour is needed
```

### Proof sketch of the Master Theorem (why the three cases)

Unroll T(n) = aT(n/b) + f(n) into a tree:
- Depth: log_b n levels.
- Level j has aʲ nodes, each costing f(n/bʲ).
- Leaves: a^(log_b n) = **n^(log_b a)** of them (using the identity a^(log_b n) = n^(log_b a)).

> T(n) = Θ(n^(log_b a)) + Σ_{j=0}^{log_b n − 1} aʲ · f(n/bʲ)

- **Case 1:** the level sums **grow geometrically** going down, so the sum is dominated by the leaves.
- **Case 2:** every level sum is about the same, Θ(n^(log_b a)), and there are log_b n of them, which gives the log factor.
- **Case 3:** the level sums **shrink geometrically** (that's what the regularity condition guarantees), so the sum is dominated by the root, f(n).

---

## E. Implementation

The program evaluates recurrences **exactly** (with memoisation) and divides by the claimed closed form. If the ratio settles to a constant, the Θ bound is confirmed numerically.

```java
import java.util.*;
import java.util.function.LongUnaryOperator;

/** Evaluates recurrences exactly and checks T(n)/g(n) settles to a constant, confirming T = Theta(g). */
public class RecurrenceDemo {

    static Map<Long, Double> memo = new HashMap<>();

    /** T(n) = a*T(n/b) + f(n), T(1) = 1, n a power of b. */
    static double master(long n, int a, int b, LongUnaryOperator fScaled, Map<Long, Double> m) {
        if (n <= 1) return 1;
        Double hit = m.get(n);
        if (hit != null) return hit;
        double v = a * master(n / b, a, b, fScaled, m) + fScaled.applyAsLong(n);
        m.put(n, v);
        return v;
    }

    static double log2(double x) { return Math.log(x) / Math.log(2); }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        System.out.printf("%-28s %-16s %12s %12s %12s%n", "recurrence", "claimed Theta", "n=2^10", "n=2^16", "n=2^20");
        Object[][] rows = {
            {"T = 2T(n/2) + n",      "n log n",     2, 2, (LongUnaryOperator) n -> n,     (java.util.function.DoubleUnaryOperator) n -> n * log2(n)},
            {"T = T(n/2) + 1",       "log n",       1, 2, (LongUnaryOperator) n -> 1,     (java.util.function.DoubleUnaryOperator) n -> log2(n)},
            {"T = 2T(n/2) + 1",      "n",           2, 2, (LongUnaryOperator) n -> 1,     (java.util.function.DoubleUnaryOperator) n -> n},
            {"T = 7T(n/2) + n^2",    "n^log2(7)",   7, 2, (LongUnaryOperator) n -> n * n, (java.util.function.DoubleUnaryOperator) n -> Math.pow(n, log2(7))},
            {"T = 3T(n/2) + n",      "n^log2(3)",   3, 2, (LongUnaryOperator) n -> n,     (java.util.function.DoubleUnaryOperator) n -> Math.pow(n, log2(3))},
            {"T = T(n/2) + n",       "n",           1, 2, (LongUnaryOperator) n -> n,     (java.util.function.DoubleUnaryOperator) n -> n},
        };
        for (Object[] r : rows) {
            int a = (int) r[2], b = (int) r[3];
            LongUnaryOperator f = (LongUnaryOperator) r[4];
            java.util.function.DoubleUnaryOperator g = (java.util.function.DoubleUnaryOperator) r[5];
            double[] ratio = new double[3];
            int[] exps = {10, 16, 20};
            for (int k = 0; k < 3; k++) {
                long n = 1L << exps[k];
                ratio[k] = master(n, a, b, f, new HashMap<>()) / g.applyAsDouble(n);
            }
            System.out.printf("%-28s %-16s %12.4f %12.4f %12.4f%n", r[0], r[1], ratio[0], ratio[1], ratio[2]);
            check(Math.abs(ratio[2] - ratio[1]) / ratio[2] < 0.15, r[0] + " ratio has settled -> Theta(" + r[1] + ")");
        }

        // Subtractive recurrences (no Master Theorem): unroll exactly.
        long t = 0; for (int n = 1; n <= 1000; n++) t += n;        // T(n) = T(n-1) + n
        System.out.println("T(n) = T(n-1) + n at n=1000: " + t + "  (n(n+1)/2 = " + 1000L * 1001 / 2 + ")");
        check(t == 500500, "T(n) = T(n-1) + n = n(n+1)/2 = Theta(n^2)");

        long hanoi = 0; for (int n = 1; n <= 20; n++) hanoi = 2 * hanoi + 1;   // T(n) = 2T(n-1) + 1
        System.out.println("Hanoi T(20) = " + hanoi + "  (2^20 - 1 = " + ((1 << 20) - 1) + ")");
        check(hanoi == (1 << 20) - 1, "T(n) = 2T(n-1) + 1 = 2^n - 1 = Theta(2^n)");
    }
}
```

**Output:**

```
recurrence                   claimed Theta          n=2^10       n=2^16       n=2^20
T = 2T(n/2) + n              n log n                1.1000       1.0625       1.0500
ok   T = 2T(n/2) + n ratio has settled -> Theta(n log n)
T = T(n/2) + 1               log n                  1.1000       1.0625       1.0500
ok   T = T(n/2) + 1 ratio has settled -> Theta(log n)
T = 2T(n/2) + 1              n                      1.9990       2.0000       2.0000
ok   T = 2T(n/2) + 1 ratio has settled -> Theta(n)
T = 7T(n/2) + n^2            n^log2(7)              2.3284       2.3332       2.3333
ok   T = 7T(n/2) + n^2 ratio has settled -> Theta(n^log2(7))
T = 3T(n/2) + n              n^log2(3)              2.9653       2.9970       2.9994
ok   T = 3T(n/2) + n ratio has settled -> Theta(n^log2(3))
T = T(n/2) + n               n                      1.9990       2.0000       2.0000
ok   T = T(n/2) + n ratio has settled -> Theta(n)
T(n) = T(n-1) + n at n=1000: 500500  (n(n+1)/2 = 500500)
ok   T(n) = T(n-1) + n = n(n+1)/2 = Theta(n^2)
Hanoi T(20) = 1048575  (2^20 - 1 = 1048575)
ok   T(n) = 2T(n-1) + 1 = 2^n - 1 = Theta(2^n)
```

**Reading the output:** each ratio column converges to a constant (1, 2, 2.33 = 7/3, 3, …), confirming the Θ bound. For merge sort, T(n)/(n log₂ n) = 1 + 1/log₂ n → 1, which matches the exact solution T(n) = n log₂ n + n.

**Java-specific notes:** the memo `HashMap` turns an exponential number of recursive evaluations into one per distinct n (O(log n) of them here). Use `double` for values like 7^20 that overflow `long`.

---

## F. Time complexity

The point of this page is *producing* time complexities. Summary of the patterns:

| Pattern | Solution |
|---|---|
| T(n) = T(n/2) + Θ(1) | Θ(log n) |
| T(n) = T(n/2) + Θ(n) | Θ(n) |
| T(n) = 2T(n/2) + Θ(1) | Θ(n) |
| T(n) = 2T(n/2) + Θ(n) | Θ(n log n) |
| T(n) = 2T(n/2) + Θ(n log n) | Θ(n log² n) |
| T(n) = aT(n/b) + Θ(nᵈ) | d < log_b a: Θ(n^(log_b a)); d = log_b a: Θ(nᵈ log n); d > log_b a: Θ(nᵈ) |
| T(n) = T(n − 1) + Θ(1) | Θ(n) |
| T(n) = T(n − 1) + Θ(n) | Θ(n²) |
| T(n) = 2T(n − 1) + Θ(1) | Θ(2ⁿ) |
| T(n) = T(αn) + T((1 − α)n) + Θ(n), 0 < α < 1 | Θ(n log n) |

The **aT(n/b) + Θ(nᵈ)** row is the simplified Master Theorem most interviewers use.

## G. Space complexity

Recurrences also describe space. Recursion **depth** for T(n) = aT(n/b) + … is log_b n, so the stack is Θ(log n) when each frame is Θ(1). Merge sort's auxiliary *array* space is S(n) = Θ(n), because a buffer of size n is reused at every level rather than being allocated afresh each time.

## H. Complexity summary

See the table in Section F and the "Common recurrences" section of [complexity-comparison.md](../15-Practice-and-Revision/complexity-comparison.md#4-common-recurrences).

## I. Advantages, limitations, and comparisons

| Method | Pros | Cons |
|---|---|---|
| Recursion tree | Visual, builds intuition, handles unequal splits | Informal unless backed by a proof |
| Substitution | Fully rigorous, works for anything | You need a good guess, and the induction can be fiddly |
| Master Theorem | Instant for the common form | Only covers aT(n/b) + f(n), and has gaps between cases |
| Akra–Bazzi (advanced) | Handles unequal splits like T(n/3) + T(2n/3) | Needs calculus |

**Interview follow-ups:** "Derive merge sort's complexity", "Why is binary search log n?", "What's the complexity of this recursive function?" (write the recurrence, then solve it).

---

## J. Practice

**Beginner**
1. Solve T(n) = 4T(n/2) + n with the Master Theorem.
2. Solve T(n) = T(n/2) + n².
3. Unroll T(n) = T(n − 1) + 2.

**Intermediate**
4. Solve T(n) = 2T(n/4) + √n. *(Case 2 → Θ(√n log n).)*
5. Draw the recursion tree for T(n) = 3T(n/3) + n and sum it.
6. Prove T(n) = 2T(n/2) + n is Ω(n log n) by substitution.

**Advanced**
7. Solve T(n) = 2T(√n) + 1 by changing variables (m = log n).
8. Solve T(n) = T(n/4) + T(3n/4) + n with a recursion tree, and give upper and lower bounds.

**Interview questions**

<details><summary>Q1. Derive the time complexity of merge sort.</summary>

T(n) = 2T(n/2) + Θ(n): two half-size recursive sorts plus a linear merge. The recursion tree has log₂ n levels, and each level does Θ(n) total work, so the answer is Θ(n log n). With the Master Theorem: a = 2, b = 2, n^(log₂ 2) = n, f = Θ(n), which is Case 2, giving Θ(n log n).
</details>

<details><summary>Q2. What does the Master Theorem compare?</summary>

The combine cost f(n) against n^(log_b a), which is the number of leaves in the recursion tree. Whichever is polynomially larger dominates. If they're equal, every level contributes equally and you get an extra log n factor.
</details>

<details><summary>Q3. Why is worst-case quicksort Θ(n²) but best-case Θ(n log n)?</summary>

The worst case splits into sizes n − 1 and 0: T(n) = T(n − 1) + Θ(n), which unrolls to Θ(n²). The best case splits evenly: T(n) = 2T(n/2) + Θ(n), which is Θ(n log n).
</details>

**Worked problem: analyse `power(x, n)` by repeated squaring.**
```
power(x, n): if n == 0 return 1; h = power(x, n/2); return n even ? h*h : h*h*x
```
T(n) = T(n/2) + Θ(1) → Master Case 2 (a = 1, b = 2, f = Θ(1) = Θ(n⁰)) → **Θ(log n)** multiplications. Calling power(x, n/2) *twice* instead would give T(n) = 2T(n/2) + Θ(1), which is Θ(n), and that's a classic interview mistake. This algorithm reappears as [modular exponentiation](../09-Number-Theoretic-Algorithms/modular-exponentiation.md).

**Coding problems**
- LeetCode 50 · Pow(x, n) *(verify link)*
- LeetCode 912 · Sort an Array (implement merge sort) *(verify link)*
- LeetCode 241 · Different Ways to Add Parentheses *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. Recursive algorithms produce recurrences. Solve them to get a closed-form complexity.
2. Recursion tree: sum each level, then sum over the levels.
3. Master Theorem: compare f(n) with n^(log_b a), which gives leaves, equal levels or root.
4. Subtractive recurrences (T(n − 1)) aren't covered by the Master Theorem. Unroll them instead.
5. Substitution proofs must prove the **exact** inductive hypothesis.

**Formulas**
- a^(log_b n) = n^(log_b a)
- Geometric series: Σ rʲ = Θ(1) if r < 1, Θ(k) if r = 1, Θ(rᵏ) if r > 1 (for k terms)
- Simplified Master Theorem: aT(n/b) + Θ(nᵈ)

**Common mistakes**
- Applying the Master Theorem to T(n − 1) forms.
- Forgetting the regularity condition in Case 3.
- Calling a function twice when you could reuse its result (exponential slowdown).

**Quiz**
1. T(n) = 9T(n/3) + n?
2. T(n) = T(2n/3) + 1?
3. T(n) = 2T(n/2) + n²?
4. Why can't the Master Theorem solve T(n) = 2T(n/2) + n/log n?

**Answers**

<details><summary>Show answers</summary>

1. Θ(n²): Case 1, since n^(log₃ 9) = n² and f = n is polynomially smaller.
2. Θ(log n): Case 2, with a = 1, b = 3/2, n⁰ = 1, f = 1.
3. Θ(n²): Case 3. The regularity check is 2(n/2)² = n²/2 ≤ ½·n². ✓
4. n/log n is smaller than n, but not by a polynomial factor nᵉ, so it falls in the gap between Case 1 and Case 2. (The answer is Θ(n log log n).)
</details>

**Related topics:** [Divide and conquer](divide-and-conquer.md) · [Merge sort](../01-Sorting-and-Order-Statistics/comparison-sorts/merge-sort.md) · [Quicksort analysis](../01-Sorting-and-Order-Statistics/quicksort/complexity-analysis.md) · [Strassen](../08-Advanced-Algorithm-Topics/matrix-operations/strassens-algorithm.md) · [Summations](../14-Mathematical-Foundations/summations.md)
