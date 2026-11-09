---
name: mcnemar-significance-testing
description: Use when two classifiers predict the same labeled sample set (e.g., DeepCC
  CMS labels vs RF/SVM/GBM/MLR on 13 independent CRC validation sets) and you need
  a paired significance test on error counts.
when_to_use_negative:
- The classifiers were evaluated on different sample sets, so errors are not paired
  and a two-sample test is required instead of McNemar's test
- The goal is unsupervised grouping of patients — DeepCC is specifically developed
  for classification, not clustering of cancer patients
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: DeepCC
  role: Trained ANN classifier whose per-sample labels on the 13 CRC validation sets
    form one side of each McNemar comparison
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: R
  role: Environment in which the McNemar tests and performance metrics are computed
    and assembled into table_s2_performance.csv
- name: keras
  role: Backend used to train and load the DeepCC ANN model whose errors are tested
    against baselines
  repo: https://keras.rstudio.com/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:fcac5ce9feefb8dcbd62ffcffbac2bbc3764a9fa5d6f0bdee4c542cd340b3784
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CityUHK-CompBio/DeepCC
  tools:
  - DeepCC
  - R
  - keras
  - msigdbr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# mcnemar-significance-testing

## Summary

Applies McNemar's test on paired per-sample classification errors to test whether the DeepCC ANN classifier's error rate on CRC validation sets is significantly lower than signature gene-based baselines (RF, SVM, GBM, multinomial logistic regression), yielding P-values for the Fig. 2 / Table S2 performance comparison.

## When to use

Use when two classifiers predict the same labeled sample set (e.g., DeepCC CMS labels vs RF/SVM/GBM/MLR on 13 independent CRC validation sets) and you need a paired significance test on error counts.

## When NOT to use

- The classifiers were evaluated on different sample sets, so errors are not paired and a two-sample test is required instead of McNemar's test
- The goal is unsupervised grouping of patients — DeepCC is specifically developed for classification, not clustering of cancer patients

## Inputs

- Per-sample predicted labels from the trained DeepCC model on each of the 13 independent CRC validation sets (CRCSC Synapse syn2623706)
- Per-sample predicted labels from baseline classifiers (RF, SVM, GBM, multinomial logistic regression, CRCSC 273-gene RF) on the same validation sets
- True CMS subtype labels for each validation sample

## Outputs

- McNemar test P-values comparing DeepCC classification errors against each baseline classifier per validation set
- table_s2_performance.csv summarizing sensitivity, specificity, accuracy, and significance per validation set

## How to apply

After training the DeepCC model with train_DeepCC_model on TCGA-COADREAD functional spectra (MSigDB v7, 22,596 gene sets) and obtaining predicted labels via get_DeepCC_label on each of the 13 independent CRC validation sets, tabulate per-sample correct/incorrect calls for DeepCC and each baseline classifier (random forests, SVM, GBM, multinomial logistic regression, and the CRCSC 273-gene RF CMS classifier). McNemar's test compares the discordant error pairs on the 2x2 table of DeepCC vs baseline correctness, which is appropriate because both classifiers are evaluated on identical samples. Run one test per baseline per validation set, collecting the McNemar P-value alongside balanced accuracy, overall accuracy, sensitivity, and specificity. Assemble the per-validation-set P-values into the Table S2 comparison and save as table_s2_performance.csv.

## Related tools

- **DeepCC** (Trained ANN classifier whose per-sample labels on the 13 CRC validation sets form one side of each McNemar comparison) — https://github.com/CityUHK-CompBio/DeepCC
- **R** (Environment in which the McNemar tests and performance metrics are computed and assembled into table_s2_performance.csv)
- **keras** (Backend used to train and load the DeepCC ANN model whose errors are tested against baselines) — https://keras.rstudio.com/

## Evaluation signals

- DeepCC shows higher sensitivity, specificity, and accuracy than each baseline (RF, SVM, GBM, multinomial logistic regression) on the 13 validation sets
- Each DeepCC-vs-baseline comparison carries a McNemar P-value quantifying the significance of the error-rate difference
- The 2x2 discordance table per comparison is built from identical sample sets so paired counts sum to the validation set size
- Output file table_s2_performance.csv contains per-validation-set metrics matching the Fig. 2 / Table S2 summary structure

## Limitations

- Significance conclusions depend on the gene set prior knowledge being comprehensive — the prior knowledge of gene sets needs to be comprehensive to avoid potential bias
- DeepCC is a supervised classification framework, not applicable to clustering of cancer patients, so the paired comparison only covers classification performance
- For single microarray samples referenced against TCGA RNA-Seq profiles, distribution differences require inverseRescale, which can affect downstream label agreement

## Evidence

- [other] McNemar test step grounding (task workflow): [evidence span withheld — anchor status: not_found]
- [abstract] Performance superiority finding: "DeepCC classifiers and DeepCC single sample predictors both achieved overall higher sensitivity, specificity, and accuracy compared with other widely used classification methods"
- [readme] Model training call: "# train DeepCC model
deepcc_model <- train_DeepCC_model(fs, labels)"
- [methods] Validation set prediction call: "get_DeepCC_label(DeepCCModel, newData, cutoff, prob_mode, prob_raw)"
