# Lower Bounds for Sorting

> **CLRS:** 8.1 (Lower bounds for sorting) · **Status:** ✅ Written · **Prerequisites:** [Asymptotic notation](../../00-Foundations/asymptotic-notation.md), [Trees](../../14-Mathematical-Foundations/trees.md) · **Next:** [Counting sort](counting-sort.md) · **Section:** [Sorting](../README.md)

---

## A. Introduction

**Theorem:** any **comparison sort** must make **Ω(n log n)** comparisons in the worst case. More precisely, it needs at least **⌈log₂(n!)⌉** comparisons, which is about n log₂ n − 1.44n.

**What problem does it solve?** It answers "**can we sort faster than n log n?**" with a definitive **no**, *if* the only way to learn about the data is by comparing pairs. That tells us:
- [Merge sort](../comparison-sorts/merge-sort.md) and [heapsort](../heapsort/heapsort.md) are **asymptotically optimal** comparison sorts. Searching for a faster comparison sort is pointless.
- To beat n log n, you must use **more than comparisons**: look at the keys' digits or values, as [counting](counting-sort.md), [radix](radix-sort.md) and [bucket sort](bucket-sort.md) do.

**Why it matters beyond sorting:** it's the model example of a **lower-bound proof**, a statement about *every possible algorithm*, not just one. The same decision-tree argument gives Ω(log n) for searching a sorted array, and Ω(n log n) for problems that sorting reduces to (element distinctness in the comparison model, convex hull).

**Prerequisites:** binary trees (a tree of height h has ≤ 2ʰ leaves), logarithms, and factorials.

---

## B. Intuition

**Analogy: twenty questions.** A friend thinks of one of the n! possible orderings of n cards. You may only ask yes/no questions of the form "is card i smaller than card j?". Each answer can at best **halve** the number of orderings still possible. To get from n! possibilities down to 1, you need at least log₂(n!) questions.

| n | Possible orderings n! | Minimum comparisons ⌈log₂ n!⌉ |
|---|---|---|
| 3 | 6 | 3 |
| 4 | 24 | 5 |
| 5 | 120 | 7 |
| 10 | 3,628,800 | 22 |
| 20 | 2.4 × 10¹⁸ | 62 |

And log₂(n!) grows like **n log₂ n**. That's the whole idea.

---

## C. How it works internally: the decision-tree model

### Decision trees

Any comparison sort, run on inputs of a fixed size n, can be drawn as a **decision tree**:
- each **internal node** is a comparison aᵢ : aⱼ;
- the **left branch** means aᵢ ≤ aⱼ and the **right branch** means aᵢ > aⱼ;
- each **leaf** is a final answer: the permutation that sorts the input;
- running the algorithm on one input means following **one root-to-leaf path**;
- **the number of comparisons on that input = the length of that path**.

**Example: insertion sort on 3 elements** (CLRS Figure 8.1):

```
                          a1:a2
                    ≤ /          \ >
                 a2:a3            a1:a3
              ≤ /    \ >        ≤ /    \ >
        ⟨1,2,3⟩     a1:a3    ⟨2,1,3⟩    a2:a3
                  ≤ /   \ >          ≤ /    \ >
             ⟨1,3,2⟩  ⟨3,1,2⟩    ⟨2,3,1⟩  ⟨3,2,1⟩
```

There are 6 leaves, one per permutation of 3 elements (3! = 6). The height is 3, so the worst case is 3 comparisons.

### The proof (CLRS Theorem 8.1)

