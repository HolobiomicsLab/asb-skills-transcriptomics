---
name: cms-classifier-benchmark-comparison
description: Use when comparing a trained CMS/DeepCC classifier against baseline machine-learning
  classifiers on independent CRC expression validation sets, or when reporting per-dataset
  sensitivity/specificity/accuracy with paired significance tests.
when_to_use_negative:
- The goal is unsupervised grouping of cancer patients — DeepCC is built for classification,
  not clustering of cancer patients
- Samples cannot be represented as gene expression profiles with Entrez-identifier
  columns, or no trained model/baseline signatures exist for the cancer type
- You need to detect novel subtypes rather than assign samples to the four established
  CMS classes
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: DeepCC
  role: Computes functional spectra (getFunctionalSpectra), trains the ANN classifier
    (train_DeepCC_model), and predicts labels (get_DeepCC_label) on validation sets
  repo: github.com/CityUHK-CompBio/DeepCC
- name: R
  role: Implementation environment for the DeepCC pipeline and baseline classifier
    benchmarking
- name: msigdbr
  role: Supplies MSigDB gene sets (get_msigdbr()) used to generate functional spectra
    before classification
- name: keras
  role: Deep-learning backend required by train_DeepCC_model and load_model_hdf5 for
    the DeepCC classifier
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:230db25cf0767307fa771a134b4719fa669a796c7e51f0fc67c771dda9d05626
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepCC
  - R
  - keras
  - msigdbr
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cms-classifier-benchmark-comparison

## Summary

Benchmark a DeepCC deep-learning subtype classifier against widely used signature gene-based baselines (RF, SVM, GBM, multinomial logistic regression) on independent colorectal cancer validation sets, using sensitivity, specificity, accuracy, and McNemar's test to quantify significance.

## When to use

Use when comparing a trained CMS/DeepCC classifier against baseline machine-learning classifiers on independent CRC expression validation sets, or when reporting per-dataset sensitivity/specificity/accuracy with paired significance tests.

## When NOT to use

- The goal is unsupervised grouping of cancer patients — DeepCC is built for classification, not clustering of cancer patients
- Samples cannot be represented as gene expression profiles with Entrez-identifier columns, or no trained model/baseline signatures exist for the cancer type
- You need to detect novel subtypes rather than assign samples to the four established CMS classes

## Inputs

- TCGA-COADREAD RNA-Seq training expression profiles (data.frame, rows = samples, columns = Entrez IDs) with CRCSC CMS subtype labels
- 13 independent microarray-based CRC validation data sets from CRCSC Synapse (syn2623706)
- MSigDB v7 gene sets (22,596 gene sets) via msigdbr / get_msigdbr()
- Pre-trained or newly trained DeepCC keras model (.hdf5) with subtype label levels (.RData)

## Outputs

- Per-validation-set performance table (balanced accuracy, accuracy, sensitivity, specificity) for DeepCC and each baseline, saved as table_s2_performance.csv
- McNemar test P-values comparing DeepCC vs each baseline classifier per validation set
- Predicted CMS labels per sample per classifier (get_DeepCC_label outputs)

## How to apply

Compute functional spectra from the TCGA-COADREAD training expression profiles with getFunctionalSpectra using default MSigDB v7 gene sets (22,596 gene sets), then train the DeepCC ANN with train_DeepCC_model on the functional spectra and CRCSC CMS labels. Apply the same functional-spectrum transform to the 13 independent CRC validation sets (CRCSC Synapse syn2623706) and call get_DeepCC_label on the trained model; also label each validation set with the baselines (random forest, SVM, GBM, multinomial logistic regression) and the CRCSC 273-gene random-forest CMS classifier. Per validation set, compute balanced accuracy, overall accuracy, sensitivity, and specificity for every classifier. Run McNemar tests comparing DeepCC classification errors against each baseline to obtain paired P-values, since all classifiers predict the same sample set. Assemble the per-validation-set comparison into the Fig. 2 / Table S2 format and save it as a CSV.

## Related tools

- **DeepCC** (Computes functional spectra (getFunctionalSpectra), trains the ANN classifier (train_DeepCC_model), and predicts labels (get_DeepCC_label) on validation sets) — github.com/CityUHK-CompBio/DeepCC
- **R** (Implementation environment for the DeepCC pipeline and baseline classifier benchmarking)
- **msigdbr** (Supplies MSigDB gene sets (get_msigdbr()) used to generate functional spectra before classification)
- **keras** (Deep-learning backend required by train_DeepCC_model and load_model_hdf5 for the DeepCC classifier)

## Examples

```
library(DeepCC); library(keras); fs <- getFunctionalSpectra(eps); deepcc_model <- train_DeepCC_model(fs, labels); pred_labels <- get_DeepCC_label(deepcc_model, getFunctionalSpectra(new_eps))
```

## Evaluation signals

- DeepCC must show higher sensitivity, specificity, and accuracy than RF, SVM, GBM, and multinomial logistic regression across the 13 independent CRC validation sets
- McNemar's test against each baseline must yield the reported significant P-values for DeepCC's classification errors
- Predicted labels must use the posterior probability > 0.5 criterion (cutoff = 0.5 from get_DeepCC_label); samples at or below 0.5 are counted as unclassifiable
- Unclassifiable fraction must be ~5% across CRCSC-analyzed data sets, and 531/557 classified samples on CIT/GSE39582, versus 117 unclassifiable for the 273-gene RF CMS classifier
- Output CSV must contain one row per validation set per classifier with the four performance metrics

## Limitations

- DeepCC is classification-only and not applicable to clustering of cancer patients
- The prior knowledge of gene sets must be comprehensive to avoid potential bias; major characteristic signaling pathways for each subtype should be included in the gene set database
- Functional spectra depend on Entrez identifier annotation; DeepCC requires reference expression profiles for single-sample classification and warns that microarray single samples should use inverseRescale or a customized same-platform reference

## Evidence

- [abstract] DeepCC outperforms baselines across CRC validation sets: "DeepCC classifiers and DeepCC single sample predictors both achieved overall higher sensitivity, specificity, and accuracy compared with other widely used classification methods"
- [readme] Default functional spectra from MSigDB v7: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
- [readme] Classification workflow for new validation data: "new_fs <- getFunctionalSpectra(new_eps)
pred_labels <- get_DeepCC_label(deepcc_model, new_fs)"
- [methods] Training interface for the DeepCC ANN: "deepcc_model <- train_DeepCC_model(fs, labels)"
