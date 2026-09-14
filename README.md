# SME-Ledger — ADTC 2026 Laptop LLM Submission

## Africa Deep Tech Challenge 2026

**SME-Ledger** is a lightweight, offline-first financial transaction intelligence system designed to transform unstructured transaction messages into structured financial records and useful financial insights.

The project began with a focus on **M-Pesa SMS messages**, addressing the problem of valuable financial information being trapped inside unstructured mobile-money notifications. During the Africa Deep Tech Challenge semifinal evaluation and subsequent model analysis, however, an important limitation became clear: a model trained on a narrow set of transaction examples could struggle to generalize beyond the formats it had seen during training.

SME-Ledger was therefore expanded beyond its original M-Pesa-only training scope.

The current model is trained on approximately **20,000 diverse examples**, including synthetic financial transaction messages, banking notifications, mobile-money messages, payment messages, Pochi transactions, Paybill transactions, Till payments, transfers, Fuliza-related transactions, reversals, failures, and capability-oriented instruction examples.

The result is a more general **financial transaction understanding model**, with M-Pesa as an important use case and starting point rather than the only environment in which the model is relevant.

> **Your Data. Your Device. Your Financial Intelligence.**

---

#  Competition Submission

| Field | Details |
|---|---|
| **Challenge** | Africa Deep Tech Challenge 2026 |
| **Track** | Laptop LLM |
| **Team ID** | `sme-ledger-from-messy-massages-logs` |
| **Primary Domain** | Financial Transaction Intelligence |
| **Model** | Gemma 3 270M Financial Intelligence |
| **Quantization** | GGUF Q4_K_M |
| **Runtime** | llama.cpp |
| **Target Hardware** | 8 GB RAM budget laptop |
| **Inference** | Fully offline |
| **Training Dataset** | SME-Ledger V2 Synthetic Financial Intelligence Dataset |
| **Training Scale** | ~20,000 examples |
| **Initial Focus** | M-Pesa and mobile-money transaction messages |
| **Current Scope** | Diverse financial and transaction messaging |

---

#  The Problem

A large amount of financial information exists as unstructured messages.

Depending on the service and country, users may receive notifications for:

- Mobile-money transactions
- Bank deposits
- Bank withdrawals
- Account transfers
- Merchant payments
- Bill payments
- Wallet transactions
- Loan disbursements
- Loan repayments
- Transaction reversals
- Failed transactions
- Account balance updates

These messages often contain valuable financial information, but they are difficult to analyze automatically because they exist as free-form or semi-structured text.

For example, a transaction notification may contain:

- A transaction reference
- Sender or recipient information
- Transaction amount
- Account or wallet balance
- Date and time
- Transaction cost
- Merchant information
- Transaction status

Without structured processing, this information remains a **wall of text**.

Users may have hundreds or thousands of transaction notifications but still struggle to answer questions such as:

> How much money came in this month?

> How much did I spend?

> What types of transactions are happening most frequently?

> What is my net cash flow?

> Are there duplicate or inconsistent transactions?

> Can these messages be converted into a usable financial ledger?

SME-Ledger addresses this problem using a small, specialized language model that can understand diverse transaction-message patterns while operating locally on constrained hardware.

---

# 🔄 From Semifinal Feedback to a More General Model

A major part of the SME-Ledger development journey was learning from the limitations observed during the semifinal stage.

The earlier version of the model was trained on approximately **200 examples**, primarily focused on M-Pesa transaction messages.

This approach demonstrated that a small model could learn the desired extraction task, but it also revealed an important limitation:

> **A model trained on a narrow distribution of examples may perform well on familiar message formats while generalizing poorly to different transaction domains, formats, or capability-oriented questions.**

During evaluation and subsequent testing, it became clear that the model needed more diversity than simply adding more examples of the same M-Pesa message structure.

The solution was therefore not just to increase the dataset size.

The training strategy itself was redesigned.

The updated dataset was expanded to approximately **20,000 examples** covering multiple transaction domains, message structures, linguistic variations, extraction tasks, financial reasoning contexts, and capability-oriented instructions.

This helped move the model from a narrowly specialized M-Pesa parser toward a broader **financial transaction intelligence model**.

---

# Training Dataset

The current SME-Ledger V2 training dataset contains approximately **20,000 examples**.

A significant portion of the additional data is synthetically generated to create controlled diversity across transaction types and message formats while avoiding the use of real personal financial records.

The dataset covers multiple categories of financial communication.

## Transaction and Financial Message Coverage

Examples include:

### Mobile Money

- Money received
- Money sent
- Wallet payments
- Merchant transactions
- Cash withdrawals
- Airtime purchases
- Balance notifications

### M-Pesa Ecosystem

- Send Money
- Receive Money
- Paybill payments
- Till payments
- Pochi transactions
- Fuliza drawdowns
- Fuliza repayments
- Fuliza fees
- Reversals
- Failed transactions
- Balance updates

### Banking and Transfers

