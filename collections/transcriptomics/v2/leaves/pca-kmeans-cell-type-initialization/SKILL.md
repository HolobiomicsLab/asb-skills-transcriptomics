---
name: pca-kmeans-cell-type-initialization
description: Use when running DIEM on 10X droplet-based snRNA-seq (or scRNA-seq) count
  data and you need to initialize k cell-type clusters via PCA and k-means before
  run_em debris-score estimation and call_targets filtering.
when_to_use_negative:
- Input is not droplet-level raw count data (e.g., already-filtered, normalized, or
  aggregated cell-by-gene matrices) — DIEM's debris test set and clustering strategy
  require all droplets, including those with a single mapped read
- Data contain too few genes enriched in the background distribution — DIEM's debris
  scoring, which the clustering feeds, will likely be inaccurate when little or no
  genes are increased in the background
- Goal is filtering single-cell data where you intend to drop the debris cluster directly
  via call_targets(clusters = "debris", thresh = NULL) rather than initialize a multinomial
  mixture for snRNA-seq scoring
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: diem
  role: R package supplying get_pcs and init to perform PCA projection and k-means
    initialization of cell-type clusters within the SCE pipeline
  repo: https://github.com/marcalva/diem
- name: R
  role: Runtime environment; raw counts are stored in a column-wise sparse Matrix
    using the R Matrix package
- name: Matrix
  role: Stores the droplet count matrix as a column-wise sparse Matrix consumed by
    get_pcs/init
- name: irlba
  role: Used by get_pcs for the PCA computation on the sparse count matrix
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:90929176daf0088c50f149d6c992c1f9609b3490f210a0fc5a40a466618e940f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - CellRanger
  - DIEM
  - Matrix
  - R
  - diem
  - irlba
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# PCA + k-means initialization of cell-type clusters for DIEM debris filtering

## Summary

Initializes DIEM's semi-supervised multinomial mixture model by projecting droplets onto principal components of variable genes and partitioning them with k-means into k cell-type clusters (plus a fixed debris cluster), providing the starting point from which the EM algorithm estimates debris and cell-type RNA distributions.

## When to use

Use when running DIEM on 10X droplet-based snRNA-seq (or scRNA-seq) count data and you need to initialize k cell-type clusters via PCA and k-means before run_em debris-score estimation and call_targets filtering.

## When NOT to use

- Input is not droplet-level raw count data (e.g., already-filtered, normalized, or aggregated cell-by-gene matrices) — DIEM's debris test set and clustering strategy require all droplets, including those with a single mapped read
- Data contain too few genes enriched in the background distribution — DIEM's debris scoring, which the clustering feeds, will likely be inaccurate when little or no genes are increased in the background
- Goal is filtering single-cell data where you intend to drop the debris cluster directly via call_targets(clusters = "debris", thresh = NULL) rather than initialize a multinomial mixture for snRNA-seq scoring

## Inputs

- SCE object from the diem R package built from a 10X CellRanger sparse Matrix (genes x droplets), with quality metrics (pct.mt, MALAT1) optionally added via get_gene_pct
- Gene-filtered droplet count matrix after set_debris_test_set and filter_genes
- Initialization parameters: k = 20 initial cell-type clusters, n_pcs = 30, minimum 200 genes per droplet, minimum cluster size 10 droplets

## Outputs

- SCE object annotated with cluster assignments: k putative cell-type clusters plus the fixed debris cluster, ready for run_em
- PCA embedding of droplets used for k-means initialization

## How to apply