1. **Every permutation must appear as a reachable leaf.** If two different input orderings reached the same leaf, the algorithm would apply the same rearrangement to both, and at least one of them would come out unsorted. So there are **at least n! leaves**.
2. **A binary tree of height h has at most 2ʰ leaves.** Each level at most doubles the number of nodes.
3. Combining: n! ≤ (number of leaves) ≤ 2ʰ, so **h ≥ log₂(n!)**.
4. **log₂(n!) = Ω(n log n).** n! ≥ (n/2)^(n/2), because the top half of the factors 1·2·…·n are each ≥ n/2. So:

   log₂(n!) ≥ (n/2)·log₂(n/2) = Ω(n log n). ∎

   (Stirling's approximation gives the precise value: log₂(n!) = n log₂ n − n log₂ e + O(log n) ≈ n log₂ n − 1.443n.)

**The worst-case height h is the worst-case number of comparisons, so every comparison sort makes Ω(n log n) comparisons in the worst case.**

### Stronger versions

- **Average case too.** Even the *average* path length to a leaf, over all n! inputs, is ≥ log₂(n!). A binary tree with N leaves has average leaf depth ≥ log₂ N. So comparison sorts are Ω(n log n) **on average**, not just in the worst case.
- **Randomised algorithms too.** A randomised comparison sort is a probability distribution over decision trees, so its expected comparisons on a random input are still Ω(n log n).
- **Corollary:** heapsort and merge sort are **asymptotically optimal**. Merge sort's worst case, n⌈lg n⌉ − 2^⌈lg n⌉ + 1, is within about 0.44n of the bound.

### What the bound does NOT say

| Misreading | Reality |
|---|---|
| "No sort can beat n log n" | Only **comparison** sorts. Counting, radix and bucket sort use key values and can be **linear** under assumptions. |
| "Every input needs n log n comparisons" | Specific inputs can be fast: insertion sort is Θ(n) on sorted input. The bound is for the worst and average case. |
| "It applies to integers in a small range" | In that case you shouldn't be using comparisons. Counting sort is Θ(n + k). |

---

## D. Algorithm and pseudocode: the adversary argument

An equivalent, more "algorithmic" proof is the **adversary**. It doesn't fix the input in advance. It answers each comparison **so as to keep as many orderings consistent as possible**:

```
ADVERSARY-COMPARE(i, j)
1  S ← the set of permutations consistent with every answer given so far
2  S≤ ← { π ∈ S : π puts element i before element j }
3  S> ← S \ S≤
4  if |S≤| ≥ |S>| then answer "i ≤ j", S ← S≤
5  else answer "i > j", S ← S>
```

The adversary always keeps **at least half** of S. The sorting algorithm can only finish when |S| = 1 (otherwise two orderings remain, and its output would be wrong for one of them). Starting from n!, halving each time, that takes **at least ⌈log₂ n!⌉ comparisons**, whatever algorithm is used.

---

## E. Implementation

The program (1) tabulates ⌈log₂ n!⌉ against merge sort's worst case and the best known sorting networks or algorithms for small n, and (2) runs **real sorting algorithms against the adversary** on all 8! = 40,320 permutations, showing they're forced to use at least ⌈log₂ 8!⌉ = 16 comparisons.

```java
import java.util.*;

/** Comparison-sort lower bound: ceil(log2 n!) vs real algorithms, and an adversary that forces it. */
public class LowerBound {

    // ---------- Adversary over all permutations of n items ----------
    static final class Adversary {
        final List<int[]> consistent = new ArrayList<>();   // position[v] = where item v sits in the true order
        int comparisons;
        Adversary(int n) { permute(new int[n], new boolean[n], 0); }
        private void permute(int[] p, boolean[] used, int k) {
            if (k == p.length) { consistent.add(p.clone()); return; }
            for (int v = 0; v < p.length; v++) if (!used[v]) { used[v] = true; p[k] = v; permute(p, used, k + 1); used[v] = false; }
        }
        /** Answers "is item i smaller than item j?" so as to keep the larger half of the orderings. */
        boolean less(int i, int j) {
            comparisons++;
            List<int[]> yes = new ArrayList<>(), no = new ArrayList<>();
            for (int[] rank : consistent) (rank[i] < rank[j] ? yes : no).add(rank);
            consistent.clear();
            if (yes.size() >= no.size()) { consistent.addAll(yes); return true; }
            consistent.addAll(no); return false;
        }
    }

    interface Cmp { boolean less(int i, int j); }

    static void insertionSort(Integer[] items, Cmp c) {
        for (int j = 1; j < items.length; j++) {
            int key = items[j], i = j - 1;
            while (i >= 0 && c.less(key, items[i])) { items[i + 1] = items[i]; i--; }
            items[i + 1] = key;
        }
    }

    static void mergeSort(Integer[] items, Cmp c) {
        if (items.length < 2) return;
        Integer[] l = Arrays.copyOfRange(items, 0, items.length / 2), r = Arrays.copyOfRange(items, items.length / 2, items.length);
        mergeSort(l, c); mergeSort(r, c);
        int i = 0, j = 0, k = 0;
        while (i < l.length && j < r.length) items[k++] = c.less(r[j], l[i]) ? r[j++] : l[i++];
        while (i < l.length) items[k++] = l[i++];
        while (j < r.length) items[k++] = r[j++];
    }

    static int ceilLog2Factorial(int n) {
        double s = 0; for (int k = 2; k <= n; k++) s += Math.log(k) / Math.log(2);
        return (int) Math.ceil(s - 1e-9);
    }

    static long mergeWorst(int n) {
        if (n < 2) return 0;
        int c = 32 - Integer.numberOfLeadingZeros(n - 1);
        return (long) n * c - (1L << c) + 1;
    }

    static void check(boolean ok, String what) { System.out.println((ok ? "ok   " : "FAIL ") + what); }

    public static void main(String[] args) {
        // Best possible worst-case comparisons known for small n (Ford-Johnson merge-insertion achieves these up to n = 11)
        int[] optimalKnown = {0, 0, 1, 3, 5, 7, 10, 13, 16, 19, 22, 26, 30};
        System.out.printf("%4s %16s %18s %20s %18s%n", "n", "n!", "ceil(log2 n!)", "best known worst", "merge sort worst");
        boolean bound = true;
        for (int n = 1; n <= 12; n++) {
            long f = 1; for (int k = 2; k <= n; k++) f *= k;
            int lb = ceilLog2Factorial(n);
            System.out.printf("%4d %16d %18d %20d %18d%n", n, f, lb, optimalKnown[n], mergeWorst(n));
            if (optimalKnown[n] < lb || mergeWorst(n) < lb) bound = false;
        }
        check(bound, "no algorithm beats ceil(log2 n!) (merge sort is within a few comparisons)");

        int n = 8;
        int lb = ceilLog2Factorial(n);
        Adversary adv = new Adversary(n);
        Integer[] items = new Integer[n]; for (int i = 0; i < n; i++) items[i] = i;
        insertionSort(items, adv::less);
        System.out.printf("%nadversary vs insertion sort (n=%d): %d comparisons, %d ordering(s) left%n", n, adv.comparisons, adv.consistent.size());
        check(adv.comparisons >= lb && adv.consistent.size() == 1, "insertion sort forced to >= ceil(log2 8!) = " + lb + " comparisons");

        adv = new Adversary(n);
        for (int i = 0; i < n; i++) items[i] = i;
        mergeSort(items, adv::less);
        System.out.printf("adversary vs merge sort     (n=%d): %d comparisons, %d ordering(s) left%n", n, adv.comparisons, adv.consistent.size());
        check(adv.comparisons >= lb && adv.consistent.size() == 1, "merge sort forced to >= " + lb + " comparisons too");

        // Stirling: log2(n!) ~ n log2 n - 1.443 n
        for (int m : new int[]{1000, 1_000_000}) {
            double exact = 0; for (int k = 2; k <= m; k++) exact += Math.log(k) / Math.log(2);
            double approx = m * Math.log(m) / Math.log(2) - m / Math.log(2);
            System.out.printf("n=%,d: log2(n!) = %,.0f   n log2 n - 1.443n = %,.0f%n", m, exact, approx);
        }
    }
}
```

**Output:**

```
   n               n!      ceil(log2 n!)     best known worst   merge sort worst
   1                1                  0                    0                  0
   2                2                  1                    1                  1
   3                6                  3                    3                  3
   4               24                  5                    5                  5
   5              120                  7                    7                  8
   6              720                 10                   10                 11
   7             5040                 13                   13                 14
   8            40320                 16                   16                 17
   9           362880                 19                   19                 21
  10          3628800                 22                   22                 25
  11         39916800                 26                   26                 29
  12        479001600                 29                   30                 33
ok   no algorithm beats ceil(log2 n!) (merge sort is within a few comparisons)

adversary vs insertion sort (n=8): 28 comparisons, 1 ordering(s) left
ok   insertion sort forced to >= ceil(log2 8!) = 16 comparisons
adversary vs merge sort     (n=8): 17 comparisons, 1 ordering(s) left
ok   merge sort forced to >= 16 comparisons too
n=1,000: log2(n!) = 8,529   n log2 n - 1.443n = 8,523
n=1,000,000: log2(n!) = 18,488,885   n log2 n - 1.443n = 18,488,874
```

**What the output shows**
- **The adversary forces insertion sort to its worst case (28 = 8·7/2 comparisons)** and merge sort to 17. Both are at least ⌈log₂ 8!⌉ = 16, as the theorem says they must be.
- For n = 12, the information-theoretic bound is 29, but the best possible is **30**, proven by exhaustive search. The bound isn't always achievable. It's a floor.
- Stirling's approximation n log₂ n − 1.443n is accurate to within a few units even at n = 10⁶.

**Java-specific detail:** `adv::less` passes the adversary's method as a `Cmp` lambda. The sorting algorithms never see real values, only answers, which is exactly the comparison model.

---

## F. Time complexity

| Statement | Bound |
|---|---|
| Worst-case comparisons, any comparison sort | ≥ ⌈log₂ n!⌉ = **Ω(n log n)** |
| Average-case comparisons, any comparison sort | ≥ log₂ n! − O(1) = **Ω(n log n)** |
| Merge sort worst case | n⌈lg n⌉ − 2^⌈lg n⌉ + 1 ≤ n lg n − n + 1, so **optimal up to about 0.44n** |
| Heapsort | ≈ 2n lg n, **optimal up to the constant factor 2** |
| Searching a sorted array (comparisons) | ≥ ⌈log₂(n + 1)⌉, which binary search achieves |

**The adversary program's cost** (not the bound itself) is Θ(n! · comparisons) to filter the permutation list. It's only feasible for tiny n, which is fine for a demonstration.

## G. Space complexity

The theorem says nothing about space. In-place optimal comparison sorts exist (heapsort: Θ(1) extra).

## H. Complexity summary

| Model | Lower bound for sorting | Achieved by |
|---|---|---|
| Comparisons only | Ω(n log n) | merge sort, heapsort, quicksort (expected) |
| Integer keys in [0, k] | Ω(n) (you must read the input) | [counting sort](counting-sort.md) Θ(n + k) |
| d-digit keys | Ω(n) | [radix sort](radix-sort.md) Θ(d(n + k)) |
| Uniformly random reals | Ω(n) expected | [bucket sort](bucket-sort.md) Θ(n) expected |

## I. Advantages, limitations, and comparisons

- **The power of the result:** it ends the search for faster comparison sorts and explains *why* the non-comparison sorts need extra assumptions.
- **Its limitation:** it's a model-dependent bound. Real computers can do more than compare, such as index arrays by key value or use bit tricks. With w-bit integer keys, there are algorithms faster than n log n (for example, O(n log log n), Han 2002). Those are theoretical and rarely practical.
- **Where it also applies:** element distinctness (Ω(n log n) comparisons), convex hull (reduces from sorting), constructing a BST from unsorted data (an inorder traversal would sort, so Ω(n log n)).

**Interview follow-ups:** "Why can't we sort faster than n log n?", "Then how is counting sort O(n)?", "Prove that building a BST requires Ω(n log n)."

---

## J. Practice

**Beginner**
1. Draw the decision tree for insertion sort on 3 elements and verify that it has 3! leaves.
2. What's the minimum depth of a leaf in a comparison sort's decision tree? (CLRS Exercise 8.1-1)
3. Compute ⌈log₂ 6!⌉ by hand.

**Intermediate**
4. Prove log₂(n!) = Θ(n log n) without Stirling (CLRS Exercise 8.1-2).
5. Show there's no comparison sort whose running time is linear for even half of the n! inputs (CLRS Exercise 8.1-3).
6. Show Ω(n log k) for sorting n/k groups of k elements each, where the groups are already in order relative to each other (CLRS Exercise 8.1-4).

**Advanced**
7. Prove the average-case Ω(n log n) bound (the average leaf depth is ≥ log₂ N).
8. Show that merging two sorted lists of size n needs 2n − 1 comparisons in the worst case (CLRS Problem 8-6).

**Interview questions**

<details><summary>Q1. Why is O(n log n) the best possible for comparison sorting?</summary>

There are n! possible input orderings, and the algorithm must distinguish all of them. Each comparison has 2 outcomes, so h comparisons can distinguish at most 2ʰ cases. We need 2ʰ ≥ n!, so h ≥ log₂ n! = Θ(n log n).
</details>

<details><summary>Q2. How does counting sort get around the lower bound?</summary>

It isn't a comparison sort. It uses each key as an array index to count occurrences, getting information about the absolute value of a key in O(1), more than one bit per operation. This needs integer keys in a small range [0, k], and runs in Θ(n + k).
</details>

<details><summary>Q3. Does the bound mean every input takes n log n comparisons?</summary>

No. It's a worst-case (and average-case) bound. Some inputs are easy: insertion sort uses n − 1 comparisons on sorted input. But no algorithm can be fast on most inputs.
</details>

**Worked problem: lower bound for finding duplicates (element distinctness).** In the comparison model, deciding whether n numbers are all distinct requires Ω(n log n) comparisons (a classic result using algebraic decision trees). Sorting and checking neighbours matches it. Hash sets get Θ(n) expected only because they're **not** comparison-based: hashing uses the value directly.

**Coding problems:** no direct problems. This bound tells you which approaches can work (any comparison-only solution to LeetCode 912 must be Ω(n log n); use counting sort when values are bounded). *(verify problem constraints for value ranges)*

---

## K. Revision notes

**Five key takeaways**
1. A comparison sort is a decision tree with ≥ n! leaves.
2. Height ≥ log₂ n! = Ω(n log n), so the worst case (and even the average case) needs Ω(n log n) comparisons.
3. Merge sort and heapsort are therefore asymptotically optimal.
4. The adversary argument: each answer keeps at least half of the consistent orderings.
5. Non-comparison sorts (counting, radix, bucket) escape the bound by using the key's value.

**Formulas:** log₂ n! ≈ n log₂ n − 1.443n. A tree of height h has ≤ 2ʰ leaves. n! ≥ (n/2)^(n/2).

**Common mistakes:** applying the bound to non-comparison sorts, and reading it as an "every input" bound.

**Quiz**
1. ⌈log₂ 5!⌉?
2. Number of leaves in the decision tree for n = 4?
3. Is radix sort subject to the Ω(n log n) bound?
4. Which classic algorithm comes closest to ⌈log₂ n!⌉ comparisons?

**Answers**

<details><summary>Show answers</summary>

1. ⌈log₂ 120⌉ = 7.
2. At least 4! = 24 reachable leaves.
3. No. It isn't a comparison sort.
4. Ford–Johnson merge insertion (it's optimal for n ≤ 11). Merge sort is within about 0.44n.
</details>

**Related topics:** [Counting sort](counting-sort.md) · [Radix sort](radix-sort.md) · [Bucket sort](bucket-sort.md) · [Merge sort](../comparison-sorts/merge-sort.md) · [Heapsort](../heapsort/heapsort.md) · [Asymptotic notation](../../00-Foundations/asymptotic-notation.md)
