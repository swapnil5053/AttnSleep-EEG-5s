# Final Program Status Report: Capstone_AttnSleep

**Generated:** October 23, 2025  


---

# Executive Summary

This capstone project implements the AttnSleep architecture optimized for 5-second truncated epochs, achieving **72.57% accuracy** in sleep stage classification. The project demonstrates the trade-off between computational efficiency and classification performance while maintaining a clean and production-ready research codebase.

---

# 1. Project Cleanup Status

The repository has been cleaned and optimized to remove unnecessary experimental files and legacy data.

## Files Removed

- `results/Exp_30s/`
- `results/Exp_30s_15min/`
- `results/Exp_5s/`
- `results/Exp_true_5s/`
- `saved/`
- `AttnSleep/`
- `edf_true_30s_npz/`
- `data_processed/`
- `prepare_datasets/`
- Outdated configuration and utility scripts

## Files Retained

- `results/Exp_5s_best/`
- `edf_true_5s_npz/`
- Core project modules
- Training and configuration files
- Documentation files

---

# 2. Performance Results

| Metric | 5s Truncated Epoch | 30s Baseline Comparison |
|---|---|---|
| Accuracy | 72.57% | -8.33% |
| Weighted F1-Score | 72.97% | -2.53% |
| Cohen's Kappa (κ) | 0.6262 | -0.1248 |
| Training Time | 20 min 19 sec | 1.28× faster |
| Dataset Size / Memory | 16.7% of baseline | 83.3% reduction |
| Epoch Length | 5 seconds (500 samples) | 6× smaller |
| Cross-Validation | 10-fold | Equivalent |
| Total Epochs | 42,308 | Equivalent |

---

# 3. Project Structure

```text
Capstone_AttnSleep/
├── edf_true_5s_npz/
├── results/Exp_5s_best/
├── model/
│   ├── model.py
│   ├── loss.py
│   └── metric.py
├── trainer/
│   └── trainer.py
├── data_loader/
│   └── data_loaders.py
├── base/
│   └── base_trainer.py
├── logger/
│   ├── logger.py
│   └── logger_config.json
├── utils/
│   ├── util.py
│   └── r_permute_*.npy
├── imgs/
│   └── AttnSleep.png
├── train_Kfold_CV.py
├── config.json
├── parse_config.py
├── requirements.txt
├── README.md
├── LICENSE
└── PROJECT_STATUS_REPORT.md
```

---

# 4. Model Architecture & Training Configuration

## AttnSleep Adaptation

The original AttnSleep architecture was adapted for shorter EEG sequence processing using 5-second truncated epochs.

### Components

### Multi-scale Residual CNN (MRCNN)
Captures EEG features at multiple temporal scales.

### Attention Feature Refinement (AFR)
Improves feature selection using attention mechanisms.

### Temporal Context Encoder (TCE)
Processes temporal relationships within EEG sequences.

---

## Input Configuration

```python
(batch_size, 1, 500)
```

- 5-second EEG window
- 100 Hz sampling rate
- 500 samples per epoch

---

## Training Parameters

| Parameter | Value |
|---|---|
| Dataset | Sleep-EDF Database |
| Subjects | 39 |
| Cross Validation | 10-Fold |
| Epochs per Fold | 30 |
| Batch Size | 128 |
| GPU | NVIDIA GeForce RTX 4060 Laptop GPU |
| Framework | PyTorch 2.2.0 + CUDA 11.8 |

---

# 5. Objectives & Key Insights

## Primary Objective

Evaluate the trade-off between computational efficiency and sleep stage classification performance using shorter EEG windows.

---

## Key Insights

- Short EEG windows still retain meaningful sleep-stage information.
- Dataset size and computational requirements were significantly reduced.
- The model maintains acceptable classification performance despite aggressive truncation.
- The approach is suitable for edge devices, wearable systems, and resource-constrained environments.

---

# 6. Repository Status & Deployment

The repository is fully cleaned, documented, and ready for deployment.

## GitHub Push Commands

```bash
# Add remote repository
git remote add origin https://github.com/YOUR_USERNAME/Capstone_AttnSleep.git

# Rename branch
git branch -M main

# Push repository
git push -u origin main
```

---

# 7. Quality Assurance

## Codebase Verification

- All modules import successfully
- No syntax errors detected
- Training pipeline verified

---

## Environment Verification

Verified on:

- PyTorch 2.2.0+cu118
- CUDA-enabled GPU environment
- Python 3.x

---

## Dataset Verification

- All 39 dataset files verified
- File formatting validated
- Cross-validation pipeline functional

---

## Documentation

Completed documentation includes:

- `README.md`
- Project structure documentation
- Performance metrics documentation
- Final project report

---

# 8. Summary Statistics

| Category | Count |
|---|---|
| Total Files | 163 |
| Code Files | 15 |
| Data Files | 39 |
| Result Files | 109 |
| Documentation Files | 4 |

---
