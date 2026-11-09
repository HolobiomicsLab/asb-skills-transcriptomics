---
name: single-cell-expression-normalization-and-hvg-selection
description: Use when a raw single-cell RNA-seq count matrix (.h5ad or GEO counts)
  must be quality-filtered, TPM-normalized, log1p-transformed, and reduced to ~2000
  highly variable genes before DELVE feature-module identification.
when_to_use_negative:
- The input is already a preprocessed .h5ad object downloaded from the authors' Zenodo
  repository — re-filtering would discard valid cells/genes
- The task requires selecting features on noisy data without preprocessing (e.g.,
  unfiltered matrices where noisy features dominate variance) is the goal — the article
  notes similarity-based methods fail on the full unfiltered data, so skipping normalization
  undermines DELVE
- The data are bulk transcriptomics or non-single-cell profiles where per-cell filtering,
  TPM normalization, and HVG selection do not apply
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DELVE
  role: Downstream unsupervised feature selection method that consumes the normalized
    HVG matrix to identify dynamic feature modules recapitulating cellular trajectories
  repo: https://github.com/jranek/delve
- name: Python
  role: Implementation environment for the filtering, TPM normalization, log1p transformation,
    and highly variable gene selection steps (scanpy-based .h5ad handling)
  repo: https://github.com/jranek/delve
- name: UMAP (uniform manifold approximation and projection)
  role: Embedding used downstream of the preprocessed matrix to visualize cell states
    and trajectory structure
- name: sketchKH
  role: Optional kernel-herding subsampling of cells from preprocessed .had samples,
    providing sketched neighborhoods analogous to DELVE's num_subsamples step
  repo: https://github.com/CompCy-lab/SketchKH
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:dbbf7a7b85211c009646fedc3c21acdc880e4ec971c550e2f7949cff9b3c1aed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - Python
  - UMAP (uniform manifold approximation and projection)
  - sketchKH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-expression-normalization-and-hvg-selection

## Summary

Quality-filter, TPM-normalize, log1p-transform, and restrict a single-cell RNA-seq count matrix (e.g., mouse splenic CD8+ T cell LCMV data from GEO GSE131847) to a small set of highly variable genes so that downstream trajectory-preserving feature selection with DELVE operates on informative features rather than noise.

## When to use

Use when a raw single-cell RNA-seq count matrix (.h5ad or GEO counts) must be quality-filtered, TPM-normalized, log1p-transformed, and reduced to ~2000 highly variable genes before DELVE feature-module identification.

## When NOT to use

- The input is already a preprocessed .h5ad object downloaded from the authors' Zenodo repository — re-filtering would discard valid cells/genes
- The task requires selecting features on noisy data without preprocessing (e.g., unfiltered matrices where noisy features dominate variance) is the goal — the article notes similarity-based methods fail on the full unfiltered data, so skipping normalization undermines DELVE
- The data are bulk transcriptomics or non-single-cell profiles where per-cell filtering, TPM normalization, and HVG selection do not apply

## Inputs

- Raw single-cell RNA-seq count matrix (e.g., GEO accession GSE131847 mouse splenic CD8+ T cell LCMV counts, cells x genes)
- Cell-level metadata including mitochondrial read fractions
- Optionally, a preprocessed .h5ad file from the authors' Zenodo repository (https://zenodo.org/records/10534873)

## Outputs

- Filtered and normalized single-cell expression matrix (log1p-transformed, HVG-restricted; e.g., 29893 cells x 2000 highly variable genes)
- List of retained highly variable genes used as features for downstream DELVE module identification

## How to apply

Start from the raw counts (e.g., GSE131847 mouse splenic CD8+ T cell LCMV data) and filter low-quality cells and genes: drop cells with >20% mitochondrial reads and genes observed in <3 cells or with <400 counts. TPM-normalize each cell, apply a log(x+1) transform, and then run highly variable gene selection to retain a compact feature set (the CD8+ T cell workflow yields 29893 cells x 2000 genes). This filtering matters because, as the paper argues, 'appropriate filtering of noisy, information-poor, or irrelevant features can serve as a crucial and necessary step' prior to trajectory inference, and variance-based selection alone 'can be overwhelmed by noisy or irrelevant features that dominate data variance'. The preprocessed matrix is then the direct input to DELVE module identification (k=10 nearest neighbors, num_subsamples=1000 sketched neighborhoods, n_clusters=5, after PCA with n_pcs=50). Decision point: if a preprocessed .h5ad is already available from the authors' Zenodo repository (records/10534873), it can be read in directly instead of re-running this preprocessing.

## Related tools

- **DELVE** (Downstream unsupervised feature selection method that consumes the normalized HVG matrix to identify dynamic feature modules recapitulating cellular trajectories) — https://github.com/jranek/delve
- **Python** (Implementation environment for the filtering, TPM normalization, log1p transformation, and highly variable gene selection steps (scanpy-based .h5ad handling)) — https://github.com/jranek/delve
- **UMAP (uniform manifold approximation and projection)** (Embedding used downstream of the preprocessed matrix to visualize cell states and trajectory structure)
- **sketchKH** (Optional kernel-herding subsampling of cells from preprocessed .had samples, providing sketched neighborhoods analogous to DELVE's num_subsamples step) — https://github.com/CompCy-lab/SketchKH

## Examples

```
sc.pp.filter_genes(adata, min_cells=3); sc.pp.normalize_total(adata, target_sum=1e4); sc.pp.log1p(adata); sc.pp.highly_variable_genes(adata, n_top_genes=2000)
```

## Evaluation signals

- Cell/gene filter thresholds are respected: no retained cell has >20% mitochondrial reads; every retained gene appears in >=3 cells and has >=400 counts
- Expression values are non-negative and consistent with a log(x+1) transform of TPM-normalized counts
- HVG selection retains ~2000 features; the CD8+ T cell LCMV case yields a 29893 cells x 2000 genes matrix
- Downstream DELVE run on this matrix recovers three dynamic modules plus one static module (k=10 NN, num_subsamples=1000, n_clusters=5, n_pcs=50, 10 KMeans++ initializations)
- The normalized matrix loads cleanly as an .h5ad anndata object with cells x features dimensions matching expectations

## Limitations

- Feature selection can greatly bias interpretation of the underlying cellular trajectory; careful consideration should be made when performing feature selection for trajectory analysis
- Unsupervised variance-based selection can be overwhelmed by noisy or irrelevant features that dominate data variance, and is insensitive to lineage-specific features with small variance and gradual expression progression
- Similarity-based methods rely on the cell-similarity graph from the full dataset and can fail to identify relevant features when the number of noisy features outweighs the number of informative ones — hence normalization/HVG filtering is a prerequisite rather than optional

## Evidence

- [intro] filtering is crucial before trajectory inference: "appropriate filtering of noisy, information-poor, or irrelevant features can serve as a crucial and necessary step for cell type identification, inference of dynamic phenotypes"
- [intro] variance-based selection is overwhelmed by noisy features: "they can be overwhelmed by noisy or irrelevant features that dominate data variance, and (2) are insensitive to lineage-specific features"
- [readme] preprocessed .h5ad files available on Zenodo: "You can download all of the preprocessed single-cell datasets (`.h5ad` files) from the [Zenodo](https://zenodo.org/records/10534873) repository."
- [readme] DELVE installed as an open-source python package: "You can install the package and necessary dependencies with `pip` by,
``npip install delve-fs
"
- [abstract] DELVE is an open-source python feature selection package: "DELVE is available as an open-source python package"
