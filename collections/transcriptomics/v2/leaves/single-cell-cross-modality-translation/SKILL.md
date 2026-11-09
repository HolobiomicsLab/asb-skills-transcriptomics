---
name: single-cell-cross-modality-translation
description: Use when raw count AnnData matrices from a single modality must be translated
  into another (scRNA-seq, scATAC-seq peaks, or ADT), including unseen cell types,
  batch-shifted data, or chained multi-modality inference.
when_to_use_negative:
- When the modalities of interest are spatial transcriptomics or single-cell Hi-C
  — the article names these as future directions not yet supported
- When multi-modal paired measurements already exist and no translation/imputation
  is needed — translation is for inferring/synthesizing profiles from single-modality
  data
- When cell heterogeneity information from public bulk omics data or annotated cell
  atlases must be incorporated — this is only proposed as a future improvement
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scButterfly
  role: Core dual-aligned VAE framework providing load_data, data_preprocessing, augmentation,
    construct_model, train_model, and test_model APIs for cross-modality translation
  repo: https://github.com/BioX-NKU/scButterfly
- name: BABEL
  role: Baseline autoencoder-based translator used for benchmarking on identical cell-type-disjoint
    splits
- name: Polarbear
  role: Baseline VAE-stitching translator used for benchmarking translation performance
- name: JAMIE
  role: Baseline joint-VAE method for multi-modal imputation used in cross-validation
    comparisons
- name: sciPENN
  role: Baseline RNA-to-protein predictor used in combination with BABEL/Polarbear
    for consecutive translation comparison
- name: MultiVI
  role: scvi-tools model providing cluster labels for scButterfly-C MultiVI_augmentation
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:8f5cd0070912faa2333c400a96aac017b79544b67905bdd0ae6bd6c8137b4a72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - BABEL
  - JAMIE
  - MultiVI
  - Polarbear
  - Python
  - scButterfly
  - sciPENN
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9459459459459459
n_claims: 37
---

# single-cell-cross-modality-translation

## Summary

Translates single-cell profiles between modalities (scRNA-seq ↔ scATAC-seq, scRNA-seq ↔ scADT/ADT) using scButterfly's dual-aligned variational autoencoders, with data augmentation variants (B/T/C) to preserve cellular heterogeneity, and supports chained consecutive translation (epigenome → transcriptome → proteome).

## When to use

Use when raw count AnnData matrices from a single modality must be translated into another (scRNA-seq, scATAC-seq peaks, or ADT), including unseen cell types, batch-shifted data, or chained multi-modality inference.

## When NOT to use

- When the modalities of interest are spatial transcriptomics or single-cell Hi-C — the article names these as future directions not yet supported
- When multi-modal paired measurements already exist and no translation/imputation is needed — translation is for inferring/synthesizing profiles from single-modality data
- When cell heterogeneity information from public bulk omics data or annotated cell atlases must be incorporated — this is only proposed as a future improvement

## Inputs

- Raw count AnnData objects (n_obs × n_vars) of scRNA-seq genes and scATAC-seq peaks (e.g., MB, MCC, MK, PBMC, BMMC datasets)
- Paired scRNA-seq/ADT CITE-seq AnnData (e.g., CITE_BMMC)
- Train/test/validation cell-ID lists (validation defaults to 20% of train_id)
- chrom_list: list of peak counts per chromosome, peaks sorted by chromosome
- Ground-truth cell-type labels for clustering evaluation

## Outputs

- Predicted profiles in the target modality (A2R_predict, R2A_predict from Butterfly.test_model)
- Chained predicted ADT profiles from consecutive scATAC-seq → scRNA-seq → ADT translation
- Clustering metrics (AMI, ARI, HOM, NMI) per method/dataset/fold (e.g., clustering_metrics_comparison.csv, consecutive_translation_metrics.tsv)
- t-SNE embeddings rendered via draw_cluster.draw_tsne (e.g., fig3a_clustering_comparison.png)

## How to apply

