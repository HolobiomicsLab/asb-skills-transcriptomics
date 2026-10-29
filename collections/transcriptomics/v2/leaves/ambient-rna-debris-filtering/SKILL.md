---
name: ambient-rna-debris-filtering
description: Use when filtering a raw 10X droplet count matrix (feature-barcode MTX)
  for ambient RNA/debris contamination, especially snRNA-seq from frozen tissue, or
  when comparing DIEM vs EmptyDrops kept-droplet sets using MT% and MALAT1% metrics.
when_to_use_negative:
- When the input is already a filtered feature table or Seurat object of retained
  cells — DIEM operates on the full raw droplet matrix during pre-processing
- When debris-enriched genes are absent (little or no genes increased in the background
  distribution), the debris score will likely be inaccurate
- When comparing debris scores across independent samples for integration — scores
  are normalized within-sample and may not be on the same scale
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: DIEM
  role: 'Primary filtering method: clusters all droplets via Dirichlet-multinomial
    mixture with semi-supervised EM, scores them, and removes debris-contaminated
    droplets'
  repo: https://github.com/marcalva/diem
- name: EmptyDrops
  role: 'Comparison baseline: estimates a Dirichlet-Multinomial ambient RNA profile
    and tests droplets against it; benchmarked against DIEM kept-droplet sets'
- name: diem
  role: R package implementing read_10x, create_SCE, get_gene_pct, set_debris_test_set,
    filter_genes, init, run_em, assign_clusters, estimate_dbr_score, call_targets,
    and convert_to_seurat
  repo: https://github.com/marcalva/diem
- name: Matrix
  role: R package used to store raw counts as a column-wise sparse matrix
- name: CellRanger
  role: Alignment pipeline producing the 10X raw_feature_bc_matrix input consumed
    by read_10x
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:3ead9f000112bf5e16c6ef8c34c1cd8413d4bf293636f98dc0202264b14dfdbf
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7916666666666666
n_claims: 24
---

# ambient-rna-debris-filtering

## Summary

Filter debris-contaminated droplets from droplet-based scRNA-seq or snRNA-seq data using DIEM's likelihood-based multinomial mixture model with semi-supervised EM, optionally benchmarked against EmptyDrops. DIEM identifies droplets containing extranuclear/ambient RNA that hard UMI cutoffs retain, improving downstream cluster quality.

## When to use

Use when filtering a raw 10X droplet count matrix (feature-barcode MTX) for ambient RNA/debris contamination, especially snRNA-seq from frozen tissue, or when comparing DIEM vs EmptyDrops kept-droplet sets using MT% and MALAT1% metrics.

## When NOT to use

- When the input is already a filtered feature table or Seurat object of retained cells — DIEM operates on the full raw droplet matrix during pre-processing
- When debris-enriched genes are absent (little or no genes increased in the background distribution), the debris score will likely be inaccurate
- When comparing debris scores across independent samples for integration — scores are normalized within-sample and may not be on the same scale

## Inputs

- Raw 10X feature-barcode count matrix (CellRanger output, e.g. raw_feature_bc_matrix) read as a column-wise sparse Matrix
- Gene annotations supporting mitochondrial ('^mt-') and MALAT1 ('^malat1$') pattern matching
- Optional: percent spliced reads per droplet for nuclear-vs-background classification (snRNA-seq)

## Outputs

- SCE object annotated with per-droplet debris scores, cluster assignments, pct.mt, and MALAT1%
- Set of kept (target) droplets after call_targets filtering
- Per-method kept-droplet counts and per-group mean MT%/MALAT1% summaries (e.g. diem_emptydrops_pbmc_comparison.csv)

## How to apply

