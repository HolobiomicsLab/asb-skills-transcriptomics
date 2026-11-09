---
name: rna-5hmc-site-classification
description: Use when RNA sequences of length 41 need to be classified as 5hmC site
  vs non-site using position-specific sequence descriptors, or when reproducing the
  iRNA5hmC-PS benchmark (78.3% accuracy, 0.56 MCC) on Dataset.csv.
when_to_use_negative:
- The goal is combining these position-specific features with frequency-based k-mer
  features in a single model — the authors report this does not work well and defer
  it to a future feature subspace ensemble approach
- Sequences are not 41 nt long or not in FASTA format — the webserver and pipeline
  assume 41-nt RNA sequences
- Input is already an engineered feature table from a different descriptor family;
  the group-wise feature selection strategy is tailored to the paper's PSG k-mer feature
  groups
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: iRNA5hmC-PS
  role: Proposed prediction framework combining PSG k-mer and position-specific k-mer
    features with Logistic Regression; also available as a webserver for 41-nt FASTA
    RNA sequence queries
  repo: https://github.com/zahid6454/iRNA5hmC-PS
- name: scikit-learn
  role: Provides the LR, SVM, and Gaussian Naive Bayes classifiers, 5-fold cross-validation
    machinery, and metrics (Accuracy, Precision, Sensitivity, Specificity, F1, MCC)
- name: pandas
  role: Loads Dataset.csv and All_Features_Dataset.csv into DataFrames for feature
    assembly
- name: matplotlib
  role: Plots AUROC and AUPR curves for cross-validation and independent-test evaluations
- name: Jupyter
  role: Execution environment for Feature Generation.ipynb and Model.ipynb
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2020.10.032
    title: Accurate prediction of RNA 5-hydroxymethylcytosine modification by utilizing
      novel position-specific gapped k-mer descriptors
  source_content_hash: sha256:be079b24838facd6f289c4441317fa3633a18b142347afa2b0197e3525bce46d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zahid6454/iRNA5hmC-PS
  tools:
  - Anaconda
  - Jupyter
  - Python
  - iRNA5hmC-PS
  - imblearn
  - matplotlib
  - numpy
  - pandas
  - scikit-learn
  - tabulate
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.55
n_claims: 20
---

# rna-5hmc-site-classification

## Summary

Reproduce iRNA5hmC-PS: classify RNA 5-hydroxymethylcytosine (5hmC) sites using position-specific k-mer and position-specific gapped k-mer (PSG k-mer) features with group-wise feature selection and a Logistic Regression classifier, evaluated by repeated 5-fold cross-validation and an independent test set.

## When to use

Use when RNA sequences of length 41 need to be classified as 5hmC site vs non-site using position-specific sequence descriptors, or when reproducing the iRNA5hmC-PS benchmark (78.3% accuracy, 0.56 MCC) on Dataset.csv.

## When NOT to use

- The goal is combining these position-specific features with frequency-based k-mer features in a single model — the authors report this does not work well and defer it to a future feature subspace ensemble approach
- Sequences are not 41 nt long or not in FASTA format — the webserver and pipeline assume 41-nt RNA sequences
- Input is already an engineered feature table from a different descriptor family; the group-wise feature selection strategy is tailored to the paper's PSG k-mer feature groups

## Inputs

- Dataset.csv — benchmark RNA 5hmC dataset of 41-nt sequences with labels
- Feature Generation.ipynb — notebook generating all proposed position-specific features
- Model.ipynb — notebook training and evaluating LR, SVM, and GNB classifiers

## Outputs

- All_Features_Dataset.csv — engineered feature matrix from Dataset.csv
- Cross-validation and independent test metric tables (Accuracy, Precision, Sensitivity, Specificity, F1-Score, MCC) with standard deviations
- AUROC and AUPR curves for cross-validation and independent-set results

## How to apply

