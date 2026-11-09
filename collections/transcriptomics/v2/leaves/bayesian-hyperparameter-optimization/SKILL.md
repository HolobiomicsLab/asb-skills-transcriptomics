---
name: bayesian-hyperparameter-optimization
description: Use when base classifiers were built with pre-set, experience-derived
  hyperparameters and you need to systematically tune them to maximize k-fold cross-validation
  accuracy before stacking or comparison.
when_to_use_negative:
- Input is already an optimized or stacked model (e.g., the logistic-regression meta-model)
  — the skill targets the individual base classifiers only
- Goal is prediction on new sequences via the shipped m5C_predict.py script — that
  uses the fixed trained models, not re-tuning
- Working on the independent test dataset for final benchmarking — tuning must be
  driven by 10-fold cross-validation on the training set to avoid test-set leakage
tools:
- name: Staem5
  role: Source of the A. thaliana m5C training data, fused feature encodings, and
    base-classifier setup that the Bayesian optimization tunes
  repo: https://github.com/Cxd-626/Staem5.git
- name: scikit-learn
  role: Implements the five base classifiers and the 10-fold cross-validation objective
    that Bayesian optimization maximizes
- name: Bayesian optimization
  role: Search strategy over classifier hyperparameters (e.g., SVM C and gamma) selected
    by 10-fold cross-validation accuracy
- name: XGBoost / LightGBM
  role: Gradient-boosting base classifiers whose hyperparameters are optimized alongside
    SVM, GBDT, and ExtraTree
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:fc0b211a0c071f64bb7deea661faa3554564022b97409d92282cc7ae6a3b30e3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - Bayesian optimization
  - Staem5
  - XGBoost / LightGBM
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# bayesian-hyperparameter-optimization

## Summary

Bayesian optimization of hyperparameters for the five Staem5 base classifiers (SVM-RBF, GBDT, XGBoost, LightGBM, ExtraTree), selecting settings that maximize 10-fold cross-validation accuracy on the A. thaliana m5C dataset. It replaces experience-based pre-set hyperparameters with a systematic search, yielding the largest gain for SVM (62.40% -> 73.62%).

## When to use

Use when base classifiers were built with pre-set, experience-derived hyperparameters and you need to systematically tune them to maximize k-fold cross-validation accuracy before stacking or comparison.

## When NOT to use

- Input is already an optimized or stacked model (e.g., the logistic-regression meta-model) — the skill targets the individual base classifiers only
- Goal is prediction on new sequences via the shipped m5C_predict.py script — that uses the fixed trained models, not re-tuning
- Working on the independent test dataset for final benchmarking — tuning must be driven by 10-fold cross-validation on the training set to avoid test-set leakage

## Inputs

- A. thaliana m5C training dataset from the Staem5 repository (5,298 positive and 5,298 negative sequences) encoded with fused PSP, k-mer, PCPseDNC, and PseEIIP features
- Pre-optimized base classifier definitions with hyperparameters from supplementary Table S2 (SVM-RBF, GBDT, XGBoost, LightGBM, ExtraTree)

## Outputs

- Before/after 10-fold cross-validation accuracies for all five base classifiers (e.g., base_classifier_accuracies.csv)
- Optimized hyperparameter values per classifier (analogous to Table S2)
- Ranked list of classifiers by post-optimization gain for downstream base-classifier selection

## How to apply

Train each base classifier (SVM with radial basis kernel, GBDT, XGBoost, LightGBM, ExtraTree) in scikit-learn on the fused PSP, k-mer (k=1,2,3), PCPseDNC, and PseEIIP features and record baseline 10-fold cross-validation accuracy. Apply Bayesian optimization to tune parameters for each classifier — for the SVM this means searching the regularization parameter C and the kernel parameter gamma — choosing candidates that maximize 10-fold cross-validation accuracy rather than held-out metrics. Re-run 10-fold cross-validation with the optimized settings and tabulate before/after accuracies side by side. Use the improvement pattern as a selection signal: in Staem5, SVM improved from 62.40% to 73.62% and GBDT from 65.69% to 71.77%, with SVM showing the largest post-optimization gain, and SVM/XGBoost/LightGBM were then chosen as base classifiers for the stacked model. Record the selected parameters (as in the paper's supplementary Table S2) so the tuning is reproducible.

## Related tools

- **Staem5** (Source of the A. thaliana m5C training data, fused feature encodings, and base-classifier setup that the Bayesian optimization tunes) — https://github.com/Cxd-626/Staem5.git
- **scikit-learn** (Implements the five base classifiers and the 10-fold cross-validation objective that Bayesian optimization maximizes)
- **Bayesian optimization** (Search strategy over classifier hyperparameters (e.g., SVM C and gamma) selected by 10-fold cross-validation accuracy)
- **XGBoost / LightGBM** (Gradient-boosting base classifiers whose hyperparameters are optimized alongside SVM, GBDT, and ExtraTree)

## Examples

```
from skopt import BayesSearchCV; from sklearn.svm import SVC; opt = BayesSearchCV(SVC(kernel='rbf'), {'C': (1e-4, 1e4, 'log-uniform'), 'gamma': (1e-5, 1, 'log-uniform')}, cv=10, scoring='accuracy'); opt.fit(X_fused, y)
```

## Evaluation signals

- Optimized SVM 10-fold CV accuracy should reach ~73.62% (up from 62.40% pre-optimization)
- Optimized GBDT 10-fold CV accuracy should reach ~71.77% (up from 65.69%)
- All five base classifiers must show improved accuracy after optimization, with SVM having the largest gain
- SVM should end with the best 10-fold CV accuracy among the five base classifiers
- Optimized parameter values should be logged per classifier (equivalent to the paper's Table S2) so the run is reproducible

## Limitations

- The paper's pre-optimization hyperparameters were set from previous experience, not systematically derived, so the before/after comparison depends on a possibly suboptimal baseline
- Only parameter values (Table S2), detailed metrics (Table S1), and stacking results (Table S3) appear in supplementary files, not the main text
- Stacked Staem5 (73.89% accuracy, 0.479 MCC for A. thaliana) was still inferior to iRNA-m5C_SVM and m5CPred-SVM on the independent A. thaliana test set; optimizing base classifiers does not guarantee better independent-test performance
- No general cross-species model: tuning and models are species-specific, so re-optimization is required for other species

## Evidence

- [results] Bayesian optimization was applied to tune parameters for each classifier.: "Bayesian optimization was applied to tune parameters for each classifier."
- [discussion] the hyperparameters were pre-set according to previous experience: "the hyperparameters were pre-set according to previous experience28, 29, 30"
- [discussion] the selected parameters are listed in Table S2: "the selected parameters are listed in Table S2"
- [discussion] performance improvement of SVM was the largest among the five base classifiers: "the performance of the five base classifiers enhanced after parameter optimization and performance improvement of SVM was the largest among the five base classifiers"
- [readme] Python dependency versions for re-running the classifier stack: "numpy == 1.19.5
    pandas == 1.2.4
    sklearn == 0.21.3
    xgboost == 0.90
    lightgbm == 2.3.0"
