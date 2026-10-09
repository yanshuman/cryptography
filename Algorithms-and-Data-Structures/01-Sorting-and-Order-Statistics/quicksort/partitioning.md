# Partitioning (Lomuto, Hoare, 3-way)

> **CLRS:** 7.1 (Lomuto PARTITION), Problem 7-1 (Hoare partition) · **Status:** ✅ Written · **Prerequisites:** [Quicksort basics](quicksort-basics.md) · **Next:** [Randomized quicksort](randomized-quicksort.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

**Partitioning** rearranges an array around a **pivot** value x so that small elements end up on one side and large elements on the other. It's the heart of [quicksort](quicksort-basics.md) and [quickselect](../order-statistics/quickselect.md).

There are three classic schemes, each with different strengths:

| Scheme | Inventor | Regions produced | Key property |
|---|---|---|---|
| **Lomuto** | Nico Lomuto (popularised by Bentley, and CLRS) | ≤ x \| x \| > x | Simplest. The pivot ends in its final spot. Bad with duplicates. |
| **Hoare** | Tony Hoare (1961, the original) | ≤ x \| ≥ x | ~3× fewer swaps. Balanced splits on duplicates. Trickier. |
| **3-way** (Dutch national flag) | Edsger Dijkstra | < x \| = x \| > x | **Θ(n) on all-equal input.** The best choice with many duplicates. |

**What problem does it solve?** It's the "divide" step of quicksort: Θ(n) work that puts the pivot in its final place. The choice of scheme affects speed, behaviour on duplicates and correctness at the edges.

**Where it's used:** quicksort and quickselect, `Arrays.sort` (Java's dual-pivot partitions into **three** parts using two pivots), LeetCode's "Sort Colors", "Move Zeroes" and "Partition Array", and the partition stage of database hash joins.

---

## B. Intuition

**Lomuto is "one scanning finger".** One pointer j scans left to right. Another pointer i marks the end of the "small" zone. When j finds a small element, the small zone grows by one and the element is swapped in.

**Hoare is "two fingers moving towards each other".** The left finger skips over elements that are already small, and the right finger skips over elements that are already large. When both stop, they've each found an element on the wrong side, so swap them. Repeat until the fingers cross. Each swap fixes **two** misplaced elements at once, which is why it needs fewer swaps.

**3-way is "sorting a flag with three colours".** Keep red (< x) at the front, blue (> x) at the back and white (= x) in the middle, with an unknown zone shrinking between white and blue. Elements equal to the pivot are **gathered in the middle and excluded from recursion**, which is why many duplicates become cheap.

---

## C. How it works internally

### Lomuto on [2, 8, 7, 1, 3, 5, 6, 4] (pivot 4)

