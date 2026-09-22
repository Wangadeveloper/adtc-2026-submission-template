# Technical Report — Gemma-SME Ledger

**Team ID:** `sme-ledger-from-messy-massages-logs`  
**Domain:** `corporate_enterprise`  
**Model:** `Gemma 3 270M Financial Intelligence Q4_K_M`  

---

## Problem

**Gemma-SME Ledger** is an on-device financial intelligence system designed to transform messy transaction messages into structured financial records and actionable insights for individuals and small businesses in Africa.

In Kenya and across Africa, mobile money (such as M-Pesa with 40.99 million active users) and digital banking notifications form the primary paper trail for informal micro-enterprises and individuals. These messages contain critical transaction data—income, expenses, merchant paybills, Till payments, transfers, loans, and reversals—trapped inside unstructured SMS text.

Manual tracking is error-prone, while cloud-based analytical apps impose bandwidth costs, require persistent connectivity, and expose sensitive personal transaction logs to remote servers. By executing model inference 100% offline on low-spec consumer laptops, SME-Ledger maintains complete data privacy, eliminates cloud API fees, and operates reliably regardless of network availability.

---

## Design Decisions

- **Base model:** `google/gemma-3-270m-it` (270M parameter instruction-tuned model)
- **Quantization:** `GGUF Q4_K_M` chosen for optimal balance between structured JSON output fidelity and small memory footprint (~253 MB weight file)
- **Alternatives considered:**
  - `Q8_0`: Excessively large memory footprint with negligible accuracy gains for entity extraction.
  - `Q2_K`: Degraded JSON syntax validation rates and caused hallucination in date/amount numeric formatting.
  - `Gemma 2B / 7B`: Exceeded 8 GB budget laptop hardware limits for rapid CPU-only real-time token generation.

### Architecture & Multi-Stage Pipeline

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
             │  Income / Expenses           │
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

1. **Division of Concerns (LLM + Python):** A 270M parameter model cannot perform multi-step financial accounting arithmetic reliably. The model handles unstructured text extraction and natural language interpretation, while deterministic Python/Pandas logic computes balances, net cash flow, and financial health indicators.
2. **LoRA Fine-Tuning & Quantization:** The base model `google/gemma-3-270m-it` was fine-tuned using Low-Rank Adaptation (LoRA) to specialize in diverse financial transaction message extraction and capability queries. Weights were merged and quantized to `GGUF Q4_K_M` for `llama.cpp` inference.
3. **JSON Validation Layer:** A validation engine ensures malformed outputs are gracefully handled or repaired before entering the ledger pipeline.

---

## Model Provenance

- **Base model source:** `huggingface:google/gemma-3-270m-it`
- **Base model commit SHA:** `7cd8243b95650cc655b5c5af9fa74d4efbf5a9b1`
- **Fine-tuning method:** `lora`
- **Training datasets:** `SME-Ledger V2 Synthetic Financial Intelligence Dataset` (~20,000 synthetic examples covering Send Money, Receive Money, Paybill, Till, Pochi, Fuliza, Bank Transfers, Reversals, and Capability Instructions)

### Fine-Tuning Details

| Parameter | Value |
|---|---|
| LoRA Rank (`r`) | 8 |
| LoRA Alpha | 16 |
| Target Modules | `q_proj`, `k_proj`, `v_proj`, `o_proj`, `gate_proj`, `up_proj`, `down_proj` |
| Trainable Parameters | 1,898,496 (0.70% of total) |
| Total Parameters | 269,996,672 |
| Training Epochs | 5 |
| Learning Rate | 1e-4 |
| Effective Batch Size | 8 |
| Final Train Loss | 0.02411 |
| Final Eval Loss | 0.06386 |

### Before / After Comparison

#### Example 1 — Send Money SMS (`tp_001`)

**Prompt:**
```
Extract the M-Pesa transaction from the SMS below and return ONLY valid JSON.
The JSON MUST contain exactly these five fields: entity, amount, balance, date, type.

SMS:
QGH7K3M2P1 Confirmed. You have received Ksh20,000.00 from Ann Mueni 0712***456 on 05/03/2026 at 10:42 AM. New M-PESA balance is Ksh159,583.00. Transaction cost, Ksh0.00.

Return ONLY the JSON object.
```

**Base model (`google/gemma-3-270m-it`) — unmodified output:**
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
*Issues: Non-conforming schema, raw string amounts with currency prefixes, non-standard date format, conversational wrapper text.*

**Fine-tuned model (`sme-ledger-v2-Q4_K_M`) — output:**
```json
{"entity": "Ann Mueni", "amount": 20000.0, "balance": 159583.0, "date": "2026-03-05", "type": "income"}
```
*Correct: Strict 5-field JSON, numeric values, ISO date, zero conversational prose.*

---

#### Example 2 — Capability Query (`tp_002`)

**Prompt:**
```
How do you handle missing information or ambiguity in a financial transaction message, such as when the sender/merchant name, transaction date, balance, or transaction ID is omitted?
```

**Base model (`google/gemma-3-270m-it`) — unmodified output:**
```
If information is missing from a text message, I might try to guess or make assumptions based on the context, or prompt the user to provide the missing details manually.
```
*Issues: Generic, ungrounded response that suggests hallucinating missing data.*

**Fine-tuned model (`sme-ledger-v2-Q4_K_M`) — output:**
```
When a transaction message contains missing information or ambiguity, I follow deterministic schema extraction rules:
1. Optional missing fields (such as balance, fee, or reference) are set explicitly to null rather than hallucinating values.
2. If entity/merchant names are incomplete, I extract the available entity string without inventing unstated names.
3. For failed or unconfirmed transactions, I set type: "unknown" and set unconfirmed financial quantities to null.
4. The output strictly maintains valid JSON structure so downstream Python validation can inspect, log, or prompt for manual reconciliation.
```
*Correct: Grounded, precise explanation matching the operational constraints.*

### Proof-of-Training Artifacts

| File | Description |
|---|---|
| `provenance/adapter_config.json` | PEFT LoRA configuration |
| `provenance/adapter_model.safetensors` | LoRA adapter weights (~7.3 MB) |
| `provenance/training_script.py` | Training pipeline script |
| `provenance/convert_to_gguf.py` | GGUF conversion & quantization script |
| `provenance/training_logs_summary.csv` | Loss/LR metrics per step |
| `provenance/training_manifest.json` | Hardware & run metadata |
| `provenance/dataset_sample.jsonl` | 50-example representative dataset sample |
| `provenance/checksums.sha256` | SHA256 checksums |

---

## Constraints

- **Target Hardware:** 8 GB RAM, 4 vCPU, integrated GPU, Ubuntu 22.04
- **No GPU acceleration:** Pure CPU inference via `llama.cpp`
- **Offline operation:** Zero external network connectivity required during runtime
- **Data privacy:** Financial transaction messages remain strictly on-device

---

## Benchmarks

Development benchmarks measured using `llama.cpp` on the target development machine:

| Metric | Value |
|---|---|
| Machine | Intel Core i5 (4 Cores), 8 GB RAM, Ubuntu 22.04 |
| RAM at peak | ~320 MB |
| Time to first token | ~28 ms |
| Generation speed | ~82.4 t/s |
| Thermal throttling | None observed |

These are self-reported development benchmarks. Official scores are measured by the ADTC profiler on the standard evaluation machine.
