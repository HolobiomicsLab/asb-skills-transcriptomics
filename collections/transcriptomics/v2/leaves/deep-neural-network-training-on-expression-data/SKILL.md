---
name: deep-neural-network-training-on-expression-data
description: Use when training a supervised cancer subtype classifier from bulk gene
  expression (RNA-Seq or microarray) with known labels (e.g., PAM50 breast cancer
  intrinsic subtypes) and when robustness to cross-platform missing genes is required.
when_to_use_negative:
- The goal is unsupervised clustering or de novo discovery of cancer patient groups
  — the article states DeepCC is 'specifically developed for addressing current challenges
  in classification but not clustering of cancer patients'.
- A labeled, well-annotated feature table of pathway scores already exists and only
  a simple model is needed — DeepCC's value is in the functional spectra transformation,
  not training on arbitrary precomputed features.
- The cancer type's characteristic pathways are poorly represented in the MSigDB gene
  set database — the article warns 'the prior knowledge of gene sets needs to be comprehensive
  to avoid potential bias'.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: DeepCC
  role: R package providing getFunctionalSpectra, train_DeepCC_model, get_DeepCC_features,
    and get_DeepCC_label for the spectral transformation, ANN training, and classification
    steps
  repo: github.com/CityUHK-CompBio/DeepCC
- name: keras
  role: deep learning backend required by DeepCC's ANN training and model serialization
    (load_model_hdf5); must be installed and verified to run in R
  repo: https://keras.rstudio.com/
- name: msigdbr
  role: supplies the MSigDBr gene set collection used as the functional spectra basis
    when the default bundled MSigDB version is not used
  repo: github.com/CityUHK-CompBio/DeepCC
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:bfd872a430f877d1ef3a843e241ef83b142b7f947805f1c30807db101b879fff
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepCC
  - R
  - keras
  - msigdbr
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# deep-neural-network-training-on-expression-data

## Summary

Trains a DeepCC artificial neural network (ANN) on pathway-level 'functional spectra' derived from tumor gene expression (e.g., TCGA BRCA RNA-Seq with PAM50 labels) to learn a supervised molecular subtype classifier. The deep features overcome platform dependency and batch effects that limit signature-gene classifiers.

## When to use

Use when training a supervised cancer subtype classifier from bulk gene expression (RNA-Seq or microarray) with known labels (e.g., PAM50 breast cancer intrinsic subtypes) and when robustness to cross-platform missing genes is required.

## When NOT to use

- The goal is unsupervised clustering or de novo discovery of cancer patient groups — the article states DeepCC is 'specifically developed for addressing current challenges in classification but not clustering of cancer patients'.
- A labeled, well-annotated feature table of pathway scores already exists and only a simple model is needed — DeepCC's value is in the functional spectra transformation, not training on arbitrary precomputed features.
- The cancer type's characteristic pathways are poorly represented in the MSigDB gene set database — the article warns 'the prior knowledge of gene sets needs to be comprehensive to avoid potential bias'.

## Inputs

- gene expression profile data.frame (rows = patient samples, columns = Entrez identifiers of genes), e.g., TCGA BRCA RNA-Seq matrix (n=517)
- character vector of training labels (e.g., PAM50 intrinsic subtypes; NA allowed)
- MSigDB gene set collection (default MSigDB v7, 22,596 gene sets) or MSigDBr from R package msigdbr

## Outputs

- trained DeepCC ANN model object (keras hdf5 model plus label levels, loadable via load_DeepCC_model)
- deep feature matrix for training samples (from get_DeepCC_features)
- predicted subtype labels with probabilities for new cohorts (get_DeepCC_label / get_DeepCC_prob)
- per-cohort sensitivity, specificity, and balanced accuracy metrics

## How to apply

