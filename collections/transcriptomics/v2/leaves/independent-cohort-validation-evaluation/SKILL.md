---
name: independent-cohort-validation-evaluation
description: Use when a trained subtype classifier must be assessed on external validation
  cohorts with known labels, or when cross-platform robustness to missing genes must
  be quantified.
when_to_use_negative:
- Input is single-sample classification without a labeled independent cohort — use
  single-sample prediction with a reference expression profile (e.g., getFunctionalSpectrum(new_ep,
  refExp = "BRCA")) instead
- Goal is unsupervised grouping of patients — DeepCC is developed for classification,
  not clustering of cancer patients
- The validation cohort lacks coherent subtype labels (e.g., BluePrint data that cannot
  distinguish Luminal A from Luminal B), making balanced accuracy against PAM50 classes
  undefined
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: DeepCC
  role: Provides getFunctionalSpectra, train_DeepCC_model, get_DeepCC_label used to
    transform cohort expression data, retrain under gene subsampling, and predict
    subtype labels on independent cohorts
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: R
  role: Implementation environment for loading cohort matrices, running the DeepCC
    pipeline, and computing per-cohort metrics
- name: msigdbr
  role: Supplies the MSigDB gene set collection (via get_msigdbr()) used to build
    functional spectra from validation-cohort expression
- name: PAM50
  role: Reference intrinsic-subtyping system whose labels define the breast cancer
    classes for sensitivity/specificity/balanced-accuracy evaluation
- name: keras
  role: Deep learning backend (keras R framework) on which train_DeepCC_model and
    loaded .hdf5 models depend
  repo: https://github.com/CityUHK-CompBio/DeepCC
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:ed9a1097ed50f85260a63346e5f4a9752a8f7865259e6f0d9ba6f877f99e73e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CityUHK-CompBio/DeepCC
  tools:
  - DeepCC
  - PAM50
  - R
  - keras
  - msigdbr
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Independent cohort validation evaluation

## Summary

Evaluates a trained DeepCC molecular subtype classifier (e.g., breast cancer PAM50-label ANN trained on TCGA RNA-Seq, n=517) on completely independent expression cohorts (TANSBIG, UNT, UPP, NKI), recording sensitivity, specificity, and balanced accuracy per cohort, and stress-tests robustness by random gene subsampling down to 1000 genes.

## When to use

Use when a trained subtype classifier must be assessed on external validation cohorts with known labels, or when cross-platform robustness to missing genes must be quantified.

## When NOT to use

- Input is single-sample classification without a labeled independent cohort — use single-sample prediction with a reference expression profile (e.g., getFunctionalSpectrum(new_ep, refExp = "BRCA")) instead
- Goal is unsupervised grouping of patients — DeepCC is developed for classification, not clustering of cancer patients
- The validation cohort lacks coherent subtype labels (e.g., BluePrint data that cannot distinguish Luminal A from Luminal B), making balanced accuracy against PAM50 classes undefined

## Inputs

- TCGA BRCA gene expression matrix (n=517) with PAM50 subtype labels (data.frame, rows = samples, columns = Entrez gene IDs)
- Independent validation-cohort expression matrices: TANSBIG, UNT, UPP, NKI (data.frame, Entrez identifiers)
- Trained DeepCC model (keras ANN) for breast cancer
- MSigDB v7 gene set collection (22,596 gene sets)

## Outputs

- Per-cohort confusion-matrix metrics: sensitivity, specificity, and balanced accuracy for TANSBIG, UNT, UPP, NKI
- Predicted subtype labels for each validation cohort sample
- Gene-subsampling robustness curve (balanced accuracy vs. number of genes, down to 1000 genes)
- Comparison table against random forests, SVM, GBM, and multinomial logistic regression

## How to apply

