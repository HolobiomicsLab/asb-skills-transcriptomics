---
name: semi-supervised-em-mixture-model-clustering
description: Use when filtering droplet-based snRNA-seq data where hard count cutoffs
  or EmptyDrops fail to remove ambient/debris RNA, and you need expression-profile-based
  clustering that fixes low-count droplets as debris before fitting a mixture model.
when_to_use_negative:
- The data are already a filtered feature/barcode matrix or an aggregated per-cell
  table — DIEM needs raw droplet-level counts including background droplets
- There are few or no genes enriched in the background distribution (little extranuclear
  RNA), since debris scoring relies on accurate estimation of debris-enriched genes
  and the resulting score will likely be inaccurate
- Integrating multiple samples whose distributions of extranuclear RNA differ, because
  debris scores cannot be assumed to be on the same scale across independent experiments
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DIEM
  role: R package providing the semi-supervised EM multinomial mixture model pipeline
    (read_10x, create_SCE, get_pcs, init, run_em, assign_clusters, estimate_dbr_score,
    call_targets)
  repo: https://github.com/marcalva/diem
- name: Matrix
  role: Stores the raw droplet counts as a column-wise sparse matrix read in by read_10x
- name: irlba
  role: Used by get_pcs for PCA-based dimensionality reduction (default n_pcs = 30)
    prior to k-means initialization
- name: EmptyDrops
  role: Benchmark filtering method (89.8% nuclear droplets on mouse brain nuclei_2k)
    compared against DIEM's EM-based filtering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:e93079ed3281fb473001c8da24f3eb60a7ac06a527f8f3d400836543b83d0930
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
  - irlba
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7916666666666666
n_claims: 24
---

# semi-supervised-em-mixture-model-clustering

## Summary

DIEM's core clustering approach: a likelihood-based multinomial (Dirichlet-multinomial) mixture model whose parameters are estimated by semi-supervised EM, where all droplets below a hard count threshold are fixed as debris, to separate debris/background droplets from nuclear or cell-type droplets in droplet-based snRNA-seq data.

## When to use

Use when filtering droplet-based snRNA-seq data where hard count cutoffs or EmptyDrops fail to remove ambient/debris RNA, and you need expression-profile-based clustering that fixes low-count droplets as debris before fitting a mixture model.

## When NOT to use

- The data are already a filtered feature/barcode matrix or an aggregated per-cell table — DIEM needs raw droplet-level counts including background droplets
- There are few or no genes enriched in the background distribution (little extranuclear RNA), since debris scoring relies on accurate estimation of debris-enriched genes and the resulting score will likely be inaccurate
- Integrating multiple samples whose distributions of extranuclear RNA differ, because debris scores cannot be assumed to be on the same scale across independent experiments

## Inputs

- Raw 10X CellRanger output directory (matrix.mtx, barcodes.tsv, genes.tsv) for a droplet-based snRNA-seq experiment, e.g. the 10X nuclei_2k fresh mouse brain dataset
- diem SCE object created from a column-wise sparse counts Matrix including all droplets (even those with a single mapped read)

## Outputs

- Per-droplet cluster assignments (debris cluster vs. putative cell-type clusters) on the SCE object
- Per-droplet debris scores and target/background calls at thresh_score = 0.5
- Filtered droplet set (e.g. 1,850 nuclear of 2,010 passing droplets for nuclei_2k) usable for downstream Seurat conversion and clustering

## How to apply

Load the 10X CellRanger output into a column-wise sparse Matrix (R Matrix package) with diem's read_10x and wrap it in a SCE object with create_SCE, computing quality metrics (percent mitochondrial reads, percent MALAT1) via get_gene_pct. Fix the debris/test split with set_debris_test_set (min_counts = 100 for full snRNA-seq data; min_counts = 10 in the tutorial vignette), then initialize k = 20 cell-type clusters by PCA on top variable genes (get_pcs, default n_pcs = 30) followed by k-means (init), merging clusters smaller than 10 droplets. Fit the multinomial mixture with run_em (semi-supervised EM with the debris cluster fixed), assign droplets with assign_clusters, then rank droplets by a debris score from estimate_dbr_score, which uses DE genes between clusters (Welch's t-test at FDR p < 0.05) and counts clusters with mean genes < 200 as debris. Call passing droplets with call_targets (thresh_score = 0.5, min_genes = 0), removing droplets whose debris score exceeds the threshold; in the mouse brain nuclei_2k data this retained 1,850 of 2,010 passing droplets (92.0%) that were nuclear, versus 89.8% for EmptyDrops and 87.6% for quantile filtering.