First transform raw expression profiles (data.frame, rows = samples, columns = Entrez gene IDs) into functional spectra with getFunctionalSpectra(eps), which by default uses MSigDB v7 (22,596 gene sets); other versions (MSigDB v5.0, v7.0/17,779 gene sets, or MSigDBr from msigdbr::get_msigdbr()) can be passed via geneSets. Then call train_DeepCC_model(fs, labels), tuning epochs, dropout, activation_func, and validation_split; labels are a character vector (NA allowed, ignored automatically), and the model requires the keras R framework to be installed and functional. Inspect learned representations with get_DeepCC_features(deepcc_model, fs) to confirm compact within-subtype and separated between-subtype distributions, then apply the model to validation cohorts (e.g., TANSBIG, UNT, UPP, NKI for breast cancer) via get_DeepCC_label(deepcc_model, new_fs), recording sensitivity, specificity, and balanced accuracy. Rationale: learning on functional spectra rather than signature genes makes the classifier platform independent and robust to missing data, as demonstrated by random gene subsampling down to 1000 genes.

## Related tools

- **DeepCC** (R package providing getFunctionalSpectra, train_DeepCC_model, get_DeepCC_features, and get_DeepCC_label for the spectral transformation, ANN training, and classification steps) — github.com/CityUHK-CompBio/DeepCC
- **keras** (deep learning backend required by DeepCC's ANN training and model serialization (load_model_hdf5); must be installed and verified to run in R) — https://keras.rstudio.com/
- **msigdbr** (supplies the MSigDBr gene set collection used as the functional spectra basis when the default bundled MSigDB version is not used) — github.com/CityUHK-CompBio/DeepCC

## Examples

```
fs <- getFunctionalSpectra(eps); deepcc_model <- train_DeepCC_model(fs, labels); df <- get_DeepCC_features(deepcc_model, fs)
```

## Evaluation signals

- Balanced accuracy on validation cohorts (e.g., TANSBIG, UNT, UPP, NKI) exceeds 80%, including when trained with only 1000 randomly subsampled genes (Fig. S6 ablation).
- Functional spectra matrix has 22,596 columns (default MSigDB v7 gene sets) with one row per training sample.
- Deep features from get_DeepCC_features show compact within-subtype distribution and between-subtype separation, with fewer unclassifiable samples than baseline methods.
- Sensitivity, specificity, and balanced accuracy are higher than random forests, SVM, GBM, and multinomial logistic regression run on the same cohorts.
- Training completes without keras errors and the saved model loads via keras::load_model_hdf5 with matching label levels.

## Limitations

- DeepCC performs classification only, not clustering of cancer patients.
- Performance depends on comprehensive gene set prior knowledge; major characteristic signaling pathways or biological processes for each subtype should be included in the gene set database.
- Batch effects and unbalanced experimental designs limit expression correction methods (e.g., ComBat), which is why the functional spectra transformation is used instead.
- For single microarray samples, the inverseRescale parameter is suggested since the pre-defined TCGA references are RNA-Seq, and a customized reference profile is recommended for better results.

## Evidence

- [methods] train_DeepCC_model(trainData, trainLabels, epochs, dropout, activation_func, validation_split): "train_DeepCC_model(trainData, trainLabels, epochs, dropout, activation_func, validation_split)"
- [abstract] DeepCC provides a novel cancer classification framework that is platform independent, robust to missing data, and can be used for single sample prediction: "DeepCC provides a novel cancer classification framework that is platform independent, robust to missing data, and can be used for single sample prediction"
- [methods] By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra.: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
- [discussion] to use a widely adopted molecular subtyping systems, such as the CMS30 for CRC and intrinsic subtyping for breast cancer (implemented by PAM5031), to train a DeepCC classifier: "to use a widely adopted molecular subtyping systems, such as the CMS30 for CRC and intrinsic subtyping for breast cancer (implemented by PAM5031), to train a DeepCC classifier"
- [readme] The current version of DeepCC dependes on keras framework, with CPU and GPU supporting.: "The current version of DeepCC dependes on keras framework, with CPU and GPU supporting."
