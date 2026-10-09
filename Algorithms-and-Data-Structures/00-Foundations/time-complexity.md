# Time Complexity

> **CLRS:** Chapter 2.2 (Analyzing algorithms), Chapter 3 · **Status:** ✅ Written · **Prerequisites:** [Algorithm basics](algorithm-basics.md) · **Next:** [Space complexity](space-complexity.md), [Asymptotic notation](asymptotic-notation.md)

---

## A. Introduction

**Time complexity** describes **how the running time of an algorithm grows as the input size grows**. It's a *function* T(n), where n is the input size, not a single number of seconds.

**What problem does it solve?** It lets us predict, before running anything, whether an algorithm will finish in time when the input is 10×, 1000× or 10⁶× bigger. It also lets us compare algorithms independently of the computer, language or compiler.

**Why we need it.** Interview questions, system design, and choosing between `ArrayList` and `LinkedList` or `HashMap` and `TreeMap` all come down to "how does the cost grow?". Online judges set time limits of about 10⁸ simple operations per second, so you must estimate complexity before you code.

**Prerequisites:** loops, recursion, and summations such as 1 + 2 + … + n = n(n + 1)/2.

---

## B. Intuition

Imagine looking for a friend's name:

| Method | Work for 100 names | Work for 1,000,000 names | Growth |
|---|---|---|---|
| Know the exact page (a hash table) | 1 step | 1 step | **constant** |
| Phone book, open in the middle and halve (binary search) | ~7 steps | ~20 steps | **logarithmic** |
| Read every name once (linear search) | 100 | 1,000,000 | **linear** |
| Compare every name with every other name (finding duplicates naively) | 10,000 | 10¹² | **quadratic** |

The *shape* of the growth is what time complexity captures. We ignore:
- **Constant factors.** 3n and 100n both grow linearly, and hardware or compiler changes affect constants anyway.
- **Lower-order terms.** In n² + 1000n, the n² term dominates once n is large.

That simplification is formalised by **Big-O, Ω and Θ** notation ([Asymptotic notation](asymptotic-notation.md)). The short version:

| Notation | Meaning | Plain English |
|---|---|---|
| **O(f(n))** | Upper bound | "grows **at most** like f(n)" |
| **Ω(f(n))** | Lower bound | "grows **at least** like f(n)" |
| **Θ(f(n))** | Tight bound | "grows **exactly** like f(n)", up to constants |

---

## C. How it works internally

### Step 1: Choose the input size n

| Problem | Natural input size |
|---|---|
| Sorting or searching an array | number of elements n |
| Multiplying two numbers | number of **bits** in the numbers |
| Graph algorithms | number of vertices **V** *and* edges **E** (two parameters) |
| String matching | text length n and pattern length m |
| Matrix multiplication | dimension n (an n × n matrix has n² entries) |

### Step 2: Count basic operations

In the RAM model, each comparison, arithmetic operation, assignment and array access costs constant time. We count how many run as a function of n.

### Step 3: Decide which case you're analysing

| Case | Meaning | Example: linear search for v |
|---|---|---|
| **Best case** | The input that makes the algorithm fastest | v is the first element: **Θ(1)** |
| **Worst case** | The input that makes it slowest. A **guarantee**. | v absent: **Θ(n)** |
| **Average case** | Expected time over a probability distribution of inputs | v at a uniformly random position: (n + 1)/2, so **Θ(n)** |
| **Expected (randomised)** | Expected time over the algorithm's **own random choices**, for the *worst* input | Randomised quicksort: **Θ(n log n)** expected on every input |
| **Amortized** | Average cost per operation over a worst-case **sequence** of operations | `ArrayList.add`: **Θ(1)** amortized, even though a resize costs Θ(n) |

**Why focus on the worst case?** (1) It's a guarantee. (2) For many algorithms the worst case happens often (searching for missing items). (3) The average case is often as bad as the worst case (for insertion sort both are Θ(n²)). (4) "Average" requires assuming an input distribution, which may not match reality.