Load raw count matrices with Butterfly.load_data(RNA_data, ATAC_data, train_id, test_id, validation_id), then preprocess via butterfly.data_preprocessing (or the finer-grained RNA_data_preprocessing, ATAC_data_preprocessing, and CLR_transform for ADT). Optionally apply augmentation — 'cell_type_augmentation' for scButterfly-T or 'MultiVI_augmentation' for scButterfly-C — since augmentation improves translation and reveals heterogeneity in complex tissues. Construct the model with butterfly.construct_model(chrom_list), where chrom_list is the per-chromosome peak counts and peaks must be sorted by chromosome, then train with butterfly.train_model (dual-aligned VAEs; train_model_cite for the RNA-to-ADT CITE direction). Run butterfly.test_model to obtain A2R_predict and R2A_predict; for consecutive translation (BMMC/CITE_BMMC), feed the epigenome-to-transcriptome Model.test predictions into the transcriptome-to-proteome Model.test. Evaluate by computing AMI, ARI, HOM, and NMI via calculate_cluster.calculate_cluster_index against ground-truth cell-type labels, and compare to baselines such as BABEL, Polarbear, JAMIE, and sciPENN.

## Related tools

- **scButterfly** (Core dual-aligned VAE framework providing load_data, data_preprocessing, augmentation, construct_model, train_model, and test_model APIs for cross-modality translation) — https://github.com/BioX-NKU/scButterfly
- **BABEL** (Baseline autoencoder-based translator used for benchmarking on identical cell-type-disjoint splits)
- **Polarbear** (Baseline VAE-stitching translator used for benchmarking translation performance)
- **JAMIE** (Baseline joint-VAE method for multi-modal imputation used in cross-validation comparisons)
- **sciPENN** (Baseline RNA-to-protein predictor used in combination with BABEL/Polarbear for consecutive translation comparison)
- **MultiVI** (scvi-tools model providing cluster labels for scButterfly-C MultiVI_augmentation)

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id, validation_id); butterfly.data_preprocessing(); butterfly.augmentation('cell_type_augmentation'); butterfly.construct_model(chrom_list); butterfly.train_model(); A2R_predict, R2A_predict = butterfly.test_model()
```

## Evaluation signals

- Clustering indices (AMI, ARI, HOM, NMI) computed with calculate_cluster.calculate_cluster_index should exceed those of baselines (BABEL, Polarbear, JAMIE, or BABEL+sciPENN / Polarbear+sciPENN) on the same splits
- In three-fold cell-type-disjoint cross-validation, all three scButterfly variants (B/T/C) should outperform baselines on unseen cell types, matching the ordering in Fig. 3a
- Predicted ADT profiles from consecutive translation should support differential expression of proteins (DEP) analysis and coherent t-SNE clustering against BMMC ground-truth labels
- Preprocessing must start from raw count matrices — verify inputs are raw counts before load_data
- Peaks must be sorted by chromosome and chrom_list counts must match the ATAC feature dimensionality

## Limitations

- Translation of single-modality data with inter-sample variations (batch effects) or different biological contexts (novel cell types) is a prevalent challenge not satisfactorily resolved in general
- Performance for cross-cell-type and cross-organ translation is flagged as needing improvement via adaptive and interpretable model parameters
- Spatial transcriptomics and single-cell Hi-C modalities are not yet supported; incorporating bulk omics/cell atlas heterogeneity or gene-regulatory prior knowledge are future directions only
- Data augmentation (cell_type_augmentation or MultiVI_augmentation) increases training time, though it promises better prediction results

## Evidence

- [abstract] scButterfly is a versatile single-cell cross-modality translation method: "Here, we propose scButterfly, a versatile single-cell cross-modality translation method based on dual-aligned variational autoencoders and data augmentation schemes."
- [abstract] generalization to consecutive translation: "scButterfly can be generalized to unpaired data training, perturbation-response analysis, and consecutive translation."
- [readme] test_model prediction API: [evidence span withheld — anchor status: external_doc]
- [readme] load_data raw counts and split IDs: [evidence span withheld — anchor status: external_doc]
- [readme] augmentation variants: [evidence span withheld — anchor status: external_doc]
- [readme] chrom_list construction requirement: [evidence span withheld — anchor status: external_doc]