First run 'Feature Generation.ipynb' in Jupyter on Dataset.csv to build all position-specific features and write All_Features_Dataset.csv. Assemble feature matrices for the four position-specific modes (A = Ps-k-mer, B = PsM(G)D, C = PsD(G)M, D = PsM(G)M(G)M) plus GC-content, apply the group-wise feature importance calculation strategy to select a small discriminative subset, then train scikit-learn Logistic Regression (best of LR, SVM, GNB) under 50-times 5-fold cross-validation and predict the held-out 132-sample stratified independent set from the 1192-sample training partition. Compute Accuracy, Precision, Sensitivity, Specificity, F1-Score and MCC with standard deviations, and render AUROC/AUPR curves for both cross-validation and independent-set evaluations.

## Related tools

- **iRNA5hmC-PS** (Proposed prediction framework combining PSG k-mer and position-specific k-mer features with Logistic Regression; also available as a webserver for 41-nt FASTA RNA sequence queries) — https://github.com/zahid6454/iRNA5hmC-PS
- **scikit-learn** (Provides the LR, SVM, and Gaussian Naive Bayes classifiers, 5-fold cross-validation machinery, and metrics (Accuracy, Precision, Sensitivity, Specificity, F1, MCC))
- **pandas** (Loads Dataset.csv and All_Features_Dataset.csv into DataFrames for feature assembly)
- **matplotlib** (Plots AUROC and AUPR curves for cross-validation and independent-test evaluations)
- **Jupyter** (Execution environment for Feature Generation.ipynb and Model.ipynb)

## Examples

```
jupyter nbconvert --to notebook --execute Feature_Generation.ipynb && jupyter nbconvert --to notebook --execute Model.ipynb
```

## Evaluation signals

- Logistic Regression cross-validation metrics should reproduce the reported values: Accuracy 78.3%, Precision 78.0%, Sensitivity 80.0%, Specificity 79.5%, F1 0.78, MCC 0.56 (within cross-validation variation)
- In the 15-combination ablation, the A+B+C+D full feature-mode combination should be the best of all 15 tested combinations
- LR should outperform SVM (75.5% accuracy, 0.49 MCC) and GNB (76.7% accuracy, 0.50 MCC) in four out of six cross-validation metrics
- Independent-set results for LR should be near the reported 78.5% accuracy, 77.3% sensitivity, 0.77 F1, and 0.53 MCC
- AUROC and AUPR curves must be rendered in the notebook output cells for both cross-validation and independent-set results

## Limitations

- Position-specific features do not work well in conjunction with existing frequency-based counterparts in single models; the authors defer combining them via a feature subspace ensemble to future work
- Efficacy on other RNA modification site identification problems has not been evaluated
- Deep learning suitability of the proposed features is claimed (convolutional/recurrent layers) but not demonstrated
- Results depend on group-wise feature selection; individual ablation combinations show variable performance across the 15 tested combinations

## Evidence

- [results] LR achieves 78.3%, 78.0%, 80.0%, 79.5%, 0.78, 0.56 in terms of Accuracy, Precision, Sensitivity, Specificity, F1-Score, and MCC, respectively: "LR achieves 78.3%, 78.0%, 80.0%, 79.5%, 0.78, 0.56 in terms of Accuracy, Precision, Sensitivity, Specificity, F1-Score, and MCC, respectively"
- [abstract] We also use a group-wise feature importance calculation strategy to select a small subset of features containing maximum discriminative information: "We also use a group-wise feature importance calculation strategy to select a small subset of features containing maximum discriminative information"
- [readme] Feature Generation.ipynb uses Dataset.csv file to generate all the proposed features and stores that into a new file titled as All_Features_Dataset.csv: ""Feature Generation.ipynb" uses "Dataset.csv" file to generate all the proposed features and stores that into a new file titled as "All_Features_Dataset.csv""
- [discussion] our proposed position-specific features do not work well in conjunction with existing frequency-based counterparts: "our proposed position-specific features do not work well in conjunction with existing frequency-based counterparts"
- [readme] The user has to copy and paste RNA sequence having length of 41. The format has to be FASTA format.: "The user has to copy and paste RNA sequence having length of 41. The format has to be FASTA format."
