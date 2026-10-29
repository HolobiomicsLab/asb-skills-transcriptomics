---
name: imbalanced-class-bootstrapping
description: Use when Ψ site training data are heavily negative-skewed (e.g. 10:1
  non-Ψ to Ψ built from RMBase v2.0) and you need bin-balanced training subsets with
  metrics averaged over bootstrap iterations.
when_to_use_negative:
- Data are already balanced 1:1 between Ψ and non-Ψ samples — binning negatives adds
  no information and only increases compute.
- You need a single estimate on the full untouched imbalanced test distribution; the
  paper treats bin-balanced (balanced dataset) and full-set (imbalanced dataset) results
  as separate regimes, not substitutes.
- Negatives are too scarce to fill even one positive-class-sized bin, leaving bootstrap
  iterations without adequate negative diversity.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: MU-PseUDeep
  role: Dual-input CNN whose training/evaluation loop bins negatives into positive-class-sized
    sets and averages metrics per iteration
  repo: https://github.com/smk5g5/MU-PseUDeep
- name: Keras
  role: Framework fitting the SGD-optimized (lr 0.0137) CNN on each bootstrap bin
    under the fixed training protocol
- name: Hyperas
  role: Hyperparameter optimization per fold before bootstrapped training, maximizing
    validation F1 with a TPE search
  repo: https://github.com/maxpumperla/hyperas
- name: Hyperopt
  role: TPE search engine underlying the Hyperas optimization used on the bootstrapped
    folds
  repo: https://github.com/hyperopt/hyperopt
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:1c19c416a89cb3f661d80b248e73140b8e45b7aa60b46c31d14d16a3f9a0e8b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/smk5g5/MU-PseUDeep
  tools:
  - Hyperas
  - Hyperopt
  - Keras
  - MU-PseUDeep
  - PseUI
  - TensorFlow
  - XG-PseU
  - iRNA-PseU
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# imbalanced-class-bootstrapping

## Summary

A resampling strategy from MU-PseUDeep for Ψ-site CNN training on negative-skewed data: negative samples are divided into bins each sized to the positive class, each bin is trained with the full positive set, and metrics are averaged across bootstrap iterations and the 10 stratified folds.

## When to use

Use when Ψ site training data are heavily negative-skewed (e.g. 10:1 non-Ψ to Ψ built from RMBase v2.0) and you need bin-balanced training subsets with metrics averaged over bootstrap iterations.

## When NOT to use

- Data are already balanced 1:1 between Ψ and non-Ψ samples — binning negatives adds no information and only increases compute.
- You need a single estimate on the full untouched imbalanced test distribution; the paper treats bin-balanced (balanced dataset) and full-set (imbalanced dataset) results as separate regimes, not substitutes.
- Negatives are too scarce to fill even one positive-class-sized bin, leaving bootstrap iterations without adequate negative diversity.

## Inputs

- 10-fold stratified train/test splits of 51-nt Ψ-centered fragments (Ψ plus 25 flanking bases) and non-Ψ negatives built at a 10:1 negative-to-positive ratio from RMBase v2.0 (human/mouse/yeast)
- One-of-K encoded sequence matrices (51×4) and secondary-structure context matrices (51×6) with binary Ψ/non-Ψ labels

## Outputs

- Per-bootstrap-iteration metric values (accuracy, F1, MCC, sensitivity, specificity) on the held-out test folds
- Fold- and iteration-averaged performance estimates with standard deviations for both balanced and imbalanced regimes (e.g. human balanced accuracy 0.726 ± 0.020, imbalanced accuracy 0.894 ± 0.011)

## How to apply

