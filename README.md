# SentinelAdvisor: AI-Powered Token Launch Governance

The **SentinelAdvisor** smart contract is a decentralized, high-integrity advisory layer designed to bridge the gap between generative AI insights and on-chain financial execution. By leveraging authorized AI oracles, the system provides a rigorous evaluation framework for new token projects on the Stacks blockchain, ensuring that only projects meeting specific safety and viability thresholds can proceed to official execution.

---

## Table of Contents
1. Overview
2. Key Features
3. Architecture
4. Functional Specification
    * Private Functions
    * Public Functions
    * Read-Only Functions
5. Technical Parameters
6. User Flow
7. Security Considerations
8. Contribution Guidelines
9. License

---

## Overview

In the rapidly evolving landscape of decentralized finance, token launches often suffer from poor tokenomics, hidden risks, or lack of expert oversight. **SentinelAdvisor** introduces an automated "Gatekeeper" mechanism. Project creators submit their parameters to the contract, which are then audited by specialized AI Oracles. 

The contract doesn't just store advice—it **enforces** it. If an AI score falls below the `MIN-SAFE-SCORE`, the contract's logic prevents the `execute-token-launch-plan` function from completing, effectively acting as an automated circuit breaker against potentially harmful or poorly designed token launches.

---

## Key Features

* **Oracle Governance:** A robust system for the contract owner to whitelist and manage highly specialized AI Oracles.
* **Threshold-Based Execution:** Automated enforcement of safety scores. A minimum score of **70** is required to launch, while a score of **90+** triggers an incentive bonus for the evaluating oracle.
* **Anti-Spam Appeal Mechanism:** A structured appeal system allows creators to challenge evaluations by paying a mandatory `APPEAL-FEE` (10 STX), preventing frivolous re-evaluations and ensuring the AI's "time" is respected.
* **State Machine Integrity:** Strict transition checks ensure a token cannot be launched twice, evaluated twice by the same instance, or appealed once it has already been deployed.
* **Transparent Logging:** Comprehensive event printing for off-chain indexing, allowing UI/UX dashboards to track "Pending," "Appealed," and "Executed" launches in real-time.

---

## Architecture

The system operates via three primary actors:
1.  **Contract Owner:** Manages the list of authorized AI Oracles.
2.  **Project Creator:** Submits projects for review, pays appeal fees, and executes the final launch.
3.  **AI Oracle:** Performs off-chain analysis of tokenomics and posts scores and advice back to the blockchain.

---

## Functional Specification

### Private Functions
These functions are internal to the contract and are used to provide logic abstraction and security checks.

* **`is-oracle (caller principal)`**: 
    * **Purpose**: I use this to validate whether a specific address is currently marked as `is-approved` within the `ai-oracles` data map.
    * **Logic**: It performs a `map-get?` and uses `default-to false` to ensure that any address not explicitly added returns a negative result.
    * **Usage**: Called by `publish-ai-evaluation` to ensure only verified AI entities can influence token scores.

### Public Functions
These functions represent the primary interface for users, oracles, and the contract owner. They modify the contract state and often require STX transfers.

* **`add-oracle (oracle principal)`**: 
    * **Access**: Restricted to the `CONTRACT-OWNER`.
    * **Action**: I update the `ai-oracles` map to grant evaluation permissions to a new principal.
* **`remove-oracle (oracle principal)`**: 
    * **Access**: Restricted to the `CONTRACT-OWNER`.
    * **Action**: I revoke evaluation permissions by setting `is-approved` to `false`.
* **`submit-token-for-review`**: 
    * **Action**: I initialize a new token entry in the `token-evaluations` map. It generates a unique `token-id`, sets the `creator` as the `tx-sender`, and defaults the status to "PENDING".
* **`publish-ai-evaluation`**: 
    * **Access**: Approved AI Oracles only.
    * **Action**: I update a pending token with a score (0-100), a risk string (e.g., "LOW"), and text-based advice. It flips the `is-evaluated` bit to `true` to prevent multiple conflicting evaluations.
* **`execute-token-launch-plan`**: 
    * **Access**: Token Creator only.
    * **Condition**: Requires an `ai-score` $\ge 70$.
    * **Action**: I finalize the launch. If the score is $\ge 90$, it triggers a `SUCCESS-BONUS` transfer of 5 STX from the creator to the oracle.
* **`appeal-ai-evaluation`**: 
    * **Access**: Token Creator only.
    * **Action**: I reset the evaluation state to allow a re-audit. This requires a 10 STX `APPEAL-FEE` paid to the contract and increments the `appeal-count` (capped at 2).

### Read-Only Functions
These functions provide a way for external applications and users to query the state of the contract without incurring gas costs or changing data.

* **`get-evaluation (token-id uint)`**: 
    * **Returns**: I provide the full evaluation record (creator, score, risk level, launch status, etc.) as a tuple, or `none` if the ID does not exist.
    * **Usage**: Essential for front-end dashboards to display the "AI Advice" and "Risk Level" to the community.

---

## Technical Parameters

| Constant | Value | Description |
| :--- | :--- | :--- |
| `MIN-SAFE-SCORE` | `u70` | The minimum score required to call the launch function. |
| `HIGH-SCORE-THRESHOLD` | `u90` | The threshold for paying out a `SUCCESS-BONUS`. |
| `SUCCESS-BONUS` | `u5000000` | 5 STX bonus paid to the Oracle for high-quality projects. |
| `APPEAL-FEE` | `u10000000` | 10 STX required to reset an evaluation. |

---

## User Flow

1.  **Submission**: The creator calls `submit-token-for-review`.
2.  **Evaluation**: An authorized oracle monitors the chain and calls `publish-ai-evaluation`.
3.  **Resolution**: 
    * **Success**: If score $\ge 70$, creator calls `execute-token-launch-plan`.
    * **Appeal**: If dissatisfied, creator calls `appeal-ai-evaluation` for a fee.

---

## Security Considerations

* **Principal Validation**: I ensure only the `creator` can execute the launch or appeal.
* **Financial Safety**: I use `try!` wrappers for all STX transfers to ensure transactions fail gracefully.
* **Oracle Trust**: The `CONTRACT-OWNER` must strictly vet all principals added to the oracle list.

---

## Contribution Guidelines

I welcome contributions to enhance the SentinelAdvisor protocol.
1.  **Fork** the repository.
2.  Create a **Feature Branch** (`git checkout -b feature/AmazingFeature`).
3.  Submit a **Pull Request** with detailed documentation of changes.

---

## License

### MIT License

Copyright (c) 2026 SentinelAdvisor Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

---
