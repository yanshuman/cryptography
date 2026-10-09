# Priority Queues

> **CLRS:** 6.5 · **Status:** ✅ Written · **Prerequisites:** [Heap basics](heap-basics.md), [MAX-HEAPIFY](maintaining-heap-property.md) · **Section:** [Sorting](../README.md) · **Used by:** [Dijkstra](../../07-Graph-Algorithms/shortest-paths/dijkstra.md), [Prim](../../07-Graph-Algorithms/minimum-spanning-trees/prim.md), [Huffman](../../06-Algorithm-Design-Techniques/greedy-algorithms/huffman-coding.md)

---

## A. Introduction

A **priority queue** is a data structure that maintains a set S of elements, each with a **key** (its priority), and supports:

| Operation (max-priority queue) | Meaning |
|---|---|
| **INSERT(S, x)** | add x |
| **MAXIMUM(S)** | return the element with the largest key, without removing it |
| **EXTRACT-MAX(S)** | remove and return the element with the largest key |
| **INCREASE-KEY(S, x, k)** | raise x's key to k (k ≥ its current key) |
| (DELETE(S, x)) | remove an arbitrary element |

A **min-priority queue** is the mirror image: INSERT, MINIMUM, EXTRACT-MIN and DECREASE-KEY.

**What problem does it solve?** "Always process the most important item next", where items keep arriving and priorities can change. A sorted list makes insertion slow, and an unsorted list makes finding the max slow. A **binary heap** makes **both O(log n)**.

**Where it's used**

