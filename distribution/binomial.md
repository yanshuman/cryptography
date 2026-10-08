# Binomial Distribution

![Binomial distribution with n = 10 and p = 0.7. Bars for 8, 9 and 10 successes are shaded and add up to 38.28%](images/binomial.png)

**[▶ Try it in the Distribution Explorer](https://yanshuman.github.io/cryptography/distribution/#binomial)** · [All distributions](README.md)

The Binomial distribution answers: **"If I try n times, how many successes will I get?"** It continues the [Bernoulli](bernoulli.md) example: the same 70% free-throw shooter now takes **10 shots**.

| | |
|---|---|
| **Type** | Discrete (0, 1, 2, …, n) |
| **Parameters** | n = number of trials, p = probability of success on each |
| **Formula** | P(X = k) = C(n, k) · pᵏ · (1 − p)ⁿ⁻ᵏ |
| **Mean / Variance** | np / np(1 − p) |
| **Running example** | 10 free throws, p = 0.7 |

## Step 1: The picture in real life

The shooter takes 10 free throws. Each shot goes in with probability 0.7, and one shot doesn't affect the next. Let

> **X = number of shots that go in** (anything from 0 to 10)

You'd expect about 7, but sometimes 6 or 8, rarely 4, and almost never 0. The Binomial distribution gives the exact chance of every count.

**The four conditions** (remember them as "BINS"):
- **B**inary: each trial is success or failure
- **I**ndependent: one shot doesn't change the next
- **N**umber of trials is fixed (n = 10)
- **S**ame probability every time (p = 0.7)

## Step 2: The chance of one particular sequence

What's the chance she makes **exactly 7**? Start with one specific order, such as making the first 7 and missing the last 3:

> ✓ ✓ ✓ ✓ ✓ ✓ ✓ ✗ ✗ ✗

Because the shots are independent, multiply the chances:

> 0.7 × 0.7 × 0.7 × 0.7 × 0.7 × 0.7 × 0.7 × 0.3 × 0.3 × 0.3 = 0.7⁷ × 0.3³

> 0.7⁷ = 0.0823543 and 0.3³ = 0.027, so 0.0823543 × 0.027 = **0.0022236**

## Step 3: How many orders give 7 makes?

The 7 makes could happen in many different orders, for example ✗ ✓ ✓ ✗ ✓ ✓ ✓ ✓ ✗ ✓. Each order has the **same** probability, 0.0022236, because it still has seven 0.7s and three 0.3s.

So we need to count the orders. That means choosing which 7 of the 10 shots are the makes, which is the **combination** "10 choose 7":

> C(10, 7) = 10! / (7! × 3!) = (10 × 9 × 8) / (3 × 2 × 1) = 720 / 6 = **120**

**Where does that come from?** There are 10 × 9 × 8 ways to pick which 3 shots are the misses in order. But the order of picking doesn't matter (missing shots 2, 5, 9 is the same as 9, 2, 5), and each group of 3 can be listed in 3 × 2 × 1 = 6 orders, so we divide by 6.

## Step 4: Put it together

> P(X = 7) = (number of orders) × (chance of each order) = 120 × 0.0022236 = **0.2668**

That's the general formula:

> **P(X = k) = C(n, k) · pᵏ · (1 − p)ⁿ⁻ᵏ**

| Piece | Meaning | For k = 7 |
|---|---|---|
| C(n, k) | how many orders | 120 |
| pᵏ | the k successes | 0.7⁷ |
| (1 − p)ⁿ⁻ᵏ | the n − k failures | 0.3³ |

Notice it's the [Bernoulli](bernoulli.md) formula pˣ(1 − p)¹⁻ˣ repeated n times, times a count of orders.

## Step 5: The whole distribution

Doing this for every k from 0 to 10:

| Makes k | 0 | 1 | 2 | 3 | 4 | 5 | 6 | **7** | 8 | 9 | 10 |
|---|---|---|---|---|---|---|---|---|---|---|---|
| C(10, k) | 1 | 10 | 45 | 120 | 210 | 252 | 210 | **120** | 45 | 10 | 1 |
| P(X = k) | 0.0000 | 0.0001 | 0.0014 | 0.0090 | 0.0368 | 0.1029 | 0.2001 | **0.2668** | 0.2335 | 0.1211 | 0.0282 |

The bars add up to 1, and the tallest is at 7. This is the chart at the top of the page.

## Step 6: Probabilities of ranges

To get the chance of a range, **add the bars**:

> P(8 or more) = P(8) + P(9) + P(10) = 0.2335 + 0.1211 + 0.0282 = **0.3828 (38.28%)**

> P(5 or fewer) = P(0) + … + P(5) = **0.1503 (15.03%)**

So she makes at least 8 out of 10 a bit more than a third of the time, and 5 or fewer about once in 7 tries.

## Step 7: Mean and standard deviation, without the table

X is the sum of 10 Bernoulli variables (one per shot), and means and variances of independent pieces simply add:

> **Mean** = n × p = 10 × 0.7 = **7**
> **Variance** = n × p(1 − p) = 10 × 0.21 = **2.1**
> **Standard deviation** = √2.1 ≈ **1.45**

So a typical result is 7 ± 1.45 makes, roughly 6 to 8.

## Step 8: What n and p do to the shape

- **p = 0.5:** perfectly symmetric (like flipping 10 coins).
- **p > 0.5:** leans right, piled up near n. **p < 0.5:** leans left, piled up near 0.
- **Large n:** the bars form a smooth **bell**. When np ≥ 10 and n(1 − p) ≥ 10, you can approximate the Binomial with a [Normal](normal.md) with μ = np and σ = √(np(1 − p)). That's the Central Limit Theorem at work.
- **Large n, tiny p:** the shape becomes a [Poisson](poisson.md) with λ = np.

Try n = 60, p = 0.5 in the Explorer to see the bell appear.

## Step 9: Summary

| Question | Answer |
|---|---|
| What is it? | Number of successes in n independent tries |
| Parameters | n (trials), p (success chance) |
| Formula | C(n, k) pᵏ (1 − p)ⁿ⁻ᵏ |
| Mean | np = 7 |
| Variance | np(1 − p) = 2.1 |
| Example result | P(exactly 7) = 26.68%, P(8 or more) = 38.28% |

**Where it's used:** exam guessing (how many multiple-choice questions you'd get right by luck), quality control (defective items in a batch of 50), medicine (patients who respond out of 20), marketing (how many of 1000 emails get opened), genetics (offspring with a trait), and election polling.

**One-line answer for exams:**
> "The Binomial distribution gives the probability of k successes in n independent trials, each with success probability p: P(X = k) = C(n, k)pᵏ(1 − p)ⁿ⁻ᵏ. Its mean is np and variance np(1 − p)."
