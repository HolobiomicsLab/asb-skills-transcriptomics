---
name: dimensionality-reduction-pca-umap
description: Use when you need to cluster droplets from a snRNA-seq count matrix (e.g.,
  after DIEM/EmptyDrops or quantile filtering) to detect residual background RNA-driven
  debris clusters via per-cluster median percent spliced reads.
when_to_use_negative:
- Input droplets have already been filtered by DIEM with a debris score threshold
  (e.g., thresh_score = 0.5), which the paper shows removes the background cluster
  without re-clustering for debris detection
- Data are single-cell PBMC RNA-seq, where the debris score's correlation with extranuclear
  contamination was less successful
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Log-normalization, variable gene selection, PCA, Louvain clustering, and UMAP
    embedding of the converted droplet object
- name: diem
  role: Provides convert_to_seurat to build the Seurat object with min.features =
    50 and carry droplet debris metadata forward
  repo: https://github.com/marcalva/diem
- name: Matrix
  role: Stores the raw droplet counts as a column-wise sparse matrix that is the input
    to clustering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:2800ece7e26bb011ea063421e49783404f146c6c4cc09e8f51f15ee5f845a811
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - DIEM
  - Matrix
  - R
  - Seurat
  - SingCellaR
  - diem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- pca-umap-dimensionality-reduction
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# dimensionality-reduction-pca-umap

## Summary

Standardized Seurat dimensionality-reduction and clustering pipeline (log-normalization, top 2,000 variable genes, 30-PC PCA, Louvain clustering, UMAP) used in the DIEM paper to detect residual background/debris clusters in filtered droplet sets.

## When to use

Use when you need to cluster droplets from a snRNA-seq count matrix (e.g., after DIEM/EmptyDrops or quantile filtering) to detect residual background RNA-driven debris clusters via per-cluster median percent spliced reads.

## When NOT to use

- Input droplets have already been filtered by DIEM with a debris score threshold (e.g., thresh_score = 0.5), which the paper shows removes the background cluster without re-clustering for debris detection
- Data are single-cell PBMC RNA-seq, where the debris score's correlation with extranuclear contamination was less successful

## Inputs

- Sparse droplet-by-gene count matrix (10X-format) of filtered/kept droplets (Matrix package column-wise sparse matrix)
- DIEM SCE/droplet metadata including debris scores and percent spliced reads

## Outputs

- Per-cluster summary CSV with one row per Seurat cluster and a median percent-spliced-reads field
- UMAP and PC embeddings for droplet visualization

## How to apply

Convert the cleaned droplet counts into a Seurat object using the diem convenience function convert_to_seurat with min.features = 50, carrying the diem droplet metadata (including the debris score) into the Seurat meta data. Restrict to droplets with at least 200 detected genes, apply log-normalization, and identify the top 2,000 variable genes as specified for the standardized pipeline. Run PCA on the variable genes retaining 30 principal components, cluster droplets with Louvain clustering on the PC embedding, and compute a UMAP embedding for visualization. For each Seurat cluster, compute the median percent spliced reads and assemble a per-cluster summary table; the rationale is that quantile-based hard count thresholds (unlike DIEM's EM-based debris scoring) can retain debris clusters in snRNA-seq data.

## Related tools

- **Seurat** (Log-normalization, variable gene selection, PCA, Louvain clustering, and UMAP embedding of the converted droplet object)
- **diem** (Provides convert_to_seurat to build the Seurat object with min.features = 50 and carry droplet debris metadata forward) — https://github.com/marcalva/diem
- **Matrix** (Stores the raw droplet counts as a column-wise sparse matrix that is the input to clustering)

## Examples

```
seur <- convert_to_seurat(sce); seur <- SCT_norm(seur_obj); seur <- RunPCA(seur, npcs = 30); seur <- FindNeighbors(seur); seur <- RunUMAP(seur, dims = 1:30)
```

## Evaluation signals

- Output CSV exists and contains exactly one row per Seurat cluster with a median-percent-spliced-reads field
- The background/debris cluster's median percent spliced reads matches the reported value of 88.5% within a small numerical tolerance (for the quantile-filtered mouse brain droplets)
- Exactly 2,000 variable genes are selected and 30 principal components are retained, per the standardized pipeline
- All analyzed droplets pass the min.features = 50 and >= 200 detected gene pre-filters

## Limitations

- Hard count quantile filtering underperformed versus DIEM and EmptyDrops, which remove droplets based on their expression distribution, so this re-clustering is expected to reveal residual debris clusters
- Debris scores carried into Seurat metadata are normalized relative to within-sample cluster means and cannot be assumed to be on the same scale in independent experiments

## Evidence

- [discussion] using a hard count threshold underperformed in comparison to EmptyDrops and DIEM: "using a hard count threshold underperformed in comparison to EmptyDrops and DIEM."
- [methods] The raw counts are stored in a column-wise sparse matrix: "The raw counts are stored in a column-wise sparse matrix using the implementation from the R package [Matrix]"
- [readme] converting to a seurat object: [evidence span withheld — anchor status: external_doc]
- [discussion] DIEM first clusters all droplets: "DIEM first clusters all droplets generated from the experiment to identify droplets belonging either to the debris cluster or putative cell type clusters."
