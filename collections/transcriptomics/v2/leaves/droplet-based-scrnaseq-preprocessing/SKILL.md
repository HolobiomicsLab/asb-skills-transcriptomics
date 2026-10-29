---
name: droplet-based-scrnaseq-preprocessing
description: Use when raw 10X droplet count matrices (CellRanger raw_feature_bc_matrix
  or 10X feature-barcode output) show extranuclear RNA contamination (elevated MT%/MALAT1%)
  or spurious cell types after hard-count-cutoff filtering of snRNA-seq data.
when_to_use_negative:
- The data are already a filtered feature-barcode matrix or an aggregated feature
  table — DIEM needs the raw droplet count matrix to model the debris distribution.
- Ambient and nuclear RNA profiles are nearly identical (little or no genes increased
  in the background distribution), e.g. scRNA-seq samples like PBMC where the debris
  score's correlation with extranuclear contamination was less successful.
- Integrating multiple samples where debris scores would be compared across experiments
  — scores are not on the same scale across independent experiments.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_2228
- http://edamontology.org/topic_3170
tools:
- name: DIEM
  role: 'Primary filtering pipeline: Dirichlet-multinomial mixture clustering plus
    debris scoring via semi-supervised EM to identify and remove debris-contaminated
    droplets'
  repo: https://github.com/marcalva/diem
- name: diem
  role: R package implementation providing read_10x, create_SCE, get_gene_pct, set_debris_test_set,
    filter_genes, get_pcs, init, run_em, assign_clusters, estimate_dbr_score, call_targets,
    and convert_to_seurat
  repo: https://github.com/marcalva/diem
- name: R
  role: Runtime environment; raw counts stored as column-wise sparse Matrix
- name: Matrix
  role: R package providing the column-wise sparse matrix storage for raw counts
- name: CellRanger
  role: Upstream 10X alignment producing the raw_feature_bc_matrix consumed by read_10x
- name: EmptyDrops
  role: Comparison filter (scRNA-seq ambient-RNA Dirichlet-Multinomial test) for benchmarking
    DIEM's kept-droplet set
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:6477db2c390d1247de75f38b7b247c51c36a3fbbc3d10afdd3efd4d87a5b5929
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - CellRanger
  - DIEM
  - EmptyDrops
  - Matrix
  - R
  - diem
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_2228
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# droplet-based-scrnaseq-preprocessing

## Summary

A pre-processing skill that removes debris-contaminated droplets from droplet-based snRNA-seq/scRNA-seq data using DIEM's likelihood-based multinomial mixture model with semi-supervised EM, replacing hard UMI/read cutoffs.

## When to use

Use when raw 10X droplet count matrices (CellRanger raw_feature_bc_matrix or 10X feature-barcode output) show extranuclear RNA contamination (elevated MT%/MALAT1%) or spurious cell types after hard-count-cutoff filtering of snRNA-seq data.

## When NOT to use

- The data are already a filtered feature-barcode matrix or an aggregated feature table — DIEM needs the raw droplet count matrix to model the debris distribution.
- Ambient and nuclear RNA profiles are nearly identical (little or no genes increased in the background distribution), e.g. scRNA-seq samples like PBMC where the debris score's correlation with extranuclear contamination was less successful.
- Integrating multiple samples where debris scores would be compared across experiments — scores are not on the same scale across independent experiments.

## Inputs

- 10X raw feature-barcode matrix (CellRanger raw_feature_bc_matrix or matrix from 10X datasets, e.g. fresh_68k_pbmc_donor_a or nuclei_2k mouse brain)
- Sparse Matrix of raw UMI counts (column-wise, R Matrix package)
- Gene name list for contamination metrics (mt- and MALAT1 patterns)

## Outputs

- SCE object with debris scores, cluster assignments, pct.mt and MALAT1 per droplet
- Set of kept (non-debris) droplets ready for convert_to_seurat or downstream clustering/DE
- Per-droplet contamination summaries (mean MT%/MALAT1% for kept vs. removed droplets, e.g. written to CSV)

## How to apply

Read the 10X output into a column-wise sparse Matrix with diem's read_10x, then build an SCE object via create_SCE and annotate per-droplet metrics with get_gene_pct for '^mt-' (pct.mt) and '^malat1$' (MALAT1). Run the DIEM pipeline: filter_genes, get_pcs, init (k-means on PCs), run_em, assign_clusters, estimate_dbr_score, then call_targets — for snRNA-seq use the debris-score threshold (thresh_score = 0.5) in debris-cluster removal mode, while for scRNA-seq use call_targets(clusters = "debris", thresh = NULL) to remove the debris cluster directly. Fix droplets below a hard count threshold as debris (e.g. set_debris_test_set with min_counts = 10 for mouse brain nuclei_2k or min_counts = 100 for PBMC scRNA-seq) since semi-supervised EM anchors the debris distribution on low-count droplets. Evaluate with summarize_clusters and plot_clust (e.g. pct.mt vs score.debris); convert kept droplets via convert_to_seurat. Rationale: DIEM models droplets with a Dirichlet-multinomial likelihood so droplets are scored on expression of genes enriched in the debris (ambient RNA) set rather than on total counts.