After loading 10X counts with read_10x into a column-wise sparse Matrix and building the SCE container with create_SCE, fix the debris/test split with set_debris_test_set (min_counts = 100 for full snRNA-seq data; the paper's small-data tutorial uses min_counts = 10) so all droplets below the hard count threshold are treated as debris, mirroring DIEM's semi-supervised EM design. Then run filter_genes (e.g., cpm_thresh = 0), get_pcs to compute the top principal components (default n_pcs = 30) from variable genes, and init to run k-means with k = 20 initial cell-type clusters, restricting to droplets with at least 200 genes and merging clusters smaller than 10 droplets. The debris cluster is held fixed while the k-means-defined putative cell-type clusters provide the initial parameters that run_em refines with the semi-supervised EM. Rationale: clustering all droplets first lets DIEM identify droplets belonging to a debris cluster versus putative cell-type clusters, which downstream assign_clusters and estimate_dbr_score (DE genes via Welch's t-test at FDR p < 0.05, clusters with mean genes < 200 counted as debris) convert into per-droplet debris scores for call_targets at thresh_score = 0.5.

## Related tools

- **diem** (R package supplying get_pcs and init to perform PCA projection and k-means initialization of cell-type clusters within the SCE pipeline) — https://github.com/marcalva/diem
- **R** (Runtime environment; raw counts are stored in a column-wise sparse Matrix using the R Matrix package)
- **Matrix** (Stores the droplet count matrix as a column-wise sparse Matrix consumed by get_pcs/init)
- **irlba** (Used by get_pcs for the PCA computation on the sparse count matrix)

## Examples

```
sce <- set_debris_test_set(sce); sce <- filter_genes(sce); sce <- get_pcs(sce); sce <- init(sce)
```

## Evaluation signals

- The pipeline order matches the README: set_debris_test_set → filter_genes → get_pcs → init → run_em → assign_clusters → estimate_dbr_score → call_targets
- summarize_clusters(sce) shows k = 20 putative cell-type clusters with the debris cluster fixed, and clusters smaller than 10 droplets merged
- Downstream estimate_dbr_score identifies DE genes between clusters via Welch's t-test at FDR p < 0.05 and flags clusters with mean genes < 200 as debris
- After call_targets(thresh_score = 0.5, min_genes = 0) on the 10X nuclei_2k mouse brain data, 2,010 passing droplets are called, of which 1,850 (92.0%) classify as nuclear by percent spliced reads
- plot_clust(sce, feat_x = 'pct.mt', feat_y = 'score.debris') shows debris clusters separated from cell-type clusters on mitochondrial percentage vs. debris score

## Limitations

- Debris scores may not be comparable across independent experiments because they are normalized relative to within-sample cluster means, so k-means-derived clusters from different samples cannot share score scales
- Scoring downstream of the clustering relies on accurate estimation of debris-enriched genes; if little or no genes are increased in the background distribution, the resulting debris score will likely be inaccurate
- The clustering strategy's correlation with extranuclear contamination was less successful in PBMC scRNA-seq data, possibly due to smaller differences between background and cell-type profiles
- Adding more initialization clusters increases compute; run multiple k_init values or multi-threading may be needed (supported since diem v2.1.0)

## Evidence

- [intro] The parameters of the model are inferred using semi-supervised EM13,14, where all droplets below a hard count threshold are fixed as debris.: "The parameters of the model are inferred using semi-supervised EM13,14, where all droplets below a hard count threshold are fixed as debris."
- [discussion] DIEM first clusters all droplets generated from the experiment to identify droplets belonging either to the debris cluster or putative cell type clusters.: "DIEM first clusters all droplets generated from the experiment to identify droplets belonging either to the debris cluster or putative cell type clusters."
- [readme] Uses Dirichlet-multinomial; Initializes with k-means; Removes background centers using a likelihood strategy: "* Uses Dirichlet-multinomial
    * Initializes with k-means
    * Removes background centers using a likelihood strategy"
- [readme] sce <- get_pcs(sce)
sce <- init(sce): "sce <- get_pcs(sce)
sce <- init(sce)"
- [methods] mb_small <- set_debris_test_set(mb_small, min_counts = 10): "mb_small <- set_debris_test_set(mb_small, min_counts = 10)"
