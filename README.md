## sBTC-NexusCover - Decentralized Insurance Contract on Stacks (Clarity)**

### 🛡 Overview

This smart contract implements a **decentralized insurance policy management system** on the **Stacks blockchain** using **Clarity**. It facilitates the creation, renewal, and claims processing of insurance policies between insurers and insured parties, enabling secure, transparent, and verifiable insurance workflows.

---

### 📂 Features

✅ **Policy Initiation**
✅ **Premium Payment & Policy Renewal**
✅ **Claim Submission & Approval Workflow**
✅ **Claim Payout & Fund Transfers**
✅ **Risk Scoring & Historical Records Tracking**

---

### ⚙ Smart Contract Modules

#### 1. 📝 `initiate-policy`

Create a new insurance policy between an insurer and an insured party. Enforces constraints on coverage limits and uniqueness of policy.

```clojure
(define-public (initiate-policy (new-insurer principal) (new-insured-party principal) (premium-amount uint) (coverage-amount uint))
```

* Rejects policies with:

  * Invalid principals
  * Premium or coverage ≤ 0
  * Coverage exceeding `max-coverage`
  * Already existing policies

---

#### 2. 💸 `submit-premium`

Activate or renew an insurance policy by paying the defined premium.

```clojure
(define-public (submit-premium (insured principal))
```

* Transfers premium from sender to insurer.
* Activates the policy for \~1 year (`block-height + 52595`).
* Can be used to renew if in the grace period (`grace-period` = 1000 blocks).

---

#### 3. 📤 `submit-claim`

File a new claim under an active insurance policy.

```clojure
(define-public (submit-claim (insured principal) (claim-amount uint))
```

* Validates:

  * Claim amount does not exceed coverage limit.
  * Policy is active and exists.
* Stores claim in `insurance-claims` map.

---

#### 4. ✅ `approve-claim`

Approve a submitted claim by setting `claim-approved` to `true`.

```clojure
(define-public (approve-claim (insured principal))
```

* Only after approval can payout be released.

---

#### 5. 🪙 `release-payout`

Transfer the approved claim amount from the insurer to the insured party.

```clojure
(define-public (release-payout (insured principal))
```

* Transfers STX from insurer to insured.
* Ensures approved claim does not breach coverage.
* Updates total claims in policy record.

---

### 📊 Data Structures

#### 🗂 `insurance-policies`

Tracks policy metadata for each insured principal.

```clojure
{ insurer, policy-premium, policy-coverage, total-claims, policy-expiration, policy-active }
```

---

#### 🗂 `insurance-claims`

Tracks claim requests and their approval status.

```clojure
{ claim-requested, claim-approved }
```

---

#### 🗂 `policy-history`

Tracks premium payment history and past claim amounts (up to 10).

```clojure
{ total-premiums-paid, claim-history, last-premium-payment }
```

---

#### 🗂 `risk-scores`

Holds a risk score for each insured party, with timestamp of last assessment.

```clojure
{ risk-score, last-assessment }
```

---

### ⚖ Constants

* `grace-period`: `u1000` → Time buffer for renewal.
* `max-coverage`: `u1000000` → Max coverage per policy.

---

### 🧪 Example Workflow

1. **Insurer** calls `initiate-policy` to create a policy for an insured party.
2. **Insured** pays using `submit-premium` to activate the policy.
3. **Insured** files a claim using `submit-claim`.
4. **Insurer** approves the claim using `approve-claim`.
5. **Insured** receives payout via `release-payout`.

---

### 📜 Logs & Events

* `insurance-policy-created`
* `premium-paid`
* `claim-filed`
* `claim-approved`
* `payout-released`

These events help indexers and front-end apps track contract activity.

---

### 🔒 Security Considerations

* Prevents double policy registration.
* Validates claim amount within policy limits.
* Ensures only approved claims are paid.
* Uses `unwrap!` to enforce fail-fast behavior.

---

### 🚀 Deployment Notes

* Ensure the contract deployer has enough STX for `stx-transfer?` operations.
* Consider role-based access control for claim approvals in future upgrades.

---