Read the 10X raw feature-barcode matrix into a column-wise sparse Matrix via diem's read_10x, then build an SCE object with create_SCE and compute contamination metrics with get_gene_pct using genes matching '^mt-' (pct.mt) and '^malat1$' (MALAT1). Define the debris test set with set_debris_test_set (a hard count floor, e.g. min_counts = 10 or 100, fixes droplets below the threshold as debris for semi-supervised EM), filter genes with filter_genes, reduce dimensions with get_pcs, initialize clusters with k-means via init, fit the Dirichlet-multinomial mixture with run_em, assign clusters, and estimate the debris score with estimate_dbr_score. Call targets with call_targets: use thresh_score = 0.5 debris-score filtering for single-nucleus data, or clusters = 'debris' mode (removing droplets in debris cluster(s)) for single-cell data such as PBMCs. For cross-method comparison, restrict both DIEM and EmptyDrops to the identical droplet set, intersect kept-droplet sets, and compare mean pct.mt and MALAT1% for shared, DIEM-only, and EmptyDrops-only droplets — EmptyDrops-only droplets tend to show elevated MT%/MALAT1% consistent with ruptured cells. Export kept-droplet counts and per-group contamination summaries (e.g. to a CSV) for validation.

## Related tools

- **DIEM** (Primary filtering method: clusters all droplets via Dirichlet-multinomial mixture with semi-supervised EM, scores them, and removes debris-contaminated droplets) — https://github.com/marcalva/diem
- **EmptyDrops** (Comparison baseline: estimates a Dirichlet-Multinomial ambient RNA profile and tests droplets against it; benchmarked against DIEM kept-droplet sets)
- **diem** (R package implementing read_10x, create_SCE, get_gene_pct, set_debris_test_set, filter_genes, init, run_em, assign_clusters, estimate_dbr_score, call_targets, and convert_to_seurat) — https://github.com/marcalva/diem
- **Matrix** (R package used to store raw counts as a column-wise sparse matrix)
- **CellRanger** (Alignment pipeline producing the 10X raw_feature_bc_matrix input consumed by read_10x)

## Examples

```
counts <- read_10x("path/to/10x"); sce <- create_SCE(counts); sce <- set_debris_test_set(sce, min_counts = 100); sce <- init(sce); sce <- run_em(sce); sce <- assign_clusters(sce); sce <- estimate_dbr_score(sce); sce <- call_targets(sce, clusters = "debris", thresh = NULL)
```

## Evaluation signals

- Pipeline reproduces expected kept-droplet counts on the 68K PBMC benchmark: 75,847 kept droplets for DIEM vs 77,585 for EmptyDrops (min_counts = 100 restricted set)
- EmptyDrops-only droplets (1,927 in the PBMC comparison) show higher mean MT% and MALAT1% than droplets kept by both methods, consistent with ruptured cells
- On the mouse brain nuclei_2k snRNA-seq set, kept droplets classify as nuclear via percent spliced reads (1,868 of 2,080 kept = 89.8% nuclear)
- Debris score thresholds are tunable: call_targets with thresh_score = 0.5 (snRNA-seq mode) vs clusters = 'debris' (scRNA-seq mode) should be applied to the matching data modality
- inspect summarize_clusters and plot_clust of pct.mt vs score.debris to confirm debris clusters show elevated extranuclear contamination before calling targets

## Limitations

- The debris score cannot be assumed to exist on the same scale in independent experiments; scores may not be comparable when integrating multiple samples, particularly if extranuclear RNA distributions differ across samples
- Debris scoring relies on accurate estimation of debris-enriched genes; if there are little or no genes increased in the background distribution, the resulting debris score will likely be inaccurate
- Debris score correlation with extranuclear contamination was less successful in PBMC scRNA-seq data, possibly due to smaller differences between background and cell type profiles in scRNA-seq
- EmptyDrops was originally developed and tested on single-cell data, so its model assumptions differ from DIEM's for snRNA-seq

## Evidence

- [abstract] Our likelihood-based approach models the gene expression distribution of debris and cell types, which are estimated using EM.: "Our likelihood-based approach models the gene expression distribution of debris and cell types, which are estimated using EM."
- [intro] DIEM fixes low-count droplets as debris before EM scoring: "The parameters of the model are inferred using semi-supervised EM13,14, where all droplets below a hard count threshold are fixed as debris."
- [methods] set_debris_test_set with min_counts parameter: "mb_small <- set_debris_test_set(mb_small, min_counts = 10)"
- [methods] call_targets with debris score threshold: "mb_small <- call_targets(mb_small,
                         thresh_score = 0.5,
                         min_genes = 0)"
- [discussion] Debris score scale caveat across experiments: [evidence span withheld — anchor status: not_found]
- [readme] README workflow for MT/MALAT1 metrics and debris-cluster removal mode: [evidence span withheld — anchor status: external_doc]