Traced in full in [Quicksort basics](quicksort-basics.md#c-how-it-works-internally). Result: [2, 1, 3, **4**, 7, 5, 6, 8], with the pivot at index 3. That took **4 swaps**, three in the loop (one of them a self-swap) plus the final one.

### Hoare on the same array (pivot = first element = 2)

```
x = A[0] = 2.  i starts before the array, j after it.
[2, 8, 7, 1, 3, 5, 6, 4]
 i→ stops at index 0 (2 ≥ 2);   j← stops at index 3 (1 ≤ 2)   → swap → [1, 8, 7, 2, 3, 5, 6, 4]
 i→ stops at index 1 (8 ≥ 2);   j← stops at index 0 (1 ≤ 2)   → i ≥ j → return j = 0
Left part A[0..0] = [1] (all ≤ 2)        Right part A[1..7] = [8, 7, 2, 3, 5, 6, 4] (all ≥ 2)
```

Notice that the **pivot (2) isn't in its final position**, and isn't even necessarily at the boundary. Hoare only guarantees left ≤ x ≤ right, so recursion is on **[p..j] and [j + 1..r]**, which *include* the boundary element.

### 3-way on [3, 1, 3, 2, 3, 5, 3] (pivot 3)

```
Invariant:  A[lo..lt−1] < x  |  A[lt..i−1] = x  |  A[i..gt] unknown  |  A[gt+1..hi] > x
start: lt=0, i=0, gt=6       [3, 1, 3, 2, 3, 5, 3]
i=0: 3 = x → i++              [3, 1, 3, 2, 3, 5, 3]          lt=0 i=1 gt=6
i=1: 1 < x → swap(lt,i), lt++, i++  [1, 3, 3, 2, 3, 5, 3]    lt=1 i=2
i=2: 3 = x → i++                                              i=3
i=3: 2 < x → swap(lt,i)       [1, 2, 3, 3, 3, 5, 3]          lt=2 i=4
i=4: 3 = x → i++                                              i=5
i=5: 5 > x → swap(i,gt), gt-- [1, 2, 3, 3, 3, 3, 5]          gt=5 (i stays at 5: the swapped-in element is unknown)
i=5: 3 = x → i++                                              i=6 > gt → stop
Result: [1, 2 | 3, 3, 3, 3 | 5]   recurse only on [1, 2] and [5]; the four 3s are done.
```

### Behaviour on all-equal input [5, 5, 5, 5, 5, 5, 5, 5]

| Scheme | Split | Result for the whole sort |
|---|---|---|
| Lomuto | everything goes ≤ x, so a split of n − 1 and 0 | **Θ(n²)** |
| Hoare | the fingers stop at every element and meet in the middle, so a split of n/2 and n/2 | Θ(n log n) |
| 3-way | everything equals x, so a split of 0, n (finished) and 0 | **Θ(n)** |

---

## D. Algorithm and pseudocode

### Lomuto partition (CLRS)

```
LOMUTO-PARTITION(A, p, r)
1  x ← A[r]
2  i ← p − 1
3  for j ← p to r − 1
4      if A[j] ≤ x
5          i ← i + 1
6          exchange A[i] ↔ A[j]
7  exchange A[i + 1] ↔ A[r]
8  return i + 1                        ▷ A[p..q−1] ≤ A[q] = x < A[q+1..r]
```

**Loop invariant** (CLRS §7.1). At the start of each iteration of lines 3–6, for any index k:
1. p ≤ k ≤ i ⇒ A[k] ≤ x
2. i + 1 ≤ k ≤ j − 1 ⇒ A[k] > x
3. k = r ⇒ A[k] = x

- **Initialization:** i = p − 1 and j = p, so both ranges are empty. A[r] = x by line 1. ✓
- **Maintenance:** if A[j] > x, then only j grows, and condition 2 now includes A[j]. ✓ If A[j] ≤ x, then i grows and A[i] (previously the first element > x, or A[j] itself) is swapped with A[j]. The new A[i] is ≤ x, and the old A[i], now at position j, is > x. ✓
- **Termination:** j = r, so every element of A[p..r − 1] is in one of the two regions. Line 7 swaps the pivot with the first element > x, which places the pivot between the regions. ✓

### Hoare partition (CLRS Problem 7-1)

```
HOARE-PARTITION(A, p, r)
1   x ← A[p]
2   i ← p − 1
3   j ← r + 1
4   while TRUE
5       repeat j ← j − 1 until A[j] ≤ x
6       repeat i ← i + 1 until A[i] ≥ x
7       if i < j
8           exchange A[i] ↔ A[j]
9       else return j                  ▷ A[p..j] ≤ x ≤ A[j+1..r], and p ≤ j < r

QUICKSORT-HOARE(A, p, r)
1  if p < r
2      q ← HOARE-PARTITION(A, p, r)
3      QUICKSORT-HOARE(A, p, q)        ▷ note: q, not q − 1
4      QUICKSORT-HOARE(A, q + 1, r)
```

**Why it's correct and terminates** (the key facts from CLRS Problem 7-1):
- i and j never go outside [p, r]. On the first round, i stops at p (A[p] = x ≥ x). After any swap, A[i] ≤ x and A[j] ≥ x act as **sentinels** that stop the scans in later rounds.
- The returned j satisfies **p ≤ j < r**, so both recursive parts are strictly smaller than [p..r]. That guarantees termination. (Using the **last** element as the pivot breaks this: j can equal r, and the recursion never ends.)
- Every element of A[p..j] is ≤ every element of A[j + 1..r].

### 3-way partition (Dutch national flag)

```
PARTITION-3WAY(A, lo, hi)
1  x ← A[lo]; lt ← lo; i ← lo; gt ← hi
2  while i ≤ gt
3      if A[i] < x:  exchange A[lt] ↔ A[i]; lt ← lt + 1; i ← i + 1
4      elif A[i] > x: exchange A[i] ↔ A[gt]; gt ← gt − 1     ▷ don't advance i: the swapped-in element is unexamined
5      else: i ← i + 1
6  return (lt, gt)                     ▷ A[lo..lt−1] < x, A[lt..gt] = x, A[gt+1..hi] > x
```

**Invariant:** A[lo..lt − 1] < x, A[lt..i − 1] = x, A[i..gt] is unknown, and A[gt + 1..hi] > x. Each iteration shrinks the unknown zone by one, so after exactly hi − lo + 1 iterations it's empty.

---

## E. Implementation

```java
import java.util.*;

/** Lomuto, Hoare and 3-way partitioning, each driving a quicksort, compared on random and all-equal input. */
public class Partitioning {

    static long comparisons, swaps;

    static void swap(int[] a, int i, int j) { int t = a[i]; a[i] = a[j]; a[j] = t; swaps++; }

    // ---------- Lomuto (CLRS) ----------
    static int lomuto(int[] a, int p, int r) {
        int x = a[r], i = p - 1;
        for (int j = p; j < r; j++) { comparisons++; if (a[j] <= x) swap(a, ++i, j); }
        swap(a, i + 1, r);
        return i + 1;
    }
    static void quickLomuto(int[] a, int p, int r) {
        while (p < r) {                                      // loop on the larger side: O(log n) stack
            int q = lomuto(a, p, r);
            if (q - p < r - q) { quickLomuto(a, p, q - 1); p = q + 1; }
            else               { quickLomuto(a, q + 1, r); r = q - 1; }
        }
    }

    // ---------- Hoare (CLRS Problem 7-1) ----------
    static int hoare(int[] a, int p, int r) {
        int x = a[p], i = p - 1, j = r + 1;
        while (true) {
            do { j--; comparisons++; } while (a[j] > x);
            do { i++; comparisons++; } while (a[i] < x);
            if (i < j) swap(a, i, j); else return j;
        }
    }
    static void quickHoare(int[] a, int p, int r) {
        while (p < r) {
            int q = hoare(a, p, r);                          // recurse on [p..q] and [q+1..r]
            if (q - p < r - q) { quickHoare(a, p, q); p = q + 1; }
            else               { quickHoare(a, q + 1, r); r = q; }
        }
    }

    // ---------- 3-way (Dutch national flag) ----------
    static int[] threeWay(int[] a, int lo, int hi) {
        int x = a[lo], lt = lo, i = lo, gt = hi;
        while (i <= gt) {
            comparisons++;
            if (a[i] < x) swap(a, lt++, i++);
            else if (a[i] > x) swap(a, i, gt--);
            else i++;
        }
        return new int[]{lt, gt};
    }
    static void quick3(int[] a, int lo, int hi) {
        while (lo < hi) {
            int[] b = threeWay(a, lo, hi);
            if (b[0] - lo < hi - b[1]) { quick3(a, lo, b[0] - 1); lo = b[1] + 1; }
            else                       { quick3(a, b[1] + 1, hi); hi = b[0] - 1; }
        }
    }

    interface Sorter { void sort(int[] a); }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        int[] l = {2, 8, 7, 1, 3, 5, 6, 4};
        int q = lomuto(l, 0, 7);
        System.out.println("Lomuto  -> " + Arrays.toString(l) + "  returns " + q);
        int[] h = {2, 8, 7, 1, 3, 5, 6, 4};
        int j = hoare(h, 0, 7);
        System.out.println("Hoare   -> " + Arrays.toString(h) + "  returns " + j + "  (left = a[0.." + j + "])");
        int[] t = {3, 1, 3, 2, 3, 5, 3};
        int[] b = threeWay(t, 0, 6);
        System.out.println("3-way   -> " + Arrays.toString(t) + "  equal block = [" + b[0] + ".." + b[1] + "]");
        check(Arrays.equals(t, new int[]{1, 2, 3, 3, 3, 3, 5}) && b[0] == 2 && b[1] == 5, "3-way trace matches Section C");

        Map<String, Sorter> sorters = new LinkedHashMap<>();
        sorters.put("Lomuto", a -> quickLomuto(a, 0, a.length - 1));
        sorters.put("Hoare",  a -> quickHoare(a, 0, a.length - 1));
        sorters.put("3-way",  a -> quick3(a, 0, a.length - 1));

        Random rnd = new Random(12);
        for (var e : sorters.entrySet()) {
            boolean ok = true;
            for (int k = 0; k < 2000; k++) {
                int[] x = rnd.ints(rnd.nextInt(300), -50, 50).toArray();   // many duplicates
                int[] exp = x.clone(); Arrays.sort(exp);
                e.getValue().sort(x);
                if (!Arrays.equals(x, exp)) ok = false;
            }
            check(ok, e.getKey() + " quicksort: 2000 random arrays with duplicates match Arrays.sort");
        }

        int n = 20_000;
        int[] random = rnd.ints(n).toArray();
        int[] equal = new int[n]; Arrays.fill(equal, 7);
        System.out.printf("%n%-8s %24s %24s%n", "scheme", "random: comps / swaps", "all-equal: comps / swaps");
        long lomutoEqual = 0, threeEqual = 0;
        for (var e : sorters.entrySet()) {
            comparisons = swaps = 0; e.getValue().sort(random.clone());
            long rc = comparisons, rs = swaps;
            comparisons = swaps = 0; e.getValue().sort(equal.clone());
            System.out.printf("%-8s %12d / %-10d %12d / %-10d%n", e.getKey(), rc, rs, comparisons, swaps);
            if (e.getKey().equals("Lomuto")) lomutoEqual = comparisons;
            if (e.getKey().equals("3-way")) threeEqual = comparisons;
        }
        check(lomutoEqual == (long) n * (n - 1) / 2, "Lomuto on all-equal input is quadratic: n(n-1)/2 comparisons");
        check(threeEqual == n, "3-way on all-equal input is linear: exactly n comparisons");
    }
}
```

**Output:**

```
Lomuto  -> [2, 1, 3, 4, 7, 5, 6, 8]  returns 3
Hoare   -> [1, 8, 7, 2, 3, 5, 6, 4]  returns 0  (left = a[0..0])
3-way   -> [1, 2, 3, 3, 3, 3, 5]  equal block = [2..5]
ok   3-way trace matches Section C
ok   Lomuto quicksort: 2000 random arrays with duplicates match Arrays.sort
ok   Hoare quicksort: 2000 random arrays with duplicates match Arrays.sort
ok   3-way quicksort: 2000 random arrays with duplicates match Arrays.sort

scheme      random: comps / swaps all-equal: comps / swaps
Lomuto         344169 / 179912        199990000 / 200009999 
Hoare          463416 / 66159            318430 / 139216    
3-way          340591 / 327245            20000 / 0         
ok   Lomuto on all-equal input is quadratic: n(n-1)/2 comparisons
ok   3-way on all-equal input is linear: exactly n comparisons
```

**What the table shows**
- **Random input:** Hoare does about **2.7× fewer swaps** than Lomuto (about 66k vs 180k), because each Hoare swap fixes two misplaced elements. It pays for this with somewhat more comparisons (its scans overlap at the meeting point), but swaps (memory writes) usually cost more than comparisons.
- **All-equal input:** Lomuto blows up to about 2 × 10⁸ comparisons and swaps (quadratic). Hoare stays around n log n, because its fingers meet in the middle. 3-way finishes in **exactly n comparisons and 0 swaps**.

**Java-specific details:** the `while (p < r)` loop that recurses on the **smaller** side keeps the stack at O(log n), even for Lomuto's quadratic all-equal case. Without it, n = 20,000 equal elements would recurse 20,000 deep. Java's own `DualPivotQuicksort` uses two pivots p₁ ≤ p₂ and a 3-region partition (< p₁, between, > p₂), plus a special path for equal pivots.

**Common mistakes**
1. **Hoare with recursion (p, q − 1), (q + 1, r)** is wrong: q isn't the pivot's final position. Use (p, q) and (q + 1, r).
2. **Hoare with the last element as pivot** can return j = r, which causes infinite recursion.
3. **3-way: advancing i after swapping with gt.** The element that came from gt hasn't been examined yet.
4. **Lomuto on data with many duplicates:** quadratic. Switch to 3-way.

---

## F. Time complexity

All three partition schemes take **Θ(n)** per call: each element is compared a constant number of times.

| Scheme | Comparisons per call (size n) | Swaps per call, on random data | Effect on the full quicksort (all-equal input) |
|---|---|---|---|
| Lomuto | n − 1 | ≈ n/2 | Θ(n²) |
| Hoare | ≈ n + O(1) | ≈ n/6 | Θ(n log n) |
| 3-way | n | depends on the duplicates | **Θ(n)** |

**3-way quicksort with many duplicates:** if there are only k distinct keys, the recursion depth is at most about k levels, and each level is Θ(n), giving **O(n·k)**. More precisely, the expected cost is Θ(n·H), where H is the entropy of the key distribution. That's optimal: no comparison sort can beat the entropy bound.

## G. Space complexity

All three are **in place**: Θ(1) auxiliary per partition call. The quicksort stack is Θ(log n) with smaller-side-first recursion.

## H. Complexity summary

| Partition scheme | Time per call | Extra space | Pivot ends in its final position? | Good with duplicates? | Stable? |
|---|---|---|---|---|---|
| Lomuto | Θ(n) | Θ(1) | ✅ | ❌ | ❌ |
| Hoare | Θ(n) | Θ(1) | ❌ | ✅ (balanced) | ❌ |
| 3-way | Θ(n) | Θ(1) | ✅ (the whole equal block) | ✅✅ (equal keys are finished) | ❌ |

## I. Advantages, limitations, and comparisons

- **Lomuto:** the easiest to write and prove, and the pivot index is useful for quickselect. But it does more swaps and is quadratic on duplicates.
- **Hoare:** the fastest in practice for distinct keys, with the fewest swaps. Its boundaries are subtle and easy to get wrong in interviews.
- **3-way:** essential when duplicates are common (sorting by country, by status code). It does slightly more swaps than Hoare on distinct keys.
- **Dual-pivot (Java):** three regions with two pivots. Fewer memory accesses per element, about 10% faster than classic quicksort on modern CPUs.

**Interview follow-ups:** "Sort an array of 0s, 1s and 2s", "Move all zeros to the end", "Partition a linked list around x" (LeetCode 86), "Why does quicksort slow down with many duplicates?"

---

## J. Practice

**Beginner**
1. Trace Hoare partition on [13, 19, 9, 5, 12, 8, 7, 4, 11, 2, 6, 21] (CLRS Problem 7-1a).
2. Run Lomuto on an all-equal array of 5 elements. What does it return?
3. Implement "move zeroes to the end", keeping the order of the non-zero elements.

**Intermediate**
4. Prove that Hoare's indices never leave [p, r] (CLRS Problem 7-1b).
5. Prove that Hoare returns p ≤ j < r (CLRS Problem 7-1c).
6. Implement 3-way quicksort and show it's linear when there are O(1) distinct keys.

**Advanced**
7. Implement Bentley–McIlroy 3-way partitioning (equal keys are swapped to the ends, then into the middle).
8. Implement Yaroslavskiy dual-pivot partitioning and count the comparisons against classic Hoare.

**Interview questions**

<details><summary>Q1. What's the difference between Lomuto and Hoare partitioning?</summary>

Lomuto uses one forward scan with the last element as pivot. It places the pivot in its final position and does about n/2 swaps on random data. Hoare uses two pointers moving towards each other, with the first element as pivot. It does about 3× fewer swaps and balances equal keys, but the pivot isn't necessarily in its final position, so you recurse on (p, j) and (j + 1, r).
</details>

<details><summary>Q2. How do you make quicksort efficient with many duplicates?</summary>

Use 3-way partitioning (< x, = x, > x) and recurse only on the < and > parts. All copies of the pivot are finished in one pass. With k distinct keys the cost is O(nk), and it's Θ(n) when all keys are equal.
</details>

<details><summary>Q3. Solve "Sort Colors" (0s, 1s, 2s) in one pass with O(1) space.</summary>

3-way partition with pivot 1. Keep lo, mid and hi pointers. If a[mid] = 0, swap it with a[lo] and advance both. If it's 2, swap it with a[hi] and decrement hi (don't advance mid). If it's 1, advance mid. Stop when mid > hi.
</details>

