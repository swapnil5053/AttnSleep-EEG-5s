# Capstone_AttnSleep

![Python](https://img.shields.io/badge/Python-3.7%2B-blue?logo=python&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-2.2.0-EE4C2C?logo=pytorch&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)
![Dataset](https://img.shields.io/badge/Dataset-Sleep--EDF%2039%20subjects-lightgrey)
![CV](https://img.shields.io/badge/Cross--Validation-10--Fold-orange)

**Automatic sleep stage classification from single-channel EEG using truncated 5-second epochs.**

This project adapts [AttnSleep](https://github.com/emadeldeen24/AttnSleep) (Eldele et al., 2021) for sleep stage classification using truncated 5-second EEG epochs instead of the standard 30-second windows.

The goal was to evaluate whether meaningful sleep staging performance could still be achieved with drastically shorter inputs for low-power or resource-constrained applications such as wearables and edge devices.

Results on Sleep-EDF (39 subjects, 10-fold cross-validation):

- **72.57% accuracy** using 5-second EEG epochs
- **83.3% reduction** in input data per epoch
- **1.28× faster** training compared to the original 30-second setup

---

## Table of Contents

- [Background](#background)
- [Results](#results)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Setup](#setup)
- [Dataset](#dataset)
- [Training](#training)
- [Configuration](#configuration)
- [Outputs](#outputs)
- [SHHS Dataset Support](#shhs-dataset-support)
- [What I Changed](#what-i-changed)
- [Key Findings](#key-findings)
- [Future Work](#future-work)
- [Citation](#citation)

---

## Background

Standard polysomnography (PSG) performs sleep staging using 30-second EEG epochs following AASM guidelines. This project investigates whether shorter temporal windows can still preserve enough discriminative information for meaningful classification.

The original AttnSleep model expects:

```text
(batch, 1, 3000)
```

corresponding to 30 seconds of EEG sampled at 100 Hz.

This project modifies the pipeline to operate on:

```text
(batch, 1, 500)
```

corresponding to only the first 5 seconds of each epoch.

Evaluation was performed on the Sleep-EDF Cassette subset (39 subjects) using 10-fold cross-validation.

---

## Results

### Performance vs Original AttnSleep

| Metric | 5s Epochs (This Work) | 30s Epochs (Original) | Difference |
|---|---|---|---|
| Accuracy | **72.57%** | 80.9% | −8.33% |
| Weighted F1-Score | **72.97%** | 75.5% | −2.53% |
| Cohen's Kappa (κ) | **0.6262** | 0.751 | −0.1248 |

### Efficiency Gains

| Metric | 5s Epochs | 30s Epochs | Improvement |
|---|---|---|---|
| Total Training Time | **20 min 19 sec** | 25 min 59 sec | **1.28× faster** |
| Avg. Time per Epoch | **4.07 sec** | 6.24 sec | — |
| Dataset Size | **16.7%** of original | 100% | **83.3% reduction** |
| Memory Footprint | **16.7%** of original | 100% | **83.3% reduction** |

---

## Architecture

The model is based on the original AttnSleep architecture:

- Multi-scale residual CNN (MRCNN)
- Adaptive feature recalibration (AFR)
- Temporal context encoder (TCE) with multi-head self-attention

The original network was designed for 30-second EEG epochs (`3000` samples at 100 Hz). This project adapts the pipeline for 5-second windows (`500` samples) while keeping the downstream transformer layers unchanged.

The main architectural modification was adding:

```python
AdaptiveAvgPool1d(80)
```

inside the `MRCNN` output pipeline so the temporal dimension remains compatible with the original TCE block.

### Model Hyperparameters

| Parameter | Value |
|---|---|
| `d_model` | 80 |
| `d_ff` | 120 |
| `h` | 5 |
| `N` | 2 |
| `afr_reduced_cnn_size` | 30 |
| `num_classes` | 5 |
| `dropout` | 0.1 |

---

## Project Structure

```text
Capstone_AttnSleep/
│
├── model/              # AttnSleep architecture and attention modules
├── trainer/            # Training and validation loops
├── data_loader/        # EEG dataset loading utilities
├── utils/              # Metrics, folds, preprocessing helpers
├── results/            # Saved checkpoints and evaluation outputs
├── imgs/               # Figures and visualisations
│
├── train_Kfold_CV.py
├── config.json
├── requirements.txt
├── README.md
└── LICENSE
```

---

## Setup

### Requirements

- Python 3.7+
- CUDA-capable GPU recommended

### Installation

```bash
git clone https://github.com/swapnil5053/Capstone_AttnSleep.git
cd Capstone_AttnSleep
pip install -r requirements.txt
```

### Dependencies

```text
torch >= 1.4.0
numpy
scikit-learn
pandas
openpyxl
mne >= 0.20.7
```

Tested on:
- NVIDIA GeForce RTX 4060 Laptop GPU
- PyTorch 2.2.0 + CUDA 11.8

---

## Dataset

### Sleep-EDF Cassette Subset

Source: https://physionet.org/content/sleep-edfx/1.0.0/

| Property | Value |
|---|---|
| Subjects | 39 |
| EEG Channel | Fpz-Cz |
| Sampling Rate | 100 Hz |
| Original Epoch Length | 30 seconds |
| Truncated Epoch Length | 5 seconds |
| Total Epochs | 42,308 |
| Sleep Stages | Wake, N1, N2, N3, REM |

The preprocessing pipeline truncates each standard 30-second epoch to its first 500 samples while preserving the original label.

Each `.npz` file contains:

- `x` → EEG signal windows
- `y` → integer sleep stage labels

---

## Training

### Run a Single Fold

```bash
python train_Kfold_CV.py \
  --config config.json \
  --fold_id 0 \
  --device 0 \
  --np_data_dir edf_true_5s_npz
```

### Run All 10 Folds

```bash
for i in {0..9}; do
  python train_Kfold_CV.py \
    --config config.json \
    --fold_id $i \
    --device 0 \
    --np_data_dir edf_true_5s_npz
done
```

### Resume from Checkpoint

```bash
python train_Kfold_CV.py \
  --resume path/to/model_best.pth \
  --fold_id 0 \
  --device 0 \
  --np_data_dir edf_true_5s_npz
```

---

## Configuration

Example training configuration:

```json
{
  "name": "Exp_5s_best",
  "n_gpu": 1,
  "arch": {
    "type": "AttnSleep",
    "args": {}
  },
  "data_loader": {
    "args": {
      "batch_size": 128,
      "num_folds": 10
    }
  },
  "optimizer": {
    "type": "Adam",
    "args": {
      "lr": 0.001,
      "weight_decay": 0.001,
      "amsgrad": true
    }
  },
  "loss": "weighted_CrossEntropyLoss",
  "metrics": ["accuracy"],
  "trainer": {
    "epochs": 30,
    "save_dir": "results/",
    "save_period": 10,
    "verbosity": 2,
    "monitor": "min val_loss"
  }
}
```

### Key Parameters

| Parameter | Value |
|---|---|
| Batch Size | 128 |
| CV Folds | 10 |
| Optimizer | Adam |
| Initial LR | 0.001 |
| Epochs | 30 |
| Save Period | Every 10 epochs |

---

## Outputs

Each fold generates:

| File | Description |
|---|---|
| `model_best.pth` | Best checkpoint |
| `outs_{N}.npy` | Predicted labels |
| `trgs_{N}.npy` | Ground-truth labels |
| `training_time_fold_{N}.txt` | Timing statistics |
| `config.json` | Saved config |
| `info.log` | Training logs |

After the final fold, results are aggregated into:

| File | Description |
|---|---|
| `classification_report.xlsx` | Precision, recall, F1 metrics |
| `confusion_matrix.torch` | Saved confusion matrix |
| `training_time_summary.txt` | Fold timing summary |

---

## SHHS Dataset Support

The repository also supports the SHHS (Sleep Heart Health Study) dataset.

To use SHHS:

1. Place SHHS `.npz` files in a directory containing `"shhs"` in its name
2. The training script automatically switches to the SHHS fold loader
3. Replace `MRCNN` with `MRCNN_SHHS` in `model.py`
4. Change `d_model` from `80` to `100`

---

## What I Changed

### Data Pipeline

- Truncated each 30-second EEG epoch to its first 5 seconds (500 samples)
- Preserved the original label structure and preprocessing format

### Model Changes

- Added `AdaptiveAvgPool1d(80)` inside `MRCNN.forward()`
- Added `adapt_pool` and `target_len = 80` inside `MRCNN.__init__`

These changes allow the shorter input sequence to remain compatible with the original transformer encoder without modifying the TCE architecture itself.

### Training Infrastructure

- Added per-fold training time tracking
- Added aggregate timing summaries across folds
- Added per-epoch timing logs

Everything else — including the attention mechanism, loss function, cross-validation pipeline, logging system, and overall architecture — remains from the original AttnSleep implementation.

---

## Key Findings

- Shorter EEG windows still preserve meaningful sleep-stage information
- The attention-based architecture remains effective despite aggressive temporal truncation
- Training becomes faster and more memory-efficient with reduced input size
- N1 remains the most difficult stage to classify due to overlap with wake and REM transitions

---

## Future Work

- Cross-dataset validation on Sleep-EDF-78 and SHHS
- Per-class performance analysis
- Hybrid adaptive epoch lengths
- ONNX export and mobile benchmarking
- Alternative truncation strategies beyond fixed first-5-second cropping

---

## Citation

```bibtex
@article{eldele2021attention,
  title={An Attention-Based Deep Learning Approach for Sleep Stage Classification With Single-Channel EEG},
  author={Eldele, Emadeldeen and Chen, Zhenghua and Liu, Chengyu and Wu, Min and Kwoh, Chee-Keong and Li, Xiaoli and Guan, Cuntai},
  journal={IEEE Transactions on Neural Systems and Rehabilitation Engineering},
  volume={29},
  pages={809--820},
  year={2021},
  publisher={IEEE}
}
```
