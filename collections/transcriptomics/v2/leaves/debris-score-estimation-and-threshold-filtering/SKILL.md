---
name: debris-score-estimation-and-threshold-filtering
description: Use when filtering contaminated (extranuclear RNA) droplets from 10X
  droplet-based snRNA-seq CellRanger output where hard count/UMI/gene cutoffs or EmptyDrops
  lose nuclear droplets or retain debris.
when_to_use_negative:
- Data is single-cell RNA-seq (e.g., PBMCs) where background and cell-type expression
  profiles differ little and debris-score correlation with contamination is less successful;
  use the debris-cluster-based call_targets(clusters = "debris") strategy instead
  of score thresholding.
- There are no genes increased in the background distribution, so debris-enriched
  genes cannot be accurately estimated and the resulting debris score will be unreliable.
- Comparing debris scores across independent samples or integrating multiple samples
  where the distribution of extranuclear RNA differs, since scores are normalized
  relative to within-sample cluster means and are not on a common scale.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: DIEM
  role: Semi-supervised EM classifier that clusters droplets, estimates per-droplet
    debris scores (estimate_dbr_score), and filters droplets via a debris score threshold
    (call_targets, thresh_score = 0.5).
  repo: https://github.com/marcalva/diem
- name: R
  role: Implementation language; raw counts are stored in a column-wise sparse Matrix
    via the Matrix package.
- name: Matrix
  role: Provides the column-wise sparse matrix representation for raw counts read
    by diem::read_10x.
- name: CellRanger
  role: Produces the 10X alignment output (raw counts per droplet) that read_10x ingests.
- name: irlba
  role: Computes the principal components (get_pcs, n_pcs = 30) used for k-means initialization
    of the multinomial mixture clusters.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:11446531d430bb43cc447afae6a29fccfaf7d6fda2209d24442d15af6a434da0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - CellRanger
  - DIEM
  - Matrix
  - R
  - irlba
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# debris-score-estimation-and-threshold-filtering

## Summary

Estimate per-droplet debris scores with DIEM's semi-supervised multinomial mixture EM and remove background-contaminated droplets by thresholding the debris score, instead of applying hard count cutoffs, in droplet-based snRNA-seq data.

## When to use

Use when filtering contaminated (extranuclear RNA) droplets from 10X droplet-based snRNA-seq CellRanger output where hard count/UMI/gene cutoffs or EmptyDrops lose nuclear droplets or retain debris.

## When NOT to use

- Data is single-cell RNA-seq (e.g., PBMCs) where background and cell-type expression profiles differ little and debris-score correlation with contamination is less successful; use the debris-cluster-based call_targets(clusters = "debris") strategy instead of score thresholding.
- There are no genes increased in the background distribution, so debris-enriched genes cannot be accurately estimated and the resulting debris score will be unreliable.
- Comparing debris scores across independent samples or integrating multiple samples where the distribution of extranuclear RNA differs, since scores are normalized relative to within-sample cluster means and are not on a common scale.

## Inputs

- Raw 10X CellRanger output directory (matrix.mtx, barcodes.tsv, genes.tsv) from a droplet-based snRNA-seq experiment
- SCE object created by diem::create_SCE from a column-wise sparse Matrix of raw counts
- Optional gene sets for quality metrics (e.g., genes matching ^mt- for percent mitochondrial reads, malat1 for percent nuclear MALAT1)

## Outputs

- SCE object annotated with per-droplet debris scores (score.debris) and cluster assignments
- Set of passing (target) droplets retained after thresholding debris scores at thresh_score = 0.5
- Quality-control summaries such as summarize_clusters output and plot_clust panels of score.debris vs. n_genes and pct.mt

## How to apply

