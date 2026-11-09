---
name: sctransform-sample-integration
description: Use when multiple scRNAseq samples (e.g., a time series such as GSE223109)
  must be normalized and integrated before joint clustering, and per-sample SCTransform
  normalization plus a shared integration feature set is needed to remove batch effects.
when_to_use_negative:
- You have only a single sample with no batch effects to correct — SCTransform alone
  suffices without SelectIntegrationFeatures-based integration.
- Input is already a normalized/integrated expression matrix or pseudobulk table rather
  than raw CellRanger output requiring per-sample SCTransform normalization.
- Prediction of circadian time from an already-integrated pseudobulk sample is the
  goal — that is tauFisher's train/test pipeline, not sample integration.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellRanger
  role: Aligns raw scRNAseq output against the MM10 reference genome (version 3.1.0)
    to produce per-sample count matrices prior to normalization
- name: Seurat
  role: Provides SCTransform per-sample normalization, SelectIntegrationFeatures (3250
    features), PCA, and Louvain clustering on the integrated dataset
- name: tauFisher
  role: Downstream package from the same paper that consumes the processed single-cell/pseudobulk
    data for circadian time prediction
  repo: github.com/micnngo/tauFisher
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48041-6
    title: tauFisher predicts circadian time from a single sample of bulk and single-cell
      pseudobulk transcriptomic data
  source_content_hash: sha256:4cb9f3c26077b274a81f3abf6098e18f8fe4852587b205dfa17aaef5bc40ae21
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellRanger
  - Seurat
  - tauFisher
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# sctransform-sample-integration

## Summary

Normalize each single-cell RNAseq sample independently with Seurat's SCTransform, then batch-integrate samples using 3250 SelectIntegrationFeatures so that downstream PCA/Louvain clustering reflects biology rather than sample-specific technical effects. Used in the tauFisher paper to build an integrated mouse dermis scRNAseq time series (GSE223109, MM10) for cell-type annotation.

## When to use

Use when multiple scRNAseq samples (e.g., a time series such as GSE223109) must be normalized and integrated before joint clustering, and per-sample SCTransform normalization plus a shared integration feature set is needed to remove batch effects.

## When NOT to use

- You have only a single sample with no batch effects to correct — SCTransform alone suffices without SelectIntegrationFeatures-based integration.
- Input is already a normalized/integrated expression matrix or pseudobulk table rather than raw CellRanger output requiring per-sample SCTransform normalization.
- Prediction of circadian time from an already-integrated pseudobulk sample is the goal — that is tauFisher's train/test pipeline, not sample integration.

## Inputs

- Raw single-cell sequencing output from GSE223109 (mouse dermis scRNAseq time series) processed by CellRanger 3.1.0 with MM10 reference
- Per-sample cell barcode matrices QC-filtered to 850-7800 detected features and <13% mitochondrial genes
- A list of Seurat objects, one per sample/time point

## Outputs

- An integrated Seurat object over 3250 SelectIntegrationFeatures features
- PCA embeddings and Louvain cluster assignments for all QC-passing cells
- Cell-type annotations (fibroblasts 12,649; immune 3,353; muscle 722; endothelial 142; 16,866 total) tabulated as cell_counts.csv

## How to apply

First align raw sequencing output with CellRanger version 3.1.0 against the MM10 reference genome. Quality-control cell barcodes per sample, keeping cells with 850-7800 detected features and less than 13% mitochondrial genes. Run the SCTransform function on each sample separately, then select 3250 integration features using SelectIntegrationFeatures across the sample list and integrate the samples. On the integrated dataset, run principal component analysis and generate clusters with the Louvain algorithm; annotate clusters with canonical markers (Col1a1 for fibroblasts, Acta1 for muscle, Fabp4 for endothelial cells, Cd52/Cd74 for immune cells). The rationale is that per-sample SCTransform removes sample-specific depth/variance effects while a shared 3250-feature space anchors cells across samples for joint clustering.

## Related tools

- **CellRanger** (Aligns raw scRNAseq output against the MM10 reference genome (version 3.1.0) to produce per-sample count matrices prior to normalization)
- **Seurat** (Provides SCTransform per-sample normalization, SelectIntegrationFeatures (3250 features), PCA, and Louvain clustering on the integrated dataset)
- **tauFisher** (Downstream package from the same paper that consumes the processed single-cell/pseudobulk data for circadian time prediction) — github.com/micnngo/tauFisher

## Examples

```
obj <- SCTransform(obj); features <- SelectIntegrationFeatures(object.list = sample_list, nfeatures = 3250)
```

## Evaluation signals

- Total QC-passing cells equal 16,866 after filtering with 850-7800 features and <13% mitochondrial reads.
- Per-sample normalization ran without error and SelectIntegrationFeatures returned exactly 3250 integration features.
- Integrated PCA + Louvain clustering separates four major cell types whose marker genes are specific: Col1a1+ fibroblasts, Acta1+ muscle, Fabp4+ endothelial, Cd52/Cd74+ immune.
- Cell counts per type match the reported composition: fibroblasts 12,649, immune 3,353, muscle 722, endothelial 142; table saved as cell_counts.csv.
- Cluster proportions are not dominated by a single sample/time point, indicating batch integration succeeded.

## Limitations

- QC thresholds (850-7800 features, <13% mitochondrial) are dataset-specific; applying them verbatim to other tissues may over- or under-filter.
- tauFisher downstream cannot overcome high scRNAseq dropout and thus cannot predict circadian time at single-cell resolution, only pseudobulk.
- README notes tauFisher requires R >=3.5.0 (tested on R 4.0.5) with packages MetaCycle, fda, nnet, etc.; Seurat/CellRanger versions must match those reported (CellRanger 3.1.0, MM10) for reproducibility.

## Evidence

- [discussion] The SCTransform function was performed on each sample and 3250 integration features were selected using SelectIntegrationFeatures: "The SCTransform function was performed on each sample and 3250 integration features were selected using SelectIntegrationFeatures"
- [discussion] Cells with 850-7800 features and less than 13% of mitochondrial genes were kept.: "Cells with 850-7800 features and less than 13% of mitochondrial genes were kept."
- [discussion] Principal component analysis was then done on the integrated dataset and the Louvain algorithm was used to generate the clusters: "Principal component analysis was then done on the integrated dataset and the Louvain algorithm was used to generate the clusters"
- [discussion] We used CellRanger version 3.1.0 with MM10 reference to process the raw sequencing output.: "We used CellRanger version 3.1.0 with MM10 reference to process the raw sequencing output."
- [readme] tauFisher requires R >=3.5.0 and has been tested on R 4.0.5.: [evidence span withheld — anchor status: external_doc]
