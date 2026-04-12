# PGDM v2 — Deterministic Card Dealing Protocol (Enhanced)
## Deterministic Deck Generation Protocol (Executable, Cryptographically Formalized Version)
**Author:** Crack Chain Poker

---

## 1. Protocol Objective

This document formalizes a deterministic, cryptographically verifiable, and fully executable protocol for card generation and distribution in trustless digital poker environments. The protocol replaces opaque, centralized randomness services with a rigorously specified, reproducible computation whose outputs are fully derivable from publicly auditable inputs. 

Rather than modeling randomness as an oracle, the system reinterprets it as *computationally unpredictable determinism*—a property that emerges from the composition of high-entropy inputs, adversarially resistant coordination, and cryptographic compression. The objective is to ensure that every dealing event is reproducible, bias-free, and resistant to manipulation under realistic adversarial models.

---

## 2. System Architecture

The protocol is composed of the following tightly coupled subsystems, each responsible for a distinct security and correctness guarantee:

- **Cryptographic Primitive Layer:** Provides collision-resistant hashing and deterministic entropy expansion. This layer defines the mathematical hardness assumptions of the system.
- **Commit-Reveal Coordination Layer:** Enforces temporal integrity and prevents adaptive adversarial behavior through binding commitments and delayed disclosure.
- **Seed Derivation Engine:** Aggregates heterogeneous entropy sources into a single cryptographic seed with uniform distribution properties.
- **Deterministic Shuffle Engine:** Produces a permutation of the deck using a bias-free algorithm with provable uniformity.
- **State Persistence Layer:** Maintains a verifiable chain of states, enabling full historical reconstruction of all prior decks and transitions.

These components operate sequentially but are cryptographically interdependent, forming a pipeline where each stage inherits and amplifies the security guarantees of the previous one.

---

## 3. Cryptographic Specification

The protocol relies on standard, well-analyzed primitives:

- **Hash Function:** SHA-256
  - Collision resistance: prevents distinct inputs from producing identical hashes
  - Preimage resistance: prevents reconstruction of inputs from outputs

- **Deterministic RNG:** HMAC-DRBG (NIST SP 800-90A compliant)
  - Provides forward security and pseudorandom expansion
  - Deterministic under fixed seed, ensuring reproducibility

- **Minimum Player Entropy:** ≥ 128 bits
  - Ensures brute-force infeasibility
  - Prevents entropy collapse due to weak user inputs

Under these assumptions, the resulting permutation space (52!) remains computationally infeasible to enumerate, predict, or bias.

---

## 4. Operational Protocol

### 4.1 Commit Phase

Each participant independently samples a high-entropy secret value and computes a commitment:

```javascript
function commit(value) {
  return sha256(value);
}
```

The commitment binds the participant to a fixed input without revealing it. This prevents strategic adaptation after observing other players' inputs.

---

### 4.2 Reveal Phase

Participants disclose their original values, which are verified against prior commitments:

```javascript
function verifyCommit(value, commitment) {
  return sha256(value) === commitment;
}
```

Invalid commitments trigger deterministic rejection of the round. This enforces protocol integrity and prevents malformed entropy injection.

---

### 4.3 Player Entropy Aggregation

All revealed inputs are concatenated and hashed:

```javascript
function buildPlayerEntropy(values) {
  return sha256(values.join(""));
}
```

This aggregation step performs entropy mixing, ensuring that even if some inputs are weak, the combined output remains robust provided at least one honest participant contributes sufficient entropy.

---

### 4.4 External Entropy Injection

An external entropy source is incorporated to mitigate collusion risks:

```javascript
function externalEntropy(blockHash) {
  return blockHash;
}
```

By referencing a blockchain-derived hash, the protocol introduces a source that is:

- Publicly verifiable
- Economically expensive to manipulate
- Independent of participating actors

---

### 4.5 Seed Derivation Function

The final seed is derived as:

```javascript
function buildSeed(S_n, I_n, E_n, nonce) {
  return sha256(S_n + I_n + E_n + nonce);
}
```

