# Elliptic Curves

Elliptic-curve cryptography (ECC) protects Bitcoin wallets, HTTPS connections, Signal messages and SSH logins. This section builds up to it one step at a time, starting from the shape the name comes from: the ellipse.

| # | Topic | What it covers | Guide | Live demo |
|---|---|---|---|---|
| 1 | [Ellipse](ellipse/) | Definition, equations, area, perimeter, reflection, Kepler orbits, real-world uses, and how the ellipse led to elliptic curves | [README](ellipse/README.md) | [Open](https://yanshuman.github.io/cryptography/elliptic-curves/ellipse/) |
| 2 | [Elliptic Curve](elliptic-curve/) | What an elliptic curve is, adding points, finite fields, ECDH key exchange, ECDSA, why ECC is used, and an attacker demo | [README](elliptic-curve/README.md) · [Step by step](elliptic-curve/step-by-step.md) · [Real-life example](elliptic-curve/real-life-example.md) | [Open](https://yanshuman.github.io/cryptography/elliptic-curves/elliptic-curve/) |

| Ellipse | Elliptic curve |
|---|---|
| [![Anatomy of an ellipse](ellipse/images/ellipse-anatomy-light.png)](ellipse/README.md) | [![Adding points on an elliptic curve](elliptic-curve/images/point-addition.png)](elliptic-curve/README.md) |

## The road to ECC

1. **Ellipse:** a closed curve where r₁ + r₂ = 2a. Its perimeter leads to *elliptic integrals*.
2. **Elliptic curves over real numbers:** y² = x³ + ax + b, and adding points with the chord-and-tangent rule.
3. **Elliptic curves over finite fields:** the same rule, done with arithmetic modulo a prime p.
4. **ECC:** key exchange (ECDH) and signatures (ECDSA), with security based on the elliptic-curve discrete logarithm problem.

Steps 1–4 are covered by the two projects above: [Ellipse](ellipse/README.md) for step 1, and [Elliptic Curve](elliptic-curve/README.md) for steps 2–4.

## Folder structure

```
elliptic-curves/
├── README.md        # This index
├── ellipse/         # Ellipse guide + Ellipse Lab
│   ├── README.md
│   ├── index.html
│   └── images/
└── elliptic-curve/  # Elliptic curve guide + Elliptic Curve Lab
    ├── README.md
    ├── step-by-step.md
    ├── real-life-example.md
    ├── index.html
    └── images/
```
