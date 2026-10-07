# Elliptic Curve Cryptography, Step by Step

A slow, beginner-friendly walkthrough on one tiny curve. Every number is worked out by hand, and every formula shows where it comes from. For the full guide, see the [Elliptic Curve README](README.md). To try each step yourself, open the [Elliptic Curve Lab](https://yanshuman.github.io/cryptography/elliptic-curves/elliptic-curve/).

## Contents

1. [Start with an equation](#step-1-start-with-an-equation)
2. [Use "mod" so we only get whole numbers](#step-2-use-mod-so-we-only-get-whole-numbers)
3. [The special property: "adding" two points](#step-3-the-special-property-adding-two-points)
4. [Where the formulas come from](#step-4-where-the-formulas-come-from)
5. [How to "divide" when using mod](#step-5-how-to-divide-when-using-mod)
6. [Calculate 2G = G + G](#step-6-calculate-2g--g--g)
7. [Calculate 3G = 2G + G](#step-7-calculate-3g--2g--g)
8. [Keep going](#step-8-keep-going)
9. [Making the keys](#step-9-making-the-keys)
10. [Why this is safe](#step-10-why-this-is-safe)
11. [Using it: two people make a shared secret](#step-11-using-it-two-people-make-a-shared-secret)
12. [Summary table](#the-whole-thing-in-one-table)
13. [Try it in the lab](#try-it-in-the-lab)

---

## Step 1: Start with an equation

Elliptic curve cryptography begins with an equation of this form:

> **y² = x³ + ax + b**

Here a and b are just fixed numbers. For our example we take **a = 2** and **b = 2**:

> **y² = x³ + 2x + 2**

**Where does this equation come from?** Mathematicians studied it long before computers existed. It has one special property, explained in Step 3, that makes cryptography possible. You don't invent your own curve. In real systems the curve is fixed and published, so everyone in the world uses the same one. Bitcoin, for example, uses the published curve y² = x³ + 7.

## Step 2: Use "mod" so we only get whole numbers

If we draw the equation normally, the points have decimals like 1.732..., and computers can't store those exactly. So we do all the math **mod a prime number**. We'll use **p = 17**.

"Mod 17" means you keep only the remainder after dividing by 17. For example, 20 mod 17 = 3, and 35 mod 17 = 1.

So our curve is now:

> **y² = x³ + 2x + 2 (mod 17)**

A **point** on the curve is any pair of whole numbers (x, y), each between 0 and 16, that makes both sides equal.

**Let's check that (5, 1) is a point:**
- Left side: y² = 1² = **1**
- Right side: 5³ + 2×5 + 2 = 125 + 10 + 2 = 137, and 137 mod 17 = 137 − 136 = **1**
- Both sides are 1, so (5, 1) is on the curve ✓

We'll call this starting point **G = (5, 1)**. G is just a point that was chosen once and published. It was found by trying values of x until the right side came out as a perfect square.

## Step 3: The special property: "adding" two points

Here's the special property: **if you draw a straight line through two points on the curve, it always hits the curve at exactly one more point.**

**Where does that come from?** A line has the equation y = λx + c, where λ is the slope. If you substitute it into the curve equation, you get an equation with x³ in it, called a **cubic equation**. A cubic equation has **3 answers** for x. We already know two of them (our two points), so the third one is forced to exist.

Using this, we define **"point addition"**:

1. Draw a line through P and Q.
2. Find the third point where it hits the curve.
3. Flip that point upside down (change y to −y).
4. The result is called **P + Q**.

**Why flip?** Without the flip, the "addition" wouldn't follow normal addition rules. The flip is what makes it behave like real addition.

## Step 4: Where the formulas come from

We need formulas so a computer can calculate this without drawing anything.

**Formula 1, the slope λ.** This is the same slope formula from school:

> λ = (y₂ − y₁) ÷ (x₂ − x₁)

**Formula 2, the new x.** When you substitute the line into the curve, you get:

> x³ − λ²x² + (other terms) = 0

There's a rule from school algebra: in a cubic x³ − Sx² + ..., **the three answers always add up to S**. Here S = λ², so:

> x₁ + x₂ + x₃ = λ², which gives **x₃ = λ² − x₁ − x₂**

**Formula 3, the new y.** The third point is on the line, so its y is λ(x₃ − x₁) + y₁. Then we flip it (multiply by −1):

> **y₃ = λ(x₁ − x₃) − y₁**

**Special case: adding a point to itself (P + P).** You can't draw a line through just one point, so you use the **tangent line**, the line that just touches the curve at that point. Its slope comes from calculus (differentiating y² = x³ + ax + b):

> **λ = (3x² + a) ÷ (2y)**

You don't need to memorize the calculus. Just remember: **two different points use the normal slope, and the same point twice uses the tangent slope.**

## Step 5: How to "divide" when using mod

The slope formula has a division, but in mod math we can't use fractions. Instead:

> **"÷ 2" means "multiply by the number that turns 2 into 1."**

Mod 17: 2 × 9 = 18, and 18 mod 17 = **1**. So in mod 17, **dividing by 2 is the same as multiplying by 9**. This number is called the **inverse** of 2 (mod 17). For big numbers, computers find inverses quickly with the Extended Euclidean Algorithm.

## Step 6: Calculate 2G = G + G

G = (5, 1), and a = 2. It's the same point twice, so we use the tangent slope.

**Slope:**
- Top: 3x² + a = 3 × 25 + 2 = **77**, and 77 mod 17 = 77 − 68 = **9**
- Bottom: 2y = 2 × 1 = **2**
- λ = 9 ÷ 2 = 9 × 9 (from Step 5) = 81, and 81 mod 17 = 81 − 68 = **13**

**New x:**
- x₃ = λ² − x₁ − x₂ = 169 − 5 − 5 = 159
- 159 mod 17 = 159 − 153 = **6**

**New y:**
- y₃ = λ(x₁ − x₃) − y₁ = 13 × (5 − 6) − 1 = −13 − 1 = −14
- −14 mod 17 = −14 + 17 = **3**

**So 2G = (6, 3).**

**Check it's on the curve:** 3² = 9. And 6³ + 2×6 + 2 = 230, with 230 mod 17 = 230 − 221 = 9 ✓

## Step 7: Calculate 3G = 2G + G

Now we add two different points, (6, 3) and (5, 1), so we use the normal slope.

**Slope:**
- λ = (1 − 3) ÷ (5 − 6) = (−2) ÷ (−1) = **2**

**New x:**
- x₃ = 2² − 6 − 5 = 4 − 11 = −7
- −7 mod 17 = **10**

**New y:**
- y₃ = 2 × (6 − 10) − 3 = −8 − 3 = −11
- −11 mod 17 = **6**

**So 3G = (10, 6).**

**Check:** 6² = 36, and 36 mod 17 = 2. Also 10³ + 20 + 2 = 1022, with 1022 mod 17 = 1022 − 1020 = 2 ✓

## Step 8: Keep going

If you keep adding G the same way, you get a chain:

> G = (5, 1) → 2G = (6, 3) → 3G = (10, 6) → 4G = (3, 1) → 5G = (9, 16) → ... → 9G = (7, 6) → ... → 18G = (5, 16) → 19G = ?

The full chain:

| k | k·G | k | k·G | k | k·G |
|---|---|---|---|---|---|
| 1 | (5, 1) | 8 | (13, 7) | 15 | (3, 16) |
| 2 | (6, 3) | 9 | (7, 6) | 16 | (10, 11) |
| 3 | (10, 6) | 10 | (7, 11) | 17 | (6, 14) |
| 4 | (3, 1) | 11 | (13, 10) | 18 | (5, 16) |
| 5 | (9, 16) | 12 | (0, 11) | **19** | **O** |
| 6 | (16, 13) | 13 | (16, 4) | 20 | (5, 1) again |
| 7 | (0, 6) | 14 | (9, 1) | | |

Look at **18G = (5, 16)**. Since −1 mod 17 = 16, this is G flipped upside down. When you add a point and its flipped twin, the line between them is **vertical**, and a vertical line never hits the curve a third time. Mathematicians call this result the **"point at infinity"**, written **O**. It acts like zero.

So **19G = O**, and then 20G = G again. The chain is a **circle of 19 steps**.

## Step 9: Making the keys

1. **Private key:** pick a secret random number. Say **k = 9**.
2. **Public key:** compute 9G by adding G to itself 9 times. That gives **(7, 6)**.

You publish (7, 6) and keep 9 secret.

## Step 10: Why this is safe

- **Going forward is easy:** if you know 9 and G, you compute (7, 6) quickly.
- **Going backward is hard:** if someone gives you (7, 6), how many times was G added? The points jump around with no pattern, so the only way is to try 1G, 2G, 3G, ... until you hit (7, 6).

In our toy example there are only 19 points, so trying them all is easy. But real curves use a prime about **77 digits long**, so the circle has about 10⁷⁷ points. Even the fastest known method would take far longer than the age of the universe. This backward problem is called the **Elliptic-Curve Discrete Logarithm Problem (ECDLP)**, and it's what keeps the private key safe.

## Step 11: Using it: two people make a shared secret

Alice and Bob both know the curve and G.

1. Alice picks a secret **3**. She computes 3G = **(10, 6)** and sends it to Bob.
2. Bob picks a secret **9**. He computes 9G = **(7, 6)** and sends it to Alice.
3. Alice takes Bob's point and adds it to itself 3 times: 3 × 9G = **27G**.
4. Bob takes Alice's point and adds it to itself 9 times: 9 × 3G = **27G**.

Both get the same point, because 3 × 9 = 9 × 3. Since the circle has 19 steps, 27G = (27 − 19)G = **8G = (13, 7)**.

An attacker only saw (10, 6) and (7, 6). To get (13, 7), they'd need to know 3 or 9, and that's the hard problem from Step 10. Alice and Bob then use (13, 7) as a key to encrypt their messages with AES.

This method is called **Elliptic-Curve Diffie–Hellman (ECDH)**. It's how HTTPS, Signal and WhatsApp set up encryption keys.

## The whole thing in one table

| Step | What happens | Where it comes from |
|---|---|---|
| Curve | y² = x³ + 2x + 2 | Fixed and published |
| mod 17 | Whole numbers only | Computers need exact numbers |
| G | (5, 1) | Fixed and published starting point |
| Adding points | Line, third point, flip | A cubic has 3 answers |
| Division | Multiply by the inverse | Mod math has no fractions |
| Private key | Secret number (9) | Picked at random |
| Public key | 9G = (7, 6) | Add G nine times |
| Shared secret | 3 × 9G = 9 × 3G = (13, 7) | Multiplication order doesn't matter |
| Security | Can't find 9 from (7, 6) | Too many points to try |

## Try it in the lab

Open the [Elliptic Curve Lab](https://yanshuman.github.io/cryptography/elliptic-curves/elliptic-curve/) and go to **section 2 (finite field)**:

1. Click **Tiny p = 17**. That loads exactly this curve, y² = x³ + 2x + 2 (mod 17), and you'll see the 18 dots (19 points with O).
2. The lab starts with a different base point, so **click the dot at (5, 1)** to make it G.
3. Slide **k** from 1 to 19 and compare each result with the table in Step 8.
4. In **section 3 (key exchange)**, set Alice's secret to **3** and Bob's to **9**. Both sides arrive at **(13, 7)**, as in Step 11.
5. In **section 4**, press **Try to find Alice's secret** to see why a toy curve can be broken instantly.

**Read next:** [ECC in Real Life: what happens when you open a website](real-life-example.md)