- Bank deposits
- Bank withdrawals
- Incoming transfers
- Outgoing transfers
- Bank-to-wallet transfers
- Wallet-to-bank transfers
- Transaction confirmations
- Account balance notifications

### Payment and Merchant Messages

- Merchant payments
- Bill payments
- Service payments
- Business transactions
- Payment confirmations

### Transaction States

- Successful transactions
- Pending transactions
- Failed transactions
- Reversed transactions
- Refunded transactions

The objective is to expose the model to the **underlying concepts of financial transactions**, rather than allowing it to memorize a small number of M-Pesa templates.

---

#  Capability Instruction Training

One weakness identified during earlier testing was that the model could perform a transaction-related task but was not always able to clearly explain what it could do when asked directly.

For example, questions such as:

> What can you do?

> What types of messages can you understand?

> Can you analyze this transaction?

> What information can you extract?

could produce responses that did not accurately represent the model's intended functionality.

The updated dataset therefore includes **capability-oriented instruction examples**.

These examples help the model understand and communicate its role, including its ability to:

- Understand transaction messages
- Extract structured financial information
- Identify transaction participants
- Extract transaction amounts
- Extract balances
- Identify dates and times
- Classify transaction types
- Recognize transaction status
- Handle multiple transaction-message formats
- Return structured outputs
- Support downstream financial analysis

This improves the model's ability to communicate its capabilities while remaining grounded in its actual specialization.

---

#  Why Synthetic Data?

The expansion from approximately 200 examples to approximately 20,000 examples required a scalable way to create diverse training data.

Synthetic generation allows the project to systematically vary:

- Transaction types
- Message structures
- Entity names
- Merchant names
- Amounts
- Dates
- Times
- Account balances
- Transaction references
- Transaction status
- Banking terminology
- Mobile-money terminology
- Payment formats

The goal is not to reproduce the financial history of real people.

Instead, the goal is to create realistic **transaction-language patterns** that teach the model how to understand the structure and meaning of financial messages.

The synthetic approach also allows the dataset to avoid including real customer financial records or personally identifiable information.

---

#  Dataset Generation & Provenance

The SME-Ledger V2 dataset was generated using a reproducible synthetic data-generation pipeline.

The dataset-generation process is documented in the Kaggle notebook:

