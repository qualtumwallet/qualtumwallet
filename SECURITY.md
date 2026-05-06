# SECURITY.md

# 🔐 QUALTUM Security Overview

QUALTUM is a post-quantum cryptographic protocol designed to secure blockchain assets against quantum-scale adversaries. This document outlines the security model, threat assumptions, core cryptographic constructions, and the guarantees the protocol provides.

---



## Philosophy

QUALTUM is built from first principles around a single question:

> **Who has the authority to move funds?**

Traditional blockchain security ties authority to a private key  a secret that can be mathematically derived by a sufficiently powerful adversary (e.g., via Shor's Algorithm on a quantum computer). QUALTUM rejects this assumption entirely.

Security in QUALTUM is governed by three pillars:

| Pillar | Description |
|---|---|
| **Non-Derivability** | Authority must not depend on secrets that can be mathematically derived by any computational model. |
| **Zero Exposure** | Public exposure of on-chain account data must not weaken the security of stored funds in any way. |
| **Future-Proofing** | The system must remain secure under future computational paradigms, including large-scale quantum computing. |

---

## Threat Model

### Adversary Capabilities

QUALTUM is designed to remain secure even against a maximally capable adversary with:

- **Full ledger access**    complete read access to all on-chain history and transaction data.
- **Unbounded storage**    ability to harvest and store ciphertext today for later decryption ("Harvest Now, Decrypt Later").
- **Quantum computing access**  access to high-qubit quantum hardware capable of running Shor's or Grover's algorithm against standard cryptographic constructions.

### What QUALTUM Does NOT Protect Against

- Compromise of the user's local environment at the time of secret generation.
- Loss or destruction of the secret `S` by the user (no recovery mechanism exists by design).
- Implementation-level bugs outside the core protocol specification.

### QUALTUM's Mitigations

| Attack Vector | Mitigation |
|---|---|
| Quantum key derivation (Shor's Algorithm) | No private key exists to derive. |
| Public key exposure from signatures | No signatures or public keys are used or published. |
| Signature nonce reuse / leakage | Signature scheme is eliminated entirely from the protocol. |
| Harvest Now, Decrypt Later | Only a hash commitment `H2` is stored on-chain; it yields no exploitable cryptographic material. |

---

## Core Security Primitives

QUALTUM's security rests on exactly **two constructions**:

### 1. Program Derived Addresses (PDAs)

Funds are held in deterministic, program-controlled accounts. A PDA has no associated private key  it is controlled exclusively by the on-chain program logic. This removes the concept of key custody from the security model entirely.

### 2. Hash-Based Commitments (Crystals Dilithium)

QUALTUM uses a post-quantum hash commitment scheme. The security guarantee reduces entirely to the **preimage resistance** of the underlying hash function  a well-understood, quantum-resistant primitive (Grover's algorithm provides at most a quadratic speedup, which is mitigated by sufficient output length).

---

## Protocol Construction

### The Commitment Chain

The user generates a secret `S` locally and computes a two-step hash chain:

```
H1 = hash(S)
H2 = hash(H1)
```

> ⚠️ Only `H2` is ever stored on-chain. The secret `S` and intermediate value `H1` remain exclusively in the user's possession until withdrawal.

### Lifecycle

#### Initialization

1. User generates secret `S` locally (never transmitted).
2. User computes `H1 = hash(S)` and `H2 = hash(H1)`.
3. `H2` is stored in the on-chain vault as the commitment.

#### Deposit

4. Funds are transferred to the Program Derived Address (PDA) associated with the vault.

#### Withdrawal

5. User submits `H1` to the on-chain program.
6. The program computes `hash(H1)` on-chain.
7. If `hash(H1) == H2`, the program releases funds to the user.
8. `H1` is now publicly revealed  the vault is considered spent and must not be reused.

```
[User] ──── H1 ────▶ [On-Chain Program]
                           │
                     hash(H1) == H2?
                           │
                    ┌──────┴──────┐
                   YES            NO
                    │              │
              Release Funds    Reject Tx
```

> ⚠️ **Critical Usage Note:** Each vault commitment is **single-use**. After a successful withdrawal, `H1` is public. The vault must be considered exhausted. Reusing a commitment after withdrawal eliminates all security guarantees.

---

## Security Properties

### Preimage Resistance

An attacker observing `H2` on-chain cannot compute `H1` or `S`. An attacker observing `H1` after withdrawal cannot compute `S`. Both directions are protected by the preimage resistance of the hash function.

### Second Preimage Resistance

An attacker cannot find an alternative `H1'` (where `H1' ≠ H1`) such that `hash(H1') == H2`. This prevents forgery of valid withdrawal proofs.

### No Cryptographic Material Leakage

Because there is no key pair, no signature, and no public key in the system, the on-chain ledger contains zero exploitable cryptographic material. An adversary  quantum or classical  observing the ledger gains nothing actionable prior to a withdrawal event.

### Quantum Resistance

The only attack surface available to a quantum adversary is Grover's algorithm applied to hash preimage search, which provides at most a quadratic speedup. Using a hash function with sufficient output length (e.g., SHA-256 or stronger) reduces this to an acceptable security margin.

---

## Summary of Security Guarantees

| Property | Status |
|---|---|
| Post-quantum secure | ✅ Yes |
| No private key required | ✅ Yes |
| No signature vulnerabilities | ✅ Yes |
| Resistant to ledger surveillance | ✅ Yes |
| Single-use commitment enforcement | ⚠️ Required by user — not enforced on re-deposit |
| Secret loss recovery | ❌ None  secret loss is permanent |

---

## Reporting Vulnerabilities

If you discover a security vulnerability in QUALTUM's protocol design or implementation, please **do not open a public issue**.

Report vulnerabilities responsibly via:

- **PGP Key:** *(attach or link your public PGP key here)*

Please include:
- A clear description of the vulnerability.
- Steps to reproduce or a proof-of-concept.
- Potential impact assessment.

We are committed to acknowledging reports within **48 hours** and resolving critical issues within **90 days** of disclosure.

---