## Related tools

- **DIEM** (Primary filtering pipeline: Dirichlet-multinomial mixture clustering plus debris scoring via semi-supervised EM to identify and remove debris-contaminated droplets) — https://github.com/marcalva/diem
- **diem** (R package implementation providing read_10x, create_SCE, get_gene_pct, set_debris_test_set, filter_genes, get_pcs, init, run_em, assign_clusters, estimate_dbr_score, call_targets, and convert_to_seurat) — https://github.com/marcalva/diem
- **R** (Runtime environment; raw counts stored as column-wise sparse Matrix)
- **Matrix** (R package providing the column-wise sparse matrix storage for raw counts)
- **CellRanger** (Upstream 10X alignment producing the raw_feature_bc_matrix consumed by read_10x)
- **EmptyDrops** (Comparison filter (scRNA-seq ambient-RNA Dirichlet-Multinomial test) for benchmarking DIEM's kept-droplet set)

## Examples

```
library(diem); counts <- read_10x("path/to/10x"); sce <- create_SCE(counts); sce <- set_debris_test_set(sce); sce <- filter_genes(sce); sce <- get_pcs(sce); sce <- init(sce); sce <- run_em(sce); sce <- assign_clusters(sce); sce <- estimate_dbr_score(sce); sce <- call_targets(sce, thresh_score = 0.5)
```

## Evaluation signals

- Kept-droplet counts: e.g. 75,847 kept droplets for DIEM vs. 77,585 for EmptyDrops on the 68K PBMC dataset (restricted to droplets with ≥100 UMIs).
- Quality-metric invariants: droplets uniquely kept by DIEM should show MT% and MALAT1% similar to shared passing droplets, while EmptyDrops-only droplets (1,927 in PBMC) show elevated MT%/MALAT1% consistent with ruptured cells.
- Benchmark classification rates: on mouse brain nuclei_2k, DIEM removed 101 droplets of which 68.4% were background (percent spliced reads), vs. 64.4% for EmptyDrops and 26.5% for quantile filtering.
- Schema check: per-droplet metrics (pct.mt, MALAT1, score.debris, cluster) present in SCE object after each pipeline step; summarize_clusters output should show debris clusters with low n_genes and high score.debris.
- Debris-score distribution check: plot_clust of pct.mt vs score.debris should show kept droplets separated from debris clusters on the contamination axis.

## Limitations

- Debris scores may not be comparable across independent experiments because they are normalized relative to within-sample cluster means, complicating multi-sample integration.
- Debris scoring relies on accurate estimation of debris-enriched genes; if few or no genes are increased in the background distribution, the resulting debris score will likely be inaccurate.
- In PBMC scRNA-seq data, DIEM's debris-score correlation with extranuclear contamination (percent spliced reads) was less successful, possibly due to smaller differences between background and cell type profiles in scRNA-seq.
- EmptyDrops was originally developed and tested on single-cell data, so its model assumptions differ from DIEM's — comparisons on snRNA-seq data should interpret kept-droplet differences carefully.

## Evidence

- [abstract] Our likelihood-based approach models the gene expression distribution of debris and cell types, which are estimated using EM.: "Our likelihood-based approach models the gene expression distribution of debris and cell types, which are estimated using EM."
- [discussion] DIEM first clusters all droplets generated from the experiment to identify droplets belonging either to the debris cluster or putative cell type clusters.: "DIEM first clusters all droplets generated from the experiment to identify droplets belonging either to the debris cluster or putative cell type clusters."
- [methods] We provide a convenience function `read_10x` that can read the 10X output into a sparse Matrix.: "We provide a convenience function `read_10x` that can read the 10X output into a sparse Matrix."
- [intro] The parameters of the model are inferred using semi-supervised EM, where all droplets below a hard count threshold are fixed as debris.: [evidence span withheld — anchor status: not_found]
- [readme] # Call targets by removing droplets in debris cluster(s) for single-cell data
sce <- call_targets(sce, clusters = "debris", thresh = NULL): "# Call targets by removing droplets in debris cluster(s) for single-cell data
sce <- call_targets(sce, clusters = "debris", thresh = NULL)"
- [discussion] the debris score, however, cannot be assumed to exist on the same scale in independent experiments.: [evidence span withheld — anchor status: not_found]
