---
name: cnn-model-performance-evaluation
description: Use when benchmarking a deep learning classifier on RNA modification-site
  prediction data across balanced and imbalanced regimes and you need per-fold averaged
  metrics comparable to published baselines.
when_to_use_negative:
- Data has not been split with sequence-identity reduction (cd-hit-est-2d threshold
  0.8 plus >60% Needleman-Wunsch filtering) — metrics would be inflated by train-test
  leakage
- The task is transcriptome-wide Ψ-site scanning at a >0.99 precision threshold rather
  than model performance benchmarking
- A direct comparison target like iPseU-CNN is needed but no source code or webserver
  is available for that method
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: MU-PseUDeep
  role: Deep learning CNN classifier whose performance is evaluated across species
    and balanced/imbalanced regimes
  repo: https://github.com/smk5g5/MU-PseUDeep
- name: XG-PseU
  role: Baseline Ψ-site prediction method run on identical train/test splits
- name: PseUI
  role: Baseline Ψ-site prediction method for comparison on balanced and imbalanced
    datasets
- name: iRNA-PseU
  role: Baseline SVM-based Ψ-site prediction method for comparison
- name: Hyperas
  role: Hyperparameter optimization wrapper using tree-structured Parzen estimator
    maximizing F1 on validation data
  repo: https://github.com/maxpumperla/hyperas
- name: Hyperopt
  role: Underlying hyperparameter search library used via Hyperas
  repo: https://github.com/hyperopt/hyperopt
- name: EDeN
  role: Converts RNAshapes dot-bracket notation into six sub-shape contexts (S, M,
    H, I, F, T) for structural encoding
  repo: https://github.com/fabriziocosta/EDeN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:3aac83a980ad804893da047e2fcd44073afa9b2b361f1f8cc7786559005c3661
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smk5g5/MU-PseUDeep
  tools:
  - EDeN
  - Hyperas
  - Hyperopt
  - MU-PseUDeep
  - PseUI
  - XG-PseU
  - iRNA-PseU
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# cnn-model-performance-evaluation

## Summary

Evaluates a 1D CNN deep learning classifier (as in MU-PseUDeep, which combines sequence 51×4 and secondary-structure 51×6 one-of-K inputs) on 10-fold stratified 80:20 splits of balanced and imbalanced RNA Ψ-site datasets, reporting accuracy, F1, MCC, sensitivity and specificity against baseline methods (iRNA-PseU, PseUI, XG-PseU).

## When to use

Use when benchmarking a deep learning classifier on RNA modification-site prediction data across balanced and imbalanced regimes and you need per-fold averaged metrics comparable to published baselines.

## When NOT to use

- Data has not been split with sequence-identity reduction (cd-hit-est-2d threshold 0.8 plus >60% Needleman-Wunsch filtering) — metrics would be inflated by train-test leakage
- The task is transcriptome-wide Ψ-site scanning at a >0.99 precision threshold rather than model performance benchmarking
- A direct comparison target like iPseU-CNN is needed but no source code or webserver is available for that method

## Inputs

- Positive Ψ-site 51-mer sequences (25 nt flanking each side) for human, mouse, yeast from RMBase v2.0
- Negative-set sequences from RNA regions lacking validated Ψ sites (10:1 negative-to-positive ratio for imbalanced evaluation)
- One-of-K encoded sequence matrices (51×4) and secondary-structure matrices (51×6) from RNAshapes/EDeN
- Baseline tool outputs on identical train/test splits (iRNA-PseU, PseUI, XG-PseU)

## Outputs

- Per-fold accuracy, F1-score and MCC averaged over bootstrap iterations and folds (e.g. human balanced: accuracy 0.726 ± 0.020, F1 0.745 ± 0.041, MCC 0.524 ± 0.043)
- Sensitivity and specificity per dataset and regime
- Comparison table of MU-PseUDeep vs PseUI, iRNA-PseU and XG-PseU on balanced and imbalanced data

## How to apply

