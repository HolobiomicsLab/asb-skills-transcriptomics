---
name: snrna-seq-droplet-quality-assessment
description: Use when raw 10X droplet snRNA-seq data (e.g. CellRanger output from
  fresh/frozen nuclei preparations) contains ambient extranuclear RNA contamination
  and droplets must be classified as nuclear vs background before downstream clustering
  or cell-type calling.
when_to_use_negative:
- Data are clean single-cell (scRNA-seq) droplet data where background/cell profiles
  differ little — DIEM's debris-score correlation with extranuclear contamination
  was less successful on PBMC scRNA-seq; prefer call_targets(clusters = "debris")
  instead of the score threshold
- There are few or no genes enriched in the background distribution — the debris score
  will likely be inaccurate since scoring relies on estimating debris-enriched genes
- Comparing debris scores across independent samples for integration — scores are
  normalized within-sample and cannot be assumed to be on the same scale if extranuclear
  RNA distributions differ
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3473
- http://edamontology.org/topic_3170
tools:
- name: DIEM
  role: Semi-supervised EM classifier that clusters droplets, estimates debris scores,
    and filters background droplets in snRNA-seq
  repo: https://github.com/marcalva/diem
- name: diem (R package)
  role: 'Implements the pipeline: read_10x, create_SCE, get_gene_pct, set_debris_test_set,
    get_pcs, init, run_em, assign_clusters, estimate_dbr_score, call_targets'
  repo: https://github.com/marcalva/diem
- name: EmptyDrops
  role: Baseline filtering method (Dirichlet-Multinomial ambient RNA model, developed
    for scRNA-seq) against which DIEM's nuclear retention is compared
- name: Matrix
  role: R package providing the column-wise sparse matrix storage for raw droplet
    counts
- name: CellRanger
  role: Upstream 10X alignment pipeline whose output directory is the required input
    format (read via read_10x)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:2036fb5e3dac126d9044fd73a7f303c5f4ff3d7ccbaceeaf7f98019f1e73f614
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
  - diem (R package)
  - irlba
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3473
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8095238095238095
n_claims: 21
---

# snrna-seq-droplet-quality-assessment

## Summary

Assesses and filters droplets in droplet-based single-nucleus RNA-seq data by estimating per-droplet debris (ambient extranuclear RNA) contamination scores with the semi-supervised EM classifier DIEM, then calling nuclear (target) droplets. Replaces hard read/UMI/gene cutoffs, which lose nuclear droplets and retain debris, with a likelihood-based multinomial expression model.

## When to use

Use when raw 10X droplet snRNA-seq data (e.g. CellRanger output from fresh/frozen nuclei preparations) contains ambient extranuclear RNA contamination and droplets must be classified as nuclear vs background before downstream clustering or cell-type calling.

## When NOT to use

- Data are clean single-cell (scRNA-seq) droplet data where background/cell profiles differ little — DIEM's debris-score correlation with extranuclear contamination was less successful on PBMC scRNA-seq; prefer call_targets(clusters = "debris") instead of the score threshold
- There are few or no genes enriched in the background distribution — the debris score will likely be inaccurate since scoring relies on estimating debris-enriched genes
- Comparing debris scores across independent samples for integration — scores are normalized within-sample and cannot be assumed to be on the same scale if extranuclear RNA distributions differ

## Inputs

- Raw 10X CellRanger output directory (matrix.mtx/barcodes.tsv/features.tsv) for a droplet-based snRNA-seq experiment, e.g. 10X nuclei_2k fresh mouse brain nuclei
- SCE object from diem's create_SCE holding all droplets as a column-wise sparse counts Matrix
- Gene annotation sets for percent metrics: mitochondrial genes (^mt-) and nuclear MALAT1 (^malat1$)

## Outputs

- Per-droplet debris scores (score.debris) quantifying ambient extranuclear RNA contamination
- Filtered set of passing (target/nuclear) droplets with background droplets removed at thresh_score = 0.5
- Cluster-level summaries (summarize_clusters) and QC plots (barcode_rank_plot, plot_clust of n_genes/pct.mt vs score.debris)
- SCE object convertible to Seurat via convert_to_seurat; result table of nuclear vs background counts among passing droplets

## How to apply

