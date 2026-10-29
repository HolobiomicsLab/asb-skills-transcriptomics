---
name: louvain-graph-clustering
description: Use when droplet-level counts (e.g., 10X sparse matrix from mouse brain
  snRNA-seq) have been filtered and converted to a Seurat object and you need data-driven
  clusters to detect residual debris/background droplets.
when_to_use_negative:
- Single-cell (not single-nucleus) PBMC-like data where background and cell-type expression
  profiles differ little, so cluster separation of debris is less successful
- Situations where no genes are increased in the background distribution, undermining
  profile-based debris identification
- Droplets have not yet been filtered or converted to a feature-space embedding (e.g.,
  raw counts without normalization and variable-gene selection)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_4040
tools:
- name: Seurat
  role: Runs log-normalization, top-2000-variable-gene selection, 30-PC PCA, Louvain
    clustering, and UMAP on the converted droplet object
- name: diem
  role: Provides convert_to_seurat convenience function (min.features = 50) that transfers
    cleaned droplets and debris metadata into Seurat for clustering
  repo: https://github.com/marcalva/diem
- name: Matrix
  role: Stores the droplet-by-gene raw counts as a column-wise sparse matrix prior
    to clustering
- name: R
  role: Implementation environment for the clustering pipeline (DIEM, Seurat, Matrix
    are R packages)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:6b78a86e449ff3a8b36649fd92a6d703eaf51e5331ba110d4006b326d6011f40
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - DIEM
  - Matrix
  - R
  - Seurat
  - diem
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_4040
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# louvain-graph-clustering

## Summary

Clusters droplets/cells by Louvain community detection on a PCA embedding (e.g., in Seurat after log-normalization and top-2000-variable-gene selection) so that background RNA-driven debris clusters can be identified and quantified by per-cluster metrics such as median percent spliced reads.

## When to use

Use when droplet-level counts (e.g., 10X sparse matrix from mouse brain snRNA-seq) have been filtered and converted to a Seurat object and you need data-driven clusters to detect residual debris/background droplets.

## When NOT to use

- Single-cell (not single-nucleus) PBMC-like data where background and cell-type expression profiles differ little, so cluster separation of debris is less successful
- Situations where no genes are increased in the background distribution, undermining profile-based debris identification
- Droplets have not yet been filtered or converted to a feature-space embedding (e.g., raw counts without normalization and variable-gene selection)

## Inputs

- Sparse droplet-by-gene count Matrix (e.g., 10X output read via diem's read_10x) restricted to passing droplets
- Seurat object created from cleaned droplets with droplet metadata (e.g., DIEM debris score)
- Per-droplet percent spliced reads values for post-clustering cluster summaries

## Outputs

- Seurat object with cluster assignments on the 30-PC embedding and a UMAP embedding
- Per-cluster summary table (CSV) with one row per cluster including median percent spliced reads
- Identification of the residual background/debris cluster

## How to apply

Start from the cleaned droplet counts (e.g., the passing droplets from quantile-based or DIEM filtering) loaded into a sparse Matrix. Convert to a Seurat object (in the DIEM workflow via convert_to_seurat with min.features = 50, carrying droplet metadata such as the debris score), restrict to droplets with at least 200 detected genes, log-normalize counts, and select the top 2,000 variable genes. Run PCA retaining 30 principal components, then run Louvain clustering on the PC embedding (Seurat FindClusters) and compute a UMAP for visualization. The rationale is that DIEM's own strategy is cluster-based — debris and cell-type profiles form separable clusters — so Louvain clustering of retained droplets exposes any residual background cluster. Finally, compute per-cluster summaries (e.g., median percent spliced reads) to flag the debris cluster; in the mouse brain quantile-filtered data this residual cluster showed a median percent spliced reads of 88.5%.

## Related tools

- **Seurat** (Runs log-normalization, top-2000-variable-gene selection, 30-PC PCA, Louvain clustering, and UMAP on the converted droplet object)
- **diem** (Provides convert_to_seurat convenience function (min.features = 50) that transfers cleaned droplets and debris metadata into Seurat for clustering) — https://github.com/marcalva/diem
- **Matrix** (Stores the droplet-by-gene raw counts as a column-wise sparse matrix prior to clustering)
- **R** (Implementation environment for the clustering pipeline (DIEM, Seurat, Matrix are R packages))

## Examples

```
seur <- convert_to_seurat(sce)  # then in Seurat: NormalizeData, FindVariableFeatures (nfeatures=2000), RunPCA (npcs=30), FindClusters, RunUMAP
```

## Evaluation signals

- Output CSV exists and contains exactly one row per Seurat cluster with a median-percent-spliced-reads field
- Residual background cluster's median percent spliced reads matches the reported value of 88.5% within a small numerical tolerance
- PCA was run on the top 2,000 variable genes retaining 30 principal components before Louvain clustering
- Only droplets passing the min.features = 50 conversion and the 200-detected-gene threshold appear in the clustered object
- UMAP embedding is computable from the same PC space used for clustering, enabling visual inspection of the debris cluster

## Limitations

- Debris scores are normalized relative to within-sample cluster means and may not be comparable across independent experiments, especially when the distribution of extranuclear RNA differs across samples
- Debris scoring and cluster-based identification rely on accurate estimation of debris-enriched genes; results are inaccurate if no genes are increased in the background distribution
- Cluster-based debris identification was less successful in PBMC single-cell RNA-seq data, possibly due to smaller differences between background and cell type profiles

## Evidence

- [discussion] Cluster-based debris identification in DIEM's design: "DIEM first clusters all droplets generated from the experiment to identify droplets belonging either to the debris cluster or putative cell type clusters."
- [abstract] Clustering yields higher quality clusters after debris removal: "When compared to filtering using these state of the art methods, DIEM better removed droplets containing high levels of extranuclear RNA and led to higher quality clusters."
- [readme] README workflow ending in Seurat object conversion: [evidence span withheld — anchor status: external_doc]
- [other] Reported debris cluster statistic in the quantile-filtered mouse brain data: [evidence span withheld — anchor status: not_found]