First establish the 10-fold stratified 80:20 train/test split, with cd-hit-est-2d (identity 0.8, word length 4) reducing train/test identity and Needleman-Wunsch alignment removing training sequences >60% identical to the test set. Then divide the negative samples into N bins each sized to the positive class, so every bootstrap iteration pairs the full Ψ-positive set with one balanced non-Ψ bin drawn from regions lacking validated Ψ sites. Train the CNN on each bin under the fixed protocol — SGD at learning rate 0.0137, binary cross-entropy loss, batch size 32, up to 500 epochs with early-stopping patience of 20 — with hyperparameters already tuned once per fold via Hyperas/Hyperopt TPE maximizing validation F1, so the optimization is not confounded by class imbalance. Evaluate accuracy, F1, MCC, sensitivity, and specificity per iteration, then average across bootstrap iterations and folds to report mean ± standard deviation for both the balanced and imbalanced regimes (human: 0.726 ± 0.020 vs 0.894 ± 0.011 accuracy). Run baselines (iRNA-PseU, PseUI, XG-PseU) on identical train/test splits so comparisons reflect the model rather than the resampling.

## Related tools

- **MU-PseUDeep** (Dual-input CNN whose training/evaluation loop bins negatives into positive-class-sized sets and averages metrics per iteration) — https://github.com/smk5g5/MU-PseUDeep
- **Keras** (Framework fitting the SGD-optimized (lr 0.0137) CNN on each bootstrap bin under the fixed training protocol)
- **Hyperas** (Hyperparameter optimization per fold before bootstrapped training, maximizing validation F1 with a TPE search) — https://github.com/maxpumperla/hyperas
- **Hyperopt** (TPE search engine underlying the Hyperas optimization used on the bootstrapped folds) — https://github.com/hyperopt/hyperopt

## Examples

```
neg_bins = np.array_split(negatives, len(negatives) // len(positives))  # bins sized to the positive class; average metrics across bins
```

## Evaluation signals

- Each bootstrap bin contains exactly n_positives negatives, and the full positive set is used in every iteration.
- Reported metrics are mean ± standard deviation across bootstrap iterations and the 10 stratified folds, not single-run values.
- Human balanced-regime results fall near accuracy 0.726 ± 0.020, F1 0.745 ± 0.041, MCC 0.524 ± 0.043, sensitivity 0.709 ± 0.061, specificity 0.810 ± 0.0203 (Table 1); imbalanced accuracy near 0.894 ± 0.011.
- On the balanced dataset the dual-input model improves over the sequence-only CNN ablation by ~3–4% accuracy/F1 and up to 9% sensitivity, confirming the resampling scheme discriminates correctly.
- Identical train/test splits are passed to iRNA-PseU, PseUI, and XG-PseU baselines as used for the bootstrapped folds.

## Limitations

- iPseU-CNN has no source code or webserver available, so the sequence-only CNN ablation is the only comparable deep baseline; a direct comparison is impossible.
- The public MU-PseUDeep README documents no code for the binning loop — the bootstrap procedure must be re-implemented from the paper's methods description.
- Bin-balanced averages can diverge from full-imbalanced-set metrics (human balanced accuracy 0.726 ± 0.020 vs imbalanced 0.894 ± 0.011) and must be reported as separate regimes.

## Evidence

- [methods] To create the negative dataset, we collected those regions of RNA that did not contain any experimentally validated Ψ sites: "To create the negative dataset, we collected those regions of RNA that did not contain any experimentally validated Ψ sites"
- [methods] We did a 10-fold stratified split of positive and negative RNA samples into an 80:20 ratio for training and testing data: "We did a 10-fold stratified split of positive and negative RNA samples into an 80:20 ratio for training and testing data"
- [methods] apply bootstrapping by binning the negative samples into bins sized to the positive class and averaging results across iterations and folds: [evidence span withheld — anchor status: not_found]
- [results] The improvement was by 3–4% for accuracy and F1 and up to 9% for sensitivity in the balanced dataset in comparison to sequence CNN: "The improvement was by 3–4% for accuracy and F1 and up to 9% for sensitivity in the balanced dataset in comparison to sequence CNN"
- [results] On balanced human data, MU-PseUDeep achieved accuracy 0.726 ± 0.020, F1 0.745 ± 0.041, MCC 0.524 ± 0.043, sensitivity 0.709 ± 0.061, and specificity 0.810 ± 0.0203 (Table 1).: [evidence span withheld — anchor status: not_found]
