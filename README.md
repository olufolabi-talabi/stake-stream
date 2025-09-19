
# 🧱 StakeStream Protocol

**StakeStream** is a next-generation, Bitcoin-aligned DeFi primitive designed for secure, capital-efficient staking and decentralized governance on **Stacks L2**. This protocol introduces **tiered staking**, **on-chain voting**, and **enterprise-grade compliance** into a modular and extensible architecture. It is purpose-built to uphold Bitcoin’s principles while leveraging Stacks' smart contract capabilities.

---

## 📚 Table of Contents

- [🧱 StakeStream Protocol](#-stakestream-protocol)
  - [📚 Table of Contents](#-table-of-contents)
  - [🧠 Overview](#-overview)
  - [✨ Key Features](#-key-features)
  - [🏗️ System Architecture](#️-system-architecture)
  - [🔩 Contract Architecture](#-contract-architecture)
    - [Data Variables](#data-variables)
    - [Key Maps](#key-maps)
  - [🔁 Staking \& Rewards Flow](#-staking--rewards-flow)
  - [🗳 Governance Flow](#-governance-flow)
  - [🧱 Tier Levels](#-tier-levels)
  - [🛡 Security \& Compliance](#-security--compliance)
  - [🧾 Constants \& Error Codes](#-constants--error-codes)
  - [🧪 Development \& Testing](#-development--testing)
  - [⚖ License](#-license)
  - [🙋‍♀️ Contribution](#️-contribution)

---

## 🧠 Overview

**StakeStream** enables users to stake STX tokens in a **multi-tiered reward system** and actively participate in protocol governance. The protocol supports dynamic APY adjustments, lock-based multipliers, and quadratic voting, all underpinned by a **Bitcoin-secured, non-custodial design** via the Stacks L2 chain.

---

## ✨ Key Features

- **Tiered Staking System**: Stake more, earn more. Tier benefits are enhanced by time-locked staking.
- **Time-Lock Multipliers**: Optional 1-month (1.25x) and 2-month (1.5x) staking periods.
- **Dynamic Reward Engine**: Base APY with compounded multipliers for tier and lock duration.
- **On-Chain Governance**: Proposal creation and voting power determined by stake weight.
- **Emergency Mode**: Enables pausing and safety withdrawals.
- **Compliance-Ready**: Built-in monitoring hooks for enterprise & regulatory support.
- **Segregated Safety Vaults**: Isolated STX pools with cooldown-based withdrawals.

---

## 🏗️ System Architecture

```
+-------------------+           +---------------------+
|    STX Stakers    |<--------->|   StakeStream SC    |
+-------------------+           +---------------------+
         |                              |
         v                              v
+------------------+       +-------------------------+
| Governance Votes |<----->|   Governance Module     |
+------------------+       +-------------------------+
         |                              |
         v                              v
+------------------+        +------------------------+
| Reward Engine    |<------>|  Tier & Lock Logic     |
+------------------+        +------------------------+
         |                              |
         v                              v
+------------------+        +------------------------+
| Safety Vaults    |<------>| Emergency/Cooldown     |
+------------------+        +------------------------+
```

---

## 🔩 Contract Architecture

### Data Variables

| Variable           | Type   | Description                              |
| ------------------ | ------ | ---------------------------------------- |
| `contract-paused`  | `bool` | Pauses contract operations               |
| `emergency-mode`   | `bool` | Enables emergency withdrawal mechanism   |
| `stx-pool`         | `uint` | Tracks total STX staked in the protocol  |
| `base-reward-rate` | `uint` | Base APY rate (e.g., 500 = 5%)           |
| `bonus-rate`       | `uint` | Lock duration bonus rate                 |
| `minimum-stake`    | `uint` | Minimum amount required to participate   |
| `cooldown-period`  | `uint` | Duration before funds can be withdrawn   |
| `proposal-count`   | `uint` | Running counter for governance proposals |

### Key Maps

| Map Name           | Purpose                                    |
| ------------------ | ------------------------------------------ |
| `UserPositions`    | Tracks user-level staking metadata         |
| `StakingPositions` | Tracks staking details and reward accrual  |
| `TierLevels`       | Defines stake thresholds and reward boosts |
| `Proposals`        | Records governance proposals and metadata  |

---

## 🔁 Staking & Rewards Flow

1. **Staking (`stake-stx`)**:

   - Transfers STX to contract
   - Determines tier level and lock-period multiplier
   - Updates staking and user position
   - Adds to the global STX pool

2. **Initiating Unstake (`initiate-unstake`)**:

   - Begins cooldown period for withdrawal
   - Sets `cooldown-start` in user’s position

3. **Completing Unstake (`complete-unstake`)**:

   - Verifies cooldown period has passed
   - Transfers STX back to user
   - Removes staking record

4. **Reward Calculation** (internal):

   - `(calculate-rewards)` computes rewards using:

     ```clarity
     (* stake * base-rate * multiplier * blocks) / 14400000
     ```

---

## 🗳 Governance Flow

1. **Proposal Creation (`create-proposal`)**:

   - Requires ≥ 1M voting power
   - Sets description, voting duration
   - Assigns unique `proposal-id`

2. **Voting (`vote-on-proposal`)**:

   - Allows casting votes with voting power
   - Uses quadratic weighting for fairness
   - Tallies `votes-for` and `votes-against`

---

## 🧱 Tier Levels

| Tier   | Min Stake (uSTX) | Multiplier | Features Enabled              |
| ------ | ---------------- | ---------- | ----------------------------- |
| Bronze | 1,000,000        | 1x         | Basic staking & rewards       |
| Silver | 5,000,000        | 1.5x       | Governance access, rewards    |
| Gold   | 10,000,000       | 2x         | Full governance & bonus perks |

> Additional tiers and features can be modularly added via `TierLevels` map.

---

## 🛡 Security & Compliance

- **Emergency Mode**: Admin-controlled toggle to pause staking and enable withdrawals.
- **Cooldown Withdrawals**: Prevents instant unstaking to mitigate flash-exit risks.
- **Transaction Monitoring Hooks** *(planned)*: For enterprise compliance use cases.
- **Non-Custodial Design**: Uses `stx-transfer?` under contract context.

---

## 🧾 Constants & Error Codes

| Constant               | Value         | Meaning                                 |
| ---------------------- | ------------- | --------------------------------------- |
| `ERR-NOT-AUTHORIZED`   | `(err u1000)` | Unauthorized caller                     |
| `ERR-INVALID-PROTOCOL` | `(err u1001)` | Input validation failed                 |
| `ERR-INSUFFICIENT-STX` | `(err u1003)` | User tried to unstake more than allowed |
| `ERR-COOLDOWN-ACTIVE`  | `(err u1004)` | Cooldown already set or not expired     |
| `ERR-NO-STAKE`         | `(err u1005)` | User has no active stake                |
| `ERR-PAUSED`           | `(err u1007)` | Contract is currently paused            |

---

## 🧪 Development & Testing

- Written in [Clarity](https://docs.stacks.co/docs/write-smart-contracts/clarity-overview)
- Compatible with [Clarinet](https://docs.hiro.so/clarinet/overview) for local testing
- Suggested testing coverage:

  - Tier level assignment
  - Lock-period reward multipliers
  - Governance edge cases (voting, proposal expiry)
  - Emergency mode handling

> Test coverage and audits recommended before mainnet deployment.

---

## ⚖ License

MIT License. See `LICENSE` file for details.

---

## 🙋‍♀️ Contribution

If you're interested in contributing, please open issues, submit PRs, or discuss improvements via the community. Governance upgrades will eventually be managed via on-chain proposals.
