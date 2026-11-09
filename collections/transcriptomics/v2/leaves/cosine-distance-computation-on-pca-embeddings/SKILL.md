---
name: cosine-distance-computation-on-pca-embeddings
description: Use when you have a Seurat object with PCA embeddings (e.g. RunPCA output
  on pbmc3k) and need cosine distances between cells to feed silhouette-score computation
  or cluster-heterogeneity evaluation.
when_to_use_negative:
- No PCA (or equivalent low-dimensional embedding) has been computed yet — computing
  cosine distances on raw counts is not the SCISSORS workflow
- The downstream analysis requires Euclidean or correlation distances other than cosine
  — CosineDist is specifically the cosine metric on PCA embeddings
- Data are not single-cell expression data in a Seurat workflow; SCISSORS is built
  as an extension to a typical Seurat workflow
tools:
- name: SCISSORS
  role: Provides the CosineDist function that computes the pairwise cosine-distance
    matrix from PCA cell embeddings
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Supplies the preprocessing pipeline (NormalizeData, FindVariableFeatures vst/2000,
    ScaleData, RunPCA) and the PCA embedding matrix used as CosineDist input
  repo: https://github.com/jr-leary7/SCISSORS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:3e5a069f85ec48a7d94d2399a1238cf7232a7cbb1b029916d5afaa65259a069d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - R
  - SCISSORS
  - Seurat
  - dplyr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cosine-distance-computation-on-pca-embeddings

## Summary

Computes a pairwise cosine-distance matrix from Seurat PCA cell embeddings using the SCISSORS CosineDist function, the distance basis for SCISSORS silhouette scoring and reclustering optimization.

## When to use

Use when you have a Seurat object with PCA embeddings (e.g. RunPCA output on pbmc3k) and need cosine distances between cells to feed silhouette-score computation or cluster-heterogeneity evaluation.

## When NOT to use

- No PCA (or equivalent low-dimensional embedding) has been computed yet — computing cosine distances on raw counts is not the SCISSORS workflow
- The downstream analysis requires Euclidean or correlation distances other than cosine — CosineDist is specifically the cosine metric on PCA embeddings
- Data are not single-cell expression data in a Seurat workflow; SCISSORS is built as an extension to a typical Seurat workflow

## Inputs

- Seurat object with a PCA dimensionality reduction (e.g. pbmc3k after RunPCA)
- PCA cell-embedding matrix (cells x PCs) from Embeddings(seu, 'pca')

## Outputs

- Pairwise cosine-distance matrix/object (cos_dist) over all cells

## How to apply

After Seurat preprocessing (NormalizeData, FindVariableFeatures with vst and 2000 features, ScaleData, RunPCA), extract the PCA cell embeddings via Embeddings(pbmc, 'pca'). Call the SCISSORS CosineDist function on this embedding matrix to obtain the pairwise cosine-distance object. The resulting cos_dist is the required distance input for silhouette calculations (a_i = mean cosine distance within-cluster, b_i = minimum mean cosine distance to any other cluster) and downstream reclustering. Do not substitute Euclidean distances: the SCISSORS silhouette formulation is defined in terms of cosine distance on the PCA embedding space.

## Related tools

- **SCISSORS** (Provides the CosineDist function that computes the pairwise cosine-distance matrix from PCA cell embeddings) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Supplies the preprocessing pipeline (NormalizeData, FindVariableFeatures vst/2000, ScaleData, RunPCA) and the PCA embedding matrix used as CosineDist input) — https://github.com/jr-leary7/SCISSORS

## Examples

```
cos_dist <- CosineDist(input = Embeddings(pbmc, "pca"))
```

## Evaluation signals

- The distance object is symmetric with zero diagonal (self-distance 0 for every cell)
- Cosine distances lie in [0, 2]; on non-negative PCA embeddings of scRNA-seq data they should be in [0, 1]
- Matrix dimensions equal n_cells x n_cells, matching the number of cells in the Seurat object
- Downstream per-cell silhouette scores computed from this distance matrix fall within the valid silhouette range of -1 to 1
- The same cos_dist input reproduces cluster::silhouette(dist = cos_dist, x = cluster_labels) reference per-cell scores within numerical tolerance

## Limitations

- Cosine distance on PCA embeddings depends on the number of PCs retained (the README workflow uses 15 PCs via PrepareData(n.PC = 15)); changing n.PC changes the distance matrix and silhouette scores
- The paper notes that 'even small differences in analysis and parameter choosing can greatly alter clustering results', so the distance matrix is sensitive to upstream preprocessing choices

## Evidence

- [methods] CosineDist called on PCA embeddings: "cos_dist <- CosineDist(input = Embeddings(pbmc, "pca"))"
- [abstract] SCISSORS silhouette scoring purpose: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [readme] README usage of silhouette scores on cells: [evidence span withheld — anchor status: external_doc]
