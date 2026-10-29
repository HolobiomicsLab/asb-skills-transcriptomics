---
name: scrna-cpm-normalization-log1p-scaling
description: Use when a raw scRNA-seq count matrix (e.g., an AnnData .h5ad such as
  the 5273-cell HBMP3-spleen-CC2 HuBMAP dataset) must be transformed to a scaled log-CPM
  representation before PCA, UMAP, or Leiden clustering.
when_to_use_negative:
- The matrix is already normalized or scaled (e.g., a prior Cellar session's .h5ad
  with log1p CPM values) — re-normalizing would double-transform the data
- The input is not a raw count matrix, e.g., CODEX/CITE-seq protein measurements or
  gene activity scores from scATAC-seq, for which this count-based CPM recipe does
  not apply
- The goal is differential expression or enrichment on raw counts rather than preparing
  an embedding/clustering input
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scanpy
  role: 'Performs the preprocessing: cell/gene filtering (50–3000 thresholds), CPM
    normalization (total count = 1e5), log1p transform, and scaling to unit variance
    and zero mean'
- name: Cellar
  role: Interactive pipeline framework that orchestrates this preprocessing step before
    PCA/UMAP and Leiden clustering
  repo: github.com/euxhenh/cellar
- name: AnnData (Annotated Data object)
  role: Holds the count matrix and normalized values, using memory-mapping mode to
    allow analysis of large datasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:6c3bf0ebeb880ec52e750c0b54ace5c9bd5515e42f1c3d79457746abc6889815
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - AnnData (Annotated Data object)
  - Cellar
  - faiss
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# scrna-cpm-normalization-log1p-scaling

## Summary

Cellar's scRNA-seq preprocessing recipe: CPM total-count normalization (total count = 1e5), log1p transformation, and scaling to zero mean and unit variance, performed via scanpy, producing a matrix ready for PCA/UMAP and Leiden clustering.

## When to use

Use when a raw scRNA-seq count matrix (e.g., an AnnData .h5ad such as the 5273-cell HBMP3-spleen-CC2 HuBMAP dataset) must be transformed to a scaled log-CPM representation before PCA, UMAP, or Leiden clustering.

## When NOT to use

- The matrix is already normalized or scaled (e.g., a prior Cellar session's .h5ad with log1p CPM values) — re-normalizing would double-transform the data
- The input is not a raw count matrix, e.g., CODEX/CITE-seq protein measurements or gene activity scores from scATAC-seq, for which this count-based CPM recipe does not apply
- The goal is differential expression or enrichment on raw counts rather than preparing an embedding/clustering input

## Inputs

- Raw scRNA-seq count matrix in AnnData (.h5ad) format, e.g., the 5273-cell HBMP3-spleen-CC2 HuBMAP spleen count matrix

## Outputs

- CPM-normalized (total count = 1e5), log1p-transformed expression matrix
- Scaled expression matrix (zero mean, unit variance) suitable for PCA/UMAP input
- Filtered AnnData object retaining cells/genes passing the 50–3000 thresholds

## How to apply

First filter cells expressing fewer than 50 or more than 3000 genes and genes expressed in fewer than 50 or more than 3000 cells using scanpy preprocessing. Then CPM-normalize the data matrix to a total count of 1e5 and log1p-transform it, followed by scaling to unit variance and zero mean. This preprocessing is done via scanpy in the Cellar pipeline and prepares the expression matrix so that scikit-learn PCA (randomized SVD, 40 dimensions) and faiss-based 15-neighbor graphs for Leiden clustering operate on comparable feature scales. The normalized matrix is carried in an AnnData object, which Cellar uses in memory-mapping mode for large datasets.

## Related tools

- **scanpy** (Performs the preprocessing: cell/gene filtering (50–3000 thresholds), CPM normalization (total count = 1e5), log1p transform, and scaling to unit variance and zero mean)
- **Cellar** (Interactive pipeline framework that orchestrates this preprocessing step before PCA/UMAP and Leiden clustering) — github.com/euxhenh/cellar
- **AnnData (Annotated Data object)** (Holds the count matrix and normalized values, using memory-mapping mode to allow analysis of large datasets)

## Examples

```
import scanpy as sc; sc.pp.normalize_total(adata, target_sum=1e5); sc.pp.log1p(adata); sc.pp.scale(adata)
```

## Evaluation signals

- After normalize_total, per-cell counts sum to the target 1e5 before log1p
- Post-scaling, each gene feature has zero mean and unit variance
- Cell/gene filters (min 50, max 3000) leave 5273 cells for the HBMP3-spleen-CC2 dataset with no cells dropped in error
- Downstream Leiden clustering at resolution 1 on the scaled matrix reproduces the 16 clusters reported in Supplementary Fig. 1a
- Output AnnData session file contains cluster assignments for all 5273 cells in adata.obs['labels']

## Limitations

- Filter cutoffs (cells: 50–3000 genes; genes: 50–3000 cells) are dataset-tuned for the HuBMAP spleen data and may discard valid cells/genes in other datasets
- No software versions are stated for Cellar, Python, or scanpy, so exact numerical reproduction depends on tool versions
- Reproducibility may depend on the external Cellar web server rather than a locally installable archived artifact

## Evidence

- [methods] CPM normalization with log1p and scaling: "The data matrix was then CPM total count normalized (total count = 1e5) and log1p-transformed."
- [methods] Scaling step: "Finally, we scale the data down to unit variance and zero-mean."
- [methods] Tool used for preprocessing: "Preprocessing of the data was done via scanpy"
- [methods] Filtering thresholds preceding normalization: "we filtered cells with less than 50 or more than 3000 expressed genes"
