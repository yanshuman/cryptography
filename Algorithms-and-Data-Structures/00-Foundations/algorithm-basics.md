# Algorithm Basics

> **CLRS:** Chapter 1 (The Role of Algorithms in Computing), Chapter 2.1–2.2 · **Status:** ✅ Written · **Next:** [Time complexity](time-complexity.md)

---

## A. Introduction

### What is an algorithm?

An **algorithm** is a finite, well-defined sequence of steps that takes some **input** and produces an **output** that solves a specific **problem**.

- A **problem** describes *what* we want. For example, the **sorting problem**: given a sequence of n numbers ⟨a₁, a₂, …, aₙ⟩, output a permutation ⟨a′₁, a′₂, …, a′ₙ⟩ such that a′₁ ≤ a′₂ ≤ … ≤ a′ₙ.
- An **instance** is one concrete input, for example ⟨31, 41, 59, 26, 41, 58⟩.
- An **algorithm** describes *how* to get the answer, and must work for **every** instance, not just one.
- An algorithm is **correct** if, for every input instance, it **halts** with the correct output. An incorrect algorithm might stop with the wrong answer, or never stop at all.

### What problem does "studying algorithms" solve?

Computers are fast, but not infinitely fast, and memory is cheap but not free. Two correct algorithms for the same problem can differ enormously:

| Sorting n = 10 million numbers | Steps (roughly) | At 10⁹ simple steps per second |
|---|---|---|
| Insertion sort, about n²/2 steps | 5 × 10¹³ | ≈ **14 hours** |
| Merge sort, about n log₂ n steps | 2.3 × 10⁸ | ≈ **0.23 seconds** |

No hardware upgrade closes a gap like that. **Choosing the right algorithm matters more than choosing a faster computer.**

### Where algorithms appear in real software

| Area | Algorithm examples |
|---|---|
| Databases | B-trees for indexes, hash joins, external merge sort |
| Maps and navigation | Dijkstra, A* shortest paths |
| Networking | Routing (Bellman-Ford, Dijkstra), max-flow for capacity planning |
| Security | RSA, modular exponentiation, hashing |
| Compression | Huffman coding (ZIP, JPEG), FFT (audio and image) |
| Search engines | String matching, sorting, PageRank (linear algebra) |
| Compilers | Graph colouring for register allocation, topological sort for build order |

### Prerequisites

Basic Java (loops, arrays, methods) and mathematical induction (useful for Section D, explained below).

---

## B. Intuition

Think of an algorithm as a **recipe**:

| Recipe | Algorithm |
|---|---|
| Ingredients | Input |
| Steps in order | Instructions |
| The finished dish | Output |
| "Works in any kitchen" | Works for every valid input |
| "Ready in 30 minutes" | Running time |
| "Needs one big bowl" | Memory (space) |

A good recipe is **unambiguous** ("stir for 2 minutes", not "stir a bit") and **finishes**. Algorithms have the same requirements.

