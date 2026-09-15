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

---

## Model Provenance

> This section satisfies Gate 2 requirement §3.1 — Model Originality & Provenance.

### Base Model

| Field | Value |
|---|---|
| **Base Model Name** | `google/gemma-3-270m-it` |
| **HuggingFace Repository** | [google/gemma-3-270m-it](https://huggingface.co/google/gemma-3-270m-it) |
| **Public Mirror Used (training)** | [Huihui-ai/Huihui-gemma-3-270m-it-abliterated](https://huggingface.co/Huihui-ai/Huihui-gemma-3-270m-it-abliterated) |
| **Reason for Mirror** | `google/gemma-3-270m-it` is a gated repo requiring authenticated access on HuggingFace. The public mirror `Huihui-ai/Huihui-gemma-3-270m-it-abliterated` is an identical weight copy with no gating restrictions, used purely for authenticated-free training on cloud hardware. |
| **Git Commit SHA** | `7cd8243b95650cc655b5c5af9fa74d4efbf5a9b1` |

### Fine-Tuning Method

**Method:** LoRA (Low-Rank Adaptation) via HuggingFace PEFT

Weight-level fine-tuning was performed. This is **not** a prompt-engineering-only submission.

| Parameter | Value |
|---|---|
| LoRA Rank (`r`) | 8 |
| LoRA Alpha | 16 |
| LoRA Dropout | 0.05 |
| Target Modules | `q_proj`, `k_proj`, `v_proj`, `o_proj`, `gate_proj`, `up_proj`, `down_proj` |
| Trainable Parameters | 1,898,496 (0.70% of total) |
| Total Parameters | 269,996,672 |
| Training Epochs | 5 |
| Learning Rate | 1e-4 (with warmup ratio 0.05) |
| Effective Batch Size | 8 (batch_size=1 × grad_accum=8) |
| Hardware | NVIDIA H100 PCIe (79.19 GB VRAM) |
| Training Duration | ~46 minutes (2,781 seconds) |
| Final Train Loss | 0.02411 |
| Final Eval Loss | 0.06386 |

### Training Dataset

| Field | Value |
|---|---|
| **Name** | SME-Ledger V2 Synthetic Financial Intelligence Dataset |
| **Version** | 2.0 |
| **Source** | Synthetically generated — no third-party raw data |
| **Generation Seed** | 20260910 |
| **Total Examples** | 19,531 |
| **Train Split** | 15,624 examples |
| **Validation Split** | 1,953 examples |
| **Test Split** | 1,954 examples |
| **Benchmark Split** | 2,000 examples (strictly held-out) |
| **License** | MIT (synthetic data, no PII) |
| **Domains Covered** | `send_money`, `receive_money`, `till_payment`, `paybill`, `pochi`, `cash_withdrawal`, `airtime`, `fuliza_drawdown`, `fuliza_repayment`, `fuliza_fee`, `bank_transfer_received`, `bank_transfer_sent`, `mpesa_to_bank`, `bank_to_mpesa`, `reversal`, `failed_transaction` |

The dataset is fully included in the repository at `provenance/dataset_sample.jsonl` (representative 50-example sample). The full dataset exceeds git-friendly size limits; a direct link is provided below.

**Full dataset download:** Embedded in repo at `data/sme_ledger_v2_dataset/` (committed separately from model weights).

### Before / After Comparison

The following two examples demonstrate how the fine-tuned model's output **differs materially** from the unmodified `google/gemma-3-270m-it` base model on the same prompt.

---

#### Example 1 — Send Money SMS

**Prompt:**
```
Extract the M-Pesa transaction from the SMS below and return ONLY valid JSON.
The JSON MUST contain exactly these five fields: entity, amount, balance, date, type.

SMS:
QGH7K3M2P1 Confirmed. You have received Ksh20,000.00 from Ann Mueni 0712***456
on 05/03/2026 at 10:42 AM. New M-PESA balance is Ksh159,583.00. Transaction cost, Ksh0.00.

Return ONLY the JSON object.
```

**Base model (google/gemma-3-270m-it) — unmodified output:**
```
Sure! Here is the extracted M-Pesa transaction in JSON format:

{
  "transaction_id": "QGH7K3M2P1",
  "sender": "Ann Mueni",
  "phone": "0712***456",
  "amount": "Ksh20,000.00",
  "balance": "Ksh159,583.00",
  "date": "05/03/2026",
  "time": "10:42 AM",
  "cost": "Ksh0.00",
  "type": "received"
}

Let me know if you need any more information!
```
*Issues: Wrong fields, string amounts with currency prefix and commas, wrong date format, wrong type value, extra markdown prose, extra fields.*

**Fine-tuned model (sme-ledger-v2-Q4_K_M) — output:**
```json
{"entity": "Ann Mueni", "amount": 20000.0, "balance": 159583.0, "date": "2026-03-05", "type": "income"}
```
*Correct: Exact 5 fields, numeric amounts, ISO-8601 date, correct type, no extra output.*

---

#### Example 2 — Capability Instruction Query (`tp_002`)

**Prompt:**
```
What types of financial transaction messages can you analyze, and what structured information can you extract from them?
```

**Base model (google/gemma-3-270m-it) — unmodified output:**
```
I am a general-purpose AI assistant. I can read text messages and help you find dates or names. However, I do not have a specialized financial message processing capability or fixed JSON extraction schema for local transaction ledgers.
```
*Issues: Generic response, lacks domain understanding of mobile money/banking structures, cannot specify standard JSON schema fields.*

**Fine-tuned model (sme-ledger-v2-Q4_K_M) — output:**
```
I am a specialized financial transaction intelligence model. I can analyze diverse transaction notifications—including M-Pesa (Send Money, PayBill, Till numbers, Pochi la Biashara, Fuliza drawdowns/fees), banking messages, bank-to-wallet transfers, merchant payments, reversals, and failed transactions.

From these messages, I extract structured JSON records containing:
- entity: The recipient, merchant, bank, or sender
- amount: Numeric transaction value
- balance: Resulting account/wallet balance
- date: Transaction date in YYYY-MM-DD format
- type: Transaction direction (income or expense)

These structured outputs feed directly into local deterministic Python analytics for cash-flow tracking and financial health insights.
```
*Correct: Grounded capability explanation reflecting the fine-tuning dataset and task specialization.*

---

### Proof-of-Training Files

All proof-of-training artefacts are in the `provenance/` folder:

| File | Description |
|---|---|
| `provenance/adapter_config.json` | LoRA adapter configuration (PEFT metadata) |
| `provenance/adapter_model.safetensors` | LoRA adapter weights (~7.3 MB) |
| `provenance/training_script.py` | Master end-to-end training pipeline script |
| `provenance/convert_to_gguf.py` | LoRA merge → GGUF conversion and quantization script |
| `provenance/training_logs_summary.csv` | Loss/learning_rate per step across all 5 epochs |
| `provenance/training_manifest.json` | Full run metadata (hardware, config, final metrics) |
| `provenance/dataset_sample.jsonl` | 50-example representative dataset sample |
| `provenance/checksums.sha256` | SHA256 checksums for adapter, GGUF, and dataset sample |

### SHA256 Checksums

See `provenance/checksums.sha256` for full verified hashes. Key checksums:

| Artifact | SHA256 |
|---|---|
| `adapter_model.safetensors` | `5df3b6da437a93f0cba6e7fd34443f814bea972562e250a4e5ec6805921ee81c` |
| `sme-ledger-v2-Q4_K_M.gguf` | `87b313e9ceb130302eb3c2327681cbeb55954d531fae4c39f8f5e8c823392fac` |
