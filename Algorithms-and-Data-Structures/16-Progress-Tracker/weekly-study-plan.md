# Weekly Study Plan

A 24-week plan at about **1–1.5 hours a day, 6 days a week**, following the [dependency map](../README.md#topic-dependency-map). Day 7 each week is for review and catching up. Weeks 1–4 are planned day by day because their pages are already written. Later weeks list topics, and they'll be planned in detail as those pages are completed.

**Daily routine (for each topic)**
1. **Read** sections A–D (intuition, trace, pseudocode) and **trace the example by hand** (20–30 min).
2. **Code** the Java implementation yourself without looking, then compare it and run the tests (20–30 min).
3. **Analyse:** re-derive the complexity in section F on paper (10 min).
4. **Practise:** at least 2 exercises from section J, and 1 coding problem (20–30 min).
5. **Revise:** do the section K quiz the *next* morning (5 min).

Update **My status** in the [syllabus checklist](syllabus-checklist.md) as you go.

---

## Phase 1: Foundations (Weeks 1–2)

### Week 1: Analysing algorithms
| Day | Topic | Goal |
|---|---|---|
| 1 | [Algorithm basics](../00-Foundations/algorithm-basics.md) | Write loop invariants for find-max and linear search |
| 2 | [Time complexity](../00-Foundations/time-complexity.md), Sections A–C | Growth-rate table, best/worst/average/expected/amortized |
| 3 | [Time complexity](../00-Foundations/time-complexity.md), Section D | Analyse all 14 code patterns. Run `ComplexityCounter`. |
| 4 | [Space complexity](../00-Foundations/space-complexity.md) | Recursion depth vs number of calls; in place |
| 5 | [Asymptotic notation](../00-Foundations/asymptotic-notation.md) | Prove Θ bounds with constants and with the limit test |
| 6 | Practice | Exercises from all four pages. LeetCode 704 and 1. |
| 7 | Review | All the quizzes from this week |

### Week 2: Recursion and divide and conquer
| Day | Topic | Goal |
|---|---|---|
| 1 | [Recurrence relations](../00-Foundations/recurrence-relations.md): recursion trees | Draw trees for 2T(n/2) + n and T(n/3) + T(2n/3) + n |
| 2 | Recurrences: substitution | Prove merge sort is O(n log n) by induction |
| 3 | Recurrences: Master Theorem | Solve all 10 table examples without looking |
| 4 | [Divide and conquer](../00-Foundations/divide-and-conquer.md) | Maximum subarray and counting inversions |
| 5 | Practice | LeetCode 53, 50, 493 |
| 6 | [Summations](../14-Mathematical-Foundations/summations.md) *(page coming)* | Arithmetic and geometric series, harmonic numbers |
| 7 | Review | Foundations quizzes, and the [complexity reference](../15-Practice-and-Revision/complexity-comparison.md) Sections 4, 7 and 9 |

## Phase 2: Sorting (Weeks 3–4)

### Week 3: Comparison sorts and heaps
| Day | Topic | Goal |
|---|---|---|
| 1 | [Sorting overview](../01-Sorting-and-Order-Statistics/README.md) + [Insertion sort](../01-Sorting-and-Order-Statistics/comparison-sorts/insertion-sort.md) | Invariant; shifts = inversions |
| 2 | [Bubble](../01-Sorting-and-Order-Statistics/comparison-sorts/bubble-sort.md) + [Selection](../01-Sorting-and-Order-Statistics/comparison-sorts/selection-sort.md) | Stability counterexample; why bubble is slowest |
| 3 | [Merge sort](../01-Sorting-and-Order-Statistics/comparison-sorts/merge-sort.md) | Implement top-down and bottom-up; prove stability |
| 4 | [Heap basics](../01-Sorting-and-Order-Statistics/heapsort/heap-basics.md) + [MAX-HEAPIFY](../01-Sorting-and-Order-Statistics/heapsort/maintaining-heap-property.md) | Index formulas; sift-down |
| 5 | [Build heap](../01-Sorting-and-Order-Statistics/heapsort/building-a-heap.md) + [Heapsort](../01-Sorting-and-Order-Statistics/heapsort/heapsort.md) | The O(n) build proof |
| 6 | [Priority queues](../01-Sorting-and-Order-Statistics/heapsort/priority-queues.md) | LeetCode 215, 23, 295 |
| 7 | Review | Implement merge sort and heapsort from memory, timed |

### Week 4: Quicksort, linear sorts, selection
| Day | Topic | Goal |
|---|---|---|
| 1 | [Quicksort basics](../01-Sorting-and-Order-Statistics/quicksort/quicksort-basics.md) + [Partitioning](../01-Sorting-and-Order-Statistics/quicksort/partitioning.md) | Lomuto, Hoare, 3-way; LeetCode 75 |
| 2 | [Randomized quicksort](../01-Sorting-and-Order-Statistics/quicksort/randomized-quicksort.md) + [Analysis](../01-Sorting-and-Order-Statistics/quicksort/complexity-analysis.md) | The indicator-variable proof |
| 3 | [Lower bounds](../01-Sorting-and-Order-Statistics/linear-time-sorting/lower-bounds.md) + [Counting sort](../01-Sorting-and-Order-Statistics/linear-time-sorting/counting-sort.md) | The decision-tree proof |
| 4 | [Radix](../01-Sorting-and-Order-Statistics/linear-time-sorting/radix-sort.md) + [Bucket](../01-Sorting-and-Order-Statistics/linear-time-sorting/bucket-sort.md) | Why stability matters; E[Σnᵢ²] = 2n − 1 |
| 5 | [Min & max](../01-Sorting-and-Order-Statistics/order-statistics/minimum-and-maximum.md) + [Quickselect](../01-Sorting-and-Order-Statistics/order-statistics/quickselect.md) | LeetCode 215, 973 |
| 6 | [Median of medians](../01-Sorting-and-Order-Statistics/order-statistics/median-of-medians.md) | Derive T(n) ≤ T(n/5) + T(7n/10) + O(n) |
| 7 | Review | The sorting table in [complexity-comparison](../15-Practice-and-Revision/complexity-comparison.md#1-sorting-algorithms), and LeetCode 912 with 3 different sorts |

## Phase 3: Core data structures (Weeks 5–7)
- **Week 5:** stacks, queues, linked lists, pointers and objects, rooted trees (Ch. 10)
- **Week 6:** hash tables: direct addressing, chaining, hash functions, open addressing, perfect hashing (Ch. 11)
- **Week 7:** binary search trees (Ch. 12), and red-black properties and rotations (Ch. 13.1–13.2)

## Phase 4: Design techniques (Weeks 8–10)
- **Week 8:** red-black insertion and deletion (13.3–13.4), disjoint sets (Ch. 21)
- **Week 9:** dynamic programming (Ch. 15): fundamentals, assembly-line, matrix-chain, LCS, optimal BST
- **Week 10:** greedy (Ch. 16), amortized analysis (Ch. 17)

## Phase 5: Graphs (Weeks 11–14)
- **Week 11:** representations, BFS, DFS
- **Week 12:** topological sort, SCCs, MST (Kruskal, Prim)
- **Week 13:** shortest paths: Bellman-Ford, DAG, Dijkstra, Floyd-Warshall, Johnson
- **Week 14:** maximum flow: Ford-Fulkerson, Edmonds-Karp, bipartite matching, push-relabel

## Phase 6: Advanced data structures (Weeks 15–16)
- **Week 15:** augmented trees (order statistics, interval trees), B-trees
- **Week 16:** binomial heaps, Fibonacci heaps

## Phase 7: Specialised algorithms (Weeks 17–19)
- **Week 17:** number theory → RSA → primality (Ch. 31). This pairs with the repo's [RSA project](../../rsa/README.md).
- **Week 18:** string matching (Ch. 32), computational geometry (Ch. 33)
- **Week 19:** sorting networks, matrix operations, linear programming, FFT (Ch. 27–30)

## Phase 8: Theory (Weeks 20–21)
- **Week 20:** P, NP, reductions, NP-completeness proofs (Ch. 34)
- **Week 21:** approximation algorithms (Ch. 35)

## Phase 9: Interview revision (Weeks 22–24)
- **Week 22:** [complexity reference](../15-Practice-and-Revision/complexity-comparison.md) and algorithm comparisons. Implement the core 10 from memory.
- **Week 23:** interview questions and coding problems by topic, with mock interviews (45 min, explaining aloud)
- **Week 24:** common mistakes, revision notes, and every section K quiz once more

---

**Falling behind?** Skip the ★ optional sections (marked in the CLRS contents) on a first pass, but **never skip the Week 1–4 foundations**. Everything later depends on them.