Load the raw CellRanger matrix with diem's read_10x (column-wise sparse Matrix from the R package Matrix, retaining all droplets including single-read barcodes) and build an SCE object with create_SCE. Compute per-droplet quality metrics — percent mitochondrial reads and percent nuclear MALAT1 — using get_gene_pct (mt genes grepped via '^mt-', MALAT1 via '^malat1$'), and inspect barcode_rank_plot. Fix the debris/test split with set_debris_test_set (min_counts = 100 is the default for full snRNA-seq data; the tutorial uses 10 on the small set), optionally filter_genes (cpm_thresh = 0), then initialize k = 20 putative cell-type clusters by PCA on top variable genes (get_pcs, default n_pcs = 30, droplets with ≥200 genes) followed by k-means via init (merging clusters smaller than 10 droplets). Run the semi-supervised EM (run_em, debris cluster fixed, Dirichlet-multinomial with a prior added in v2.4.0), assign_clusters, estimate per-droplet debris scores with estimate_dbr_score (DE genes via Welch's t-test at FDR p < 0.05; clusters with mean genes < 200 counted as debris), and call passing droplets with call_targets(thresh_score = 0.5, min_genes = 0). Validate by checking debris score correlation with pct.mt/MALAT1 via plot_clust and summarize_clusters; for the 10X nuclei_2k mouse brain set, expect ~1,850 of 2,010 passing droplets (92.0%) classified nuclear, exceeding EmptyDrops (89.8%) and quantile filtering (87.6%).

## Related tools

- **DIEM** (Semi-supervised EM classifier that clusters droplets, estimates debris scores, and filters background droplets in snRNA-seq) — https://github.com/marcalva/diem
- **diem (R package)** (Implements the pipeline: read_10x, create_SCE, get_gene_pct, set_debris_test_set, get_pcs, init, run_em, assign_clusters, estimate_dbr_score, call_targets) — https://github.com/marcalva/diem
- **EmptyDrops** (Baseline filtering method (Dirichlet-Multinomial ambient RNA model, developed for scRNA-seq) against which DIEM's nuclear retention is compared)
- **Matrix** (R package providing the column-wise sparse matrix storage for raw droplet counts)
- **CellRanger** (Upstream 10X alignment pipeline whose output directory is the required input format (read via read_10x))

## Examples

```
counts <- read_10x("path/to/10x"); sce <- create_SCE(counts); sce <- set_debris_test_set(sce); sce <- get_pcs(sce); sce <- init(sce); sce <- run_em(sce); sce <- assign_clusters(sce); sce <- estimate_dbr_score(sce); sce <- call_targets(sce, thresh_score = 0.5)
```

## Evaluation signals

- Reproduction check on 10X nuclei_2k mouse brain: DIEM filtering keeps 1,850 of 2,010 passing droplets (92.0%) nuclear, vs EmptyDrops 89.8% and quantile 87.6%
- Debris scores should correlate with extranuclear contamination markers — plot pct.mt and MALAT1 against score.debris and confirm expected ordering via plot_clust
- Structural check: droplets below the set_debris_test_set count threshold (min_counts = 100 for full snRNA-seq) are fixed as debris and excluded from the test set
- Passing-droplet classification: each droplet above thresh_score = 0.5 (default) is background; verify call_targets used min_genes = 0 and that passing droplets are then classifiable as nuclear vs background by percent spliced reads
- Schema check: output table (e.g. diem_nuclei2k_filtering_results.csv) contains counts and proportions of nuclear and background droplets among the total passing droplets

## Limitations

- Debris scores may not be comparable across independent experiments or when integrating multiple samples, since they are normalized relative to within-sample cluster means and the distribution of extranuclear RNA may differ across samples
- The scoring approach relies on accurate estimation of debris-enriched genes; if few or no genes are increased in the background distribution, the debris score will likely be inaccurate
- Correlation of debris score with extranuclear contamination was less successful in PBMC scRNA-seq data, likely because background and cell-type RNA profiles differ less in scRNA-seq
- The debris score threshold can be adjusted according to the desired tolerance for contamination — a fixed threshold is not universally optimal
- EmptyDrops was originally developed and tested on single-cell data, so its model assumptions differ from DIEM's when applied to snRNA-seq

## Evidence

- [intro] The parameters of the model are inferred using semi-supervised EM13,14, where all droplets below a hard count threshold are fixed as debris.: "The parameters of the model are inferred using semi-supervised EM13,14, where all droplets below a hard count threshold are fixed as debris."
- [methods] mb_small <- call_targets(mb_small, thresh_score = 0.5, min_genes = 0): "mb_small <- call_targets(mb_small,
                         thresh_score = 0.5,
                         min_genes = 0)"
- [methods] set_debris_test_set with min_counts parameter: "mb_small <- set_debris_test_set(mb_small, min_counts = 10)"
- [readme] README DIEM workflow call chain: "sce <- set_debris_test_set(sce)
sce <- filter_genes(sce)
sce <- get_pcs(sce)
sce <- init(sce)
sce <- run_em(sce)"
- [readme] README threshold guidance for snRNA-seq data: "# Call targets using debris score for single-nucleus data
sce <- call_targets(sce, thresh_score = 0.5)"
- [discussion] Debris score cross-sample comparability caveat: [evidence span withheld — anchor status: not_found]
- [discussion] Threshold adjustability caveat: "The debris score threshold that removes droplets can be adjusted according to the desired tolerance for contamination."