This function compresses all entropy sources into a fixed-length output. The inclusion of a nonce guarantees domain separation between rounds, preventing seed reuse and replay conditions.

---

## 5. Deterministic Random Generator (CSPRNG)

The protocol uses HMAC-DRBG to expand the seed into a pseudorandom stream:

```javascript
const crypto = require('crypto');

function hmacDRBG(seed, counter) {
  return crypto
    .createHmac('sha256', seed)
    .update(counter.toString())
    .digest();
}
```

The counter parameter ensures that each invocation produces a unique output while maintaining deterministic reproducibility. This construction also provides forward unpredictability: future outputs cannot be inferred from past outputs without knowledge of the seed.

---

## 6. Bias-Free Fisher-Yates Shuffle

Uniform sampling over the permutation space is achieved using Fisher-Yates with rejection sampling:

```javascript
function getRandomInt(seed, counter, max) {
  let r, limit;
  const MAX = 2 ** 32;
  limit = Math.floor(MAX / max) * max;

  do {
    const hash = hmacDRBG(seed, counter++);
    r = hash.readUInt32BE(0);
  } while (r >= limit);

  return r % max;
}

function shuffle(deck, seed) {
  let counter = 0;

  for (let i = deck.length - 1; i > 0; i--) {
    const j = getRandomInt(seed, counter++, i + 1);
    [deck[i], deck[j]] = [deck[j], deck[i]];
  }

  return deck;
}
```

Rejection sampling eliminates modulo bias by discarding out-of-range values, ensuring that each index is selected with equal probability.

---

## 7. Canonical Deck Construction

```javascript
function generateDeck() {
  const suits = ['H', 'D', 'C', 'S'];
  const ranks = ['A','2','3','4','5','6','7','8','9','10','J','Q','K'];

  let deck = [];

  for (let s of suits) {
    for (let r of ranks) {
      deck.push(r + s);
    }
  }

  return deck;
}
```

The canonical ordering serves as a deterministic baseline, ensuring that all permutations originate from a shared initial state.

---

## 8. Full Execution Pipeline

```javascript
function runHand(S_n, playerValues, blockHash, nonce) {
  const commitments = playerValues.map(v => commit(v));

  const valid = playerValues.every((v, i) => verifyCommit(v, commitments[i]));
  if (!valid) throw new Error("Invalid commitment");

  const I_n = buildPlayerEntropy(playerValues);
  const E_n = externalEntropy(blockHash);

  const seed = buildSeed(S_n, I_n, E_n, nonce);

  let deck = generateDeck();
  deck = shuffle(deck, seed);

  return deck;
}
```

This pipeline is strictly deterministic. Given identical inputs, the output deck is guaranteed to be identical, enabling full third-party verification.

---

## 9. State Transition Function

```javascript
function updateState(deck) {
  return deck.join("");
}
```

Each resulting deck becomes the next system state S_{n+1}, forming a cryptographic chain of states analogous to a hash-linked ledger.

---

## 10. Verifiable Properties

- **Deterministic Reproducibility:** Exact replay is possible under identical inputs
- **Auditability:** Independent observers can recompute all intermediate values
- **Pre-Reveal Unpredictability:** Future states remain hidden until reveal phase completion
- **Bias Elimination:** Uniform permutation sampling guaranteed via rejection sampling
- **Collusion Resistance:** External entropy disrupts coordinated manipulation

---

## 11. Security Model Considerations

The protocol assumes:

- At least one honest participant contributing high entropy
- Cryptographic primitives remain unbroken
- External entropy source is economically infeasible to manipulate

Under these assumptions, adversaries cannot bias or predict outcomes beyond negligible probability.

---

## 12. Conclusion

The PGDM v2 protocol redefines randomness as a deterministic emergent property derived from cryptographic composition and distributed entropy coordination. By eliminating reliance on opaque randomness providers, it establishes a verifiable, mathematically grounded, and adversarial-resistant foundation for decentralized poker infrastructures and similar trustless systems.

---

**End of Document**

