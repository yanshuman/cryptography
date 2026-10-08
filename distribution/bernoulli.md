# Bernoulli Distribution

![Bernoulli distribution with p = 0.7: a bar of 0.3 at 0 (miss) and a bar of 0.7 at 1 (make)](images/bernoulli.png)

**[▶ Try it in the Distribution Explorer](https://yanshuman.github.io/cryptography/distribution/#bernoulli)** · [All distributions](README.md)

The simplest distribution of all: **one try, two possible outcomes**. Every other "counting" distribution in this folder is built from it. The running example is **one free throw in basketball**.

| | |
|---|---|
| **Type** | Discrete (only 0 or 1) |
| **Parameter** | p = probability of success |
| **Formula** | P(X = x) = pˣ (1 − p)¹⁻ˣ, x ∈ {0, 1} |
| **Mean / Variance** | p / p(1 − p) |
| **Running example** | A player who makes 70% of free throws takes one shot (p = 0.7) |

## Step 1: The picture in real life

A basketball player has made 70 of her last 100 free throws. She steps up to take **one** more shot. There are only two possibilities:

- the ball goes in (**success**), or
- it misses (**failure**).

Any situation with exactly **one try and two outcomes** is a **Bernoulli trial**, named after the Swiss mathematician Jacob Bernoulli (1655–1705). Other examples: one coin flip, one email (opened or not), one patient (cured or not), one product off the line (defective or fine).

## Step 2: Turn the outcome into a number

Maths needs numbers, so we write

> **X = 1** if the shot goes in, **X = 0** if it misses.

This 0/1 coding is the whole trick. It makes the maths in the next steps very simple.

## Step 3: The one parameter, p

From her record, the chance of success is

> p = 70 / 100 = **0.7**

The chance of failure must be whatever is left, because the two chances have to add up to 1 (something always happens):

> 1 − p = 1 − 0.7 = **0.3**

So the whole distribution is just two bars:

| Outcome | X | Probability |
|---|---|---|
| Miss | 0 | **0.3** |
| Make | 1 | **0.7** |

## Step 4: One formula for both bars

It's handy to write both cases as **one** formula:

> **P(X = x) = pˣ · (1 − p)¹⁻ˣ**

Check that it works:

- **x = 1:** p¹ · (1 − p)⁰ = p · 1 = **0.7** ✓ (anything to the power 0 is 1)
- **x = 0:** p⁰ · (1 − p)¹ = 1 · 0.3 = **0.3** ✓

The exponents act like switches: when x = 1 the p part is "on", and when x = 0 the (1 − p) part is "on". This same pattern appears again inside the [Binomial](binomial.md) formula.

## Step 5: The mean (expected value)

The **mean** is the average result if she took this shot many, many times. Multiply each value by its probability and add:

> E[X] = 0 × 0.3 + 1 × 0.7 = **0.7**

So the mean of a Bernoulli variable is just **p**. That makes sense: over 100 shots, the average of all the 0s and 1s is the fraction that went in, which is 0.7.

## Step 6: The variance and standard deviation

Variance = average of (value − mean)²:

| X | X − 0.7 | (X − 0.7)² | Probability | Product |
|---|---|---|---|---|
| 0 | −0.7 | 0.49 | 0.3 | 0.147 |
| 1 | +0.3 | 0.09 | 0.7 | 0.063 |
| | | | **Sum** | **0.21** |

> Variance = 0.147 + 0.063 = **0.21**, and this always equals **p(1 − p)** = 0.7 × 0.3 ✓
> Standard deviation σ = √0.21 ≈ **0.458**

**What does p(1 − p) tell us?** It's largest when p = 0.5 (0.25), so a 50/50 shot is the most unpredictable. It's 0 when p = 0 or p = 1, because then the outcome is certain. Try moving p in the Explorer and watch the variance.

| p | 0.1 | 0.3 | 0.5 | 0.7 | 0.9 |
|---|---|---|---|---|---|
| p(1 − p) | 0.09 | 0.21 | **0.25** | 0.21 | 0.09 |

## Step 7: Why it matters: it's the building block

On its own, one shot isn't very interesting. But repeat Bernoulli trials and you get the other counting distributions:

| Question about repeated shots | Distribution |
|---|---|
| How many go in out of 10? | [Binomial](binomial.md) (a sum of 10 Bernoullis) |
| How many shots until the first one goes in? | [Geometric](geometric.md) |
| Very many tries, very small p, counting the rare successes | [Poisson](poisson.md) |

## Step 8: Summary

| Question | Answer |
|---|---|
| What is it? | One trial with two outcomes, coded 1 (success) and 0 (failure) |
| Parameter | p = P(success) |
| Formula | P(X = x) = pˣ(1 − p)¹⁻ˣ |
| Mean | p = 0.7 |
| Variance | p(1 − p) = 0.21 |
| Most uncertain when | p = 0.5 |

**Where it's used:** coin flips, A/B tests (did the visitor click?), medical trials (did the treatment work?), quality control (is the item defective?), spam filters (spam or not), and logistic regression in machine learning, which predicts p for each case.

**One-line answer for exams:**
> "A Bernoulli distribution models a single trial with two outcomes, success (1) with probability p and failure (0) with probability 1 − p. Its mean is p and its variance is p(1 − p)."