## Related tools

- **DIEM** (R package providing the semi-supervised EM multinomial mixture model pipeline (read_10x, create_SCE, get_pcs, init, run_em, assign_clusters, estimate_dbr_score, call_targets)) — https://github.com/marcalva/diem
- **Matrix** (Stores the raw droplet counts as a column-wise sparse matrix read in by read_10x)
- **irlba** (Used by get_pcs for PCA-based dimensionality reduction (default n_pcs = 30) prior to k-means initialization)
- **EmptyDrops** (Benchmark filtering method (89.8% nuclear droplets on mouse brain nuclei_2k) compared against DIEM's EM-based filtering)

## Examples

```
counts <- read_10x("path/to/10x"); sce <- create_SCE(counts); sce <- set_debris_test_set(sce); sce <- init(sce); sce <- run_em(sce); sce <- assign_clusters(sce); sce <- estimate_dbr_score(sce); sce <- call_targets(sce, thresh_score = 0.5, min_genes = 0)
```

## Evaluation signals

- DIEM row reproduces 1,850 of 2,010 passing droplets (92.0%) classified as nuclear on the 10X nuclei_2k mouse brain dataset, exceeding EmptyDrops (89.8%) and quantile filtering (87.6%)
- All droplets below the hard count threshold of set_debris_test_set (min_counts) are fixed as debris and never change cluster during EM
- Clusters with mean genes < 200 are counted as debris, and debris-enriched genes are those DE via Welch's t-test at FDR p < 0.05
- Called targets at thresh_score = 0.5 show increasing percent spliced reads (decreasing contamination) with decreasing debris score
- Droplet assignments are stable across repeated run_em fits and across multiple k_init values (v2.1.0 supports running multiple k_init simultaneously)

## Limitations

- Debris scores may not be comparable across independent experiments or when integrating multiple samples, particularly if the distribution of extranuclear RNA differs across samples
- If there are little or no genes increased in the background distribution, the resulting debris score will likely be inaccurate
- Correlation of debris score with percent reads spliced was less successful in PBMC single-cell RNA-seq data, possibly because background and cell-type profiles differ less in scRNA-seq
- EmptyDrops' model assumptions differ from DIEM's since it was originally developed and tested on single-cell data

## Evidence

- [intro] semi-supervised EM with fixed debris cluster: "The parameters of the model are inferred using semi-supervised EM13,14, where all droplets below a hard count threshold are fixed as debris."
- [intro] likelihood-based multinomial model: "we built a fast filtering pipeline that uses a likelihood-based approach to model debris and cell type RNA distributions with a multinomial distribution."
- [discussion] clustering to identify debris cluster: "DIEM first clusters all droplets generated from the experiment to identify droplets belonging either to the debris cluster or putative cell type clusters."
- [methods] set_debris_test_set tutorial parameter: "mb_small <- set_debris_test_set(mb_small, min_counts = 10)"
- [methods] read_10x convenience function: "We provide a convenience function `read_10x` that can read the 10X output into a sparse Matrix."
- [methods] call_targets threshold: "mb_small <- call_targets(mb_small,
                         thresh_score = 0.5,
                         min_genes = 0)"
- [readme] README DIEM workflow steps: "sce <- set_debris_test_set(sce)
sce <- filter_genes(sce)
sce <- get_pcs(sce)
sce <- init(sce)
sce <- run_em(sce)"
- [readme] README call_targets for snRNA data: "# Call targets using debris score for single-nucleus data
sce <- call_targets(sce, thresh_score = 0.5)"
- [readme] v2.3.0 multinomial clustering switch: "* Quantifies amount of contamination in droplets. Filtering is
      performed using this debris score.
    * Clustering switched to multinomial mixture model to increase speed."