**Average case vs expected time.** These are easy to confuse. **Average-case** analysis averages over *random inputs*, so a malicious input can still be slow. **Expected** running time of a *randomised* algorithm averages over its own coin flips, so it holds for **every** input. Randomised quicksort is Θ(n log n) expected even on sorted input. Deterministic quicksort is Θ(n log n) on average but Θ(n²) on sorted input.

### Step 4: Simplify to the dominant term

T(n) = 3n² + 10n + 50 → keep 3n², drop the constant → **Θ(n²)**.

### Growth rates you must know

For n = 1,000,000 at about 10⁸ operations per second:

| Complexity | Name | Operations at n = 10⁶ | Roughly | Typical example |
|---|---|---|---|---|
| Θ(1) | constant | 1 | instant | array access, hash lookup (expected) |
| Θ(log n) | logarithmic | 20 | instant | binary search, balanced-BST lookup |
| Θ(√n) | square root | 1,000 | instant | trial-division primality test |
| Θ(n) | linear | 10⁶ | 0.01 s | one scan, linear search |
| Θ(n log n) | linearithmic | 2 × 10⁷ | 0.2 s | merge sort, heapsort |
| Θ(n²) | quadratic | 10¹² | ~3 hours | insertion sort, all pairs |
| Θ(n³) | cubic | 10¹⁸ | ~300 years | naive matrix multiplication, Floyd-Warshall |
| Θ(2ⁿ) | exponential | 2^(10⁶) | never | all subsets |
| Θ(n!) | factorial | — | never | all permutations (brute-force TSP) |

**Rule of thumb for interviews and online judges** (1–2 seconds):

| Max n | Acceptable complexity |
|---|---|
| ≤ 10–12 | Θ(n!), Θ(2ⁿ · n) |
| ≤ 20–25 | Θ(2ⁿ) |
| ≤ 500 | Θ(n³) |
| ≤ 5,000 | Θ(n²) |
| ≤ 10⁶ | Θ(n log n) |
| ≤ 10⁸ | Θ(n) |
| larger | Θ(log n) or Θ(1) |

---

## D. Algorithm and pseudocode: a guide to analysing code

This is the core skill. Each pattern below shows the code, the derivation and the result. The Java program in Section E checks every result by counting.

### Pattern 1: Single loop, Θ(n)

```java
for (int i = 0; i < n; i++) { sum += a[i]; }      // body is Θ(1)
```
The body runs n times at constant cost each, so the total is **Θ(n)**.

### Pattern 2: Consecutive loops, add them

```java
for (int i = 0; i < n; i++) { ... }               // Θ(n)
for (int j = 0; j < n * n; j++) { ... }           // Θ(n²)
```
Θ(n) + Θ(n²) = **Θ(n²)**. For sequential blocks, the **largest** term wins.

### Pattern 3: Nested independent loops, multiply them

```java
for (int i = 0; i < n; i++)
    for (int j = 0; j < m; j++) { ... }           // body Θ(1)
```
The inner loop runs m times for each of n outer iterations: n · m, so **Θ(n·m)**, or Θ(n²) if m = n.

### Pattern 4: Dependent nested loops, use a summation

```java
for (int i = 0; i < n; i++)
    for (int j = i + 1; j < n; j++) { ... }       // inner runs n-1-i times
```
Total = Σᵢ₌₀ⁿ⁻¹ (n − 1 − i) = (n − 1) + (n − 2) + … + 0 = **n(n − 1)/2**, which is **Θ(n²)**.
This is the "all pairs" pattern (bubble, selection and insertion sort). Even though the inner loop shrinks, the total is still quadratic, just half of n².

### Pattern 5: Halving or doubling loop, Θ(log n)

