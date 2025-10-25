
# STX-TrustValue: Decentralized Escrow & Trust Protocol

**TrustValue** is a Clarity-based smart contract that enables **secure peer-to-peer (P2P) transactions** with **built-in trust and rating mechanics**.
It facilitates **escrowed payments**, **deal tracking**, and **reputation scoring**, ensuring fair and transparent exchanges without centralized intermediaries.

---

## 📜 Overview

Trustvalue provides a decentralized trust layer for users transacting on the Stacks blockchain.

It allows:

* Users to **initiate secure deals** with others.
* Funds to be **locked in escrow** until a transaction is completed.
* Participants to **rate each other**, building a verifiable **trust score** over time.

This design aims to minimize fraud and encourage fair behavior in P2P ecosystems such as marketplaces, freelance services, and digital asset exchanges.

---

## ⚙️ Core Features

| Feature                  | Description                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------- |
| 🧾 **Deal Creation**     | Initiate a deal specifying a counterparty and value. The deal is stored immutably on-chain. |
| 💰 **Escrow Payment**    | Funds are transferred securely upon completion of a deal.                                   |
| ✅ **Payment Completion** | The initiator finalizes the transaction, transferring the funds to the counterparty.        |
| ⭐ **Trust Ratings**      | Counterparties can rate each other post-deal to build on-chain trust profiles.              |
| 📊 **Reputation System** | Each user accumulates a cumulative trust score and deal count.                              |
| 🔍 **Transparency**      | Anyone can query deal or payment details directly on-chain.                                 |

---

## 🧩 Contract Architecture

### Constants

| Name             | Description                                                               |
| ---------------- | ------------------------------------------------------------------------- |
| `CONTRACT-OWNER` | Contract deployer / owner address.                                        |
| `ADMIN`          | Alias for the deployer used in authorization checks.                      |
| `ERR-*`          | Error codes for consistent contract-wide validation and failure handling. |

---

### Data Structures

#### 1. **Payments Map**

Stores escrowed payment details.

| Field         | Type        | Description                            |
| ------------- | ----------- | -------------------------------------- |
| `id`          | `uint`      | Unique payment identifier              |
| `from`        | `principal` | Initiator (payer)                      |
| `to`          | `principal` | Counterparty (receiver)                |
| `amount`      | `uint`      | STX value locked in escrow             |
| `is-complete` | `bool`      | Indicates whether payment is finalized |
| `created-at`  | `uint`      | Block height of payment creation       |

#### 2. **Deals Map**

Tracks every deal and its metadata.

| Field          | Type                | Description                            |
| -------------- | ------------------- | -------------------------------------- |
| `deal-id`      | `uint`              | Unique deal identifier                 |
| `initiator`    | `principal`         | Deal creator                           |
| `counterparty` | `principal`         | Other party in the transaction         |
| `value`        | `uint`              | Deal value                             |
| `state`        | `string-ascii (20)` | Deal state (e.g., "OPEN")              |
| `timestamp`    | `uint`              | Block height of creation               |
| `trust-score`  | `uint`              | Rating value assigned after completion |

#### 3. **Trust Profiles Map**

Stores cumulative trust scores for each participant.

| Field              | Type        | Description                 |
| ------------------ | ----------- | --------------------------- |
| `address`          | `principal` | User address                |
| `cumulative-score` | `uint`      | Sum of all ratings received |
| `deal-count`       | `uint`      | Number of completed deals   |

---

### Data Variables

| Variable             | Type   | Purpose                            |
| -------------------- | ------ | ---------------------------------- |
| `payment-id-counter` | `uint` | Sequential counter for payment IDs |
| `deal-counter`       | `uint` | Sequential counter for deal IDs    |

---

## 🚀 Public Functions

### 1. `initiate-deal (counterparty principal) (value uint)`

Creates a new deal between the sender and another party.

**Requirements:**

* `counterparty` cannot be the sender or admin.
* `value` must be greater than zero.

**Behavior:**

* Registers a new deal and payment record.
* Returns the new `deal-id`.

**Returns:**
`(ok deal-id)` or error code:

* `ERR-INVALID-USER`
* `ERR-LOW-VALUE`
* `ERR-SELF-DEAL`
* `ERR-ZERO-AMOUNT`

---

### 2. `complete-payment (payment-id uint)`

Transfers escrowed funds from the initiator to the counterparty.

**Requirements:**

* Caller must be the payment sender.
* Payment must exist.

**Behavior:**

* Transfers `amount` STX using `stx-transfer?`.
* Marks payment as complete.

**Returns:**
`(ok true)` or error:

* `ERR-NO-PAYMENT`
* `ERR-NO-AUTH`

---

### 3. `rate-counterparty (deal-id uint) (rating uint)`

Allows a counterparty to rate the initiator after deal completion.

**Requirements:**

* Only the counterparty can rate.
* `rating` must be > 0.
* `deal-id` must exist and be valid.

**Behavior:**

* Updates the initiator’s trust profile (cumulative score and count).
* Updates deal record with the rating.

**Returns:**
`(ok true)` or error:

* `ERR-INVALID-DEAL-ID`
* `ERR-DEAL-NOT-EXIST`
* `ERR-NOT-AUTHORIZED`
* `ERR-BAD-RATING`

---

## 📖 Read-Only Functions

| Function                                | Description                                                 |
| --------------------------------------- | ----------------------------------------------------------- |
| `get-trust-profile (address principal)` | Returns the trust score and deal count for a given address. |
| `get-payment-info (payment-id uint)`    | Retrieves payment details by ID.                            |
| `get-deal-info (deal-id uint)`          | Retrieves deal information by ID.                           |

---

## 🧪 Example Workflow

### 1. Initiate a Deal

```clarity
(contract-call? .trustbridge initiate-deal 'ST2ABC...  u1000)
;; → (ok u1)
```

### 2. Complete Payment

```clarity
(contract-call? .trustbridge complete-payment u1)
;; → (ok true)
```

### 3. Rate Counterparty

```clarity
(contract-call? .trustbridge rate-counterparty u1 u5)
;; → (ok true)
```

### 4. Query Trust Profile

```clarity
(contract-call? .trustbridge get-trust-profile 'ST2ABC...)
;; → { cumulative-score: u5, deal-count: u1 }
```

---

## 🔒 Security Considerations

* **No self-deals:** The initiator cannot open a deal with themselves or the admin.
* **Value validation:** Prevents zero-value or low-value transactions.
* **Authorization checks:** Only authorized participants can complete payments or rate deals.
* **Immutable audit trail:** Every deal and rating is permanently stored on-chain.

---

## 📈 Potential Extensions

Future improvements could include:

* Support for **multi-signature escrow** (both parties must approve before release).
* **Dispute resolution** via a third-party arbiter.
* **Weighted trust scores** (recent deals count more).
* **NFT or token-based incentives** for high-trust participants.

---

## 🧠 Developer Notes

* Written in **Clarity** (Stacks smart contract language).
* Compatible with **Stacks 2.1+** network.
* Designed for **decentralized marketplaces, freelance platforms, and microtransactions.**

---

## 📄 License

MIT License — freely usable, modifiable, and distributable with attribution.
