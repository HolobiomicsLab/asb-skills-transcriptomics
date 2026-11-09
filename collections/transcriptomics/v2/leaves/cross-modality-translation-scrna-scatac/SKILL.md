---
name: cross-modality-translation-scrna-scatac
description: Use when paired or unpaired scRNA-seq and scATAC-seq data are available
  and ATAC-only profiles must be converted to RNA-like profiles, e.g., to annotate
  scATAC-seq cell types via translated RNA expression.
when_to_use_negative:
- Data are single-modality with no paired or unpaired counterpart modality available
  for translator training — scButterfly requires RNA and ATAC (or other modality)
  inputs
- Spatial transcriptomics or single-cell Hi-C data are the target modalities — the
  article names these only as future extensions not yet supported
- The goal is annotating RNA profiles directly; the documented annotation pipeline
  targets scATAC-seq cell type annotation via translated RNA profiles
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: scButterfly
  role: 'Core cross-modality translator: dual-aligned variational autoencoders plus
    data augmentation, used via load_data, data_preprocessing, augmentation, construct_model,
    train_model and test_model APIs'
  repo: https://github.com/BioX-NKU/scButterfly
- name: MultiVI
  role: Joint multi-omics analysis from scvi-tools used to derive cluster labels for
    the scButterfly-C data augmentation scheme
  repo: https://docs.scvi-tools.org/en/stable/tutorials/notebooks/MultiVI_tutorial.html
- name: scanpy
  role: Single-cell Python library whose AnnData objects and Leiden clustering feed
    the scButterfly pipeline inputs and augmentation labels
  repo: https://scanpy.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:20ecbe26a020c5b2d5d8da505445ceb36f4823c2737f35d4b0f3829373a57672
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - MultiVI
  - Python
  - scButterfly
  - scanpy
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cross-modality-translation-scrna-scatac

## Summary

Translate single-cell ATAC-seq profiles into RNA expression using scButterfly's dual-aligned variational autoencoder, enabling downstream tasks such as automatic cell type annotation of scATAC-seq data (e.g., PBMC multiome) without direct RNA measurement.

## When to use

Use when paired or unpaired scRNA-seq and scATAC-seq data are available and ATAC-only profiles must be converted to RNA-like profiles, e.g., to annotate scATAC-seq cell types via translated RNA expression.

## When NOT to use

- Data are single-modality with no paired or unpaired counterpart modality available for translator training — scButterfly requires RNA and ATAC (or other modality) inputs
- Spatial transcriptomics or single-cell Hi-C data are the target modalities — the article names these only as future extensions not yet supported
- The goal is annotating RNA profiles directly; the documented annotation pipeline targets scATAC-seq cell type annotation via translated RNA profiles

## Inputs

- AnnData object of raw scRNA-seq count matrix (n_obs x n_vars, cells x genes)
- AnnData object of raw scATAC-seq count matrix (n_obs x n_vars, cells x peaks)
- train_id, test_id, and optional validation_id lists of cell IDs
- chrom_list: a list of peaks count for each chromosome, with peaks sorted by chromosomes
- augmentation type: 'cell_type_augmentation', 'MultiVI_augmentation', or None

## Outputs

- A2R_predict: translated RNA expression profiles from ATAC-seq test cells
- R2A_predict: translated ATAC profiles from RNA-seq test cells
- Cell type annotation metrics: accuracy, Cohen's kappa, F1-macro and F1-weighted (e.g., pbmc_annotation_metrics.csv)
- Trained scButterfly model checkpoints (default './model')

## How to apply

