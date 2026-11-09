---
name: svm-cell-type-annotation-cross-validation
description: Use when you have scATAC-seq profiles (e.g., PBMC multiome held-out test
  cells) whose RNA equivalents were translated by scButterfly-C and you need automatic
  cell type annotation with cross-validated performance metrics.
when_to_use_negative:
- When annotated scRNA-seq is already available and no cross-modality translation
  is needed — apply standard single-modality annotation tools instead
- When the modality pair is not RNA/ATAC but, e.g., scADT-seq or unpaired/perturbation
  data — scButterfly has separate schemes (scButterfly-T, scButterfly-B, scButterfly-T
  perturbation prediction)
- When the goal is spatial transcriptomics or single-cell Hi-C — the article lists
  these as future directions not yet supported
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3168
tools:
- name: scButterfly
  role: Dual-aligned VAE translator that converts held-out scATAC-seq profiles to
    RNA expression (scButterfly-C scheme with MultiVI_augmentation) prior to SVM classification
  repo: https://github.com/BioX-NKU/scButterfly
- name: MultiVI
  role: scvi-tools joint multi-omics model supplying cluster labels used by Butterfly.augmentation
    with MultiVI_augmentation
- name: Cellcano
  role: Alternative classifier optionally substituted for the SVM to annotate translated
    RNA profiles; also serves as a state-of-the-art baseline for comparison
- name: EpiAnno
  role: Dedicated scATAC-seq annotation method used as the baseline benchmark in Fig.
    4d,e
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:05ff6028ae2bdcf8edb8118e0720977bccd85f13ff88aa9c07ae3361e9a89299
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - Cellcano
  - EpiAnno
  - MultiVI
  - Python
  - scButterfly
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# svm-cell-type-annotation-cross-validation

## Summary

Annotates scATAC-seq cell types (including subtypes) by training an SVM classifier with five-fold cross-validation on scButterfly-C-translated RNA profiles, then scoring accuracy, Cohen's kappa, and macro/weighted F1 against dedicated scATAC-seq annotation baselines such as EpiAnno and Cellcano.

## When to use

Use when you have scATAC-seq profiles (e.g., PBMC multiome held-out test cells) whose RNA equivalents were translated by scButterfly-C and you need automatic cell type annotation with cross-validated performance metrics.

## When NOT to use

- When annotated scRNA-seq is already available and no cross-modality translation is needed — apply standard single-modality annotation tools instead
- When the modality pair is not RNA/ATAC but, e.g., scADT-seq or unpaired/perturbation data — scButterfly has separate schemes (scButterfly-T, scButterfly-B, scButterfly-T perturbation prediction)
- When the goal is spatial transcriptomics or single-cell Hi-C — the article lists these as future directions not yet supported

## Inputs

- Translated RNA expression profiles from scButterfly-C test_model on held-out ATAC-only cells (AnnData/matrix)
- MultiVI-integrated cell-type/cluster labels for the paired PBMC multiome training split
- Raw count AnnData objects of paired scRNA-seq and scATAC-seq data (n_obs x n_vars genes/peaks)

## Outputs

- Predicted cell type labels (including subtypes) for scATAC-seq test cells
- Cross-validated metrics table pbmc_annotation_metrics.csv with accuracy, Cohen's kappa, F1-macro, F1-weighted
- Comparison against EpiAnno and Cellcano baselines (Fig. 4d,e)

## How to apply

First run the scButterfly pipeline: load paired PBMC scRNA-seq/scATAC-seq multiome data with Butterfly.load_data, preprocess via Butterfly.data_preprocessing using RNA_data_preprocessing, ATAC_data_preprocessing, inverse_TFIDF and CLR_transform, then augment the training split with Butterfly.augmentation using MultiVI-derived cluster labels (the scButterfly-C scheme), and train the dual-aligned variational autoencoder with construct_model/train_model. Translate the held-out ATAC-only test cells to RNA expression with Butterfly.test_model, yielding A2R/R2A predictions. Train an SVM classifier on the translated RNA profiles under five-fold cross-validation to assign cell type labels, optionally repeating the classification with the Cellcano classifier as a sensitivity check. Compute accuracy, Cohen's kappa, F1-macro and F1-weighted with calculate_cluster_index, and compare against the EpiAnno and Cellcano baselines reported in Fig. 4d,e before writing the metrics to pbmc_annotation_metrics.csv.

## Related tools

- **scButterfly** (Dual-aligned VAE translator that converts held-out scATAC-seq profiles to RNA expression (scButterfly-C scheme with MultiVI_augmentation) prior to SVM classification) — https://github.com/BioX-NKU/scButterfly
- **MultiVI** (scvi-tools joint multi-omics model supplying cluster labels used by Butterfly.augmentation with MultiVI_augmentation)
- **Cellcano** (Alternative classifier optionally substituted for the SVM to annotate translated RNA profiles; also serves as a state-of-the-art baseline for comparison)
- **EpiAnno** (Dedicated scATAC-seq annotation method used as the baseline benchmark in Fig. 4d,e)

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id); butterfly.data_preprocessing(); butterfly.augmentation('MultiVI_augmentation'); butterfly.construct_model(chrom_list); butterfly.train_model(); A2R_predict, R2A_predict = butterfly.test_model()
```

## Evaluation signals

- pbmc_annotation_metrics.csv contains all four cross-validated metrics (accuracy, Cohen's kappa, F1-macro, F1-weighted) for the scButterfly-C + SVM pipeline
- Each metric is the result of five-fold cross-validation of the SVM on translated RNA profiles
- Metrics are consistent with the EpiAnno/Cellcano comparison shown in Fig. 4d,e, with scButterfly-C annotation achieving higher accuracy, kappa, and macro/weighted F1
- Annotations recover PBMC cell subtypes, not just coarse-level types
- Peaks must be sorted by chromosome and chrom_list passed to construct_model for the ATAC encoder to train correctly

## Limitations

- Annotation quality depends on the scButterfly-C translation step; performance for cross-cell-type and cross-organ translation is flagged as needing improvement via advanced machine learning with adaptive and interpretable parameters
- Using MultiVI_augmentation or cell_type_augmentation increases training time (README: 'this will cause more training time used, but promise better result for predicting')
- Augmentation needs cell_type in adata.obs; if absent, cell_type_augmentation silently falls back to MultiVI_augmentation
- Incorporating cell heterogeneity information from public bulk omics data or annotated cell atlases is only a proposed future improvement, not implemented

## Evidence

- [other] Combining scButterfly-C translation with a classifier accurately annotates scATAC-seq cell types, including subtypes, achieving higher accuracy, Cohen's kappa, and macro/weighted F1 scores than state-of-the-art methods (Fig. 4d,e).: [evidence span withheld — anchor status: not_found]
- [readme] methods API augmentation: [evidence span withheld — anchor status: external_doc]
- [readme] methods API test_model returns predictions: [evidence span withheld — anchor status: external_doc]
- [methods] methods calculate_cluster_index evaluation function: "calculate_cluster.calculate_cluster_index"
- [readme] readme raw count requirement for load_data: [evidence span withheld — anchor status: external_doc]
- [abstract] abstract claim of superiority over baselines: "we provide compelling evidence of scButterfly’s superiority over baseline methods in preserving cellular heterogeneity while translating datasets of various contexts and in revealing cell"
