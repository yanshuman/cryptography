# Asymptotic Notation (O, Ω, Θ, o, ω)

> **CLRS:** Chapter 3 (Growth of Functions) · **Status:** ✅ Written · **Prerequisites:** [Time complexity](time-complexity.md) · **Next:** [Recurrence relations](recurrence-relations.md)

---

## A. Introduction

**Asymptotic notation** is the precise mathematical language for "how fast a function grows" as n → ∞. It lets us say *"merge sort is Θ(n log n)"* and mean something exact.

**What problem does it solve?** Exact operation counts like T(n) = 3n² + 7n + 12 depend on details we don't care about (the cost of one comparison, the compiler). Asymptotic notation keeps only the **growth rate**, so we can compare algorithms cleanly and prove statements about them.

**Why we need it:** every complexity claim in this repository, in interviews and in papers uses O, Ω or Θ. Using them precisely avoids common errors, such as saying "Big-O means worst case" (it doesn't).

**Prerequisites:** functions, inequalities, logarithms and exponents.

---

## B. Intuition

Picture two runners where only **eventually** matters:

- **f = O(g):** "f grows no faster than g". Past some starting line, f stays below a constant multiple of g. It's a **ceiling**.
- **f = Ω(g):** "f grows at least as fast as g". It's a **floor**.
- **f = Θ(g):** both at once, so f is **sandwiched** between two multiples of g. It's the **exact growth rate**.
- **f = o(g):** f becomes **negligible** compared with g (strictly slower).
- **f = ω(g):** f **dominates** g (strictly faster).

It's like comparing real numbers:

| Asymptotic | Like comparing numbers |
|---|---|
| f = O(g) | a ≤ b |
| f = Ω(g) | a ≥ b |
| f = Θ(g) | a = b |
| f = o(g) | a < b |
| f = ω(g) | a > b |

*"Eventually"* matters because of small inputs: 100n is bigger than n² for n < 100, but for every n ≥ 100, n² wins. Asymptotics only care about large n.

---

## C. How it works internally: the formal definitions

### Θ-notation (tight bound)

> **Θ(g(n))** = { f(n) : there exist positive constants c₁, c₂, n₀ such that 0 ≤ c₁·g(n) ≤ f(n) ≤ c₂·g(n) for all n ≥ n₀ }

```
  f(n) is trapped between c₁·g(n) and c₂·g(n) once n ≥ n₀

  value
    │                      c₂·g(n)
    │                 ....''''
    │            ..''      f(n)
    │        .'' ___----'''
    │     .'_--''        c₁·g(n)
    │   .-'  ___----''''
    │ _-___--'
    └──────┼────────────────────── n
          n₀
```

### O-notation (upper bound)

> **O(g(n))** = { f(n) : there exist positive constants c, n₀ such that 0 ≤ f(n) ≤ c·g(n) for all n ≥ n₀ }

### Ω-notation (lower bound)

> **Ω(g(n))** = { f(n) : there exist positive constants c, n₀ such that 0 ≤ c·g(n) ≤ f(n) for all n ≥ n₀ }

### o and ω (strict, "not tight")

> **o(g(n))**: for **every** constant c > 0 there is an n₀ with 0 ≤ f(n) < c·g(n) for all n ≥ n₀. Equivalently, lim f/g = 0.
> **ω(g(n))**: for **every** c > 0 there is an n₀ with f(n) > c·g(n) ≥ 0. Equivalently, lim f/g = ∞.

The difference between O and o: O needs the inequality for **some** c, while o needs it for **every** c. So 2n² = O(n²) but 2n² ≠ o(n²). On the other hand, 2n = o(n²).

### Key theorem

> **f(n) = Θ(g(n)) if and only if f(n) = O(g(n)) and f(n) = Ω(g(n)).**

### Worked proof 1: 3n² + 10n + 5 = Θ(n²)

We need c₁, c₂, n₀ with c₁n² ≤ 3n² + 10n + 5 ≤ c₂n² for n ≥ n₀.

- **Lower bound:** 3n² + 10n + 5 ≥ 3n² for all n ≥ 1, so take **c₁ = 3**.
- **Upper bound:** for n ≥ 1, 10n ≤ 10n² and 5 ≤ 5n². So 3n² + 10n + 5 ≤ 3n² + 10n² + 5n² = 18n². Take **c₂ = 18**.
- **n₀ = 1.** ∎

The constants aren't unique. Any valid choice proves the claim.

### Worked proof 2: n² ≠ O(n)

Suppose n² ≤ c·n for all n ≥ n₀. Dividing by n gives n ≤ c, but n grows without bound while c is fixed. That's a contradiction once n > c. ∎

### Worked proof 3: ½n² − 3n = Θ(n²) (from CLRS)

We need c₁n² ≤ ½n² − 3n ≤ c₂n². Divide by n²: c₁ ≤ ½ − 3/n ≤ c₂.
- The right side holds for c₂ = ½ (because ½ − 3/n ≤ ½).
- The left side: at n = 7, ½ − 3/7 = 1/14. The expression increases with n, so c₁ = 1/14 works for n ≥ 7.
- So c₁ = 1/14, c₂ = ½, n₀ = 7. ∎

### The limit test (the fastest method in practice)

If L = lim_{n→∞} f(n)/g(n) exists:

| L | Conclusion |
|---|---|
| 0 | f = o(g), so also O(g) but not Θ |
| 0 < L < ∞ | f = Θ(g) |
| ∞ | f = ω(g), so also Ω(g) but not Θ |

Use **L'Hôpital's rule** when both functions go to ∞. Example: lim (ln n)/n = lim (1/n)/1 = 0, so **log n = o(n)**.

### The growth hierarchy (from slowest to fastest)

> 1 ≺ log log n ≺ log n ≺ (log n)² ≺ √n ≺ n ≺ n log n ≺ n² ≺ n³ ≺ 2ⁿ ≺ 3ⁿ ≺ n! ≺ nⁿ

Here ≺ means "is o() of". Three facts you must know:

1. **Any power of log is smaller than any power of n:** (log n)ᵏ = o(nᵉ) for every k and every ε > 0. Even (log n)¹⁰⁰ = o(n^0.01).
2. **Any polynomial is smaller than any exponential:** nᵏ = o(aⁿ) for every k and every a > 1.
3. **The base of a log doesn't matter:** log_a n = log_b n / log_b a, which only differs by a constant. So we write just "log n". But **the base of an exponential does matter**: 2ⁿ = o(3ⁿ), because 3ⁿ/2ⁿ = 1.5ⁿ → ∞.

### Standard functions and identities

| Identity | Use |
|---|---|
| a^(log_b c) = c^(log_b a) | Master theorem manipulations |
| log(n!) = Θ(n log n) | Sorting lower bound ([Lower bounds](../01-Sorting-and-Order-Statistics/linear-time-sorting/lower-bounds.md)) |
| n! ≈ √(2πn)·(n/e)ⁿ (Stirling) | Bounds on n! |
| log₂(2ⁿ) = n, 2^(log₂ n) = n | Converting between log and exponent |
| ⌈n/2⌉ + ⌊n/2⌋ = n | Splitting arrays in divide and conquer |
| lg* n (iterated log) ≤ 5 for every n < 2^65536 | Union-find analysis |

### Properties

| Property | Holds for | Example |
|---|---|---|
| **Transitivity** | O, Ω, Θ, o, ω | f = O(g), g = O(h) ⇒ f = O(h) |
| **Reflexivity** | O, Ω, Θ | f = Θ(f) |
| **Symmetry** | Θ only | f = Θ(g) ⇔ g = Θ(f) |
| **Transpose symmetry** | O ↔ Ω, o ↔ ω | f = O(g) ⇔ g = Ω(f) |
| **Sum rule** | | O(f) + O(g) = O(max(f, g)) |
| **Product rule** | | O(f)·O(g) = O(f·g) |

**Not every pair of functions is comparable.** For example, n and n^(1 + sin n) oscillate, so neither is O of the other. This almost never comes up with real algorithms.

---

## D. Algorithm and pseudocode: how to use notation correctly

### A procedure for proving f = Θ(g)

```
PROVE-THETA(f, g)
1  Try the limit L = lim f(n)/g(n).
2  if 0 < L < ∞ then f = Θ(g)  ▷ done
3  otherwise, find constants directly:
4     Lower bound: drop positive lower-order terms → f(n) ≥ c₁·g(n)
5     Upper bound: bound each lower-order term by a multiple of g(n) → f(n) ≤ c₂·g(n)
6     Choose n₀ so both hold, e.g. so negative terms are small enough
```

### Using notation correctly with algorithms

| Statement | Correct? | Why |
|---|---|---|
| "Insertion sort is O(n²)" | ✅ | Its running time on **every** input is ≤ c·n² |
| "Insertion sort is O(n³)" | ✅ but useless | O is just an upper bound. Loose bounds are true but uninformative. |
| "Insertion sort is Θ(n²)" | ❌ as stated | Its best case is Θ(n). Say "**worst-case** running time is Θ(n²)". |
| "Insertion sort is Ω(n)" | ✅ | Every input needs at least ~n steps |
| "Merge sort is Θ(n log n)" | ✅ | Best and worst case are both Θ(n log n) |
| "Big-O means worst case" | ❌ | O is an upper bound on a **function**. You can give O, Ω or Θ for best, worst or average cases independently. |
| "The worst case of quicksort is Ω(n log n)" | ✅ but weak | Its worst case is actually Θ(n²) |

**Rule:** always say *which case* (worst, best, average, expected, amortized) **and** which bound (O, Ω, Θ). For example: "quicksort's worst-case running time is Θ(n²), and its expected running time is Θ(n log n)."

### Notation in equations

- **T(n) = 2T(n/2) + Θ(n)** means "plus some function that is Θ(n)".
- **2n² + Θ(n) = Θ(n²)** means "whatever the anonymous Θ(n) function is, the left side is Θ(n²)".
- The "=" here really means "∈" (is a member of the set). It's a one-way statement, so don't flip it: Θ(n²) = 2n² + Θ(n) isn't how it's written.

---

## E. Implementation

The program checks the constants from the worked proofs numerically, and prints growth tables that show the hierarchy and the limit test in action.

```java
/** Numerically checks asymptotic-bound constants and shows the growth hierarchy. */
public class AsymptoticDemo {

    static double f1(double n) { return 3 * n * n + 10 * n + 5; }   // proof 1: Theta(n^2), c1=3, c2=18, n0=1
    static double f3(double n) { return 0.5 * n * n - 3 * n; }       // proof 3: Theta(n^2), c1=1/14, c2=1/2, n0=7

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        boolean p1 = true, p3 = true;
        for (long n = 1; n <= 1_000_000; n++) {
            double g = (double) n * n;
            if (!(3 * g <= f1(n) && f1(n) <= 18 * g)) p1 = false;
            if (n >= 7 && !(g / 14 <= f3(n) && f3(n) <= 0.5 * g)) p3 = false;
        }
        check(p1, "3n^2 + 10n + 5 is between 3n^2 and 18n^2 for all 1 <= n <= 10^6");
        check(p3, "n^2/2 - 3n is between n^2/14 and n^2/2 for all 7 <= n <= 10^6");
        check(f3(6) < 36.0 / 14, "...and the lower bound really fails at n = 6 (so n0 = 7 is needed)");

        System.out.println();
        System.out.println("Limit test: f(n)/g(n) as n grows");
        System.out.printf("%10s %14s %14s %14s %14s%n", "n", "log2n / n", "f1 / n^2", "n^3 / 2^n", "2^n / 3^n");
        for (int n : new int[]{10, 20, 40, 80, 160}) {
            double log2 = Math.log(n) / Math.log(2);
            System.out.printf("%10d %14.6f %14.6f %14.3e %14.3e%n",
                    n, log2 / n, f1(n) / ((double) n * n), Math.pow(n, 3) / Math.pow(2, n), Math.pow(2.0 / 3, n));
        }
        check(Math.log(1e9) / 1e9 < 1e-7, "log n / n -> 0, so log n = o(n)");
        check(Math.abs(f1(1e9) / 1e18 - 3) < 1e-6, "f1 / n^2 -> 3, a positive constant, so f1 = Theta(n^2)");
        check(Math.pow(160, 3) / Math.pow(2, 160) < 1e-40, "n^3 / 2^n -> 0, so polynomials = o(exponentials)");

        System.out.println();
        System.out.println("Growth table (values rounded)");
        System.out.printf("%8s %8s %10s %12s %14s %16s %12s%n", "n", "log2n", "sqrt n", "n log2n", "n^2", "n^3", "2^n");
        for (int n : new int[]{8, 64, 1024, 65536}) {
            double lg = Math.log(n) / Math.log(2);
            System.out.printf("%8d %8.0f %10.0f %12.0f %14.0f %16.3e %12s%n",
                    n, lg, Math.sqrt(n), n * lg, (double) n * n, Math.pow(n, 3),
                    n <= 64 ? String.format("%.3e", Math.pow(2, n)) : "astronomical");
        }

        // The base of a logarithm only changes a constant factor:
        double ratio = (Math.log(1e6) / Math.log(2)) / Math.log10(1e6);
        System.out.printf("%nlog2(n) / log10(n) = %.4f for every n (= log2 10)%n", ratio);
        check(Math.abs(ratio - Math.log(10) / Math.log(2)) < 1e-12, "log bases differ by a constant factor");
    }
}
```

**Output:**

```
ok   3n^2 + 10n + 5 is between 3n^2 and 18n^2 for all 1 <= n <= 10^6
ok   n^2/2 - 3n is between n^2/14 and n^2/2 for all 7 <= n <= 10^6
ok   ...and the lower bound really fails at n = 6 (so n0 = 7 is needed)

Limit test: f(n)/g(n) as n grows
         n      log2n / n       f1 / n^2      n^3 / 2^n      2^n / 3^n
        10       0.332193       4.050000      9.766e-01      1.734e-02
        20       0.216096       3.512500      7.629e-03      3.007e-04
        40       0.133048       3.253125      5.821e-08      9.044e-08
        80       0.079024       3.125781      4.235e-19      8.179e-15
       160       0.045762       3.062695      2.803e-42      6.690e-29
ok   log n / n -> 0, so log n = o(n)
ok   f1 / n^2 -> 3, a positive constant, so f1 = Theta(n^2)
ok   n^3 / 2^n -> 0, so polynomials = o(exponentials)

Growth table (values rounded)
       n    log2n     sqrt n      n log2n            n^2              n^3          2^n
       8        3          3           24             64        5.120e+02    2.560e+02
      64        6          8          384           4096        2.621e+05    1.845e+19
    1024       10         32        10240        1048576        1.074e+09 astronomical
   65536       16        256      1048576     4294967296        2.815e+14 astronomical

log2(n) / log10(n) = 3.3219 for every n (= log2 10)
ok   log bases differ by a constant factor
```

**Reading the output:**
- f1/n² approaches **3**, the leading coefficient. That's a positive constant, so the limit test says **Θ(n²)**.
- n³/2ⁿ collapses to about 10⁻⁴² by n = 160: exponentials crush polynomials.
- 2⁶⁴ ≈ 1.8 × 10¹⁹. That's why exponential algorithms die before n ≈ 60.

**Java-specific notes:** use `double` (or `BigInteger`) when evaluating fast-growing functions, because `long` overflows at 2⁶³. `Math.log` is the natural log, so log₂ x = `Math.log(x) / Math.log(2)`. For integers, the exact ⌊log₂ x⌋ is `31 - Integer.numberOfLeadingZeros(x)`.

---

## F. Time complexity

Asymptotic notation *describes* time complexity rather than having one. How each bound is used:

| You want to say… | Use |
|---|---|
| "It never takes longer than…" (a guarantee) | **O**, on the worst case |
| "It always takes at least…" (a lower bound for an algorithm, or for *every* algorithm solving a problem) | **Ω** |
| "This is exactly how it grows" | **Θ** |
| "It's strictly faster than…" | **o** |

**Lower bounds for problems vs algorithms:** "sorting is Ω(n log n)" (for comparison sorts) is a statement about **every possible algorithm**. It's much stronger than saying one algorithm is Ω(n log n). See [Lower bounds for sorting](../01-Sorting-and-Order-Statistics/linear-time-sorting/lower-bounds.md).

## G. Space complexity

The same notation applies to memory: "merge sort uses Θ(n) auxiliary space", "heapsort uses O(1)".

## H. Complexity summary

| Notation | Definition (for some/all constants) | Limit f/g | Analogy |
|---|---|---|---|
| f = O(g) | ∃c, n₀: f ≤ cg | < ∞ | ≤ |
| f = Ω(g) | ∃c, n₀: f ≥ cg | > 0 | ≥ |
| f = Θ(g) | ∃c₁, c₂, n₀: c₁g ≤ f ≤ c₂g | 0 < L < ∞ | = |
| f = o(g) | ∀c ∃n₀: f < cg | = 0 | < |
| f = ω(g) | ∀c ∃n₀: f > cg | = ∞ | > |

## I. Advantages, limitations, and comparisons

**Advantages:** machine-independent, it focuses on what matters at scale, and it makes proofs possible.

**Limitations:**
- **It hides constants.** 10⁶·n is "better" than n², but slower for every n < 10⁶. Examples: [median of medians](../01-Sorting-and-Order-Statistics/order-statistics/median-of-medians.md) vs quickselect, and Fibonacci heaps vs binary heaps.
- **It hides lower-order terms** that can matter for moderate n.
- **It's about n → ∞.** Real inputs are finite: for n = 20, a 2ⁿ algorithm (about 10⁶ steps) is perfectly fine.
- **The model matters.** "O(1) arithmetic" breaks down for huge numbers (big integers in RSA), where arithmetic costs depend on bit length.

**Interview follow-ups:** "Is O(n) always better than O(n log n)?" (asymptotically, yes; for a given n, it depends on constants), and "What's the tight bound?" (they want Θ).

---

## J. Practice

**Beginner**
1. Prove 5n + 3 = Θ(n) with explicit c₁, c₂, n₀.
2. True or false: n = O(n²)? n² = O(n)? 2ⁿ⁺¹ = O(2ⁿ)? 2²ⁿ = O(2ⁿ)?
3. Order these by growth: n log n, n^1.5, 2^(log₂ n), log(n!), n/log n.

**Intermediate**
4. Use the limit test to compare √n and log² n.
5. Show max(f, g) = Θ(f + g) for non-negative f and g.
6. Is ⌈log₂ n⌉! polynomially bounded? *(CLRS Problem 3-3 style.)*

**Advanced**
7. Prove log(n!) = Θ(n log n) *without* Stirling. *(Hint: n! ≤ nⁿ, and n! ≥ (n/2)^(n/2).)*
8. Give two functions f and g such that neither f = O(g) nor g = O(f).

**Interview questions**

<details><summary>Q1. What is the difference between O, Ω and Θ?</summary>

O is an asymptotic upper bound (f grows no faster than g), Ω is a lower bound (f grows at least as fast), and Θ is both (exactly the same growth rate, up to constants). Θ is the most informative. O is the most commonly quoted, but it's often meant as Θ informally.
</details>

<details><summary>Q2. Why don't we care about the base of the logarithm?</summary>

log_a n = log_b n / log_b a, and 1/log_b a is a constant, so log₂ n and log₁₀ n differ only by a constant factor, which asymptotic notation ignores. The base of an **exponential** does matter: 2ⁿ vs 3ⁿ.
</details>

<details><summary>Q3. Does Big-O describe the worst case?</summary>

No. O is a bound on a function. You can bound the worst-case function, the best-case function or the average-case function, each with O, Ω or Θ. For example, quicksort's best-case time is Θ(n log n) and its worst case is Θ(n²).
</details>

**Worked problem: rank f(n) = n^(log n) against g(n) = 2ⁿ.** Take log₂ of both: log₂ f = (log₂ n)², and log₂ g = n. Since (log n)² = o(n), f = o(g). n^(log n) is "quasi-polynomial": bigger than any polynomial, smaller than any exponential.

**Coding problems:** none directly. Practise by stating the tight bound for every solution you write, for example on LeetCode 70 · Climbing Stairs *(verify link)* (naive Θ(φⁿ) vs DP Θ(n)).

---

## K. Revision notes

**Five key takeaways**
1. Θ = tight, O = upper, Ω = lower, o and ω = strict versions.
2. Prove bounds by finding constants (c₁, c₂, n₀) or with the limit test.
3. log ≺ polynomial ≺ exponential ≺ factorial. The base of a log doesn't matter, but the base of an exponential does.
4. Always state both the **case** (worst, average and so on) and the **bound** (O, Ω, Θ).
5. Asymptotics hide constants, so small-n behaviour can be different.

**Formulas**
- f = Θ(g) ⇔ f = O(g) and f = Ω(g)
- log(n!) = Θ(n log n)
- log_a n = Θ(log_b n)
- nᵏ = o(aⁿ) for a > 1, and (log n)ᵏ = o(nᵉ) for ε > 0

**Common mistakes**
- "Big-O = worst case".
- Writing O when you mean Θ in a proof.
- Assuming 2^(2n) = O(2ⁿ). It's false: 4ⁿ/2ⁿ = 2ⁿ → ∞.

**Quiz**
1. Is 2ⁿ⁺¹ = Θ(2ⁿ)?
2. Is log₂ n = Θ(log₁₀ n)?
3. What does lim f/g = 0 tell you?
4. Fill in the blank: n log n = ___(n²) (use o or ω).

**Answers**

<details><summary>Show answers</summary>

1. Yes: 2ⁿ⁺¹ = 2·2ⁿ, a constant multiple.
2. Yes: they differ by the constant factor log₂ 10 ≈ 3.32.
3. f = o(g), so f = O(g) but f ≠ Θ(g).
4. o: (n log n)/n² = (log n)/n → 0.
</details>

**Related topics:** [Time complexity](time-complexity.md) · [Recurrence relations](recurrence-relations.md) · [Summations](../14-Mathematical-Foundations/summations.md) · [Lower bounds for sorting](../01-Sorting-and-Order-Statistics/linear-time-sorting/lower-bounds.md)
