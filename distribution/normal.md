# Normal (Gaussian) Distribution

![The normal distribution with mean 70 and standard deviation 5. The shaded area between 65 and 75 holds 68.27% of the values](images/normal.png)

**[▶ Try it in the Distribution Explorer](https://yanshuman.github.io/cryptography/distribution/#normal)** · [All distributions](README.md)

This guide builds the Gaussian distribution slowly, using one example all the way through: **exam marks in a class**. Every number comes from an earlier step.

| | |
|---|---|
| **Type** | Continuous |
| **Parameters** | mean μ (centre), standard deviation σ (width) |
| **Formula** | f(x) = 1 / (σ√(2π)) · e^(−(x − μ)² / (2σ²)) |
| **Mean / Variance** | μ / σ² |
| **Running example** | Exam marks with μ = 70, σ = 5 |

## Step 1: The picture in real life

Imagine 1000 students take an exam and you count how many got each mark:

- Many students score **around the average**, say around 70.
- Fewer students score 60 or 80.
- Very few score 50 or 90.

If you draw this as a graph, it looks like a **bell**: high in the middle, low on both sides, and the same on the left and right. This bell shape is called the **Gaussian distribution**, also known as the **Normal distribution**. It's named after the mathematician Carl Friedrich Gauss.

## Step 2: Why does a bell shape appear?

A student's mark isn't decided by one thing. It's the result of **many small things added together**: how much they studied (+), sleep last night (+ or −), whether the questions matched what they practised (+ or −), mood, luck in guessing, and so on.

Each small thing pushes the mark up or down a little. For most students, some pushes go up and some go down, so they **cancel out**, and the mark lands near the average. For a student to get a very high mark, almost **everything** has to go right at the same time, which is rare. The same is true for a very low mark.

This is why the middle is crowded and the edges are empty. The rule is:

> **When a result is the sum of many small random effects, it forms a bell curve.**

This rule is called the **Central Limit Theorem**, and it's the real reason the Gaussian appears everywhere: heights, weights, measurement errors, and marks.

## Step 3: The centre, called the mean (μ)

To describe a bell curve, we need just **two numbers**. The first tells us **where the centre is**.

Take a small class of 8 students with these marks:

> 63, 65, 65, 69, 71, 75, 75, 77

**Mean** = add them all and divide by how many there are:

> (63 + 65 + 65 + 69 + 71 + 75 + 75 + 77) ÷ 8 = 560 ÷ 8 = **70**

So **μ = 70**. The peak of the bell sits at 70.

## Step 4: The width, called the standard deviation (σ)

The second number tells us **how spread out** the marks are. Two classes can both average 70, but in one class everyone gets 68 to 72, while in the other the marks range from 40 to 100. We need a number for this "spread".

**Step 4a: Find how far each mark is from the mean.**

| Mark | 63 | 65 | 65 | 69 | 71 | 75 | 75 | 77 |
|---|---|---|---|---|---|---|---|---|
| Mark − 70 | −7 | −5 | −5 | −1 | +1 | +5 | +5 | +7 |

**Step 4b: Square each distance.** If we just added the distances, the minus and plus values would cancel to 0. Squaring makes them all positive.

| Distance² | 49 | 25 | 25 | 1 | 1 | 25 | 25 | 49 |
|---|---|---|---|---|---|---|---|---|

**Step 4c: Take the average of the squares.** This is called the **variance**.

> (49 + 25 + 25 + 1 + 1 + 25 + 25 + 49) ÷ 8 = 200 ÷ 8 = **25**

**Step 4d: Take the square root.** This undoes the squaring, so we're back in "marks" units.

> σ = √25 = **5**

So **σ = 5**. On average, a student is about 5 marks away from the mean.

(Note: when working with a sample instead of a whole group, statistics books divide by n − 1 instead of n. The idea is the same.)

## Step 5: What μ and σ do to the shape

- **Changing μ slides the bell left or right.** If μ = 80, the peak moves to 80. The shape stays the same.
- **Changing σ makes the bell wide or narrow.** A small σ (like 2) gives a tall, thin bell, because everyone is close to the average. A large σ (like 15) gives a short, wide bell, because the marks are very spread out.

The total area under the bell is always **1** (100% of students), so when it gets wider, it must also get shorter. Try both sliders in the [Explorer](https://yanshuman.github.io/cryptography/distribution/#normal).

## Step 6: Building the formula, one piece at a time

Here's the full formula:

> **f(x) = (1 / (σ√(2π))) × e^(−(x − μ)² / (2σ²))**

Instead of memorising it, let's **build it** in 4 small steps, using μ = 70 and σ = 5.

**Piece 1: How far is x from the centre?**

> (x − μ)

For x = 75, that's 75 − 70 = 5.

**Piece 2: Measure that distance in "σ units".**

> (x − μ) / σ

For x = 75, that's 5 / 5 = **1**, so 75 is "1 step" from the centre. This number is called **z**, and we'll use it again in Step 8.

**Piece 3: Make it a bell.**

> e^(−z² / 2)

Here e ≈ 2.718 is a special number in maths. Let's see what this piece gives:

| Mark x | 55 | 60 | 65 | **70** | 75 | 80 | 85 |
|---|---|---|---|---|---|---|---|
| z | −3 | −2 | −1 | **0** | 1 | 2 | 3 |
| e^(−z²/2) | 0.011 | 0.135 | 0.607 | **1** | 0.607 | 0.135 | 0.011 |

Look at the bottom row: it's **highest (1) at the centre**, then drops on both sides **equally**, and becomes almost 0 far away. That's already a bell.

- **Why z²?** Squaring makes −1 and +1 give the same value, so the left and right sides match (symmetry).
- **Why the minus sign?** It makes the value get **smaller** as you move away from the centre. Without it, the curve would grow instead of falling.
- **Why e?** It makes the curve smooth and fall off quickly. Mathematicians showed that this exact form is what you get when many small effects add up (Step 2).

**Piece 4: Fix the height so the total area is 1.**

The total area under Piece 3 is not exactly 1. With calculus, you can show the area is σ√(2π). So we divide by that:

> 1 / (σ√(2π)) = 1 / (5 × 2.5066) = **0.0798**

The **√(2π)** isn't chosen by anyone. It comes out of the calculus when you measure the area under e^(−z²/2).

**Putting it together:**

> f(x) = 0.0798 × e^(−z²/2)

| Mark x | 55 | 60 | 65 | **70** | 75 | 80 | 85 |
|---|---|---|---|---|---|---|---|
| f(x) | 0.0009 | 0.0108 | 0.0484 | **0.0798** | 0.0484 | 0.0108 | 0.0009 |

That is exactly the curve in the picture at the top of this page.

An important point: **the height f(x) is not the probability itself.** For a continuous distribution, probability is the **area under the curve** between two values. For example, the chance of a mark between 70 and 71 is roughly the height (about 0.08) × width (1), so about 8% (the exact area is 7.93%). The chance of exactly 70.000... is 0, because a single point has no width.

## Step 7: The 68–95–99.7 rule

If you measure the areas under the curve, you always get the same results, for **any** bell curve:

| Range | Area (share of students) | In our class (μ = 70, σ = 5) |
|---|---|---|
| μ ± 1σ | **68%** | 65 to 75 |
| μ ± 2σ | **95%** | 60 to 80 |
| μ ± 3σ | **99.7%** | 55 to 85 |

**Where does 68% come from?** It's the area under the curve from z = −1 to z = +1, computed once by mathematicians using calculus (it's 68.27%). Since every Gaussian is just the same shape slid by μ and stretched by σ, the answer is the same for all of them.

So in a class of 1000 students, about 680 score between 65 and 75, about 950 score between 60 and 80, and only about 3 are outside 55 to 85.

## Step 8: The z-score, comparing any value

> **z = (x − μ) / σ**

This is Piece 2 from Step 6. It tells you **how many σ steps** a value is from the average.

**Example: a student scored 80. How good is that?**
z = (80 − 70) / 5 = **2**, so the score is 2 steps above average.

From the rule: 95% of students are between −2 and +2. The remaining 5% is split equally into 2.5% at the low end and 2.5% at the high end. So scoring above 80 puts you in the **top 2.5%**, which means you did better than about **97.5%** of the class.

**Why z-scores are useful:** they let you compare things measured differently. Suppose you got 80 in Maths (μ = 70, σ = 5) and 85 in Physics (μ = 75, σ = 10):

- Maths: z = (80 − 70) / 5 = **2**
- Physics: z = (85 − 75) / 10 = **1**

Your raw Physics mark is higher, but your Maths result is actually **more impressive**, because you're further above the class.

## Step 9: The standard normal distribution

If you convert every value to its z-score, every Gaussian becomes the same one, with **μ = 0 and σ = 1**. This is called the **standard normal distribution**, and its formula is simpler:

> f(z) = (1 / √(2π)) × e^(−z²/2)

That's why statistics books have just **one "Z-table"**: you convert your problem to z, then look up the area. For example, the table says the area to the left of z = 1 is **0.8413**, so 84% of students scored below 75 in our class.

## Step 10: Summary

| Question | Answer |
|---|---|
| What is it? | A symmetric bell-shaped distribution |
| Why does it appear? | Many small random effects add up (Central Limit Theorem) |
| What describes it? | μ (centre) and σ (width) |
| Highest point | At x = μ |
| Total area | Always 1 |
| Quick rule | 68% within 1σ, 95% within 2σ, 99.7% within 3σ |
| Compare values | z = (x − μ) / σ |

**Where it's used:** exam grading and percentiles, heights and weights, measurement errors in science, noise in electronic signals, quality control in factories (the "Six Sigma" method is named after σ), and machine learning.

**Related distributions:** a [Binomial](binomial.md) with many trials looks normal. Taking eˣ of a normal value gives a [Log-normal](lognormal.md). Squaring and adding normals gives a [Chi-square](chi-square.md). A normal with an *estimated* σ gives a [Student's t](t-distribution.md).

**One-line answer for exams:**
> "The Gaussian distribution is a continuous, symmetric, bell-shaped distribution defined by its mean μ and standard deviation σ. It arises whenever many small independent effects add together (Central Limit Theorem), and about 68%, 95%, and 99.7% of values lie within 1, 2, and 3 standard deviations of the mean."
