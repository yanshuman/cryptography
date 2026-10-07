# ECC in Real Life: What Happens When You Open a Website

When you open a website over HTTPS, send a WhatsApp or Signal message, or make a Bitcoin transaction, elliptic curves are working in the background. This page follows one complete example, **opening https://example.com**, using the tiny curve from the [step-by-step walkthrough](step-by-step.md) so you can check every number. Then it shows what changes at real size.

**In short:**
1. Your device picks a random secret number (the **private key**).
2. It computes secret × G (the **public key**) and sends that.
3. Both sides combine the other's public key with their own secret and reach the **same shared point**.
4. That shared point becomes the **AES key** that encrypts everything.

## Contents

- [The setup](#the-setup-public-the-same-for-everyone)
- [Step 1: Each side picks a random secret](#step-1-each-side-picks-a-random-secret-private-key)
- [Step 2: Each side sends secret × G](#step-2-each-side-computes-secret--g-public-key-and-sends-it)
- [Step 3: Each side computes the shared point](#step-3-each-side-combines-the-others-public-key-with-its-own-secret)
- [Step 4: The shared point becomes the AES key](#step-4-the-shared-point-becomes-the-aes-key)
- [The same thing at real size](#the-same-thing-at-real-size)
- [How this changes in each app](#how-this-changes-in-each-app)
- [The example on one screen](#the-example-on-one-screen)
- [Try it in the lab](#try-it-in-the-lab)

---

## The setup (public, the same for everyone)

Both sides already know:
- the curve: **y² = x³ + 2x + 2 (mod 17)**
- the starting point: **G = (5, 1)**

Real browsers use a standard curve such as **X25519**, which is built into every browser and server.

---

## Step 1: Each side picks a random secret (private key)

| | Your browser | example.com server |
|---|---|---|
| Random secret | **3** | **9** |
| Who knows it | only your browser | only the server |

A fresh secret is picked **every time you connect**, and thrown away afterwards.

---

## Step 2: Each side computes secret × G (public key) and sends it

| | Your browser | Server |
|---|---|---|
| Computes | 3 × G = 3G = **(10, 6)** | 9 × G = 9G = **(7, 6)** |
| Sends | "Hello, my key is **(10, 6)**" → | ← "Hello, my key is **(7, 6)**" |

This exchange is the start of the **TLS handshake**, and it happens in plain sight.

👀 **Eve, who is watching the Wi-Fi, sees:** the curve, G = (5, 1), (10, 6) and (7, 6). She does **not** see 3 or 9.

---

## Step 3: Each side combines the other's public key with its own secret

| | Your browser | Server |
|---|---|---|
| Takes | the server's key (7, 6) = 9G | your key (10, 6) = 3G |
| Multiplies by its own secret | 3 × 9G = **27G** | 9 × 3G = **27G** |
| Result | **(13, 7)** | **(13, 7)** ✓ |

Both get the **same point** without ever sending it, because 3 × 9 = 9 × 3. (The cycle has 19 steps, so 27G = 8G = (13, 7). The full cycle is in [Step 8 of the walkthrough](step-by-step.md#step-8-keep-going).)

**Eve is stuck.** To get (13, 7) she needs 3 or 9. Getting 3 from (10, 6) is the *elliptic-curve discrete logarithm problem*. On this toy curve she could try all 19 values, but on a real curve there are about 10⁷⁷.

---

## Step 4: The shared point becomes the AES key

The browser and server take the **x-coordinate**, 13, and run it through a hash function (a key-derivation function, HKDF) to get a proper key:

```
shared point x = 13
        ↓  HKDF (hashing)
AES key = 7f3a…c91e   (256 bits of random-looking data; illustrative)
```

From now on, everything is encrypted with that AES key:

```
You type:   password = "hunter2"
Sent as:    8f1c09e4b2a7…   ← Eve sees only gibberish (illustrative)
Server decrypts it with the same AES key → "hunter2"
```

**Why switch to AES?** ECC is good at agreeing on a secret. AES is much faster at encrypting large amounts of data, such as web pages, videos and messages. So ECC sets up the key, and AES does the heavy lifting.

---

## The same thing at real size

| | Toy example | Real HTTPS (X25519) |
|---|---|---|
| Prime p | 17 | 2²⁵⁵ − 19 (77 digits) |
| Number of points | 19 | ≈ 2²⁵² ≈ 10⁷⁶ |
| Private key | 3 | a random 32-byte number |
| Public key | (10, 6) | 32 bytes |
| Time to compute | instant | about 0.05 ms |
| Eve trying every key | 19 tries | longer than the age of the universe |

---

## How this changes in each app

**🔒 HTTPS:** exactly the steps above. There's one extra piece: the server also **signs** its public key with its certificate (using ECDSA or a similar scheme). This proves you're talking to the real example.com and not Eve pretending to be it, which stops a "man-in-the-middle" attack.

**💬 WhatsApp / Signal:** the same key exchange, but repeated constantly. Every message gets a new key derived from fresh ECC exchanges (the "double ratchet"). Even if one key leaked, older and newer messages would stay safe.

**₿ Bitcoin works differently.** It uses ECC to **sign**, not to share an AES key, and nothing is encrypted:
- Your private key is a secret number d, and your public key is d × G. Your address is made from the public key.
- To spend coins you **sign** the transaction with d. Everyone checks the signature using your public key.
- Only the person who knows d can make a valid signature, so only you can spend your coins.

So "the shared point becomes the AES key" applies to **HTTPS and messaging**. Bitcoin uses the same curve maths, but for **proving ownership**.

---

## The example on one screen

```
Public:   curve y² = x³ + 2x + 2 (mod 17),  G = (5, 1)

Browser:  secret 3  →  sends 3G = (10, 6)
Server:   secret 9  →  sends 9G = (7, 6)

Browser:  3 × (7, 6)  = 27G = (13, 7)  ┐
Server:   9 × (10, 6) = 27G = (13, 7)  ┘ same, never sent

AES key = HKDF(13)  →  encrypts the whole page
Eve saw (10, 6) and (7, 6), but can't get 3, 9 or (13, 7).
```

---

## Try it in the lab

You can reproduce these exact numbers in the [Elliptic Curve Lab](https://yanshuman.github.io/cryptography/elliptic-curves/elliptic-curve/):

1. In **section 2**, click **Tiny p = 17**, then click the dot at **(5, 1)** to make it G.
2. In **section 3**, set Alice (the browser) = **3** and Bob (the server) = **9**. The shared point is **(13, 7)**.
3. In **section 4**, press **Try to find Alice's secret** to see Eve break the toy key instantly.

**Read next:** [Step-by-step walkthrough](step-by-step.md) · [Full elliptic curve guide](README.md)