Load the TCGA BRCA expression matrix with PAM50 labels and each independent cohort matrix (TANSBIG, UNT, UPP, NKI) in R, ensuring gene identifiers are Entrez IDs as required by DeepCC. Compute functional spectra for every dataset with getFunctionalSpectra(eps) using the default MSigDB v7 collection (22,596 gene sets), then obtain predictions for each validation cohort via get_DeepCC_label(deepcc_model, new_fs). Record sensitivity, specificity, and balanced accuracy per cohort and compare against random forests, SVM, GBM, and multinomial logistic regression baselines. For the robustness ablation, randomly subsample genes at decreasing set sizes down to 1000 genes, regenerate functional spectra, retrain and re-evaluate; functional-spectra transformation over MSigDB gene sets is what confers platform independence and tolerance to missing genes. Acceptance is balanced accuracy > 80% with only 1000 randomly subsampled genes (Fig. S6) and performance consistent with reported outperformance of the baseline classifiers.

## Related tools

- **DeepCC** (Provides getFunctionalSpectra, train_DeepCC_model, get_DeepCC_label used to transform cohort expression data, retrain under gene subsampling, and predict subtype labels on independent cohorts) — https://github.com/CityUHK-CompBio/DeepCC
- **R** (Implementation environment for loading cohort matrices, running the DeepCC pipeline, and computing per-cohort metrics)
- **msigdbr** (Supplies the MSigDB gene set collection (via get_msigdbr()) used to build functional spectra from validation-cohort expression)
- **PAM50** (Reference intrinsic-subtyping system whose labels define the breast cancer classes for sensitivity/specificity/balanced-accuracy evaluation)
- **keras** (Deep learning backend (keras R framework) on which train_DeepCC_model and loaded .hdf5 models depend) — https://github.com/CityUHK-CompBio/DeepCC

## Examples

```
fs_val <- getFunctionalSpectra(nki_eps); pred_labels <- get_DeepCC_label(deepcc_model, fs_val)
```

## Evaluation signals

- Balanced accuracy remains above 80% when the classifier is trained with only 1000 randomly subsampled genes (Fig. S6 acceptance criterion)
- Per-cohort sensitivity, specificity, and balanced accuracy are recorded for all four validation cohorts (TANSBIG, UNT, UPP, NKI) without train/test leakage
- DeepCC metrics exceed random forests, SVM, GBM, and multinomial logistic regression on the same cohorts
- Functional spectra for every cohort are computed with the same default MSigDB v7 gene set collection (22,596 gene sets)
- Predicted label levels match the PAM50 subtype classes used in training

## Limitations

- Comprehensive gene set prior knowledge is required; major characteristic signaling pathways or biological processes for each subtype must be included in the gene set database to avoid potential bias
- Validation labels must be coherent — BluePrint cannot distinguish Luminal A from Luminal B, and MammaPrint/Oncotype DX are prognostic risk tools, not subtype labels
- Microarray validation samples may need inverseRescale when computing spectra against TCGA RNA-Seq references, and a customized reference profile is recommended for divergent platforms
- Performance depends on gene identifier harmonization (Entrez IDs) across platforms; unpaired gene annotation is the portability problem DeepCC's pathway transformation mitigates rather than removes

## Evidence

- [abstract] DeepCC classifiers and DeepCC single sample predictors both achieved overall higher sensitivity, specificity, and accuracy compared with other widely used classification methods: "DeepCC classifiers and DeepCC single sample predictors both achieved overall higher sensitivity, specificity, and accuracy compared with other widely used classification methods"
- [abstract] Simulation analysis based on random subsampling of genes demonstrated the robustness of DeepCC to missing data: "Simulation analysis based on random subsampling of genes demonstrated the robustness of DeepCC to missing data"
- [methods] pred_labels <- get_DeepCC_label(deepcc_model, new_fs): "new_fs <- getFunctionalSpectra(new_eps)
pred_labels <- get_DeepCC_label(deepcc_model, new_fs)"
- [methods] By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra.: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
- [discussion] The trained DeepCC model can be subsequently used for classification of new samples: "The trained DeepCC model can be subsequently used for classification of new samples"
- [readme] a data.frame containing gene expression profiles (each row represents one patient sample and the column names should be Entrez identifiers of genes): "a data.frame containing gene expression profiles (each row represents one patient sample and the column names should be Entrez identifiers of genes)"
