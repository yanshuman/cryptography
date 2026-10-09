# Complexity Comparison Reference

> **Status:** ✅ Written · A one-stop reference for every complexity in this repository. Each entry links to the page where it's **derived**. Don't just memorise these: be able to explain where each one comes from.

**Notation:** n = elements · k = key range · d = digits · V = vertices · E = edges · h = tree height · α(n) = inverse Ackermann (≤ 4 in practice) · "exp." = expected · "amort." = amortized.

**Contents:** [1. Sorting](#1-sorting-algorithms) · [2. Selection](#2-selection-order-statistics) · [3. Data structures](#3-data-structure-operations) · [4. Recurrences](#4-common-recurrences) · [5. Graph algorithms](#5-graph-algorithms) · [6. Other algorithms](#6-other-algorithms) · [7. Growth rates](#7-growth-rates) · [8. Confused concepts](#8-frequently-confused-concepts) · [9. Estimating from Java code](#9-estimating-complexity-from-java-code)

---

## 1. Sorting algorithms

| Algorithm | Best | Average | Worst | Aux. space | Stable | In place | Comparison | Notes |
|---|---|---|---|---|---|---|---|---|
| [Bubble](../01-Sorting-and-Order-Statistics/comparison-sorts/bubble-sort.md) (early exit) | Θ(n) | Θ(n²) | Θ(n²) | Θ(1) | ✅ | ✅ | ✅ | swaps = inversions |
| [Selection](../01-Sorting-and-Order-Statistics/comparison-sorts/selection-sort.md) | Θ(n²) | Θ(n²) | Θ(n²) | Θ(1) | ❌ | ✅ | ✅ | ≤ n − 1 swaps |
| [Insertion](../01-Sorting-and-Order-Statistics/comparison-sorts/insertion-sort.md) | Θ(n) | Θ(n²) | Θ(n²) | Θ(1) | ✅ | ✅ | ✅ | Θ(n + inversions), adaptive |
| [Merge](../01-Sorting-and-Order-Statistics/comparison-sorts/merge-sort.md) | Θ(n log n) | Θ(n log n) | Θ(n log n) | Θ(n) | ✅ | ❌ | ✅ | ≤ n⌈lg n⌉ − 2^⌈lg n⌉ + 1 comparisons |
| [Heapsort](../01-Sorting-and-Order-Statistics/heapsort/heapsort.md) | Θ(n log n) | Θ(n log n) | Θ(n log n) | Θ(1) | ❌ | ✅ | ✅ | ≈ 2n lg n comparisons |
| [Quicksort](../01-Sorting-and-Order-Statistics/quicksort/quicksort-basics.md) (fixed pivot) | Θ(n log n) | Θ(n log n) | Θ(n²) | Θ(log n)–Θ(n) stack | ❌ | ✅ | ✅ | sorted input is the worst case |
| [Randomized quicksort](../01-Sorting-and-Order-Statistics/quicksort/randomized-quicksort.md) | Θ(n log n) | Θ(n log n) exp. | Θ(n²) w.p. → 0 | Θ(log n) | ❌ | ✅ | ✅ | ≈ 1.39 n lg n comparisons |
| Introsort | Θ(n log n) | Θ(n log n) | Θ(n log n) | Θ(log n) | ❌ | ✅ | ✅ | C++ `std::sort` |
| TimSort | Θ(n) | Θ(n log n) | Θ(n log n) | Θ(n) | ✅ | ❌ | ✅ | Java object sorts, Python |
| [Counting](../01-Sorting-and-Order-Statistics/linear-time-sorting/counting-sort.md) | Θ(n + k) | Θ(n + k) | Θ(n + k) | Θ(n + k) | ✅ | ❌ | ❌ | integer keys in [0, k] |
| [Radix (LSD)](../01-Sorting-and-Order-Statistics/linear-time-sorting/radix-sort.md) | Θ(d(n + k)) | Θ(d(n + k)) | Θ(d(n + k)) | Θ(n + k) | ✅ | ❌ | ❌ | stable inner sort required |
| [Bucket](../01-Sorting-and-Order-Statistics/linear-time-sorting/bucket-sort.md) | Θ(n) | Θ(n) exp. | Θ(n²) | Θ(n) | ✅ | ❌ | partly | uniform input |

**Comparison sort lower bound:** Ω(n log n) worst case *and* average case, because ⌈log₂ n!⌉ ≈ n lg n − 1.44n. See [Lower bounds](../01-Sorting-and-Order-Statistics/linear-time-sorting/lower-bounds.md).

## 2. Selection (order statistics)

| Problem | Algorithm | Time | Space |
|---|---|---|---|
| Min or max | [linear scan](../01-Sorting-and-Order-Statistics/order-statistics/minimum-and-maximum.md) | n − 1 comparisons (optimal) | Θ(1) |
| Min and max | pairs | ≤ 3⌊n/2⌋ comparisons (optimal) | Θ(1) |
| Second smallest | tournament | n + ⌈lg n⌉ − 2 | Θ(n) |
| k-th smallest | [quickselect](../01-Sorting-and-Order-Statistics/order-statistics/quickselect.md) | Θ(n) exp., Θ(n²) worst | Θ(1) |
| k-th smallest | [median of medians](../01-Sorting-and-Order-Statistics/order-statistics/median-of-medians.md) | **Θ(n) worst case** | Θ(log n) |
| Top k (streaming) | size-k heap | Θ(n log k) | Θ(k) |
| Top k (array) | build-heap + k extracts | Θ(n + k log n) | Θ(1) |

## 3. Data-structure operations

### Linear structures

| Structure | Access by index | Search | Insert | Delete | Notes |
|---|---|---|---|---|---|
| Array | Θ(1) | Θ(n) | Θ(n) | Θ(n) | contiguous, cache-friendly |
| Sorted array | Θ(1) | Θ(log n) | Θ(n) | Θ(n) | binary search |
| Dynamic array (`ArrayList`) | Θ(1) | Θ(n) | Θ(1) amort. at the end, Θ(n) in the middle | Θ(n) | doubling growth |
| Singly linked list | Θ(n) | Θ(n) | Θ(1) at head / after a node | Θ(1) after a node | |
| Doubly linked list | Θ(n) | Θ(n) | Θ(1) given the node | Θ(1) given the node | `LinkedList` |
| Stack (array or list) | — | — | push Θ(1) | pop Θ(1) | `ArrayDeque` |
| Queue / deque | — | — | Θ(1) | Θ(1) | `ArrayDeque` |

### Hashing

| Structure | Search | Insert | Delete | Worst case |
|---|---|---|---|---|
| Hash table, chaining | Θ(1 + α) exp. | Θ(1) | Θ(1 + α) exp. | Θ(n) |
| Hash table, open addressing | ≤ 1/(1 − α) probes exp. (unsuccessful) | same | same | Θ(n) |
| Java `HashMap` (Java 8+) | Θ(1) exp. | Θ(1) amort. exp. | Θ(1) exp. | **O(log n)** (long buckets become red-black trees) |
| Perfect hashing (static keys) | **Θ(1) worst case** | — | — | Θ(n) space |

α = n/m is the load factor.

### Trees and heaps

| Structure | Search | Insert | Delete | Min/max | Notes |
|---|---|---|---|---|---|
| BST (unbalanced) | Θ(h): O(log n) avg., Θ(n) worst | Θ(h) | Θ(h) | Θ(h) | random build: h = O(log n) exp. |
| Red-black tree (`TreeMap`) | Θ(log n) | Θ(log n) | Θ(log n) | Θ(log n) | h ≤ 2 lg(n + 1) |
| B-tree (degree t) | Θ(t log_t n) CPU, Θ(log_t n) disk I/O | same | same | Θ(log_t n) | databases, filesystems |
| Order-statistic tree | rank or select in Θ(log n) | Θ(log n) | Θ(log n) | | augmented red-black tree |
| Interval tree | overlap query Θ(log n) | Θ(log n) | Θ(log n) | | augmented red-black tree |
| [Binary heap](../01-Sorting-and-Order-Statistics/heapsort/priority-queues.md) | Θ(n) | Θ(log n) | extract Θ(log n) | peek Θ(1) | build Θ(n) |
| Binomial heap | Θ(n) | O(log n) (Θ(1) amort.) | Θ(log n) | Θ(log n) | union Θ(log n) |
| Fibonacci heap | Θ(n) | Θ(1) | extract-min O(log n) amort. | Θ(1) | decrease-key **Θ(1) amort.**, union Θ(1) |
| Disjoint set (union by rank + path compression) | find O(α(n)) amort. | make-set Θ(1) | — | — | union O(α(n)) amort. |

## 4. Common recurrences

| Recurrence | Solution | Example |
|---|---|---|
| T(n) = T(n/2) + Θ(1) | Θ(log n) | binary search, fast power |
| T(n) = T(n/2) + Θ(n) | Θ(n) | quickselect (lucky case) |
| T(n) = 2T(n/2) + Θ(1) | Θ(n) | tree traversal, recursive max |
| T(n) = 2T(n/2) + Θ(n) | **Θ(n log n)** | merge sort, quicksort (best case) |
| T(n) = 2T(n/2) + Θ(n log n) | Θ(n log² n) | — |
| T(n) = 3T(n/2) + Θ(n) | Θ(n^1.585) | Karatsuba |
| T(n) = 7T(n/2) + Θ(n²) | Θ(n^2.807) | Strassen |
| T(n) = 8T(n/2) + Θ(n²) | Θ(n³) | naive recursive matrix multiplication |
| T(n) = T(n/5) + T(7n/10) + Θ(n) | Θ(n) | median of medians |
| T(n) = T(αn) + T((1 − α)n) + Θ(n) | Θ(n log n) | quicksort with a constant-ratio split |
| T(n) = T(n − 1) + Θ(1) | Θ(n) | linear recursion |
| T(n) = T(n − 1) + Θ(n) | **Θ(n²)** | quicksort worst case, insertion sort |
| T(n) = 2T(n − 1) + Θ(1) | Θ(2ⁿ) | Towers of Hanoi |
| T(n) = T(n − 1) + T(n − 2) + Θ(1) | Θ(φⁿ) | naive Fibonacci |
| T(n) = 2T(√n) + Θ(log n) | Θ(log n log log n) | substitute m = lg n |

**Master Theorem, simplified:** T(n) = aT(n/b) + Θ(nᵈ):
- d < log_b a → Θ(n^(log_b a))
- d = log_b a → Θ(nᵈ log n)
- d > log_b a → Θ(nᵈ)

The full derivation is in [Recurrence relations](../00-Foundations/recurrence-relations.md).

## 5. Graph algorithms

| Problem | Algorithm | Time | Space | Requirements |
|---|---|---|---|---|
| Traversal | BFS / DFS (adjacency list) | **Θ(V + E)** | Θ(V) | — |
| Traversal | BFS / DFS (adjacency matrix) | Θ(V²) | Θ(V²) matrix | — |
| Unweighted shortest paths | BFS | Θ(V + E) | Θ(V) | unit weights |
| Topological sort | DFS / Kahn's algorithm | Θ(V + E) | Θ(V) | a DAG |
| Strongly connected components | Kosaraju / Tarjan | Θ(V + E) | Θ(V) | directed graph |
| MST | Kruskal | O(E log E) = O(E log V) | Θ(V) | sorting + union-find |
| MST | Prim (binary heap) | O(E log V) | Θ(V) | |
| MST | Prim (Fibonacci heap) | O(E + V log V) | Θ(V) | |
| Single-source shortest paths | Dijkstra (binary heap) | O((V + E) log V) | Θ(V) | **non-negative weights** |
| Single-source shortest paths | Dijkstra (Fibonacci heap) | O(E + V log V) | Θ(V) | non-negative weights |
| Single-source shortest paths | Dijkstra (array, dense) | O(V²) | Θ(V) | non-negative weights |
| Single-source shortest paths | Bellman-Ford | Θ(VE) | Θ(V) | negative edges OK, detects negative cycles |
| Single-source shortest paths | DAG relaxation | Θ(V + E) | Θ(V) | a DAG (negative edges OK) |
| All-pairs shortest paths | Floyd-Warshall | Θ(V³) | Θ(V²) | no negative cycles |
| All-pairs shortest paths | Johnson | O(VE log V) | Θ(V²) | sparse graphs, negative edges OK |
| All-pairs shortest paths | repeated squaring | Θ(V³ log V) | Θ(V²) | |
| Max flow | Ford-Fulkerson | O(E · \|f*\|) | Θ(V + E) | integer capacities |
| Max flow | Edmonds-Karp | O(VE²) | Θ(V + E) | BFS augmenting paths |
| Max flow | Relabel-to-front | O(V³) | Θ(V + E) | |
| Bipartite matching | via max flow | O(VE) | Θ(V + E) | |

## 6. Other algorithms

| Area | Algorithm | Time |
|---|---|---|
| Dynamic programming | LCS (lengths m, n) | Θ(mn) |
| | Matrix-chain multiplication | Θ(n³) time, Θ(n²) space |
| | Optimal BST | Θ(n³) (Θ(n²) with Knuth's optimisation) |
| | 0/1 knapsack (capacity W) | Θ(nW), pseudo-polynomial |
| Greedy | Activity selection (sorted) | Θ(n) (+ Θ(n log n) to sort) |
| | Huffman coding | Θ(n log n) |
| Strings (text n, pattern m) | Naive | O((n − m + 1)m) |
| | Rabin-Karp | O(n + m) exp., O(nm) worst |
| | Finite automaton | Θ(n) matching + O(m³\|Σ\|) preprocessing (O(m\|Σ\|) improved) |
| | KMP | **Θ(n + m)** |
| Number theory (β-bit numbers) | Euclid's GCD | O(log min(a, b)) steps |
| | Modular exponentiation | O(log e) multiplications |
| | Miller-Rabin (s rounds) | O(sβ³) bit operations |
| Geometry | Convex hull (Graham scan) | O(n log n) |
| | Convex hull (Jarvis march) | O(nh) |
| | Closest pair | O(n log n) |
| | Any segment intersection | O(n log n) |
| Matrices | Naive multiplication | Θ(n³) |
| | Strassen | Θ(n^2.807) |
| | LUP decomposition / solving | Θ(n³) |
| Polynomials | FFT multiplication | Θ(n log n) |
| Linear programming | Simplex | exponential worst case, fast in practice |
| NP-complete problems | brute-force subset / TSP | Θ(2ⁿ) / Θ(n!) |
| Approximation | Vertex cover (2-approx) | O(V + E) |
| | Metric TSP (2-approx) | O(V²) (via MST) |
| | Set cover (greedy, ln n approx.) | polynomial |

## 7. Growth rates

At about 10⁸ simple operations per second:

| f(n) | n = 10 | n = 10³ | n = 10⁶ | Max n in ~1 s |
|---|---|---|---|---|
| 1 | 1 | 1 | 1 | ∞ |
| log₂ n | 3.3 | 10 | 20 | astronomical |
| √n | 3.2 | 32 | 1000 | 10¹⁶ |
| n | 10 | 10³ | 10⁶ | ~10⁸ |
| n log₂ n | 33 | 10⁴ | 2 × 10⁷ | ~5 × 10⁶ |
| n² | 100 | 10⁶ | 10¹² | ~10⁴ |
| n³ | 10³ | 10⁹ | 10¹⁸ | ~500 |
| 2ⁿ | 1024 | 10³⁰¹ | — | ~27 |
| n! | 3.6 × 10⁶ | — | — | ~11 |

**Ordering:** 1 ≺ log log n ≺ log n ≺ √n ≺ n ≺ n log n ≺ n² ≺ n³ ≺ 2ⁿ ≺ n! ≺ nⁿ. Proofs and the limit test are in [Asymptotic notation](../00-Foundations/asymptotic-notation.md).

## 8. Frequently confused concepts

| Pair | Difference |
|---|---|
| **O vs Θ** | O is an upper bound only. Θ is tight. "Insertion sort is O(n²)" is true, and "Θ(n²)" is true only for its worst case. |
| **Worst case vs Big-O** | Unrelated axes. You can state O, Ω or Θ of the best, worst or average case. |
| **Average vs expected** | Average: over random *inputs*. Expected: over the algorithm's *coin flips*, for any input. |
| **Amortized vs average** | Amortized: guaranteed average per operation over a worst-case *sequence* (no probability involved). |
| **Auxiliary vs total space** | Auxiliary excludes the input. "O(1) space" sorts mean O(1) auxiliary. |
| **Recursion: calls vs depth** | Stack space depends on the maximum *depth*, not the total number of calls. Naive Fibonacci makes about φⁿ calls but has depth n. |
| **Hash table O(1)** | Expected, under good hashing. The worst case is Θ(n), or O(log n) in Java 8+ `HashMap`. |
| **BFS O(V + E) vs O(V·E)** | Each adjacency list is scanned once in total, not once per vertex. |
| **Linear-time sorting** | Only with assumptions on the keys (range, digits, distribution). It doesn't violate the Ω(n log n) comparison bound. |
| **Pseudo-polynomial** | Θ(nW) knapsack is polynomial in the *value* W, which is exponential in W's bit length. |
| **Polynomial vs exponential in the bit length** | Trial division up to √N is Θ(√N), which is exponential in the β = log N bits. |

## 9. Estimating complexity from Java code

The full guide with worked examples and a program that verifies them is in [Time complexity, Section D](../00-Foundations/time-complexity.md#d-algorithm-and-pseudocode-a-guide-to-analysing-code). Quick reference:

| Code shape | Complexity |
|---|---|
| `for (i = 0; i < n; i++)` with an O(1) body | Θ(n) |
| Two loops one after the other (n, then n²) | Θ(n²): the larger one wins |
| Nested loops, both to n | Θ(n²) |
| `for i` … `for (j = i + 1; j < n; j++)` | n(n − 1)/2 = Θ(n²) |
| `for (i = 1; i < n; i *= 2)` | Θ(log n) |
| `for (i = 0; i < n; i++) for (j = 1; j < n; j *= 2)` | Θ(n log n) |
| `for (i = 1; i <= n; i++) for (j = 1; j <= n; j += i)` | Θ(n log n) (harmonic) |
| `for (i = 2; i < n; i = i * i)` | Θ(log log n) |
| Binary search `while (lo <= hi)`, halving each time | Θ(log n) |
| `f(n) = f(n - 1) + O(1)` | Θ(n) |
| `f(n) = f(n/2) + f(n/2) + O(n)` | Θ(n log n) |
| `f(n) = f(n - 1) + f(n - 2)` (no memo) | Θ(φⁿ) |
| The same, memoised | Θ(n) |
| n `HashSet.add` or `contains` | Θ(n) expected |
| n `TreeSet.add` | Θ(n log n) |
| n `PriorityQueue.offer` / `poll` | Θ(n log n) |
| `list.remove(0)` in a loop on an `ArrayList` | Θ(n²) (use `ArrayDeque`) |
| `s += c` in a loop on a `String` | Θ(n²) (use `StringBuilder`) |
| BFS/DFS on `List<List<Integer>>` | Θ(V + E) |
| 2-D DP `dp[i][j]` with O(1) per cell | Θ(nm) |
| 3-D DP / triple loop `k, i, j` | Θ(n³) |
| DP with s states and t transitions each | Θ(s · t) |
| Sort, then a linear scan | Θ(n log n) |
| Backtracking over all subsets / permutations | Θ(2ⁿ · n) / Θ(n! · n) |

**Rules of thumb:** sequential code adds, nested code multiplies, halving gives a log, recursion needs a recurrence, and memoisation turns overlapping recursion into states × transitions.

---

*Entries for chapters not yet written in full (data structures, graphs, DP and so on) are standard CLRS results. Each will link to its derivation once that topic page is complete. Progress is tracked in the [syllabus checklist](../16-Progress-Tracker/syllabus-checklist.md).*
