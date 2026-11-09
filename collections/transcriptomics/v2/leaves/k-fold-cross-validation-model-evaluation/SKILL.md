---
name: k-fold-cross-validation-model-evaluation
description: Use when comparing classifier candidates (SVM, XGBoost, LightGBM, ExtraTree,
  GBDT) or stacking-vs-voting ensembles on balanced RNA modification training data
  to select the optimal model before independent testing.
when_to_use_negative:
- The input is an independent test set (e.g., Test1157, Met1320, Met1900) — cross-validation
  is only for model selection on training data, and Staem5 was shown to be inferior
  to iRNA-m5C_SVM and m5CPred-SVM on the independent A. thaliana test set
- The goal is cross-species prediction — the article notes a general cross-species
  model is difficult and species-specific models are required, so a single pooled
  cross-validation model is the wrong choice
- The feature matrix is not fused/reduced (no PSP, k-mer, PCPseDNC, PseEIIP fusion
  and no F score selection) — results will not match the published cross-validation
  figures
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: Staem5
  role: Stacked ensemble framework whose base classifiers and meta-model are evaluated
    via 10-fold cross-validation
  repo: https://github.com/Cxd-626/Staem5.git
- name: scikit-learn
  role: Trains base classifiers (SVM with RBF kernel, GBDT, XGBoost, LightGBM, ExtraTree)
    and generates out-of-fold predictions under 10-fold cross-validation
- name: mlxtend
  role: Implements the stacking ensemble with logistic regression as the meta-classifier
    over the cross-validation predictions
- name: m5C-PseDNC
  role: Prior SVM-based m5C predictor included as a benchmark in cross-validation
    comparisons
- name: iRNA-m5C
  role: Prior RF-based m5C predictor included as a benchmark in cross-validation comparisons
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:159eddc41310435ca59ea0a14042f1aef42387ac500c48b9b655b2207034fade
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - Staem5
  - iRNA-m5C
  - m5C-PseDNC
  - mlxtend
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# k-fold-cross-validation-model-evaluation

## Summary

Evaluate candidate base classifiers and stacked/voting ensembles for m5C site prediction using 10-fold cross-validation, computing Sn, Sp, accuracy, MCC, and AUC on the balanced A. thaliana and M. musculus training datasets before any independent testing.

## When to use

Use when comparing classifier candidates (SVM, XGBoost, LightGBM, ExtraTree, GBDT) or stacking-vs-voting ensembles on balanced RNA modification training data to select the optimal model before independent testing.

## When NOT to use

- The input is an independent test set (e.g., Test1157, Met1320, Met1900) — cross-validation is only for model selection on training data, and Staem5 was shown to be inferior to iRNA-m5C_SVM and m5CPred-SVM on the independent A. thaliana test set
- The goal is cross-species prediction — the article notes a general cross-species model is difficult and species-specific models are required, so a single pooled cross-validation model is the wrong choice
- The feature matrix is not fused/reduced (no PSP, k-mer, PCPseDNC, PseEIIP fusion and no F score selection) — results will not match the published cross-validation figures

## Inputs

- Balanced 41-bp m5C training dataset for A. thaliana (5,298 positive / 5,298 negative sequences)
- Balanced 41-bp m5C training dataset for M. musculus (4,563 positive / 4,563 negative sequences)
- Fused feature vectors from four encoding schemes: PSP, k-mer (k=1,2,3), PCPseDNC, and PseEIIP
- F score-selected reduced-dimension feature matrix (step of 5)

## Outputs

- 10-fold cross-validation performance metrics (Sn, Sp, accuracy, MCC, AUC) for each base classifier before and after parameter optimization
- Stacking-vs-voting accuracy comparison table over all base-classifier combinations (e.g., table2_stacking_vs_voting_cv_results.csv)
- Selected base classifier set for the stacked model (SVM, XGBoost, LightGBM)

## How to apply

Load the balanced 41-bp m5C training datasets (A. thaliana: 5,298 positive/5,298 negative; M. musculus: 4,563 positive/4,563 negative), then train base classifiers — SVM with radial basis kernel, GBDT, XGBoost, LightGBM, ExtraTree — in scikit-learn, first tuning hyperparameters via Bayesian optimization selected on 10-fold cross-validation accuracy. Generate out-of-fold predictions under 10-fold cross-validation, then aggregate them either by majority voting over each combination of base classifiers or by stacking with logistic regression as the meta-classifier implemented in mlxtend. Compute Sn, Sp, accuracy, MCC, and AUC from the 10-fold cross-validation results for every candidate and tabulate the comparisons (e.g., stacking 73.89% vs voting 73.68% accuracy for A. thaliana; 77.12% vs 76.83% for M. musculus). Use the accuracy of each stacked combination as the decision criterion, adding base classifiers one at a time until accuracy decreases, which led to selecting SVM, XGBoost, and LightGBM as the final base classifiers.

## Related tools

- **Staem5** (Stacked ensemble framework whose base classifiers and meta-model are evaluated via 10-fold cross-validation) — https://github.com/Cxd-626/Staem5.git
- **scikit-learn** (Trains base classifiers (SVM with RBF kernel, GBDT, XGBoost, LightGBM, ExtraTree) and generates out-of-fold predictions under 10-fold cross-validation)
- **mlxtend** (Implements the stacking ensemble with logistic regression as the meta-classifier over the cross-validation predictions)
- **m5C-PseDNC** (Prior SVM-based m5C predictor included as a benchmark in cross-validation comparisons)
- **iRNA-m5C** (Prior RF-based m5C predictor included as a benchmark in cross-validation comparisons)

## Examples

```
python m5C_predict.py --input query.fasta --species Arabidopsis
```

## Evaluation signals

- Stacking accuracy should slightly exceed voting accuracy for every base combination (A. thaliana 73.89% vs 73.68%; M. musculus 77.12% vs 76.83%)
- The final stacked model (SVM + XGBoost + LightGBM with logistic regression meta-model) should reach 73.89% accuracy and 0.479 MCC on A. thaliana 10-fold cross-validation
- Base classifier performance should improve after Bayesian optimization, with SVM showing the largest improvement and achieving the best 10-fold CV performance for A. thaliana
- The comparison table should contain Sn, Sp, accuracy, MCC, and AUC for every voting and stacking model, written to table2_stacking_vs_voting_cv_results.csv
- Base classifier selection should stop when adding an additional classifier decreases cross-validation accuracy

## Limitations

- Detailed performance metrics are only in supplementary Table S1 and detailed stacking comparison results only in supplementary Table S3, not the main text
- Staem5 underperformed competing methods on the independent A. thaliana test set, so cross-validation success does not guarantee independent-test superiority
- Hyperparameters of some competing predictors were pre-set according to previous experience rather than systematically derived, which may bias benchmark comparisons

## Evidence

- [other] Stacking should slightly outperform voting for each base combination (e.g., A. thaliana stacking 73.89% vs voting 73.68% accuracy; M. musculus stacking 77.12% vs voting 76.83%).: [evidence span withheld — anchor status: not_found]