```java
for (int i = 1; i < n; i *= 2) { ... }            // i = 1, 2, 4, 8, ...
```
After k iterations, i = 2ᵏ. The loop stops when 2ᵏ ≥ n, so k = ⌈log₂ n⌉. The result is **Θ(log n)**.
Any loop that multiplies or divides by a constant factor > 1 each time is logarithmic. The base doesn't matter asymptotically.

### Pattern 6: Binary search, Θ(log n)

```java
int lo = 0, hi = n - 1;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;   // avoids int overflow of (lo + hi)
    if (a[mid] == key) return mid;
    if (a[mid] < key) lo = mid + 1; else hi = mid - 1;
}
```
Each iteration halves the search range: n → n/2 → n/4 → … → 1. That's at most ⌊log₂ n⌋ + 1 iterations, so **Θ(log n)** worst case and Θ(1) best case.

### Pattern 7: Linear outer loop, logarithmic inner loop, Θ(n log n)

```java
for (int i = 0; i < n; i++)
    for (int j = 1; j < n; j *= 2) { ... }
```
n × log n = **Θ(n log n)**.

**A trap:** `for (i = 1; i <= n; i++) for (j = 1; j <= n; j += i)` looks like n², but the inner loop runs n/i times. The total is n(1 + 1/2 + 1/3 + … + 1/n) = n·Hₙ ≈ n ln n, so **Θ(n log n)** (the harmonic series).

### Pattern 8: Simple recursion, write a recurrence

```java
int sum(int[] a, int i) {                // sum of a[i..n-1]
    if (i == a.length) return 0;         // Θ(1)
    return a[i] + sum(a, i + 1);         // Θ(1) work + T(n-1)
}
```
T(n) = T(n − 1) + Θ(1), so T(n) = **Θ(n)**.

### Pattern 9: Branching recursion, Θ(2ⁿ) vs memoised Θ(n)

```java
long fib(int n) { return n < 2 ? n : fib(n - 1) + fib(n - 2); }
```
T(n) = T(n − 1) + T(n − 2) + Θ(1). The call tree roughly doubles at each level, so T(n) = Θ(φⁿ) with φ ≈ 1.618, which is **exponential**. With memoisation each fib(k) is computed once, giving **Θ(n)**. That's the essence of [dynamic programming](../06-Algorithm-Design-Techniques/dynamic-programming/fundamentals.md).

### Pattern 10: Divide and conquer, use the Master Theorem

```java
void mergeSort(int[] a, int lo, int hi) {   // sorts a[lo..hi]
    if (lo >= hi) return;
    int mid = (lo + hi) >>> 1;
    mergeSort(a, lo, mid); mergeSort(a, mid + 1, hi);   // 2 T(n/2)
    merge(a, lo, mid, hi);                              // Θ(n)
}
```
T(n) = 2T(n/2) + Θ(n), so **Θ(n log n)**. See [Recurrence relations](recurrence-relations.md) and [Divide and conquer](divide-and-conquer.md).

### Pattern 11: Hash table operations, Θ(1) expected

```java
Set<Integer> seen = new HashSet<>();
for (int x : a) if (!seen.add(x)) return true;   // duplicate found
```
`HashSet.add` and `contains` take **Θ(1) expected** time (assuming a good hash function and a bounded load factor). n operations give **Θ(n) expected**. The worst case is worse: Θ(n) per operation for an old chained table with all keys colliding. Java 8+ `HashMap` turns a long bucket into a red-black tree, giving O(log n) per operation in the worst case.

### Pattern 12: Graph traversal, Θ(V + E)

```java
// BFS/DFS on an adjacency list
for (int u : order)                // every vertex processed once:  Θ(V)
    for (int v : adj.get(u))       // every edge examined once (twice if undirected): Θ(E)
        ...
```
It looks like nested loops, but the inner loop's total work across *all* outer iterations is the sum of all adjacency-list lengths, which is E (or 2E). So the result is **Θ(V + E)**, not V · E. With an adjacency **matrix**, the inner loop scans all V columns, so the cost is **Θ(V²)**.