**Worked problem: Partition List (LeetCode 86).** Given a linked list and x, put all nodes < x before nodes ≥ x, preserving the relative order. Build two lists, "less" and "greater-or-equal", by relinking nodes as you scan, then join them. That's Θ(n) time and Θ(1) space, and it's **stable** (unlike array partitioning), because linked lists don't need swaps.

**Coding problems**
- LeetCode 75 · Sort Colors *(verify link)*
- LeetCode 283 · Move Zeroes *(verify link)*
- LeetCode 86 · Partition List *(verify link)*
- LeetCode 905 · Sort Array By Parity *(verify link)*
- LeetCode 2161 · Partition Array According to Given Pivot *(verify link)*

---

## K. Revision notes

**Five key takeaways**
1. All partition schemes are Θ(n) and in place.
2. Lomuto: simple, the pivot ends in its final place, more swaps, quadratic with duplicates.
3. Hoare: two pointers, about 3× fewer swaps. Recurse on (p, j) and (j + 1, r). Never pivot on the last element.
4. 3-way: < | = | >. Linear on all-equal input, and optimal with many duplicates.
5. Java's primitive sort uses dual-pivot partitioning.

**Common mistakes:** Hoare recursion bounds, advancing i after the gt-swap, using Lomuto on duplicate-heavy data.

**Quiz**
1. Which scheme guarantees that the pivot is in its final position: Lomuto or Hoare?
2. How many comparisons does 3-way partitioning make on n equal keys?
3. Why must Hoare not use A[r] as the pivot?
4. Which scheme does "Sort Colors" use?

**Answers**

<details><summary>Show answers</summary>

1. Lomuto (and 3-way).
2. n.
3. It could return j = r, so the recursive call (p, r) would have the same size, which means infinite recursion.
4. 3-way (the Dutch national flag).
</details>

**Related topics:** [Quicksort basics](quicksort-basics.md) · [Randomized quicksort](randomized-quicksort.md) · [Complexity analysis](complexity-analysis.md) · [Quickselect](../order-statistics/quickselect.md)