Load the raw paired PBMC scRNA-seq/scATAC-seq multiome count matrices as AnnData objects with Butterfly.load_data, splitting cells into train/test/validation IDs, then preprocess with Butterfly.data_preprocessing, which internally applies RNA_data_preprocessing, ATAC_data_preprocessing, inverse_TFIDF and CLR_transform from scButterfly.data_processing. Decide between augmentation schemes: scButterfly-C uses MultiVI-derived cluster labels via butterfly.augmentation(aug_type='MultiVI_augmentation') to expand the training split (preferred when cell_type labels are unavailable), while scButterfly-T uses cell_type labels and scButterfly-B (aug_type=None) uses original data only. Construct the dual-aligned VAE with Butterfly.construct_model(chrom_list), supplying a list of peak counts per chromosome with peaks sorted by chromosome, then train with Butterfly.train_model and translate held-out ATAC-only test profiles using Butterfly.test_model, which returns A2R_predict and R2A_predict. For the annotation application, classify the translated RNA profiles with an SVM in five-fold cross-validation and score accuracy, Cohen's kappa, F1-macro and F1-weighted using calculate_cluster_index, comparing against EpiAnno and Cellcano baselines.

## Related tools

- **scButterfly** (Core cross-modality translator: dual-aligned variational autoencoders plus data augmentation, used via load_data, data_preprocessing, augmentation, construct_model, train_model and test_model APIs) — https://github.com/BioX-NKU/scButterfly
- **MultiVI** (Joint multi-omics analysis from scvi-tools used to derive cluster labels for the scButterfly-C data augmentation scheme) — https://docs.scvi-tools.org/en/stable/tutorials/notebooks/MultiVI_tutorial.html
- **scanpy** (Single-cell Python library whose AnnData objects and Leiden clustering feed the scButterfly pipeline inputs and augmentation labels) — https://scanpy.readthedocs.io/en/stable/

## Examples

```
from scButterfly.butterfly import Butterfly; b = Butterfly(); b.load_data(RNA_data, ATAC_data, train_id, test_id); b.data_preprocessing(); b.augmentation('MultiVI_augmentation'); b.construct_model(chrom_list); b.train_model(); A2R_predict, R2A_predict = b.test_model()
```

## Evaluation signals

- pbmc_annotation_metrics.csv contains four cross-validated metrics (accuracy, Cohen's kappa, F1-macro, F1-weighted) for the scButterfly-C SVM annotation pipeline
- Translated-profile annotation metrics are higher than the EpiAnno and Cellcano baselines reported in Fig. 4d,e, including cell subtype annotation
- butterfly.test_model returns both A2R_predict and R2A_predict without error on held-out test cells
- ATAC peaks are sorted by chromosome and chrom_list peak counts match the ATAC AnnData variable groups before construct_model
- t-SNE visualization via draw_tsne shows translated profiles preserving cellular heterogeneity consistent with the source labels

## Limitations

- Incorporating cell heterogeneity information from public bulk omics data and/or annotated cell atlas is only a proposed future improvement, not implemented
- Prior knowledge of gene regulatory mechanisms to bridge modalities is proposed as a future improvement
- Spatial transcriptomics and single-cell Hi-C modalities are named as future directions not yet supported
- Performance for cross-cell-type and cross-organ translation is flagged as needing improvement via advanced machine learning techniques with adaptive and interpretable model parameters

## Evidence

- [abstract] scButterfly is a versatile single-cell cross-modality translation method based on dual-aligned variational autoencoders and data augmentation schemes: "Here, we propose scButterfly, a versatile single-cell cross-modality translation method based on dual-aligned variational autoencoders and data augmentation schemes."
- [abstract] the extensive applications of scButterfly for integrative multi-omics analysis of single-modality data, data enhancement of poor-quality single-cell multi-omics, and automatic cell type annotation: "the extensive applications of scButterfly for integrative multi-omics analysis of single-modality data, data enhancement of poor-quality single-cell multi-omics, and automatic cell type annotation of"
- [methods] butterfly.Butterfly.load_data: "butterfly.Butterfly.load_data"
- [readme] chrom_list: a list of peaks count for each chromosome, remember to sort peaks with chromosomes: [evidence span withheld — anchor status: external_doc]
- [readme] If you choose MultiVI_augmentation, scButterfly-C (Cluster) will train a MultiVI model first: [evidence span withheld — anchor status: external_doc]
- [discussion] we can extend it to flexibly accommodate additional modalities, such as spatial transcriptomics55 and single-cell Hi-C data56: "we can extend it to flexibly accommodate additional modalities, such as spatial transcriptomics55 and single-cell Hi-C data56"