### Pattern 13: Multi-dimensional dynamic programming

```java
// 2-D DP, e.g. LCS of strings of length n and m
for (int i = 1; i <= n; i++)
    for (int j = 1; j <= m; j++)
        dp[i][j] = ...;                   // Θ(1) work per cell
```
Cost = (number of states) × (work per state) = n · m · Θ(1) = **Θ(nm)**.
For a 3-D DP such as Floyd-Warshall (`for k, for i, for j`), the cost is **Θ(n³)**.
For matrix-chain multiplication there are Θ(n²) states but each one tries Θ(n) split points, so the cost is **Θ(n³)**.

> **Universal DP formula:** time = (#states) × (#transitions per state) × (cost per transition).

### Pattern 14: Hidden costs in Java library calls

| Code | Looks like | Actually costs |
|---|---|---|
| `s = s + c` inside a loop (String) | Θ(n) | **Θ(n²)**: each `+` copies the whole string. Use `StringBuilder`. |
| `list.remove(0)` on an `ArrayList` | Θ(1) | **Θ(n)**: shifts every element. Use `ArrayDeque`. |
| `list.get(i)` on a `LinkedList` | Θ(1) | **Θ(n)**: walks from an end |
| `list.contains(x)` | Θ(1) | **Θ(n)**: linear scan. Use a `HashSet`. |
| `Arrays.sort(a)` | — | **Θ(n log n)** |
| `str.substring(i, j)` (Java 7u6+) | Θ(1) | **Θ(j − i)**: it copies |

---

## E. Implementation

This program **counts** the basic operations for each pattern and checks them against the formulas derived above. It shows that the analysis really predicts the counts.

```java
import java.util.*;

/** Empirically checks the operation counts derived in time-complexity.md. */
public class ComplexityCounter {

    static long single(int n)        { long c = 0; for (int i = 0; i < n; i++) c++; return c; }
    static long nested(int n)        { long c = 0; for (int i = 0; i < n; i++) for (int j = 0; j < n; j++) c++; return c; }
    static long triangular(int n)    { long c = 0; for (int i = 0; i < n; i++) for (int j = i + 1; j < n; j++) c++; return c; }
    static long doubling(int n)      { long c = 0; for (int i = 1; i < n; i *= 2) c++; return c; }
    static long nLogN(int n)         { long c = 0; for (int i = 0; i < n; i++) for (int j = 1; j < n; j *= 2) c++; return c; }
    static long harmonic(int n)      { long c = 0; for (int i = 1; i <= n; i++) for (int j = 1; j <= n; j += i) c++; return c; }

    static long binarySearchSteps(int[] a, int key) {
        long steps = 0; int lo = 0, hi = a.length - 1;
        while (lo <= hi) {
            steps++;
            int mid = lo + (hi - lo) / 2;
            if (a[mid] == key) return steps;
            if (a[mid] < key) lo = mid + 1; else hi = mid - 1;
        }
        return steps;
    }

    static long calls;
    static long fibNaive(int n) { calls++; return n < 2 ? n : fibNaive(n - 1) + fibNaive(n - 2); }
    static long fibMemo(int n, long[] memo) {
        calls++;
        if (n < 2) return n;
        if (memo[n] != 0) return memo[n];
        return memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
    }

    /** BFS-style traversal of an adjacency list; counts vertex visits + edge scans. */
    static long graphWork(List<List<Integer>> adj) {
        long work = 0;
        for (List<Integer> nbrs : adj) { work++; for (int v : nbrs) work++; }
        return work;
    }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        int n = 1024;
        System.out.printf("n = %d%n", n);
        System.out.printf("single loop       %,12d   expected n            = %,d%n", single(n), n);
        System.out.printf("nested loops      %,12d   expected n^2          = %,d%n", nested(n), (long) n * n);
        System.out.printf("triangular loops  %,12d   expected n(n-1)/2     = %,d%n", triangular(n), (long) n * (n - 1) / 2);
        System.out.printf("doubling loop     %,12d   expected log2 n       = %d%n", doubling(n), 10);
        System.out.printf("n * log loop      %,12d   expected n log2 n     = %,d%n", nLogN(n), (long) n * 10);
        double hn = 0; for (int i = 1; i <= n; i++) hn += (double) n / i;
        System.out.printf("harmonic loop     %,12d   approx  n * H(n)      = %,.0f%n", harmonic(n), hn);

        check(single(n) == n, "single loop = n");
        check(nested(n) == (long) n * n, "nested = n^2");
        check(triangular(n) == (long) n * (n - 1) / 2, "triangular = n(n-1)/2");
        check(doubling(n) == 10, "doubling loop = log2(1024) = 10");
        check(nLogN(n) == (long) n * 10, "n log n loop = 10240");
        check(Math.abs(harmonic(n) - hn) < n, "harmonic loop within n of n*H(n)");

        int[] sorted = new int[1 << 20];
        for (int i = 0; i < sorted.length; i++) sorted[i] = 2 * i;   // even numbers
        long worst = binarySearchSteps(sorted, -1);                  // absent key
        System.out.printf("binary search on 2^20 elements, absent key: %d steps (bound: floor(log2 n) + 1 = 21)%n", worst);
        check(worst <= 21, "binary search <= floor(log2 n) + 1 steps");

        calls = 0; fibNaive(25); long naive = calls;
        calls = 0; fibMemo(25, new long[26]); long memo = calls;
        System.out.printf("fib(25): naive calls = %,d   memoised calls = %d%n", naive, memo);
        check(naive == 242785, "naive fib(25) makes 242,785 calls (exponential)");
        check(memo == 49, "memoised fib(25) makes 2n - 1 = 49 calls (linear)");

        // A graph with V = 1000 vertices and E = 5000 directed edges.
        int V = 1000, E = 5000; Random rnd = new Random(1);
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < V; i++) adj.add(new ArrayList<>());
        for (int e = 0; e < E; e++) adj.get(rnd.nextInt(V)).add(rnd.nextInt(V));
        long g = graphWork(adj);
        System.out.printf("graph traversal work = %,d   expected V + E = %,d (not V*E = %,d)%n", g, V + E, (long) V * E);
        check(g == V + E, "adjacency-list traversal = V + E");
    }
}
```

**Output:**

```
n = 1024
single loop              1,024   expected n            = 1,024
nested loops         1,048,576   expected n^2          = 1,048,576
triangular loops       523,776   expected n(n-1)/2     = 523,776
doubling loop               10   expected log2 n       = 10
n * log loop            10,240   expected n log2 n     = 10,240
harmonic loop            8,275   approx  n * H(n)      = 7,689
ok   single loop = n
ok   nested = n^2
ok   triangular = n(n-1)/2
ok   doubling loop = log2(1024) = 10
ok   n log n loop = 10240
ok   harmonic loop within n of n*H(n)
binary search on 2^20 elements, absent key: 20 steps (bound: floor(log2 n) + 1 = 21)
ok   binary search <= floor(log2 n) + 1 steps
fib(25): naive calls = 242,785   memoised calls = 49
ok   naive fib(25) makes 242,785 calls (exponential)
ok   memoised fib(25) makes 2n - 1 = 49 calls (linear)
graph traversal work = 6,000   expected V + E = 6,000 (not V*E = 5,000,000)
ok   adjacency-list traversal = V + E
```

**What the output shows:** going from naive to memoised Fibonacci cuts 242,785 calls to 49. Graph traversal is V + E = 6,000, not V·E = 5,000,000.

**Java-specific notes:**
- **Measuring time in Java is tricky.** The JIT compiler warms up, garbage collection pauses, and `System.nanoTime()` differences on tiny inputs are noise. For real benchmarks use **JMH**. For learning, *counting operations* (as above) is cleaner than timing.
- **`int` overflow in counts:** n² overflows `int` when n > 46,340. Use `long` for counters and products (note the `(long) n * n` cast).
- **`(lo + hi) / 2` can overflow** when lo + hi > 2³¹ − 1. Use `lo + (hi - lo) / 2` or `(lo + hi) >>> 1`.

---

## F. Time complexity of this topic's examples (summary of derivations)

| Pattern | Derivation | Result |
|---|---|---|
| Single loop | n × Θ(1) | Θ(n) |
| Consecutive loops | Θ(f) + Θ(g) = Θ(max(f, g)) | dominant term |
| Nested independent loops | n × m | Θ(nm) |
| Dependent nested loops | Σ(n − 1 − i) = n(n − 1)/2 | Θ(n²) |
| Halving or doubling | 2ᵏ ≥ n ⇒ k = log₂ n | Θ(log n) |
| Harmonic loop | n(1 + 1/2 + … + 1/n) = n·Hₙ | Θ(n log n) |
| Linear recursion | T(n) = T(n − 1) + c | Θ(n) |
| Fibonacci recursion | T(n) = T(n − 1) + T(n − 2) + c | Θ(φⁿ) |
| Divide and conquer (merge sort) | T(n) = 2T(n/2) + cn | Θ(n log n) |
| Hash set over n items | n × Θ(1) expected | Θ(n) expected |
| Graph on an adjacency list | Σ(1 + deg(v)) | Θ(V + E) |
| 2-D DP | states × work | Θ(nm) |

**Assumptions behind these results:** the RAM model (constant-time arithmetic on word-sized integers), a hash function that spreads keys uniformly, and adjacency-list graphs unless stated otherwise.

## G. Space complexity

Time and space are analysed separately. See [Space complexity](space-complexity.md). One quick link between them: **an algorithm can't use more space than time**. Touching each memory cell costs at least one step, so space ≤ time.

## H. Complexity summary

The central reference with every algorithm's complexity is [complexity-comparison.md](../15-Practice-and-Revision/complexity-comparison.md).

## I. Advantages, limitations, and comparisons

**Theory vs practice:**
- **Constants matter for small n.** Insertion sort (Θ(n²)) beats merge sort (Θ(n log n)) for n below about 20–50, which is why Java's TimSort uses insertion sort on small runs.
- **Caches matter.** A Θ(n) scan of an array can be 10× faster than a Θ(n) walk of a linked list, because array elements sit next to each other in memory.
- **Big-O hides huge constants sometimes.** Some "linear-time" algorithms (like [median of medians](../01-Sorting-and-Order-Statistics/order-statistics/median-of-medians.md)) are slower in practice than "worse" Θ(n²)-worst-case alternatives like quickselect.
- **Worst-case inputs can be deliberate.** Attackers send inputs that trigger worst cases ("algorithmic complexity attacks"), such as hash collisions. Randomisation defends against this.

**Common interview follow-ups:** "Can you do better?", "What's the bottleneck?", "What if the data doesn't fit in memory?", "Is that amortized or worst case?"

---

## J. Practice

**Beginner**
1. Give the complexity of: `for (i = n; i > 0; i /= 3)`.
2. Give the complexity of two separate loops, the first to n and the second to n².
3. What is `for (i = 0; i < n; i++) for (j = 0; j < 5; j++)`?

**Intermediate**
4. `for (i = 0; i < n; i++) for (j = 0; j < i * i; j++)`. *Hint: Σ i² ≈ n³/3.*
5. What does `String s = ""; for (i = 0; i < n; i++) s += "x";` cost, and why?
6. Recursion `f(n) = f(n/2) + 1`. What is its time complexity?

**Advanced**
7. `for (i = 2; i < n; i = i * i)`. How many iterations? *Hint: i takes the values 2, 4, 16, 256, …, so the answer is Θ(log log n).*
8. Show that `for (i = 1; i <= n; i++) for (j = 1; j <= n; j += i)` is Θ(n log n) using the bound Hₙ ≤ ln n + 1.

**Interview questions**

<details><summary>Q1. Why is BFS O(V + E) and not O(V × E)?</summary>

Each vertex is dequeued once, and when a vertex is processed only *its own* edges are scanned. Summed over all vertices, that's the total of all adjacency-list lengths, which is E (2E undirected). So the total is V + E. Multiplying would assume every vertex scans every edge, which it doesn't.
</details>

<details><summary>Q2. What is the difference between worst-case and amortized complexity?</summary>

Worst case bounds a **single** operation. Amortized bounds the **average over a sequence** of operations in the worst case. `ArrayList.add` is Θ(n) worst case for the one add that triggers a resize, but Θ(1) amortized, because resizes double the capacity and so happen rarely enough that the total for n adds is Θ(n).
</details>

<details><summary>Q3. Is O(n) always faster than O(n²)?</summary>

Only for large enough n. Big-O ignores constants. 1000n is slower than n² for n < 1000. Also, Big-O is an upper bound, so an "O(n²)" algorithm might actually run in Θ(n) on your inputs. Compare Θ bounds, and remember the constants for small inputs.
</details>

**Worked problem.** *Complexity of checking whether any two elements of an array sum to k.*
- Brute force over all pairs: n(n − 1)/2 checks, so **Θ(n²)**.
- Sort first, then two pointers: Θ(n log n) + Θ(n) = **Θ(n log n)**.
- Hash set (`if set contains k − x`): **Θ(n) expected**, at the cost of Θ(n) extra space.

**Coding problems**
- LeetCode 1 · Two Sum *(verify link)*
- LeetCode 704 · Binary Search *(verify link)*
- LeetCode 509 · Fibonacci Number (naive vs memo) *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. Time complexity is the *growth rate* of operations as a function of input size, independent of hardware.
2. Drop constants and lower-order terms: 3n² + 10n is Θ(n²).
3. Sequential code: add, so the largest term wins. Nested loops: multiply, or use a summation if they're dependent.
4. Halving or doubling gives log n. Recursion needs a recurrence. DP cost = states × transitions.
5. Know which case you mean: best, worst, average (random input), expected (random algorithm) or amortized (a sequence).

**Formulas**
- 1 + 2 + … + n = n(n + 1)/2, which is Θ(n²)
- 1 + 2 + 4 + … + 2ᵏ = 2ᵏ⁺¹ − 1
- Hₙ = 1 + 1/2 + … + 1/n ≈ ln n
- log₂ n = ln n / ln 2. The base doesn't matter in Θ.

**Common mistakes**
- Calling BFS O(V·E).
- Forgetting hidden library costs (`String +=`, `ArrayList.remove(0)`).
- Saying "hash lookup is O(1)" without "expected/average".
- Treating `j += i` loops as n².

**Quiz**
1. Complexity of `for (i = 1; i < n; i *= 2) for (j = 0; j < n; j++)`?
2. T(n) = T(n − 1) + n solves to?
3. Why is naive Fibonacci exponential?
4. What's the time limit rule of thumb for n = 10⁵?

**Answers**

<details><summary>Show answers</summary>

1. Θ(n log n): log n outer iterations × n inner iterations.
2. Θ(n²): it's n + (n − 1) + … + 1.
3. Each call makes two more calls, and the same sub-problems are recomputed many times. The call tree has about φⁿ nodes.
4. Θ(n log n) or better. Θ(n²) = 10¹⁰ is too slow.
</details>

**Related topics:** [Asymptotic notation](asymptotic-notation.md) · [Recurrence relations](recurrence-relations.md) · [Space complexity](space-complexity.md) · [Complexity comparison](../15-Practice-and-Revision/complexity-comparison.md) · [Amortized analysis](../06-Algorithm-Design-Techniques/amortized-analysis/aggregate-method.md)
