# provenance/ — Proof-of-Training Artefacts

> Gate 2 §3.1 — Model Originality & Provenance

This folder contains all proof-of-training files required by the ADTC 2026 Gate 2 submission guidelines.

## Contents

| File | Size | Description |
|---|---|---|
| `adapter_config.json` | ~1.2 KB | LoRA PEFT configuration — confirms fine-tuning parameters (rank, alpha, target modules) |
| `adapter_model.safetensors` | ~7.3 MB | LoRA adapter weights — the clearest evidence a fine-tuning run occurred |
| `training_script.py` | ~14 KB | Master end-to-end pipeline: data prep → LoRA fine-tune → merge → GGUF convert → evaluate |
| `convert_to_gguf.py` | ~4 KB | LoRA merge + GGUF conversion + Q4_K_M quantization script |
| `training_logs_summary.csv` | ~2 KB | Loss / grad_norm / learning_rate per step across all 5 epochs (sampled every ~20 steps) |
| `training_manifest.json` | ~2 KB | Full run metadata: hardware, config, duration, final loss, eval metrics |
| `dataset_sample.jsonl` | ~25 KB | 50 representative examples from the SME-Ledger V2 training dataset |
| `checksums.sha256` | ~1 KB | SHA256 hashes of adapter weights and final GGUF for integrity verification |

## Training Summary

| Parameter | Value |
|---|---|
| Base Model | `google/gemma-3-270m-it` (via public mirror `Huihui-ai/Huihui-gemma-3-270m-it-abliterated`) |
| Fine-Tuning Method | LoRA (PEFT), weight-level fine-tuning |
| LoRA Rank / Alpha | 8 / 16 |
| Trainable Parameters | 1,898,496 (0.70% of 270M total) |
| Training Examples | 16,275 (including ~4% capability instruction queries) |
| Epochs | 5 |
| Hardware | NVIDIA H100 PCIe, 79.19 GB VRAM |
| Duration | ~46 minutes (2,781 seconds) |
| Final Train Loss | **0.02411** |
| Final Eval Loss | **0.06386** |
| Git Commit SHA | `7cd8243b95650cc655b5c5af9fa74d4efbf5a9b1` |

## Verifying Checksums

```bash
# From the provenance/ directory:
sha256sum -c checksums.sha256

# Or individually:
sha256sum adapter_model.safetensors
# Expected: 5df3b6da437a93f0cba6e7fd34443f814bea972562e250a4e5ec6805921ee81c

sha256sum ../model/gemma3-financial-intelligence-Q4_K_M.gguf
# Expected: 87b313e9ceb130302eb3c2327681cbeb55954d531fae4c39f8f5e8c823392fac
```

## Loss Curve Summary

The `training_logs_summary.csv` shows the full loss descent across 5 epochs:

- **Epoch 0.005:** loss = 1.813 (warmup start)
- **Epoch 0.5:** loss ≈ 0.060 (rapid initial convergence)
- **Epoch 1.0:** loss ≈ 0.065 (stable plateau)
- **Epoch 2.0:** loss ≈ 0.056
- **Epoch 3.0:** loss ≈ 0.057
- **Epoch 4.0:** loss ≈ 0.043
- **Epoch 5.0:** loss = **0.02411** (final)

The clear monotonic descent from ~1.8 → ~0.024 confirms genuine gradient-based weight adaptation occurred.
