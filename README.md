
---

# sBTC-Finance - On-Chain Credit Scoring & Under-Collateralized Lending Protocol (Clarity)

This smart contract implements a decentralized lending system that allows users to request **under-collateralized loans** based on a dynamic, on-chain **credit score**. The system adjusts loan requirements and risk based on borrower reputation and repayment history.

---

##  Features

* ✅ **Credit Scoring System** (min: 50, max: 100)
* 💰 **Under-Collateralized Loans** (collateral required depends on credit score)
* 🔁 **Dynamic Interest Rates** (score-based)
* ⏳ **Loan Repayment & Default Tracking**
* 🔒 **Collateral Management**
* 🛠️ **DAO/Admin Default Marking**
* 📈 **Reputation Adjustment on Repayment or Default**
* 👥 **Per-user Loan Tracking**

---

## 📐 Architecture

### 🎯 Maps

| Name         | Purpose                                          |
| ------------ | ------------------------------------------------ |
| `UserScores` | Tracks credit score, borrowing, and repayment    |
| `Loans`      | Stores each loan’s full details                  |
| `UserLoans`  | Lists all active loans per user (max 20 tracked) |

---

## ⚙️ Initialization

### `initialize-score`

* Initializes a user with the **minimum score (50)**.
* Can only be called once per user.
* Sets initial values for `score`, `total-borrowed`, `loans-repaid`, etc.

```clojure
(contract-call? .credit-score initialize-score)
```

---

## 🧠 Credit Scoring

| Action       | Effect on Score      |
| ------------ | -------------------- |
| On Repayment | +2 (up to max 100)   |
| On Default   | -10 (down to min 50) |

---

## 🏦 Loan Request Flow

### `request-loan (amount uint) (collateral uint) (duration uint)`

* User must have a credit score ≥ 70.
* Max 5 active loans per user.
* Calculates required collateral based on score:

  ```
  required-collateral = amount * (100 - (score * 0.5)) / 100
  ```
* Transfers collateral to contract.
* Creates loan record and disburses loan amount to borrower.

---

### 💸 Loan Structure

Each loan includes:

* `borrower`
* `amount`, `collateral`
* `due-height` (block when due)
* `interest-rate` (10 - (score \* 0.05))
* `repaid-amount`, `is-active`, `is-defaulted`

---

## 💼 Repayment & Scoring

### `repay-loan (loan-id uint) (amount uint)`

* Only borrower can repay.
* Repayment is cumulative.
* Full repayment (principal + interest) triggers:

  * Credit score increase
  * Return of collateral
  * Loan marked inactive

---

## ❌ Defaults

### `mark-loan-defaulted (loan-id uint)`

* Only callable by `CONTRACT-OWNER`.
* Marks a loan as defaulted if overdue.
* Credit score is decreased.
* Collateral is retained by the protocol.

---

## 📊 Score & Loan Queries

### Read-only functions:

```clojure
(get-user-score <principal>)         ;; Returns user score & stats
(get-loan <loan-id>)                 ;; View loan details
(get-user-active-loans <principal>)  ;; Lists active loans for a user
```

---

## 🧾 Constants

| Constant         | Value                        | Purpose                     |
| ---------------- | ---------------------------- | --------------------------- |
| `MIN-SCORE`      | `50`                         | Minimum credit score        |
| `MAX-SCORE`      | `100`                        | Maximum score achievable    |
| `MIN-LOAN-SCORE` | `70`                         | Minimum to qualify for loan |
| `CONTRACT-OWNER` | `tx-sender` (at deploy time) | Admin/DAO control           |

---

## 🔐 Errors

| Error Code                 | Meaning                              |
| -------------------------- | ------------------------------------ |
| `ERR-UNAUTHORIZED`         | Unauthorized access                  |
| `ERR-INSUFFICIENT-BALANCE` | Not enough collateral                |
| `ERR-INVALID-AMOUNT`       | Invalid loan or repayment amount     |
| `ERR-LOAN-NOT-FOUND`       | Loan not found or inactive           |
| `ERR-LOAN-DEFAULTED`       | Loan already defaulted               |
| `ERR-INSUFFICIENT-SCORE`   | Credit score too low                 |
| `ERR-ACTIVE-LOAN`          | Too many active loans                |
| `ERR-NOT-DUE`              | Loan not yet due for default marking |

---

## 🔄 Calculations

### Interest Rate

```clojure
base-rate = 10
interest = base-rate - (score * 0.05)
```

### Required Collateral

```clojure
required = amount * (100 - (score * 0.5)) / 100
```

---