Load the raw 10X CellRanger counts with diem's read_10x into a column-wise sparse Matrix (keeping all droplets, including those with a single mapped read), and wrap them in an SCE object with create_SCE. Fix the debris/test split with set_debris_test_set (min_counts = 100 for full snRNA-seq data), then initialize k = 20 clusters via PCA (get_pcs, n_pcs = 30) and k-means (init) and run the semi-supervised EM (run_em) with the debris cluster fixed. Assign droplets to clusters with assign_clusters and compute per-droplet debris scores with estimate_dbr_score, where DE genes are selected via Welch's t-test at FDR p < 0.05 and clusters with mean genes < 200 are treated as debris. Finally call passing droplets with call_targets at thresh_score = 0.5 (min_genes = 0), removing droplets whose debris score exceeds the threshold; the threshold can be adjusted according to the desired tolerance for contamination.

## Related tools

- **DIEM** (Semi-supervised EM classifier that clusters droplets, estimates per-droplet debris scores (estimate_dbr_score), and filters droplets via a debris score threshold (call_targets, thresh_score = 0.5).) — https://github.com/marcalva/diem
- **R** (Implementation language; raw counts are stored in a column-wise sparse Matrix via the Matrix package.)
- **Matrix** (Provides the column-wise sparse matrix representation for raw counts read by diem::read_10x.)
- **CellRanger** (Produces the 10X alignment output (raw counts per droplet) that read_10x ingests.)
- **irlba** (Computes the principal components (get_pcs, n_pcs = 30) used for k-means initialization of the multinomial mixture clusters.)

## Examples

```
sce <- call_targets(estimate_dbr_score(assign_clusters(run_em(init(get_pcs(filter_genes(set_debris_test_set(create_SCE(read_10x("path/to/10x"))))))))), thresh_score = 0.5, min_genes = 0)
```

## Evaluation signals

- Reproduce the mouse brain nuclei_2k benchmark: DIEM keeps 1,850 of 2,010 passing droplets (92.0%) classified as nuclear, exceeding EmptyDrops (89.8%) and quantile filtering (87.6%).
- Invariants: all droplets below the hard count threshold (set_debris_test_set min_counts = 100) are fixed as debris before EM; DE genes for scoring pass Welch's t-test at FDR p < 0.05; debris clusters have mean genes < 200.
- Inspect plot_clust panels of pct.mt vs. score.debris and n_genes vs. score.debris (log_x = TRUE): debris-enriched clusters should show elevated score.debris consistent with extranuclear contamination measured by percent spliced reads.
- After call_targets(thresh_score = 0.5, min_genes = 0), no retained droplet has a debris score above the threshold, and retained droplets show low percent mitochondrial/spliced-read background signal.
- Barcode rank plot (barcode_rank_plot) confirms the full droplet distribution was used, not a knee-point-filtered subset.

## Limitations

- Debris scores cannot be assumed to be on the same scale in independent experiments; they may not be comparable when integrating multiple samples with differing distributions of extranuclear RNA.
- Scoring relies on accurate estimation of debris-enriched genes; if few or no genes are increased in the background distribution, the debris score will likely be inaccurate.
- In scRNA-seq (e.g., PBMC data), debris score correlation with extranuclear contamination was less successful, likely due to smaller differences between background and cell type profiles; EmptyDrops' model assumptions also differ because it was developed for single-cell data.
- Droplets below the hard count threshold are always fixed as debris by the semi-supervised EM, so a portion of the hard-cutoff behavior is retained.

## Evidence

- [intro] semi-supervised EM fixes low-count droplets as debris: "The parameters of the model are inferred using semi-supervised EM13,14, where all droplets below a hard count threshold are fixed as debris."
- [discussion] debris score threshold tunable: "The debris score threshold that removes droplets can be adjusted according to the desired tolerance for contamination."
- [discussion] debris scores not comparable across experiments: [evidence span withheld — anchor status: not_found]
- [methods] call_targets invocation: "mb_small <- call_targets(mb_small,
                         thresh_score = 0.5,
                         min_genes = 0)"
- [methods] estimate_dbr_score function: "We estimate the debris score by ruunning the function `estimate_dbr_score`"
- [readme] call targets via debris score for snRNA-seq: "# Call targets using debris score for single-nucleus data
sce <- call_targets(sce, thresh_score = 0.5)"