**Five properties every algorithm must have** (Knuth's classic list):

1. **Finiteness:** it stops after a finite number of steps.
2. **Definiteness:** each step is precise and unambiguous.
3. **Input:** zero or more inputs from a defined set.
4. **Output:** at least one output with a defined relationship to the input.
5. **Effectiveness:** each step is basic enough to actually be carried out.

**Two questions we ask about every algorithm:**

1. **Is it correct?** Does it always give the right answer? (Proved with *loop invariants* or *induction*.)
2. **Is it efficient?** How do time and memory grow as the input grows? (Measured with *asymptotic analysis*; see [Time complexity](time-complexity.md).)

---

## C. How it works internally

We'll use one tiny problem through this page: **find the largest number in an array.**

> **Input:** A = [3, 8, 2, 9, 4]
> **Output:** 9

**Idea:** keep a "best so far". Look at each element once. If it's bigger than the best so far, it becomes the new best.

| Step | i | A[i] | Compare with best | best after this step |
|---|---|---|---|---|
| start | – | – | best = A[0] | **3** |
| 1 | 1 | 8 | 8 > 3 ✓ | **8** |
| 2 | 2 | 2 | 2 > 8 ✗ | 8 |
| 3 | 3 | 9 | 9 > 8 ✓ | **9** |
| 4 | 4 | 4 | 4 > 9 ✗ | 9 |
| end | | | | **return 9** |

**Edge cases to always think about:**

| Case | Example | What should happen |
|---|---|---|
| One element | [7] | Return 7 (the loop body never runs) |
| All equal | [5, 5, 5] | Return 5 |
| Negative numbers | [−4, −1, −9] | Return −1. **Don't** start best at 0! |
| Max at the start or end | [9, 1, 2] or [1, 2, 9] | Both must work |
| Empty array | [] | No maximum exists. Throw an exception or document a precondition. |

### The RAM model: what counts as "one step"?

To compare algorithms independently of hardware, CLRS uses the **Random-Access Machine (RAM) model**:

- Instructions run **one after another** (no parallelism).
- Each **simple operation** (+, −, ×, ÷, comparison, assignment, array access, function call or return) takes **constant time**.
- Memory access to any address takes constant time.
- Data values (integers) fit in a fixed-size word.

We count these simple steps to measure running time. That's what makes "this takes about n²/2 steps" meaningful.

---

## D. Algorithm and pseudocode

### CLRS-style pseudocode conventions

CLRS pseudocode is close to real code but ignores language details:

| Convention | Meaning |
|---|---|
| Indentation | Shows block structure (no braces) |
| `for i ← 2 to n` | Loop with i = 2, 3, …, n (inclusive) |
| `A[1 .. n]` | CLRS arrays are **1-indexed** (Java is 0-indexed, so watch out) |
| `←` | Assignment |
| `▷` | Comment |
| `length[A]` or `A.length` | Number of elements |
| Variables | Local to the procedure |
| Objects | Passed by pointer (like Java references) |

### FIND-MAX

```
FIND-MAX(A, n)
1   best ← A[1]
2   for i ← 2 to n
3       if A[i] > best
4           best ← A[i]
5   return best
```

| Line | Explanation |
|---|---|
| 1 | Start with the first element as the best seen. This avoids the "start at 0" bug with negative numbers. |
| 2 | Visit the remaining elements, left to right. |
| 3–4 | Replace best whenever a bigger element appears. |
| 5 | After the loop, best is the maximum of the whole array. |

### Proving correctness with a loop invariant

A **loop invariant** is a statement that is true **before every iteration** of a loop. It's the main tool for proving loops correct. We must show three things (just like induction):

| Part | What to show | Analogy to induction |
|---|---|---|
| **Initialization** | It's true before the first iteration | Base case |
| **Maintenance** | If it's true before an iteration, it's true before the next one | Inductive step |
| **Termination** | When the loop ends, the invariant gives us a useful property, usually that the answer is correct | Conclusion |

**Invariant for FIND-MAX:** *At the start of each iteration with index i, `best` is the maximum of A[1 .. i − 1].*

- **Initialization:** before the first iteration, i = 2, and best = A[1], which is the max of A[1 .. 1]. ✓
- **Maintenance:** suppose best = max(A[1 .. i − 1]). The body compares A[i] with best and keeps the larger. So afterwards best = max(A[1 .. i]), which is exactly the invariant for the next value i + 1. ✓
- **Termination:** the loop ends when i = n + 1. Substituting into the invariant: best = max(A[1 .. n]), the maximum of the whole array. ✓

**Why it terminates:** i starts at 2, increases by 1 each iteration, and the loop stops when i > n. That's exactly n − 1 iterations, which is finite.

### A second example: linear search

**Problem:** given A[1 .. n] and a value v, return an index i with A[i] = v, or NIL if v isn't present.

```
LINEAR-SEARCH(A, n, v)
1   for i ← 1 to n
2       if A[i] = v
3           return i
4   return NIL
```

**Invariant:** at the start of iteration i, v is not in A[1 .. i − 1]. At termination without returning, i = n + 1, so v isn't in A[1 .. n], and returning NIL is correct.

---

## E. Implementation

```java
import java.util.Arrays;

/** FIND-MAX and LINEAR-SEARCH from CLRS Ch. 2, with a runtime check of the loop invariant. */
public class AlgorithmBasics {

    /** Returns the largest element. Precondition: a.length >= 1. */
    static int findMax(int[] a) {
        if (a == null || a.length == 0) {
            throw new IllegalArgumentException("array must contain at least one element");
        }
        int best = a[0];                       // NOT 0: the array may be all negative
        for (int i = 1; i < a.length; i++) {
            // Invariant: best == max(a[0..i-1])
            assert best == maxOfPrefix(a, i) : "invariant broken at i=" + i;
            if (a[i] > best) {
                best = a[i];
            }
        }
        return best;
    }

    /** Returns the index of v in a, or -1 if v is absent (Java's version of NIL). */
    static int linearSearch(int[] a, int v) {
        for (int i = 0; i < a.length; i++) {
            if (a[i] == v) {
                return i;
            }
        }
        return -1;
    }

    // Slow helper used only to check the invariant in the assert above.
    private static int maxOfPrefix(int[] a, int len) {
        int m = a[0];
        for (int k = 1; k < len; k++) m = Math.max(m, a[k]);
        return m;
    }

    private static void check(boolean ok, String what) {
        System.out.println((ok ? "ok   " : "FAIL ") + what);
    }

    public static void main(String[] args) {
        int[] a = {3, 8, 2, 9, 4};
        System.out.println("A = " + Arrays.toString(a));
        System.out.println("findMax(A) = " + findMax(a));
        System.out.println("linearSearch(A, 9) = " + linearSearch(a, 9));
        System.out.println("linearSearch(A, 5) = " + linearSearch(a, 5));

        check(findMax(a) == 9, "max of [3,8,2,9,4] is 9");
        check(findMax(new int[]{7}) == 7, "single element");
        check(findMax(new int[]{-4, -1, -9}) == -1, "all negative");
        check(findMax(new int[]{5, 5, 5}) == 5, "all equal");
        check(linearSearch(a, 9) == 3, "9 found at index 3");
        check(linearSearch(a, 5) == -1, "5 not found");
        try {
            findMax(new int[0]);
            check(false, "empty array should throw");
        } catch (IllegalArgumentException e) {
            check(true, "empty array throws IllegalArgumentException");
        }
    }
}
```

**Output:**

```
A = [3, 8, 2, 9, 4]
findMax(A) = 9
linearSearch(A, 9) = 3
linearSearch(A, 5) = -1
ok   max of [3,8,2,9,4] is 9
ok   single element
ok   all negative
ok   all equal
ok   9 found at index 3
ok   5 not found
ok   empty array throws IllegalArgumentException
```

**Dry run of `findMax([3, 8, 2, 9, 4])`:** best = 3 → i = 1: 8 > 3, so best = 8 → i = 2: 2 > 8 is false → i = 3: 9 > 8, so best = 9 → i = 4: 4 > 9 is false → return 9.

**Java-specific notes:**
- **Indexing:** CLRS is 1-indexed (`for i ← 2 to n`). Java is 0-indexed (`for (int i = 1; i < a.length; i++)`). Translate carefully, because this is the most common source of off-by-one bugs.
- **`assert` is off by default.** Run with `java -ea` to enable it. Asserts are for checking invariants during development, not for validating user input (use exceptions for that).
- **NIL:** Java has no NIL for `int`. Use −1 for "not found" (as `String.indexOf` does), or return `Integer`/`Optional`.
- **Library equivalents:** `Arrays.stream(a).max().getAsInt()`, `Collections.max(list)`.

**Common mistakes:**
1. `int best = 0;`, which is wrong for all-negative arrays.
2. `int best = Integer.MIN_VALUE;` works but hides the empty-array case: it returns MIN_VALUE as if it were a real element.
3. Looping `i <= a.length`, which throws `ArrayIndexOutOfBoundsException`.

---

## F. Time complexity

Count the RAM-model steps for FIND-MAX on n elements:

| Line | Cost per execution | Times executed |
|---|---|---|
| 1 `best ← A[1]` | c₁ | 1 |
| 2 loop test | c₂ | n (n − 1 passes plus the final failed test) |
| 3 comparison | c₃ | n − 1 |
| 4 assignment | c₄ | between **0** and **n − 1** |
| 5 return | c₅ | 1 |

**Total:** T(n) = c₁ + c₂n + c₃(n − 1) + c₄·t + c₅, where t = number of times line 4 runs.

- **Best case** (max at the start, t = 0): T(n) = (c₂ + c₃)n + const, which is a linear function of n.
- **Worst case** (strictly increasing array, t = n − 1): T(n) = (c₂ + c₃ + c₄)n + const, which is **still linear**.

Both are **Θ(n)**: the running time grows proportionally to n. We drop constants and lower-order terms because, for large n, only the growth rate matters. The exact rules are in [Asymptotic notation](asymptotic-notation.md).

**Linear search:** the best case is Θ(1) (found at index 0), and the worst case is Θ(n) (absent, or the last element). The **average case**, if v is present and equally likely to be at any position, is (1 + 2 + … + n)/n = (n + 1)/2 comparisons, which is Θ(n).

**Can we do better than Θ(n) for max?** No. Any correct algorithm must look at every element: an element it never examines could be the maximum. So Ω(n) is a **lower bound**, and FIND-MAX is optimal.

## G. Space complexity

- **Input space:** n integers. This isn't counted as auxiliary space.
- **Auxiliary space:** two variables (best and i), so **Θ(1)**, regardless of n.
- **Recursion stack:** none, because the algorithm is iterative.
- FIND-MAX is **in place**: it uses only a constant amount of extra memory.

## H. Complexity summary

| Algorithm | Best time | Average time | Worst time | Auxiliary space |
|---|---|---|---|---|
| FIND-MAX | Θ(n) | Θ(n) | Θ(n) | Θ(1) |
| LINEAR-SEARCH | Θ(1) | Θ(n) | Θ(n) | Θ(1) |
| (Binary search, sorted input, for comparison) | Θ(1) | Θ(log n) | Θ(log n) | Θ(1) iterative |

## I. Advantages, limitations, and comparisons

- **Linear search** works on **unsorted** data and on linked lists, but is Θ(n) per query. If you search many times, sorting once (Θ(n log n)) and then using **binary search** (Θ(log n) per query) or a **hash set** (Θ(1) expected) is far better.
- **Correctness first, efficiency second.** A fast wrong answer is worthless. Loop invariants make correctness arguments rigorous.
- **The RAM model is an approximation.** Real machines have caches: scanning an array sequentially is much faster per element than chasing pointers in a linked list, even though both are Θ(n). Section I of each topic discusses these practical effects.

**Interview follow-ups:**
- "Find both max and min with fewer comparisons." Use pairs: about 3n/2 comparisons instead of 2n − 2. See [Minimum and maximum](../01-Sorting-and-Order-Statistics/order-statistics/minimum-and-maximum.md).
- "Find the second largest." Do one pass tracking the top two, Θ(n).
- "Search in a sorted array." Use binary search, Θ(log n).

---

## J. Practice

**Beginner**
1. Write `findMin` and give its loop invariant.
2. Trace FIND-MAX on [−3, −7, −1, −8]. Show `best` after each iteration.
3. Count how many times line 4 of FIND-MAX runs on [1, 2, 3, 4, 5] and on [5, 4, 3, 2, 1].

**Intermediate**
4. Write a loop invariant for "sum of the array" and prove all three parts.
5. Write `countOccurrences(a, v)`. What are its best and worst case times?
6. Modify linear search to return the **last** index of v. What's the invariant?

**Advanced**
7. *(CLRS Exercise 2.1-3.)* Give a formal loop-invariant proof that LINEAR-SEARCH is correct.
8. Suppose each element is a random permutation of 1..n. What is the **expected** number of times line 4 of FIND-MAX runs? *(Hint: element i is a new maximum with probability 1/i, so the answer is the harmonic number Hₙ − 1 ≈ ln n.)*

**Interview questions**

<details><summary>Q1. What's the difference between an algorithm and a program?</summary>

An algorithm is the abstract, language-independent method. A program is a concrete implementation of it in a specific language that runs on a real machine. One algorithm (say, merge sort) can be implemented as many programs.
</details>

<details><summary>Q2. Why do we analyse algorithms instead of just timing them?</summary>

Timings depend on hardware, the language, the compiler, other running processes and the particular input. Asymptotic analysis predicts how running time **grows** with input size, independent of the machine. That tells you whether the program will still work when the data is 1000× bigger.
</details>

<details><summary>Q3. What is a loop invariant, and why is it useful?</summary>

A property that holds before every iteration of a loop. Proving initialization, maintenance and termination shows the loop computes the right answer. It's mathematical induction applied to a loop. It's also a debugging tool: assert the invariant and the bug location reveals itself.
</details>

**Worked problem: second largest element in one pass**

Keep `first` and `second`. For each x: if x > first, then second = first and first = x. Otherwise, if x > second and x ≠ first, then second = x. The invariant is that after processing a[0..i−1], `first` and `second` are the two largest distinct values seen. Time Θ(n), space Θ(1).

**Coding problems**
- LeetCode 414 · Third Maximum Number *(verify link)*
- LeetCode 1 · Two Sum (linear scan + hash set) *(verify link)*
- LeetCode 704 · Binary Search *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. An algorithm is a finite, unambiguous procedure that must be correct for **every** input instance.
2. Algorithm choice beats hardware: Θ(n²) vs Θ(n log n) is hours vs seconds for large n.
3. Loop invariants prove correctness in three steps: initialization, maintenance, termination.
4. In the RAM model, each simple operation costs constant time, so we count operations.
5. Always check edge cases: empty, single element, all equal, negatives, extremes at either end.

**Formulas**
- FIND-MAX: exactly n − 1 comparisons, so Θ(n), which is optimal.
- Linear search average (present, uniform position): (n + 1)/2 comparisons.

**Common mistakes**
- Initialising max to 0.
- Off-by-one when translating 1-indexed pseudocode to 0-indexed Java.
- Forgetting to prove termination.

**Quiz**
1. What are the three parts of a loop-invariant proof?
2. What is FIND-MAX's best-case running time?
3. Why must any max-finding algorithm take Ω(n) time?
4. What does the RAM model assume about memory access?

**Answers**

<details><summary>Show answers</summary>

1. Initialization, maintenance, termination.
2. Θ(n). It still has to compare every element; only the number of assignments changes.
3. Any element it doesn't examine could be the maximum.
4. Any memory location can be read or written in constant time.
</details>

**Related topics:** [Time complexity](time-complexity.md) · [Asymptotic notation](asymptotic-notation.md) · [Insertion sort](../01-Sorting-and-Order-Statistics/comparison-sorts/insertion-sort.md) (CLRS's first fully analysed algorithm) · [Minimum and maximum](../01-Sorting-and-Order-Statistics/order-statistics/minimum-and-maximum.md)