[Kaggle — SME-Ledger Data Generator](https://www.kaggle.com/code/wangapa106g/sme-ledger-data-genarator/output)

The generator is responsible for creating diverse instruction-tuning examples across financial transaction categories and model capability questions.

The repository includes a `provenance/` directory containing information required to understand the origin and structure of the training data.

```text
provenance/
├── README.md
└── dataset_sample.jsonl
```

The provenance documentation should include:

- Dataset version
- Dataset generation methodology
- Generation seed
- Dataset size
- Dataset splits
- Transaction categories
- Capability-instruction categories
- Data source classification
- PII policy
- Kaggle generator reference
- Dataset license

A representative sample is included in `dataset_sample.jsonl` so the training format can be inspected without requiring the full dataset to be stored in the submission repository.

---

#  Beyond a Single Country or Payment Platform

SME-Ledger originated from a Kenyan problem: financial information embedded in M-Pesa transaction messages.

M-Pesa remains an important and practical starting point for the project.

However, the underlying problem is broader.

Financial institutions, mobile-money providers, banks, wallets, merchants, and payment platforms around the world generate transaction notifications that contain similar concepts:

```text
Who?
↓
Did what?
↓
With how much money?
↓
When?
↓
What was the result?
↓
What is the resulting balance or status?
```

The specific wording changes from one provider to another, but the underlying financial concepts remain similar.

The expanded training strategy therefore focuses on learning these concepts across diverse message structures.

This makes SME-Ledger potentially adaptable beyond:

- M-Pesa
- Kenya
- A single mobile-money provider
- A single transaction-message template

The current model should therefore be understood as a **financial transaction intelligence model with a strong mobile-money foundation**, rather than a model designed exclusively for one Kenyan messaging format.

---

#  Solution Architecture

SME-Ledger separates language understanding from deterministic financial computation.

```text
                Transaction Messages
                         │
                         ▼
              ┌─────────────────────┐
              │  SME-Ledger LLM     │
              │  Gemma 3 270M       │
              └──────────┬──────────┘
                         │
                         ▼
              Structured Transaction
                  Understanding
                         │
                         ▼
              ┌─────────────────────┐
              │ Validation Layer    │
              │ + Normalization     │
              └──────────┬──────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │ Local Ledger        │
              │ / Analytics Engine  │
              └──────────┬──────────┘
                         │
                         ▼
              Deterministic Financial
                    Computation
                         │
                         ▼
              Financial Insights
```

### The LLM handles

- Transaction-language understanding
- Entity extraction
- Amount extraction
- Balance extraction
- Date identification
- Transaction classification
- Transaction status recognition
- Structured output generation
- Capability-oriented responses

### Deterministic software handles

- Arithmetic
- Income totals
- Expense totals
- Net cash flow
- Financial summaries
- Transaction aggregation
- Balance consistency checks
- Duplicate detection
- Ledger generation

This hybrid design keeps the model focused on what language models are good at while allowing deterministic code to handle operations where consistency is critical.

---

#  Model

The final model is based on:

```text
Base Model:
Gemma 3 270M

Fine-Tuning:
LoRA / PEFT Instruction Tuning

Training Data:
~20,000 diverse financial transaction and capability examples

Final Format:
GGUF Q4_K_M

Runtime:
llama.cpp
```

The model was selected because the project requires a balance between:

- Small memory footprint
- Fast CPU inference
- Offline operation
- Financial language understanding
- Structured output generation
- Deployment on constrained hardware

The objective is not to build the largest possible financial model.

The objective is to demonstrate that a relatively small model can acquire useful, specialized financial transaction understanding when trained with sufficiently diverse and carefully designed data.

---

#  Privacy-First Architecture

Financial messages contain sensitive information.

SME-Ledger is designed around local processing:

```text
                 INTERNET
                    X
                    │
          ┌─────────▼─────────┐
          │    User Device    │
          │                   │
          │ Transaction SMS   │
          │       ↓           │
          │ Local LLM         │
          │       ↓           │
          │ Local Ledger      │
          │       ↓           │
          │ Local Analytics   │
          │       ↓           │
          │ Financial Insight │
          └───────────────────┘
```

During inference, the model is designed to run without requiring an external AI API or cloud-based language model.

---

#  Designed for the 8 GB Laptop

The ADTC environment emphasizes constrained hardware.

SME-Ledger therefore uses a small model that can run through `llama.cpp` with GGUF quantization.

The target environment includes:

- 4 CPU cores
- 8 GB RAM
- Integrated graphics
- No dedicated GPU requirement
- Offline inference

The project demonstrates that improving model capability does not necessarily require increasing parameter count.

In this case, a major improvement came from **improving training-data diversity and coverage** rather than simply using a larger model.

---

# 📁 Repository Structure

```text
adtc-2026-submission-template/
│
├── metadata.json
├── README.md
├── REPORT.md
├── download_model.sh
├── .gitignore
│
├── model/
│   └── sme-ledger-financial-intelligence-Q4_K_M.gguf
│
├── provenance/
│   ├── README.md
│   └── dataset_sample.jsonl
│
└── submission.json
```

The model weights are not committed to Git.

The `download_model.sh` script retrieves the public GGUF model before evaluation.

---

#  Project Evolution

```text
Stage 1
│
├── ~200 M-Pesa-focused examples
│
├── Demonstrated structured SMS extraction
│
└── Limitation:
    Narrow generalization outside familiar formats

                ↓

Semifinal Evaluation & Analysis

                ↓

Stage 2
│
├── ~20,000 training examples
│
├── Diverse financial transaction domains
│
├── Banking messages
│
├── Mobile-money messages
│
├── Pochi / Paybill / Till coverage
│
├── Transaction failures and reversals
│
├── Multiple message structures
│
└── Capability-oriented instruction tuning

                ↓

Current Direction

Lightweight General Financial
Transaction Intelligence Model

                ↓

Offline Deployment

8 GB Laptop → Future Android / Edge Deployment
```

---

#  Long-Term Vision

SME-Ledger begins with transaction-message intelligence.

The longer-term opportunity is broader:

```text
Transaction Messages
        ↓
Structured Financial Data
        ↓
Personal / SME Ledger
        ↓
Cash-Flow Intelligence
        ↓
Financial Health Signals
        ↓
Credit Readiness
        ↓
Privacy-Preserving Financial Intelligence
```

Potential future applications include:

- Personal finance
- SME bookkeeping
- Mobile-money analytics
- Banking message analysis
- Wallet transaction intelligence
- Cash-flow monitoring
- Financial anomaly detection
- Balance consistency validation
- Loan affordability analysis
- SACCO decision support
- Financial-institution integrations
- On-device Android deployment

---

# Project Resources

### Source Repository

[SME-Ledger — ADTC 2026 Submission](https://github.com/Wangadeveloper/adtc-2026-submission-template)

### Dataset Generation

[SME-Ledger Synthetic Data Generator — Kaggle](https://www.kaggle.com/code/wangapa106g/sme-ledger-data-genarator/output)

### ADTC

[Africa Deep Tech Challenge 2026](https://adtc-2026.devpost.com)

---

# 📄 License

This submission repository is licensed under the **GNU GPL v3 License**.

Dataset provenance and licensing information are documented in the `provenance/` directory.

---

##  Acknowledgement

SME-Ledger was developed as an iterative research and engineering project for the **Africa Deep Tech Challenge 2026**.

A key part of its development was learning from model limitations observed during evaluation and responding by improving **data diversity, task coverage, and generalization**.

The project demonstrates an important principle for edge AI:

> **Better specialization is not always achieved by making the model bigger. Sometimes the biggest improvement comes from teaching a small model a more complete view of the problem.**

**SME-Ledger**

> *Your Data. Your Device. Your Financial Intelligence.*
