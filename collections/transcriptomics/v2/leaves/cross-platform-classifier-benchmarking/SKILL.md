---
name: cross-platform-classifier-benchmarking
description: Use when a gene-expression subtype classifier trained on one platform
  (e.g., TCGA RNA-Seq) must be evaluated on independent cohorts from other platforms,
  or when robustness to unmatched/missing genes is assessed by random gene subsampling.
when_to_use_negative:
- The goal is de novo clustering or discovery of new cancer subtypes — DeepCC is specifically
  developed for classification, not clustering of cancer patients.
- Input is already a functional-spectra (pathway-activity) feature table rather than
  a gene-level expression matrix with Entrez identifiers, so gene-level subsampling
  and spectrum regeneration are impossible.
- No labeled training cohort exists for the target subtyping system (e.g., PAM50 labels),
  since DeepCC is a supervised framework.
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2648
tools:
- name: DeepCC
  role: 'Core R package: getFunctionalSpectra() projects each cohort into the MSigDB
    v7 pathway-activity space; train_DeepCC_model(), get_DeepCC_features(), and get_DeepCC_label()
    build and evaluate the cross-platform classifier'
  repo: https://github.com/CityUHK-CompBio/DeepCC
- name: keras
  role: Deep-learning backend on which train_DeepCC_model builds the ANN; pre-trained
    models are restored via keras::load_model_hdf5 in load_DeepCC_model
  repo: https://keras.rstudio.com/
- name: msigdbr
  role: Supplies the MSigDBr gene set object (via get_msigdbr()) as an alternative
    geneSets input to getFunctionalSpectra
- name: PAM50
  role: Widely adopted intrinsic breast subtyping system whose labels supervise the
    TCGA BRCA DeepCC classifier
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41389-019-0157-8
    title: 'DeepCC: a novel deep learning-based framework for cancer molecular subtype
      classification'
  source_content_hash: sha256:7e05b8f0af9b8ae5867bcd621a9cec268f7e3512a62729d4318a49bdc8452f0f
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
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2648
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cross-platform-classifier-benchmarking

## Summary

Benchmarks a DeepCC molecular subtype classifier across platforms: an ANN is trained on functional spectra of TCGA RNA-Seq data (PAM50 labels) and evaluated on independent microarray cohorts, with random gene-subsampling ablation quantifying robustness to cross-platform missing genes (balanced accuracy > 80% at 1000 genes).

## When to use

Use when a gene-expression subtype classifier trained on one platform (e.g., TCGA RNA-Seq) must be evaluated on independent cohorts from other platforms, or when robustness to unmatched/missing genes is assessed by random gene subsampling.

## When NOT to use

- The goal is de novo clustering or discovery of new cancer subtypes — DeepCC is specifically developed for classification, not clustering of cancer patients.
- Input is already a functional-spectra (pathway-activity) feature table rather than a gene-level expression matrix with Entrez identifiers, so gene-level subsampling and spectrum regeneration are impossible.
- No labeled training cohort exists for the target subtyping system (e.g., PAM50 labels), since DeepCC is a supervised framework.

## Inputs

- TCGA BRCA RNA-Seq expression matrix (n=517) as a data.frame with rows = samples and columns = Entrez gene identifiers
- Character vector of PAM50 training labels (NA allowed; DeepCC ignores them automatically)
- Independent validation-cohort expression matrices: TANSBIG, UNT, UPP, NKI (microarray, same Entrez-column format)
- MSigDB gene set collection — default MSigDB v7 (22,596 gene sets), or MSigDBr from msigdbr::get_msigdbr()
- Optional TCGA reference expression profile identifier (e.g., 'BRCA') for single-sample classification

## Outputs

- Trained keras DeepCC ANN model (deepcc_model) and deep feature matrix (df) from get_DeepCC_features
- Per-cohort predicted subtype labels (pred_labels) for TANSBIG, UNT, UPP, NKI
- Sensitivity, specificity, and balanced accuracy recorded per validation cohort
- Gene-subsampling robustness curve: balanced accuracy vs. number of genes, down to 1000 genes (Fig. S6 analogue)

## How to apply

