# AttnSleep: 5-Second Truncated Epoch Sleep Classification

A deep learning model for classifying sleep stages from EEG signals. The standard approach uses 30-second epochs, but I tested whether using just the first 5 seconds would be fast enough while maintaining reasonable accuracy.

**TL;DR:** 72.57% accuracy on 5-second windows. That's 8% worse than 30-second windows, but training is 1.28x faster and uses 83% less data. Worth it for resource-constrained systems.

## Results

| Metric | 5s Epochs | 30s Epochs (baseline) |
|--------|-----------|----------------------|
| Accuracy | 72.57% | 80.9% |
| F1-Score | 72.97% | 75.5% |
| Cohen's Kappa | 0.6262 | 0.7088 |
| Training Time | 20 min | 25.6 min |
| Dataset Size | 500 samples | 3000 samples |

The accuracy drop is predictable—less signal means harder to classify. But the speed and size benefits are real.

## Quick Start

```bash
pip install -r requirements.txt
python train_Kfold_CV.py --config config.json --fold_id 0 --device 0 --np_data_dir edf_true_5s_npz
```

Results go to `results/Exp_5s_best/`.

## The Approach

I took the AttnSleep architecture (multi-scale CNN + attention + temporal encoder) and retrained it on 5-second windows instead of 30-second ones. The network adapts to shorter sequences, and I evaluated it with 10-fold cross-validation on the Sleep-EDF database (39 subjects).

The main changes were:
- Input size: 500 samples instead of 3000
- Attention mechanism recalibrated for shorter sequences
- Everything else stayed the same

## Architecture

AttnSleep uses three components:
- **Multi-scale Residual CNN**: Captures patterns at different time scales
- **Attention Feature Refinement**: Learns which parts of the EEG matter for sleep staging
- **Temporal Context Encoder**: Models relationships between time steps

For 5-second windows, I adjusted the attention and CNN layer sizes to work with the smaller input.

## Why This Matters

Sleep staging with wearable devices or edge hardware is limited by compute. If you can get decent accuracy in 5 seconds instead of 30, that opens up applications on phones or smartwatches. The trade-off here is clear: you lose 8% accuracy but gain massive speed and storage savings.

## Setup

```
Capstone_AttnSleep/
├── edf_true_5s_npz/       # 5-second epoch dataset
├── results/Exp_5s_best/   # Output metrics and confusion matrices
├── model/                 # Neural network code
├── trainer/               # Training loop
├── data_loader/           # Dataset handling
├── train_Kfold_CV.py      # Main script
├── config.json            # Hyperparameters
└── requirements.txt
```

## Configuration

Edit `config.json` for:
- Batch size (default 128)
- Learning rate
- Number of training epochs (30)
- Fold count (10-fold CV)

## Testing

The model was tested on Sleep-EDF with 10-fold cross-validation. Each fold trains for 30 epochs on an RTX 4060. Training time is logged per fold.

## Key Findings

**It works.** 72.57% accuracy on 5-second windows is reasonable for a resource-constrained system. You're losing classification power compared to 30-second windows, but you gain:
- 83% smaller dataset
- 28% faster training
- Viable for real-time or edge deployment

**The accuracy drop isn't surprising.** Half the signal data means the model has less to work with. But the attention mechanism helps—it learns which parts of those 5 seconds matter most.

**Next steps** would be testing on other sleep databases, trying hybrid approaches (mix 5s and 30s epochs), or optimizing for actual mobile/wearable deployment.

## Requirements

- Python 3.7+
- PyTorch 2.2.0+
- NumPy, scikit-learn, Pandas, OpenPyXL

## Citation

Based on the AttnSleep architecture. If you build on this, cite the original AttnSleep paper and mention the 5-second truncation variant.

---

**Status:** Complete  
**Accuracy:** 72.57% on 5-second epochs  
**Best for:** Resource-constrained environments where speed matters more than peak accuracy
