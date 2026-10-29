---
name: breast-cancer-pam50-subtype-classification
description: Use when classifying breast cancer samples into PAM50 molecular subtypes
  from gene expression data, especially cross-platform prediction where signature
  genes are missing or single-sample classification is needed.
when_to_use_negative:
- The task is unsupervised clustering of cancer patients rather than supervised subtype
  classification — DeepCC was 'specifically developed for addressing current challenges
  in classification but not clustering of cancer patients'
- No training labels are available (DeepCC is a supervised framework requiring labels
  such as PAM50 or CMS)
- Single-sample microarray prediction without an appropriate reference expression
  profile and inverseRescale enabled — mixing microarray with TCGA RNA-Seq references
  is explicitly discouraged
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: DeepCC
  role: R framework providing getFunctionalSpectra, train_DeepCC_model, get_DeepCC_features,
    and get_DeepCC_label for the classification pipeline
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: keras
  role: Backend dependency required to train/load the DeepCC ANN models (.hdf5)
- name: msigdbr
  role: Supplies the MSigDBr gene set collection (via get_msigdbr()) used to compute
    functional spectra
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:d295a378a2179a725763225230a7adfdd832f6b8e39207769599f21157be6f46
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# breast-cancer-pam50-subtype-classification

## Summary

Trains a DeepCC artificial neural network on TCGA BRCA RNA-Seq functional spectra with PAM50 labels and applies it to breast cancer validation cohorts (TANSBIG, UNT, UPP, NKI), demonstrating robustness to cross-platform missing genes via random gene-subsampling down to 1000 genes.

## When to use

Use when classifying breast cancer samples into PAM50 molecular subtypes from gene expression data, especially cross-platform prediction where signature genes are missing or single-sample classification is needed.

## When NOT to use

- The task is unsupervised clustering of cancer patients rather than supervised subtype classification — DeepCC was 'specifically developed for addressing current challenges in classification but not clustering of cancer patients'
- No training labels are available (DeepCC is a supervised framework requiring labels such as PAM50 or CMS)
- Single-sample microarray prediction without an appropriate reference expression profile and inverseRescale enabled — mixing microarray with TCGA RNA-Seq references is explicitly discouraged

## Inputs

- TCGA BRCA RNA-Seq expression matrix (n=517) with PAM50 subtype labels
- Breast cancer validation-cohort expression matrices: TANSBIG, UNT, UPP, NKI
- data.frame of gene expression profiles with column names as Entrez identifiers; character vector of training labels (NA allowed)

## Outputs

- Trained DeepCC model with deep feature extraction (get_DeepCC_features)
- Predicted PAM50 subtype labels for TANSBIG, UNT, UPP, and NKI cohorts
- Per-cohort sensitivity, specificity, and balanced accuracy metrics; gene-subsampling robustness curve (Fig. S6 analogue)

## How to apply

Load the TCGA BRCA expression matrix (n=517) with PAM50 labels plus the four validation-cohort matrices (TANSBIG, UNT, UPP, NKI) as data.frames whose columns are Entrez identifiers. Compute functional spectra with getFunctionalSpectra(eps) using the default MSigDB v7 collection (22,596 gene sets), then train the DeepCC ANN via train_DeepCC_model(fs, labels), tuning epochs, dropout, activation_func, and validation_split. Extract deep features with get_DeepCC_features and predict validation-cohort labels with get_DeepCC_label(deepcc_model, new_fs), recording sensitivity, specificity, and balanced accuracy per cohort. For the robustness ablation, randomly subsample genes at decreasing set sizes down to 1000, regenerate functional spectra, retrain, and confirm balanced accuracy stays above 80%, reproducing Fig. S6. NA training labels are allowed and ignored automatically by DeepCC.

## Related tools

- **DeepCC** (R framework providing getFunctionalSpectra, train_DeepCC_model, get_DeepCC_features, and get_DeepCC_label for the classification pipeline) — https://github.com/CityUHK-CompBio/DeepCC
- **keras** (Backend dependency required to train/load the DeepCC ANN models (.hdf5))
- **msigdbr** (Supplies the MSigDBr gene set collection (via get_msigdbr()) used to compute functional spectra)

## Examples

```
fs <- getFunctionalSpectra(eps); deepcc_model <- train_DeepCC_model(fs, labels); pred_labels <- get_DeepCC_label(deepcc_model, getFunctionalSpectra(new_eps))
```

## Evaluation signals

- Balanced accuracy remains above 80% when only 1000 randomly subsampled genes are used for training (acceptance criterion from Fig. S6)
- Performance on the four validation cohorts outperforms random forests, SVM, GBM, and multinomial logistic regression in sensitivity, specificity, and accuracy
- Deep features show compact within-subtype distribution and clear between-subtype separation of patient samples, with fewer unclassifiable samples
- Functional spectra are computed with the MSigDB v7 default (22,596 gene sets), and input data.frames use Entrez identifiers as column names
- Training labels containing NA values are accepted and ignored automatically without pipeline failure

## Limitations

- DeepCC is classification-only and not applicable to clustering of cancer patients
- Robustness depends on comprehensive gene set prior knowledge; major characteristic signaling pathways for each subtype must be included in the gene set database to avoid potential bias
- Single-sample prediction requires a reference expression profile (e.g., a TCGA cancer-type identifier or a customized profile from the same/similar cancer type and platform); for microarray single samples the inverseRescale parameter should be enabled because TCGA is RNA-Seq data
- Platform differences and batch effects in signature-gene-based approaches are what DeepCC mitigates via pathway-level functional spectra, but mis-specification of the gene set collection can still bias results

## Evidence

- [abstract] Simulation analysis based on random subsampling of genes demonstrated the robustness of DeepCC to missing data: "Simulation analysis based on random subsampling of genes demonstrated the robustness of DeepCC to missing data"
- [methods] MSigDB v7 default for functional spectra: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
- [discussion] Better performance over PAM50: "we effectively applied DeepCC to breast cancer, and demonstrated a better performance over PAM50, which is a widely used classification system in breast cancer"
- [readme] Input format for DeepCC: "a data.frame containing gene expression profiles (each row represents one patient sample and the column names should be Entrez identifiers of genes)"
- [readme] Training hyperparameters: "deepcc_model <- train_DeepCC_model(fs, labels)"
- [readme] Microarray single-sample caveat: "if your single sample is microarray data, we strongly suggest you turn the parameter `inverseRescale` on, since TCGA is RNA-Seq data"
