# AttnSleep: 5-Second Truncated Epoch Optimization

A deep learning research project investigating the feasibility of using 5-second truncated EEG epochs for sleep stage classification to reduce computational overhead while maintaining model performance.

## Research Objective

**Can 5-second truncated EEG epochs maintain sleep stage classification performance while significantly reducing computational requirements?**

## Methodology

### Data Processing
* **Epoch Truncation:** Extracted the first 5 seconds from each standard 30-second epoch to maintain the temporal structure while aggressively reducing data volume.
* **Dataset Optimization:** Achieved an 83.3% reduction in dataset size while preserving the exact same number of training epochs as the original data.

### Technical Implementation
* **Model Adaptation:** Modified the baseline AttnSleep architecture to process 500-sample sequences.
* **Attention Mechanisms:** Adjusted spatial and temporal attention layers to accommodate shorter input sequences.
* **Validation Strategy:** Implemented robust 10-fold cross-validation to ensure reliable performance comparisons between 5-second and 30-second epochs.

## Results

The study successfully demonstrated a significant reduction in computational resource requirements with a measured, acceptable trade-off in overall classification accuracy.

### Performance vs. Efficiency Trade-off

| Metric | 30s Epoch (Baseline) | 5s Epoch (Proposed) | Impact |
|---|---|---|---|
| **Accuracy** | 80.90% | 72.57% | -8.33% |
| **F1-Score** | 75.50% | 72.97% | -2.53% |
| **Cohen's Kappa** | 0.751 | 0.626 | -0.125 |
| **Training Time** | 25 min 59 sec | 20 min 19 sec | 1.28x faster |
| **Memory / Data Size** | 100% baseline | 16.7% of baseline | 83.3% reduction |

## Practical Applications

This truncation strategy makes advanced sleep classification more viable for real-world deployments:
* **Edge Computing:** Lowers the memory and processing threshold, making deployment feasible for portable smartphones and wearable devices.
* **Clinical Environments:** Enables faster processing times for medical software requiring rapid, resource-efficient sleep stage analysis. 

## Future Work

* **Hybrid Approaches:** Investigate architectures that combine both 5s and 30s epochs to balance deep context with processing speed.
* **Real-time Implementation:** Adapt the optimized model for continuous, streaming classification systems.
* **Cross-dataset Validation:** Test model generalization across different clinical sleep databases.
* **Architectural Exploration:** Evaluate alternative attention mechanisms specifically tuned for highly truncated time-series data.

---

**Repository:** [github.com/swapnil5053/Capstone_AttnSleep](https://github.com/swapnil5053/Capstone_AttnSleep)