| Application | Queue type | What the key is |
|---|---|---|
| OS process scheduling | max | process priority |
| Event-driven simulation | min | event time |
| [Dijkstra's shortest paths](../../07-Graph-Algorithms/shortest-paths/dijkstra.md) | min | tentative distance (uses DECREASE-KEY) |
| [Prim's MST](../../07-Graph-Algorithms/minimum-spanning-trees/prim.md) | min | the cheapest edge to the tree |
| [Huffman coding](../../06-Algorithm-Design-Techniques/greedy-algorithms/huffman-coding.md) | min | symbol frequency |
| A* search, best-first search | min | f = g + h |
| Merging k sorted lists or files | min | each list's current head |
| Top-k / k-th largest in a stream | min-heap of size k | value |
| Running median | one max-heap + one min-heap | value |
| Bandwidth management, job queues, rate limiting | max or min | priority, deadline |

**Prerequisites:** the heap array layout, and sift-down.

---

## B. Intuition

**Analogy: a hospital emergency room.** Patients don't get treated first-come-first-served (that's a plain queue). The most critical patient goes next. New patients arrive at any time, and a waiting patient's condition can get worse (INCREASE-KEY).

**How a heap makes it fast:**
- **MAXIMUM:** the root. Instant.
- **EXTRACT-MAX:** remove the root, move the **last leaf** to the root, and **sift it down**. O(log n).
- **INSERT:** put the new element at the **end** (the next leaf position, so the tree stays nearly complete), then **sift it up** while it's bigger than its parent. O(log n).
- **INCREASE-KEY:** a bigger key can only violate the property **upward** (with its parent), so sift it up. O(log n).

Every operation touches only one root-to-leaf path.

---

## C. How it works internally

Start with the max-heap A = [16, 14, 10, 8, 7, 9, 3, 2, 4, 1] (0-indexed).

### INSERT(15): append, then sift up

```
Append at index 10 (parent = index 4, value 7):
[16, 14, 10, 8, 7, 9, 3, 2, 4, 1, 15]
15 > 7  → swap  → [16, 14, 10, 8, 15, 9, 3, 2, 4, 1, 7]     (now at index 4, parent index 1 = 14)
15 > 14 → swap  → [16, 15, 10, 8, 14, 9, 3, 2, 4, 1, 7]     (now at index 1, parent index 0 = 16)
15 < 16 → stop
```

### EXTRACT-MAX: take the root, move the last element to the root, sift down

```
max = 16. Move the last element (7) to the root and shrink:
[7, 15, 10, 8, 14, 9, 3, 2, 4, 1]
7 vs children 15, 10 → swap with 15 → [15, 7, 10, 8, 14, 9, 3, 2, 4, 1]
7 vs children 8, 14  → swap with 14 → [15, 14, 10, 8, 7, 9, 3, 2, 4, 1]
7 vs child 1         → stop
return 16
```

### INCREASE-KEY(index 8, value 4 → 15) (CLRS Figure 6.5)

```
[15, 14, 10, 8, 7, 9, 3, 2, 15, 1]    index 8's parent is index 3 (8): 15 > 8 → swap
[15, 14, 10, 15, 7, 9, 3, 2, 8, 1]    index 3's parent is index 1 (14): 15 > 14 → swap
[15, 15, 10, 14, 7, 9, 3, 2, 8, 1]    index 1's parent is index 0 (15): 15 > 15? no → stop
```

### DELETE(index i)

Replace A[i] with the last element and shrink. The moved element might be **too big** (so sift up) **or too small** (so sift down). Do whichever applies. O(log n).

**Edge cases:** EXTRACT on an empty queue throws "heap underflow". INCREASE-KEY with a *smaller* key is an error in CLRS (use sift-down, or DELETE + INSERT). Ties are allowed, and the order among equal priorities is **not** FIFO (a heap isn't stable).

---

## D. Algorithm and pseudocode

CLRS 6.5 (1-indexed):

```
HEAP-MAXIMUM(A)
1  return A[1]

HEAP-EXTRACT-MAX(A)
1  if heap-size[A] < 1
2      error "heap underflow"
3  max ← A[1]
4  A[1] ← A[heap-size[A]]
5  heap-size[A] ← heap-size[A] − 1
6  MAX-HEAPIFY(A, 1)
7  return max

HEAP-INCREASE-KEY(A, i, key)
1  if key < A[i]
2      error "new key is smaller than current key"
3  A[i] ← key
4  while i > 1 and A[PARENT(i)] < A[i]
5      exchange A[i] ↔ A[PARENT(i)]
6      i ← PARENT(i)

MAX-HEAP-INSERT(A, key)
1  heap-size[A] ← heap-size[A] + 1
2  A[heap-size[A]] ← −∞
3  HEAP-INCREASE-KEY(A, heap-size[A], key)
```

INSERT is cleverly expressed as "add a −∞ leaf, then increase its key". That reuses the sift-up code.

### Correctness of HEAP-INCREASE-KEY (loop invariant, CLRS Exercise 6.5-3)

> At the start of each iteration of the `while` loop, the array is a max-heap **except** that A[i] may be larger than A[PARENT(i)]. Also, if A[i] has a parent, then A[PARENT(i)] ≥ the children of i.

- **Initialization:** only A[i] changed (it got bigger), so the only possible violation is between i and its parent. Its children were ≤ the old A[i] ≤ the new key, and they were ≤ the parent too. ✓
- **Maintenance:** swapping i with its parent fixes that edge. The old parent moves down to i and is ≥ i's children (by the invariant). The only possible new violation is between the new i (one level up) and its parent. ✓
- **Termination:** either i = 1 (the root) or A[PARENT(i)] ≥ A[i]. Either way there's no violation left, so it's a max-heap. ✓

---

## E. Implementation

```java
import java.util.*;

/** A max-priority queue on a binary heap (CLRS 6.5), checked against java.util.PriorityQueue, plus classic applications. */
public class PriorityQueues {

    /** Growable max-heap of ints with insert, max, extractMax, increaseKey and delete. */
    static final class MaxPQ {
        private int[] a = new int[8];
        private int size;

        int size() { return size; }
        boolean isEmpty() { return size == 0; }

        int max() {
            if (size == 0) throw new NoSuchElementException("heap underflow");
            return a[0];
        }

        void insert(int key) {
            if (size == a.length) a = Arrays.copyOf(a, 2 * a.length);   // amortized O(1) growth
            a[size] = key;
            siftUp(size++);
        }

        int extractMax() {
            int max = max();
            a[0] = a[--size];                                           // last leaf -> root
            if (size > 0) siftDown(0);
            return max;
        }

        void increaseKey(int i, int key) {
            if (key < a[i]) throw new IllegalArgumentException("new key is smaller than current key");
            a[i] = key;
            siftUp(i);
        }

        /** Delete the element at index i: replace it with the last element, then fix in whichever direction is needed. */
        void delete(int i) {
            int last = a[--size];
            if (i == size) return;                                      // we removed the last slot itself
            int old = a[i];
            a[i] = last;
            if (last > old) siftUp(i); else siftDown(i);
        }

        private void siftUp(int i) {
            int v = a[i];
            while (i > 0 && a[(i - 1) / 2] < v) { a[i] = a[(i - 1) / 2]; i = (i - 1) / 2; }
            a[i] = v;
        }

        private void siftDown(int i) {
            int v = a[i];
            while (true) {
                int c = 2 * i + 1;
                if (c >= size) break;
                if (c + 1 < size && a[c + 1] > a[c]) c++;
                if (a[c] <= v) break;
                a[i] = a[c]; i = c;
            }
            a[i] = v;
        }

        int indexOf(int key) { for (int i = 0; i < size; i++) if (a[i] == key) return i; return -1; }
        int[] snapshot() { return Arrays.copyOf(a, size); }
        boolean isHeap() { for (int i = 1; i < size; i++) if (a[(i - 1) / 2] < a[i]) return false; return true; }
    }

    // ---------- Application 1: merge k sorted arrays in O(N log k) ----------
    static int[] mergeK(int[][] lists) {
        // min-heap of {value, listIndex, positionInList}
        PriorityQueue<int[]> pq = new PriorityQueue<>((x, y) -> Integer.compare(x[0], y[0]));
        int total = 0;
        for (int i = 0; i < lists.length; i++) {
            total += lists[i].length;
            if (lists[i].length > 0) pq.add(new int[]{lists[i][0], i, 0});
        }
        int[] out = new int[total];
        int k = 0;
        while (!pq.isEmpty()) {
            int[] top = pq.poll();
            out[k++] = top[0];
            int li = top[1], pos = top[2] + 1;
            if (pos < lists[li].length) pq.add(new int[]{lists[li][pos], li, pos});
        }
        return out;
    }

    // ---------- Application 2: k largest in a stream with a size-k MIN-heap, O(n log k) ----------
    static List<Integer> topK(int[] stream, int k) {
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        for (int x : stream) {
            if (minHeap.size() < k) minHeap.add(x);
            else if (x > minHeap.peek()) { minHeap.poll(); minHeap.add(x); }
        }
        List<Integer> res = new ArrayList<>(minHeap);
        res.sort(Collections.reverseOrder());
        return res;
    }

    // ---------- Application 3: running median with two heaps, O(log n) per insert ----------
    static final class MedianFinder {
        private final PriorityQueue<Integer> low = new PriorityQueue<>(Collections.reverseOrder()); // max-heap: smaller half
        private final PriorityQueue<Integer> high = new PriorityQueue<>();                          // min-heap: larger half
        void add(int x) {
            if (low.isEmpty() || x <= low.peek()) low.add(x); else high.add(x);
            if (low.size() > high.size() + 1) high.add(low.poll());     // rebalance: |low| - |high| in {0, 1}
            else if (high.size() > low.size()) low.add(high.poll());
        }
        double median() {
            return low.size() > high.size() ? low.peek() : (low.peek() + (long) high.peek()) / 2.0;
        }
    }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        MaxPQ pq = new MaxPQ();
        for (int x : new int[]{16, 14, 10, 8, 7, 9, 3, 2, 4, 1}) pq.insert(x);
        pq.insert(15);
        System.out.println("after insert 15:     " + Arrays.toString(pq.snapshot()));
        System.out.println("extractMax() = " + pq.extractMax() + "  -> " + Arrays.toString(pq.snapshot()));
        pq.increaseKey(pq.indexOf(4), 15);
        System.out.println("increaseKey(4->15):  " + Arrays.toString(pq.snapshot()));
        check(pq.isHeap() && pq.max() == 15, "heap property holds; max is 15");

        // Randomised differential test against java.util.PriorityQueue (as a max-heap)
        Random rnd = new Random(4);
        MaxPQ mine = new MaxPQ();
        PriorityQueue<Integer> ref = new PriorityQueue<>(Collections.reverseOrder());
        boolean same = true;
        for (int op = 0; op < 200_000; op++) {
            int r = rnd.nextInt(10);
            if (r < 5 || ref.isEmpty()) { int x = rnd.nextInt(1000); mine.insert(x); ref.add(x); }
            else if (r < 8) { if (mine.extractMax() != ref.poll()) same = false; }
            else if (r < 9) { int i = rnd.nextInt(mine.size()); int v = mine.snapshot()[i];
                              ref.remove(v); mine.delete(i); }
            else { int i = rnd.nextInt(mine.size()); int v = mine.snapshot()[i]; int nv = v + rnd.nextInt(100);
                   ref.remove(v); ref.add(nv); mine.increaseKey(i, nv); }
            if (mine.size() != ref.size() || (!ref.isEmpty() && mine.max() != ref.peek())) same = false;
        }
        check(same && mine.isHeap(), "200,000 random insert/extract/delete/increaseKey ops agree with java.util.PriorityQueue");

        try { new MaxPQ().extractMax(); check(false, "underflow should throw"); }
        catch (NoSuchElementException e) { check(true, "extractMax on an empty queue throws (heap underflow)"); }

        int[][] lists = {{1, 4, 9}, {2, 3, 10, 11}, {}, {0, 5, 6, 7, 8}};
        int[] merged = mergeK(lists);
        System.out.println("k-way merge: " + Arrays.toString(merged));
        check(Arrays.equals(merged, new int[]{0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11}), "merge of 4 sorted lists");

        int[] stream = rnd.ints(100_000, 0, 1_000_000).toArray();
        List<Integer> top = topK(stream, 5);
        int[] sorted = stream.clone(); Arrays.sort(sorted);
        List<Integer> expected = new ArrayList<>();
        for (int i = 0; i < 5; i++) expected.add(sorted[sorted.length - 1 - i]);
        check(top.equals(expected), "top-5 of 100,000 numbers via a size-5 min-heap");

        MedianFinder mf = new MedianFinder();
        List<Integer> seen = new ArrayList<>();
        boolean medOk = true;
        for (int i = 0; i < 2000; i++) {
            int x = rnd.nextInt(500); mf.add(x); seen.add(x);
            List<Integer> s = new ArrayList<>(seen); Collections.sort(s);
            int m = s.size();
            double exp = m % 2 == 1 ? s.get(m / 2) : (s.get(m / 2 - 1) + (long) s.get(m / 2)) / 2.0;
            if (exp != mf.median()) medOk = false;
        }
        check(medOk, "running median (two heaps) correct after each of 2000 inserts");
    }
}
```

**Output:**

```
after insert 15:     [16, 15, 10, 8, 14, 9, 3, 2, 4, 1, 7]
extractMax() = 16  -> [15, 14, 10, 8, 7, 9, 3, 2, 4, 1]
increaseKey(4->15):  [15, 15, 10, 14, 7, 9, 3, 2, 8, 1]
ok   heap property holds; max is 15
ok   200,000 random insert/extract/delete/increaseKey ops agree with java.util.PriorityQueue
ok   extractMax on an empty queue throws (heap underflow)
k-way merge: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
ok   merge of 4 sorted lists
ok   top-5 of 100,000 numbers via a size-5 min-heap
ok   running median (two heaps) correct after each of 2000 inserts
```

The first three lines match the hand traces in Section C exactly.

### Java's `PriorityQueue`: what you need to know

| Fact | Detail |
|---|---|
| Default ordering | **min-heap** (natural order). Use `new PriorityQueue<>(Comparator.reverseOrder())` for a max-heap. |
| `add` / `offer` | O(log n) sift-up |
| `poll` | O(log n) sift-down |
| `peek` | O(1) |
| `remove(Object)` and `contains` | **O(n)**: linear search, then O(log n) to fix the heap |
| **No decrease-key** | Changing an element's key **while it's inside the queue corrupts the heap**. Workaround: **lazy deletion**: insert a new (key, item) entry and skip stale entries when they're polled. That's what most Java Dijkstra implementations do. |
| Iteration order | **Not sorted.** `toString()` and `for-each` show the internal array order. |
| Constructor from a collection | O(n) heapify |
| Thread safety | Not thread-safe. Use `PriorityBlockingQueue` for concurrent access. |
| Ties | Order among equal priorities isn't FIFO. Add a sequence number as a tie-breaker if you need FIFO. |

**Common mistakes**
1. Writing the comparator as `(a, b) -> a - b`. It **overflows** for large or negative ints. Use `Integer.compare(a, b)`.
2. Mutating an object's priority field while it's in the queue.
3. Expecting `System.out.println(pq)` to print sorted output.
4. Using `pq.remove(x)` inside a loop, which is Θ(n) each time.

---

## F. Time complexity

| Operation | Binary heap | Derivation |
|---|---|---|
| MAXIMUM / peek | **Θ(1)** | read A[0] |
| INSERT | **O(log n)** | sift-up moves along at most the height ⌊log₂ n⌋ |
| EXTRACT-MAX | **O(log n)** | O(1) to move the last element up, plus MAX-HEAPIFY at O(log n) |
| INCREASE-KEY | **O(log n)** | sift-up |
| DELETE(index) | **O(log n)** | one sift-up or one sift-down |
| DELETE(value) without an index | O(n) | finding it is a linear search |
| Build from n elements | **Θ(n)** | [bottom-up](building-a-heap.md) |
| Merge two heaps | Θ(n) | rebuild. Use [binomial](../../04-Trees/binomial-heaps/binomial-trees.md) or [Fibonacci](../../04-Trees/fibonacci-heaps/structure.md) heaps for fast merging. |

**Applications:**
- **k-way merge of N total elements:** N polls and adds on a heap of size ≤ k, so **Θ(N log k)**.
- **Top-k of a stream of n:** **Θ(n log k)** time, Θ(k) space.
- **Running median:** O(log n) per insert, O(1) per query.

**Amortized growth:** doubling the array when it fills makes INSERT **O(log n) amortized** (the occasional O(n) copy averages out to O(1) per insert). See [Dynamic tables](../../06-Algorithm-Design-Techniques/amortized-analysis/dynamic-tables.md).

## G. Space complexity

Θ(n) for the array. Operations use Θ(1) auxiliary space (iterative sifts). Java's `PriorityQueue<Integer>` stores boxed `Integer` objects, about 5× the memory of an `int[]` heap. See [Space complexity](../../00-Foundations/space-complexity.md#memory-in-java-64-bit-jvm-with-compressed-pointers-typical).

## H. Complexity summary

| Implementation | Insert | Find max | Extract max | Increase/decrease key | Merge |
|---|---|---|---|---|---|
| Unsorted array | Θ(1) | Θ(n) | Θ(n) | Θ(1) | Θ(1) (linked) |
| Sorted array | Θ(n) | Θ(1) | Θ(1) | Θ(n) | Θ(n) |
| **Binary heap** | **Θ(log n)** | **Θ(1)** | **Θ(log n)** | **Θ(log n)** | Θ(n) |
| Balanced BST (`TreeMap`) | Θ(log n) | Θ(log n) | Θ(log n) | Θ(log n) | Θ(n) |
| [Binomial heap](../../04-Trees/binomial-heaps/binomial-trees.md) | O(log n) (O(1) amortized) | O(log n) | Θ(log n) | Θ(log n) | **Θ(log n)** |
| [Fibonacci heap](../../04-Trees/fibonacci-heaps/structure.md) | **Θ(1)** | Θ(1) | O(log n) amortized | **Θ(1) amortized** | **Θ(1)** |

Fibonacci heaps give Dijkstra its theoretical O(E + V log V) bound, but binary heaps are faster in practice for almost every real graph.

## I. Advantages, limitations, and comparisons

**Advantages:** simple, compact, fast in practice, and everything is O(log n) or better.
**Limitations:** no efficient search, no ordered iteration, slow merging. Java's version also has no decrease-key.

**Heap vs `TreeSet`/`TreeMap` as a priority queue:** a TreeMap supports O(log n) removal of *any* key and ordered iteration, plus both min and max, but it has a higher constant factor and doesn't allow duplicate keys unless you wrap them. Use a heap for "always take the min or max". Use a TreeMap when you also need arbitrary removal or range queries.

**Interview follow-ups:** "Implement a min-heap from scratch", "Merge k sorted lists", "Find the median from a data stream", "Kth largest element", "Task scheduler", "Why doesn't Java's PriorityQueue support decrease-key, and how does Dijkstra cope?"

---

## J. Practice

**Beginner**
1. Trace HEAP-EXTRACT-MAX on A = [15, 13, 9, 5, 12, 8, 7, 4, 0, 6, 2, 1] (CLRS Exercise 6.5-1).
2. Trace MAX-HEAP-INSERT(A, 10) on the same heap (CLRS Exercise 6.5-2).
3. Implement a min-priority queue with HEAP-MINIMUM, HEAP-EXTRACT-MIN, HEAP-DECREASE-KEY and MIN-HEAP-INSERT (CLRS Exercise 6.5-3).

**Intermediate**
4. Implement a FIFO queue and a stack using a priority queue (CLRS Exercise 6.5-7).
5. Implement HEAP-DELETE(A, i) in O(lg n) (CLRS Exercise 6.5-8).
6. Merge k sorted lists in O(n lg k) (CLRS Exercise 6.5-9).

**Advanced**
7. Implement an **indexed priority queue** (a heap plus a position map) that supports decrease-key by item ID in O(log n).
8. Implement a d-ary heap, and analyse insert and extract as functions of d (CLRS Problem 6-2).

**Interview questions**

<details><summary>Q1. How would you find the k-th largest element in an array?</summary>

Keep a min-heap of size k. For each element, if the heap has fewer than k elements, add it. Otherwise, if the element is bigger than the heap's minimum, replace the minimum. At the end the root is the k-th largest. That's Θ(n log k) time and Θ(k) space. Alternatively, quickselect is Θ(n) expected. See [Quickselect](../order-statistics/quickselect.md).
</details>

<details><summary>Q2. How does the two-heap running-median algorithm work?</summary>

A max-heap holds the smaller half of the numbers and a min-heap holds the larger half. Keep their sizes equal, or let the max-heap have one extra. The median is the max-heap's root (odd count), or the average of both roots (even count). Each insert is O(log n) and each query is O(1).
</details>

<details><summary>Q3. Java's PriorityQueue has no decrease-key. How do you implement Dijkstra?</summary>

Lazy deletion. When a shorter distance to v is found, insert a new pair (dist, v) instead of updating the old one. When polling, skip pairs whose distance is greater than the current best dist[v]. The heap can hold up to E entries, so the time is O(E log E) = O(E log V), the same asymptotically.
</details>

**Worked problem: Task Scheduler (LeetCode 621 style).** Count each task's frequency, and use a max-heap of counts. In each cycle of n + 1 slots, pop up to n + 1 tasks, decrement their counts, and push back any that remain. The total time is the sum of the cycle lengths (with only the actual task count in the last cycle). That's O(T log 26), where T is the number of tasks.

**Coding problems**
- LeetCode 23 · Merge k Sorted Lists *(verify link)*
- LeetCode 295 · Find Median from Data Stream *(verify link)*
- LeetCode 215 · Kth Largest Element in an Array *(verify link)*
- LeetCode 347 · Top K Frequent Elements *(verify link)*
- LeetCode 621 · Task Scheduler *(verify link)*
- LeetCode 743 · Network Delay Time (Dijkstra with a heap) *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. A priority queue supports insert, max (or min), extract and change-key. A binary heap does each in O(log n), and peek in O(1).
2. INSERT and INCREASE-KEY **sift up**. EXTRACT and HEAPIFY **sift down**. DELETE does one or the other.
3. Java's `PriorityQueue` is a **min-heap**, has **no decrease-key**, and its `remove(Object)` is O(n).
4. Classic patterns: top-k (a size-k min-heap), k-way merge, running median (two heaps), Dijkstra with lazy deletion.
5. Fibonacci heaps improve decrease-key to O(1) amortized, which matters in theory more than in practice.

**Formulas:** k-way merge Θ(N log k). Top-k Θ(n log k). Build Θ(n).

**Common mistakes:** a comparator that overflows (`a - b`), mutating keys while inside the queue, assuming iteration is sorted.

**Quiz**
1. How do you make a max-heap with Java's PriorityQueue?
2. Cost of `pq.remove(someObject)`?
3. Which direction does INCREASE-KEY sift in a max-heap?
4. What's the complexity of merging k sorted lists of total length N?

**Answers**

<details><summary>Show answers</summary>

1. `new PriorityQueue<>(Comparator.reverseOrder())`.
2. O(n): a linear search, plus O(log n) to repair the heap.
3. Up, because a larger key can only conflict with its parent.
4. Θ(N log k).
</details>

**Related topics:** [Heap basics](heap-basics.md) · [Heapsort](heapsort.md) · [Dijkstra](../../07-Graph-Algorithms/shortest-paths/dijkstra.md) · [Huffman coding](../../06-Algorithm-Design-Techniques/greedy-algorithms/huffman-coding.md) · [Fibonacci heaps](../../04-Trees/fibonacci-heaps/structure.md) · [Quickselect](../order-statistics/quickselect.md)
