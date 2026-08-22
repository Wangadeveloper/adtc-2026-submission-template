# Technical Report — Gemma-SME Ledger

**Team ID:** `sme-ledger-from-messy-massages-logs`  
**Domain:** `corporate_enterprise`  
**Model:** `Gemma 3 270M Financial Intelligence Q4_K_M`  

---

## Problem

**Gemma-SME Ledger** is an on-device financial intelligence system designed to turn messy M-Pesa SMS messages into structured financial insights for individuals and small businesses. M-Pesa is a popular mobile money platform in Kenya with 40.99 million active customers as of March 2026. 

For many individuals and small businesses (SMEs) in Kenya, a large portion of their financial history exists solely inside their phones as transaction SMS messages. These messages contain valuable information about income, expenses, merchants, transfers, withdrawals, balances, and recurring payments, but they are difficult to analyze manually. Traditional financial applications require cloud processing, accounts, or internet infrastructure that may not be practical for users with limited devices, costly cellular data, or intermittent connectivity.

Furthermore, financial transaction data is highly sensitive. Sending a user's complete financial history to a remote server for every query introduces privacy, connectivity, cost, and infrastructure barriers. By processing SMS data locally, the system keeps the financial ledger and analytical workflow entirely on the user's device, ensuring 100% offline functionality and privacy.

---

## Design Decisions

The project is designed as a hybrid system that pairs a tiny, specialized language model with deterministic code. The core architecture uses a **Gemma 3 270M** parameter model, fine-tuned with LoRA and quantized to GGUF for efficient inference via `llama.cpp`.

### Multi-Stage Pipeline Architecture

```
                 ┌──────────────────────┐
                 │   M-Pesa SMS Logs    │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │  Gemma 3 270M GGUF   │
                 │   Local Extraction   │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │    Structured JSON   │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │   Pandas Ledger      │
                 └──────────┬───────────┘
                            │
                            ▼
             ┌──────────────────────────────┐
             │  Deterministic Analytics     │
             │                              │
             │  Income                      │
             │  Expenses                    │
             │  Cash Flow                   │
             │  Spending Patterns           │
             │  Financial Signals            │
             └──────────────┬───────────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │  Financial Profile   │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │  Gemma 3 270M GGUF   │
                 │ Local Interpretation │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │    User Guidance     │
                 └──────────────────────┘
```

### Key Decisions
1. **Division of Concerns (LLM + Python):** A 270M parameter model cannot perform complex financial arithmetic or multi-step reasoning reliably. The architecture separates responsibilities:
   * **LLM (Language Understanding):** Extract structured entities and data from messy SMS text into JSON.
   * **Python (Deterministic Computation):** Compute totals, margins, growth, net cash flow, and financial health signals using Pandas.
   * **LLM (Interpretation & Guidance):** Translate the computed profile into natural, contextual advice based on the user's questions.
2. **LoRA Fine-Tuning & Quantization:** The base model `google/gemma-3-270m-it` was fine-tuned using Low-Rank Adaptation (LoRA) to specialize it specifically for Kenyan financial SMS extraction. The final weights were merged and quantized to `GGUF Q4_K_M` to optimize memory and CPU inference performance.
3. **Validation Layer:** Since small models can occasionally produce invalid JSON, a validation and normalization layer was built into the Python pipeline to correct or filter malformed outputs.

---

## Constraints

- **On-Device Target:** 8 GB RAM laptop profile (4 vCPU, integrated GPU).
- **Offline Operation:** Zero external network dependencies. All inference runs locally via `llama.cpp` using the GGUF weight file.
- **Privacy & Security:** Sensitive transactional messages never leave the device, mitigating cybersecurity risks and complying with data privacy principles.
- **Hardware Agnosticism:** Optimized to execute efficiently on standard, budget, or older laptops without requiring high-end dedicated GPUs.

---

## Benchmarks

Development benchmarks measured using the local llama.cpp environment on a standard budget development machine:

| Metric | Value |
|---|---|
| **Machine** | Intel Core i5 (4 Cores), 8 GB RAM, Ubuntu 22.04 |
| **RAM at Peak (Model Load)** | ~320 MB |
| **Time to First Token** | ~28 ms |
| **Generation Speed** | ~82.4 t/s |
| **Thermal Throttling** | None observed |

*Note: Gemma 3 270M Q4_K_M has a very light memory footprint, leaving ample resources (~7.6 GB) for OS operations, the Python analytics engine, and other user applications.*
