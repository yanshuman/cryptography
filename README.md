# Cryptography

Learning projects for classic cryptographic algorithms, built so every step can be seen and played with. Each project lives in its own folder with its own README.

## Projects

| Project | What it shows | Guide | Live demo |
|---|---|---|---|
| [RSA](rsa/) | Key generation, encryption, decryption, digital signatures, and how an attacker would try to break it | [README](rsa/README.md) | [Open](https://yanshuman.github.io/cryptography/rsa/) |
| [Elliptic Curves](elliptic-curves/) | Step-by-step path to elliptic-curve cryptography. Part 1: the ellipse, with its geometry, orbits, reflection and an interactive lab | [README](elliptic-curves/README.md) | [Ellipse Lab](https://yanshuman.github.io/cryptography/elliptic-curves/ellipse/) |

| RSA | Ellipse |
|---|---|
| [![RSA architecture](rsa/images/rsa-architecture-light.png)](rsa/README.md) | [![Anatomy of an ellipse](elliptic-curves/ellipse/images/ellipse-anatomy-light.png)](elliptic-curves/ellipse/README.md) |

## How to run

Every project is plain HTML, CSS and JavaScript, with no installation or build step. Open the project's `index.html` in any browser, or use its live demo link above.

## Folder structure

```
cryptography/
├── README.md            # This index
├── rsa/                 # RSA: interactive lab, architecture diagram, guide
│   └── README.md
└── elliptic-curves/     # Elliptic curves section
    ├── README.md
    └── ellipse/         # Ellipse guide + Ellipse Lab
        └── README.md
```

## Questions

Open an issue on GitHub: https://github.com/yanshuman/cryptography/issues

## Note

These projects are for learning. For real-world security, use a well-tested library such as OpenSSL, Python's `cryptography`, or Go's standard crypto packages.
