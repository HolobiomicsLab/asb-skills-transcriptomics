---
name: cancer-molecular-subtype-classification
description: Use when assigning established molecular subtype labels (e.g., CRCSC
  CMS, PAM50) to tumor RNA-Seq or microarray samples, especially cross-platform validation
  where signature-gene classifiers fail due to batch effects or unpaired gene annotation.
when_to_use_negative:
- The goal is de novo discovery/clustering of cancer patients — DeepCC is 'specifically
  developed for addressing current challenges in classification but not clustering
  of cancer patients' and requires pre-existing subtype labels.
- Input is not a gene expression matrix with mappable Entrez gene annotation (e.g.,
  mutation, methylation, or imaging data) — functional spectra depend on gene-set
  enrichment over gene identifiers.
- The MSigDB gene set database available is not comprehensive — the paper warns that
  'the prior knowledge of gene sets needs to be comprehensive to avoid potential bias'.
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: DeepCC
  role: Core R package providing getFunctionalSpectra, train_DeepCC_model, get_DeepCC_label,
    get_DeepCC_features, and pre-defined CRC/TCGA reference profiles
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: keras
  role: Deep learning backend used by train_DeepCC_model and to load trained models
    (load_model_hdf5); CPU and GPU supported
  repo: https://keras.rstudio.com/
- name: msigdbr
  role: Supplies MSigDBr gene sets via get_msigdbr() as an alternative geneSets source
    for functional spectra
- name: CRC_TCGA pre-trained model
  role: Pre-trained CRC DeepCC model (TCGA-COADREAD, 626 samples) loadable with load_DeepCC_model('CRC_TCGA')
    for classifying new CRC samples without retraining
  repo: https://github.com/zero19970/deepcc_model
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:3db89cc8a44b96faee8fc2cf306240c6a39b05e3626e6031da303674855beb70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CityUHK-CompBio/DeepCC
  tools:
  - CRC_TCGA pre-trained model
  - DeepCC
  - R
  - keras
  - msigdbr
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cancer-molecular-subtype-classification

## Summary

Train and apply DeepCC, an ANN-based supervised classifier that converts gene expression profiles into pathway-activity 'functional spectra' (MSigDB gene-set enrichment features) to assign cancer molecular subtype labels (e.g., CMS for CRC, PAM50 for breast cancer), platform-independently.

## When to use

Use when assigning established molecular subtype labels (e.g., CRCSC CMS, PAM50) to tumor RNA-Seq or microarray samples, especially cross-platform validation where signature-gene classifiers fail due to batch effects or unpaired gene annotation.

## When NOT to use

- The goal is de novo discovery/clustering of cancer patients — DeepCC is 'specifically developed for addressing current challenges in classification but not clustering of cancer patients' and requires pre-existing subtype labels.
- Input is not a gene expression matrix with mappable Entrez gene annotation (e.g., mutation, methylation, or imaging data) — functional spectra depend on gene-set enrichment over gene identifiers.
- The MSigDB gene set database available is not comprehensive — the paper warns that 'the prior knowledge of gene sets needs to be comprehensive to avoid potential bias'.

## Inputs

- data.frame of gene expression profiles (rows = patient samples, columns = Entrez identifiers)
- character vector of subtype training labels (e.g., CMS classes; NA permitted)
- MSigDB gene set collection (default MSigDB v7, 22,596 gene sets, or MSigDBr from the R package msigdbr)
- reference expression profile for single-sample classification (pre-defined TCGA average profiles such as 'COADREAD', or a customized one from the same/similar cancer type and platform)

## Outputs

- functional spectra matrix (pathway-activity features per sample)
- trained DeepCC ANN model (keras hdf5 model plus label levels, loadable via load_DeepCC_model)
- deep features (get_DeepCC_features) usable for sample visualization
- predicted subtype labels and per-class probabilities (get_DeepCC_label / get_DeepCC_prob)

## How to apply

Build a data.frame of gene expression profiles with rows as samples and columns named by Entrez gene identifiers, plus a character vector of training labels (NA allowed; DeepCC ignores them automatically). Compute functional spectra with getFunctionalSpectra, which by default scores MSigDB v7 gene sets (22,596 gene sets); MSigDB v5.0/v6.0 or msigdbr-derived MSigDBr can be substituted via the geneSets parameter. Train the keras-backed ANN with train_DeepCC_model(fs, labels), then classify new batches by recomputing their functional spectra and calling get_DeepCC_label; for a single microarray sample, use getFunctionalSpectrum with a reference expression profile (e.g., refExp = 'COADREAD') and enable inverseRescale to mitigate RNA-Seq-vs-microarray distribution differences. The functional-spectra transformation is what confers platform independence and robustness to missing data, so label assignment should always go through pathway-activity features rather than raw gene expression.

