# Crack Chain Poker — Visual, Functional Description and Executable Protocol

## 1. System Purpose
Crack Chain Poker is an interface and operational system for private poker tables based on wallet identity, cryptographic authentication, derived roles, and a deterministic sequence of events.

The system strictly separates:
- user visual intent,
- wallet authentication,
- role derivation,
- table creation,
- linear event logging,
- and the future randomness layer for card dealing.

---

## 2. Initial Screen: Entry

### 2.1 Visible Elements
- Main title: `Crack Chain Poker`
- Subtitle: `Enter`
- Two buttons:
  - `Host`
  - `Player`

### 2.2 Visual Function
This screen only defines access intent.
It does not assign real privileges.
It does not authenticate.
It does not create a session.

### 2.3 Functional Behavior
When selecting an option:
- the system records user intent,
- redirects to wallet connection flow,
- and stores the selected route as initial context.

```text
ENTRY_SCREEN -> ROLE_INTENT = HOST | PLAYER
```

---

## 3. Wallet Connection Screen

### 3.1 Objective
Allow identity connection through a wallet, supporting two modes:
- hierarchical deterministic derivation using words and path,
- or direct private key import.

### 3.2 Visible Elements
- Title: `Connect Wallet`
- Input options:
  - `Seed Phrase / Derivation Wallet`
  - `Private Key Import`
- Fields for:
  - recovery words,
  - passphrase,
  - derivation path,
  - hardened option,
  - BIP38 password if applicable.

### 3.3 Visual Function
Guides the user to select the appropriate wallet access method.

### 3.4 Logical Function
- If seed phrase is used, wallet is derived.
- If private key is used, wallet is imported directly.
- If key is BIP38 encrypted, password is required to decrypt.

---

## 4. Identity Logic

### 4.1 Base Model
System identity does not depend on UI text, but on validated cryptographic material.

```text
Wallet = Identity + Key Material + Access Context
```

### 4.2 Two Identity Paths

#### Path A: Deterministic derivation
Uses:
- seed phrase,
- passphrase,
- derivation path,
- hardened derivation.

#### Path B: Direct import
Uses:
- private key,
- optional BIP38,
- decryption password.

---

## 5. Derivation Path Semantics

### 5.1 Objective
Defines deterministic and auditable identity hierarchy.

### 5.2 Functional Rules
- Required for derived wallets.
- Not required for raw private keys.
- Hardened is used for security isolation.

### 5.3 Conceptual Model
```text
m / purpose' / network' / role' / license' / table_id' / index'
```

### 5.4 Hardened Derivation
Marks segments for secure derivation.

```text
m/0'/1'/2'/3'
```

---

## 6. Role Semantics

### 6.1 Host
The host creates and operates the table.

Functions:
- create table,
- configure rules,
- open/close access,
- start sessions,
- log events.

### 6.2 Player
The player participates within the table.

Functions:
- connect wallet,
- request entry,
- play,
- perform allowed actions.

### 6.3 Critical Rule
Role must not depend solely on UI.
It must be derived cryptographically and contextually.

---

## 7. Step-by-Step Protocol

### Step 1 — Entry
Display main screen.

### Step 2 — Intent Selection
User selects Host or Player.

### Step 3 — Wallet Connection
System opens connection interface.

### Step 4 — Wallet Mode Selection
User chooses derivation or private key.

### Step 5 — Validation
System validates input.

### Step 6 — Session Activation
Session is created if valid.

### Step 7 — Context Resolution
System determines role and permissions.

### Step 8 — Lobby Access
Wallet enters operational space.

---

## 8. Event Protocol

### 8.1 Principle
All actions are recorded linearly.

### 8.2 Event Structure
```text
Event = {
  event_id,
  timestamp,
  sequence_index,
  actor_id,
  table_id,
  action_type,
  payload,
  prev_event_hash
}
```

### 8.3 Rule
History is append-only.

---

## 9. Table Creation

### 9.1 Flow
Host creates table.

### 9.2 Fields
- name,
- descriptions,
- game type,
- max players,
- rules,
- initial state.

### 9.3 Initial State
```text
WAITING_PLAYERS
```

---

## 10. Executable Pseudo-Protocol

```pseudo
ON_APP_START:
  render(ENTRY_SCREEN)

ON_SELECT_ROLE(role_intent):
  store(role_intent)
  render(WALLET_CONNECT_SCREEN)

ON_SELECT_WALLET_MODE(mode):
  IF mode == "DERIVATION":
    request(seed_phrase, passphrase, derivation_path, hardened_flag)
  IF mode == "PRIVATE_KEY":
    request(private_key)
    IF encrypted:
      request(bip38_password)

ON_VALIDATE_WALLET(input):
  IF valid:
    create_session()
    resolve_context()
    render(LOBBY)
  ELSE:
    show_error()
```

---

## 11. Derivation Pseudocode

```pseudo
DERIVE_WALLET(seed, passphrase, path, hardened):
  root = KDF(seed, passphrase)
  parsed_path = parse(path)
  IF hardened:
    parsed_path = apply_hardened(parsed_path)
  return derive(root, parsed_path)
```

---

## 12. Private Key Import Pseudocode

```pseudo
IMPORT_PRIVATE_KEY(key, bip38_password):
  IF encrypted:
    key = decrypt_bip38(key, bip38_password)
  validate(key)
  return key
```

---

## 13. Design Criteria

- UI must not replace authentication
- Wallet must not depend on visual role
- Role must be verifiable
- Events must be sequential
- Randomness is added later

---

## 14. Expected Output

After connection:
- active identity
- role
- session state
- lobby access

---

## 15. Operational Summary

```text
Entry → Intent → Wallet → Validation → Session → Role → Lobby → Table → Events
```

---

## 16. Consistency Note

The system must keep separate:
- visual intent,
- cryptographic identity,
- protocol execution.

This separation ensures scalability and determinism.