Compute functional spectra with getFunctionalSpectra(eps) for the training cohort (TCGA BRCA RNA-Seq, n=517, PAM50 labels) and each validation cohort (TANSBIG, UNT, UPP, NKI), keeping the default MSigDB v7 collection (22,596 gene sets) so all platforms are projected into the same pathway-activity space — this transformation, not raw gene expression, is what confers platform independence. Train the ANN with train_DeepCC_model(fs, labels), tuning epochs, dropout, activation_func, and validation_split, and inspect deep features from get_DeepCC_features(deepcc_model, fs) for compact within-subtype and separated between-subtype distributions. Predict labels for the four validation cohorts with get_DeepCC_label(deepcc_model, new_fs), recording sensitivity, specificity, and balanced accuracy per cohort, and benchmark against random forests, SVM, GBM, and multinomial logistic regression. For the robustness ablation, randomly subsample genes at decreasing set sizes down to 1000, regenerate functional spectra, and retrain/re-evaluate at each size; the acceptance criterion from Fig. S6 is balanced accuracy > 80% at 1000 genes. For single microarray samples, use getFunctionalSpectrum(new_ep, refExp = 'BRCA') with inverseRescale enabled, since TCGA references are RNA-Seq.

## Related tools

- **DeepCC** (Core R package: getFunctionalSpectra() projects each cohort into the MSigDB v7 pathway-activity space; train_DeepCC_model(), get_DeepCC_features(), and get_DeepCC_label() build and evaluate the cross-platform classifier) — https://github.com/CityUHK-CompBio/DeepCC
- **keras** (Deep-learning backend on which train_DeepCC_model builds the ANN; pre-trained models are restored via keras::load_model_hdf5 in load_DeepCC_model) — https://keras.rstudio.com/
- **msigdbr** (Supplies the MSigDBr gene set object (via get_msigdbr()) as an alternative geneSets input to getFunctionalSpectra)
- **PAM50** (Widely adopted intrinsic breast subtyping system whose labels supervise the TCGA BRCA DeepCC classifier)

## Examples

```
fs <- getFunctionalSpectra(eps); deepcc_model <- train_DeepCC_model(fs, labels); pred_labels <- get_DeepCC_label(deepcc_model, getFunctionalSpectra(new_eps))
```

## Evaluation signals

- Acceptance criterion: balanced accuracy > 80% when the pipeline is rerun with only 1000 randomly subsampled genes (Fig. S6 reproduction).
- DeepCC sensitivity, specificity, and accuracy on all four validation cohorts (TANSBIG, UNT, UPP, NKI) should exceed random forests, SVM, GBM, and multinomial logistic regression.
- Schema check: fs is a data.frame with one row per sample and 22,596 columns under default MSigDB v7; expression inputs use Entrez identifiers as column names.
- get_DeepCC_features output shows compact within-subtype distribution and between-subtype separation (inspectable with vis_samples), with fewer unclassifiable samples than baseline classifiers.
- Performance degrades gracefully as genes are randomly removed — no collapse when subsampling from full gene set down to 1000 genes.

## Limitations

- DeepCC addresses classification only, not de novo clustering of cancer patients; it cannot discover new subtypes.
- Results depend on a comprehensive gene set prior: major subtype-characteristic pathways must be present in MSigDB to avoid bias, and results vary with MSigDB version (v5.0: 10,348 sets; v7: 22,596 sets).
- For single microarray samples scored against TCGA RNA-Seq references, distribution mismatch occurs; inverseRescale mitigates it only partially, and a customized reference from the same/similar platform is preferable.

## Evidence

- [abstract] robustness to missing data: "Simulation analysis based on random subsampling of genes demonstrated the robustness of DeepCC to missing data"
- [abstract] benchmark outperformance: "DeepCC classifiers and DeepCC single sample predictors both achieved overall higher sensitivity, specificity, and accuracy compared with other widely used classification methods"
- [methods] default gene set collection: "By default, DeepCC will use MSigDB v7 (22, 596 gene sets) to generate functional spectra."
- [methods] tunable hyperparameters: "train_DeepCC_model(trainData, trainLabels, epochs, dropout, activation_func, validation_split)"
- [discussion] label source for training: "to use a widely adopted molecular subtyping systems, such as the CMS30 for CRC and intrinsic subtyping for breast cancer (implemented by PAM5031), to train a DeepCC classifier"
- [readme] cross-platform single-sample caveat: "Note: if your single sample is microarray data, we strongly suggest you turn the parameter `inverseRescale` on, since TCGA is RNA-Seq data"
- [readme] required input format: "a data.frame containing gene expression profiles (each row represents one patient sample and the column names should be Entrez identifiers of genes)"