## Related tools

- **DeepCC** (Core R package providing getFunctionalSpectra, train_DeepCC_model, get_DeepCC_label, get_DeepCC_features, and pre-defined CRC/TCGA reference profiles) — https://github.com/CityUHK-CompBio/DeepCC
- **keras** (Deep learning backend used by train_DeepCC_model and to load trained models (load_model_hdf5); CPU and GPU supported) — https://keras.rstudio.com/
- **msigdbr** (Supplies MSigDBr gene sets via get_msigdbr() as an alternative geneSets source for functional spectra)
- **CRC_TCGA pre-trained model** (Pre-trained CRC DeepCC model (TCGA-COADREAD, 626 samples) loadable with load_DeepCC_model('CRC_TCGA') for classifying new CRC samples without retraining) — https://github.com/zero19970/deepcc_model

## Examples

```
fs <- getFunctionalSpectra(eps); deepcc_model <- train_DeepCC_model(fs, labels); pred_labels <- get_DeepCC_label(deepcc_model, getFunctionalSpectra(new_eps))
```

## Evaluation signals

- Functional spectra dimensionality matches the chosen gene set database (e.g., 22,596 features for default MSigDB v7; 10,348 for v5.0; 17,779 for v6.0).
- On the CRC benchmark (TCGA-COADREAD training, 13 independent microarray validation sets from CRCSC Synapse syn2623706), DeepCC shows higher sensitivity, specificity, and accuracy than RF, SVM, GBM, and multinomial logistic regression baselines, with significant McNemar's test P-values against each baseline.
- get_DeepCC_label returns predictions for every sample in a new data set, and get_DeepCC_prob returns per-class probabilities for thresholding via the cutoff/prob_mode parameters.
- Deep features (get_DeepCC_features) should show more compact within-subtype and better between-subtype separation than raw expression, and reduced numbers of unclassifiable samples.
- Robustness check: simulation with random subsampling of genes should show the classifier tolerates missing data.

## Limitations

- DeepCC performs supervised classification only, not clustering of cancer patients into novel subtypes.
- Classification quality depends on a comprehensive gene set database; characteristic signaling pathways for each subtype must be present in the gene set collection.
- Single-sample microarray classification against the RNA-Seq-based TCGA reference suffers distribution differences; inverseRescale only partially overcomes this and a customized reference profile is recommended.
- Existing batch-correction approaches such as ComBat are limited to balanced experimental designs, which is part of why functional spectra are used instead.

## Evidence

- [readme] By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra.: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra. You can also use MSigDB v5.0 (10, 348 gene sets) and MSigDB v7.0 (17, 779 gene sets)."
- [abstract] DeepCC classifiers and DeepCC single sample predictors both achieved overall higher sensitivity, specificity, and accuracy compared with other widely used classification methods: "DeepCC classifiers and DeepCC single sample predictors both achieved overall higher sensitivity, specificity, and accuracy compared with other widely used classification methods"
- [readme] if your single sample is microarray data, we strongly suggest you turn the parameter `inverseRescale` on, since TCGA is RNA-Seq data, compared with microarray.: "Note: if your single sample is microarray data, we strongly suggest you turn the parameter `inverseRescale` on, since TCGA is RNA-Seq data, compared with microarray."
- [discussion] specifically developed for addressing current challenges in classification but not clustering of cancer patients: "specifically developed for addressing current challenges in classification but not clustering of cancer patients"
- [discussion] we effectively applied DeepCC to breast cancer, and demonstrated a better performance over PAM50, which is a widely used classification system in breast cancer: "we effectively applied DeepCC to breast cancer, and demonstrated a better performance over PAM50, which is a widely used classification system in breast cancer"
- [methods] MSigDBr attained from R package `msigdbr` with function get_msigdbr() in DeepCC: "Add parameter `geneSets = "MSigDBvx"` (x = 5/6/7) or `geneSets = MSigDBr` (MSigDBr attained from R package `msigdbr` with function get_msigdbr() in DeepCC)"