Perform a 10-fold stratified 80:20 train/test split with pandas/Scikit-learn, then reduce train-test sequence identity with cd-hit-est-2d (threshold 0.8, word length 4), remove training sequences with >60% Needleman-Wunsch global-alignment identity to the test set, and cluster the test set internally with cd-hit-est at the same threshold to avoid leakage. Train the Keras CNN on paired one-of-K encoded inputs (sequence 51×4, secondary structure 51×6 from RNAshapes/EDeN sub-shape contexts S, M, H, I, F, T) using SGD (learning rate 0.0137), binary cross-entropy, batch size 32, 500 epochs with early stopping patience of 20 and model checkpointing. Optimize dropout rates, kernel regularizer weights, optimizer and learning rate with Hyperas/Hyperopt tree-structured Parzen estimator maximizing F1 on validation data for a single fold. For imbalanced data, bin negative samples into bins sized to the positive class (bootstrap) and average metric values over iterations and folds. Compute accuracy, F1 and MCC per fold, plus sensitivity and specificity, and run iRNA-PseU, PseUI and XG-PseU on identical splits for baseline comparison.

## Related tools

- **MU-PseUDeep** (Deep learning CNN classifier whose performance is evaluated across species and balanced/imbalanced regimes) — https://github.com/smk5g5/MU-PseUDeep
- **XG-PseU** (Baseline Ψ-site prediction method run on identical train/test splits)
- **PseUI** (Baseline Ψ-site prediction method for comparison on balanced and imbalanced datasets)
- **iRNA-PseU** (Baseline SVM-based Ψ-site prediction method for comparison)
- **Hyperas** (Hyperparameter optimization wrapper using tree-structured Parzen estimator maximizing F1 on validation data) — https://github.com/maxpumperla/hyperas
- **Hyperopt** (Underlying hyperparameter search library used via Hyperas) — https://github.com/hyperopt/hyperopt
- **EDeN** (Converts RNAshapes dot-bracket notation into six sub-shape contexts (S, M, H, I, F, T) for structural encoding) — https://github.com/fabriziocosta/EDeN

## Examples

```
cd-hit-est-2d -i train.fa -j test.fa -o train_nr.fa -c 0.8 -n 4; python train_mu_pseudeep.py --epochs 500 --batch_size 32 --lr 0.0137 --patience 20
```

## Evaluation signals

- Reproduced human balanced accuracy should match 0.726 ± 0.020 and imbalanced accuracy 0.894 ± 0.011 (Table 1)
- MU-PseUDeep should outperform iRNA-PseU, PseUI and XG-PseU on both balanced and imbalanced datasets
- Combined sequence+structure model should improve over sequence-only CNN by 3–4% accuracy/F1 and up to 9% sensitivity on balanced data, and ~2% accuracy, 2% F1, up to 4% MCC, up to ~7% sensitivity on imbalanced data
- cd-hit-est-2d at identity threshold 0.8 with word length 4 must remove train-test overlap before split evaluation
- Per-fold metrics should be averaged over bootstrap iterations with negative bins sized to the positive class

## Limitations

- No source code or webserver was available for iPseU-CNN, so a direct comparison with that deep learning approach is impossible
- Hyperparameter optimization was performed on a single fold's validation F1, not across all folds
- Negative datasets are constructed from regions lacking experimentally validated Ψ sites, so unannotated true positives may bias specificity estimates

## Evidence

- [methods] Balanced split metrics: "We did a 10-fold stratified split of positive and negative RNA samples into an 80:20 ratio for training and testing data"
- [methods] Identity filtering: "using cd-hit-est-2d with a minimum sequence identity threshold (0.8)"
- [methods] Needleman-Wunsch filter: "removed the sequences from the training set that had >60% sequence identity with the test set"
- [methods] Encoding: "Sequence data was converted into a one-of-K encoding binary matrix of size 51 × 4"
- [results] Table 1 result: [evidence span withheld — anchor status: not_found]
- [results] Structure improvement: "The improvement was by 3–4% for accuracy and F1 and up to 9% for sensitivity in the balanced dataset in comparison to sequence CNN"
