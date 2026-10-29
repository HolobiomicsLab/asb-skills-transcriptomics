---
name: single-cell-quality-control-preprocessing
description: Use when raw single-cell counts (10x/h5ad) of representative samples
  must be quality-filtered and normalized, or when bulk cohort data must be preprocessed
  (normalize_total, log1p, top-HVG + markers, PCA) before scSemiProfiler VAE-GAN training
  or active-learning clustering.
when_to_use_negative:
- Input single-cell data has already been preprocessed by the dataset provider (e.g.,
  the COVID-19 dataset 'arrives already preprocessed'; the iMGL dataset passed the
  provider's stringent quality controls) — re-running cell filtering would be redundant
- Bulk deconvolution with a single-cell reference is the goal and the user already
  holds a curated reference; the QC here is specific to semi-profiling representative
  selection
- Data are not scRNA-seq counts — extension to proteomics or other modalities is future
  work and would require algorithm adjustments
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scSemiProfiler
  role: 'Provides the preprocessing utilities (initial setup for bulk HVG/PCA selection,
    and representative single-cell QC: background noise removal, <200-gene cell filter,
    10,000-count normalization, gene set scoring, variance weighting)'
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: MSigDB
  role: Source of all curated gene sets whose scores augment the preprocessed single-cell
    features
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:a57c80fd7a77ee6b8e677cd5e5a9767823a23a836956977723f0be1bdd0f8206
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - MSigDB
  - scSemiProfiler
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# single-cell-quality-control-preprocessing

## Summary

Quality-control and preprocessing routine used in the scSemiProfiler pipeline to clean raw single-cell counts of representative samples (background noise removal, low-quality cell and doublet filtering, normalization) and to prepare bulk cohort data (library-size normalization, log1p, HVG selection, PCA) for deep generative inference.

## When to use

Use when raw single-cell counts (10x/h5ad) of representative samples must be quality-filtered and normalized, or when bulk cohort data must be preprocessed (normalize_total, log1p, top-HVG + markers, PCA) before scSemiProfiler VAE-GAN training or active-learning clustering.

## When NOT to use

- Input single-cell data has already been preprocessed by the dataset provider (e.g., the COVID-19 dataset 'arrives already preprocessed'; the iMGL dataset passed the provider's stringent quality controls) — re-running cell filtering would be redundant
- Bulk deconvolution with a single-cell reference is the goal and the user already holds a curated reference; the QC here is specific to semi-profiling representative selection
- Data are not scRNA-seq counts — extension to proteomics or other modalities is future work and would require algorithm adjustments

## Inputs

- Raw single-cell count matrix of representative samples (h5ad, e.g., from 10x sequencing)
- Preprocessed bulk sequencing data for the cohort (e.g., the 124-sample COVID-19 bulk cohort)
- Gene set collections: MSigDB curated gene sets and Ernst et al. gene sets
- Quality-control plots used to manually flag doublets and low-quality cells

## Outputs

- Cleaned, normalized, log1p-transformed representative single-cell matrices annotated with MSigDB/Ernst gene set scores and variance-based feature weights
- Dimensionality-reduced bulk cohort representation (top 6000 HVGs + markers, PCA to ~100 PCs) ready for K-Means clustering into B clusters
- Preprocessed AnnData objects consumed by scSemiProfiler's VAE-GAN pretraining and active learning modules

## How to apply

For the bulk cohort, run the scSemiProfiler initial setup: library-size normalization followed by log1p transformation, select the top 6000 highly variable genes plus manually added markers, and reduce dimensionality with PCA (e.g., to 100 PCs when the COVID-19 cohort has 124 samples). For representative single-cell data, zero out expression values below 0.1% of library size (background noise removal), remove cells expressing fewer than 200 genes per SCANPY's default settings, drop doublets expressing an unusually high number of genes identified via quality-control plots, then normalize to 10,000 counts per cell with log1p. Finally, augment the cleaned matrix with gene set scores from all curated MSigDB and Ernst et al. gene sets and apply variance-based feature importance weighting so higher-variance features carry more weight. These steps are the foundation ('Representative Single-Cell Profiling And Processing') for subsequent representative reconstruction and in silico inference.

## Related tools

- **scSemiProfiler** (Provides the preprocessing utilities (initial setup for bulk HVG/PCA selection, and representative single-cell QC: background noise removal, <200-gene cell filter, 10,000-count normalization, gene set scoring, variance weighting)) — https://github.com/mcgilldinglab/scSemiProfiler
- **MSigDB** (Source of all curated gene sets whose scores augment the preprocessed single-cell features)

## Examples

```
import scanpy as sc; sc.pp.filter_cells(adata, min_genes=200); sc.pp.normalize_total(adata, target_sum=10000); sc.pp.log1p(adata)
```

## Evaluation signals

- No cells remain in the filtered representative matrices that express fewer than 200 genes; values below 0.1% of each cell's library size are set to zero
- After normalization, every cell sums to 10,000 counts and values are log1p-transformed
- The bulk feature space contains the top 6000 highly variable genes plus manually added markers, and PCA yields at most 100 components (≤ sample count)
- Cleaned AnnData objects carry MSigDB/Ernst gene set score columns and variance-based feature importance weights before VAE-GAN pretraining
- Doublets flagged by quality-control plots (unusually high gene counts) are excluded from the representative reference data

## Limitations

- Doublet and low-quality cell removal depends on manual inspection of quality-control plots
- Datasets arriving already preprocessed (e.g., the COVID-19 dataset per the source publication) bypass much of this QC
- The preprocessing scheme is scRNA-seq-specific; adapting it to other modalities (e.g., proteomics) would require algorithm adjustments and new benchmarking

## Evidence

- [methods] library size normalization + log1p + HVG selection: "we perform library size normalization followed by a log1p transformation"
- [methods] highly variable genes and markers: "we identify and select highly variable genes and relevant markers for further analysis"
- [methods] background noise removal threshold: "expression values that are extremely low (below 0.1% of the library size) are adjusted to zero"
- [methods] low-quality cell filter: [evidence span withheld — anchor status: not_found]
- [methods] gene set score augmentation: "gene set scores are calculated using all curated gene sets from MSigDB59 and Ernst et al.91"
- [methods] variance-based feature weighting: "we adopt a feature weighting method, giving more weight to features with higher variance"
- [readme] README overview of semi-profiling cost scenario: [evidence span withheld — anchor status: external_doc]
