---
name: pseudotime-benchmark-evaluation
description: Use when pseudotime orderings from redPATH and baseline methods must
  be quantitatively compared on the same filtered expression matrix against known
  cell type/stage labels using CI, BI, KC, and POS scores.
when_to_use_negative:
- The expression matrix has not yet been reduced to GO development-related genes and
  dispersion-filtered, since comparisons must run on identical filtered matrices
- The goal is single-method pseudotime inference rather than head-to-head benchmarking
  against baselines with known cell type labels
- The dataset lacks known cell type/stage labels, making CI/BI/KC/POS evaluation against
  ground truth impossible
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: redPATH
  role: Primary trajectory inference method; run via redpath() with threadnum = 4
    and base_path_range = c(4:8) to obtain the pseudotime being benchmarked
  repo: https://github.com/tinglabs/redPATH
- name: SCORPIUS
  role: Baseline pseudotime method run on the identical filtered matrices for score
    comparison
- name: TSCAN
  role: Baseline pseudotime method run on the identical filtered matrices for score
    comparison
- name: Monocle2
  role: Baseline pseudotime method run on the identical filtered matrices for score
    comparison
- name: DPT
  role: Baseline pseudotime method run on the identical filtered matrices for score
    comparison
- name: reCAT
  role: Supplies mean cell cycle scores used by kmeans_clustering (k = 5) and statistical_test
    (threshold = 0.001) to flag G0-like cells before subsetting
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:878beaaabbd2818f60f878c668d00d118cb973cec710379296f726fb3ac67472
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - DPT
  - Monocle2
  - R
  - SCORPIUS
  - TSCAN
  - reCAT
  - redPATH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# pseudotime-benchmark-evaluation

## Summary

Benchmark redPATH against SCORPIUS, TSCAN, Monocle2, and DPT on single-cell RNA-seq datasets by computing CI, BI, KC, and POS metrics from inferred pseudotemporal orderings against known cell type labels, and reproduce reported scores (e.g., redPATH CI 0.868, BI 0.973, KC 0.832 on the 56-cell G0-removed NSC-Llorens dataset).

## When to use

Use when pseudotime orderings from redPATH and baseline methods must be quantitatively compared on the same filtered expression matrix against known cell type/stage labels using CI, BI, KC, and POS scores.

## When NOT to use

- The expression matrix has not yet been reduced to GO development-related genes and dispersion-filtered, since comparisons must run on identical filtered matrices
- The goal is single-method pseudotime inference rather than head-to-head benchmarking against baselines with known cell type labels
- The dataset lacks known cell type/stage labels, making CI/BI/KC/POS evaluation against ground truth impossible

## Inputs

- Raw single-cell expression matrix (genes by cells), e.g. llorens_exprs (145 NSC cells) or MGH107 (252 cells)
- Known cell type/stage labels, e.g. llorens_labels
- reCAT sample data with mean cell cycle scores, e.g. llorens_reCAT_sample_data (for G0-like cell removal)

## Outputs

- Pseudotime orderings per method (redPATH, SCORPIUS, TSCAN, Monocle2, DPT) on the same filtered matrices
- CI, BI, KC, and POS scores per method-dataset pair, e.g. figure2b_benchmark_scores.csv or pseudotime_evaluation_metrics.csv

## How to apply

Preprocess each raw genes-by-cells matrix by subsetting to development-related Gene Ontology genes with get_GO_exp (gene_type = 'gene_names', organism = 'mouse', takelog = F) and filtering with filter_exp (dispersion_threshold = 10, threads = 2). Run redPATH on the filtered matrix with redpath(filtered_GO_data, threadnum = 4, base_path_range = c(4:8)), where the range spans the estimated number of cell types k0 to k0 + 4, and run SCORPIUS (and optionally TSCAN, Monocle2, DPT) on the identical filtered matrices. For G0-removal variants, first identify quiescent cells via kmeans_clustering on reCAT mean scores (k = 5) and statistical_test (threshold = 0.001), then remove the flagged G0 cluster (leaving 56 cells in NSC-Llorens). Compute cell-index (CI), biologically-informed index (BI), Kendall correlation (KC), and POS for every method-dataset pair against known labels such as llorens_labels, assemble the scores into a comparison table (e.g., pseudotime_evaluation_metrics.csv), and confirm redPATH attains the best values.

## Related tools

- **redPATH** (Primary trajectory inference method; run via redpath() with threadnum = 4 and base_path_range = c(4:8) to obtain the pseudotime being benchmarked) — https://github.com/tinglabs/redPATH
- **SCORPIUS** (Baseline pseudotime method run on the identical filtered matrices for score comparison)
- **TSCAN** (Baseline pseudotime method run on the identical filtered matrices for score comparison)
- **Monocle2** (Baseline pseudotime method run on the identical filtered matrices for score comparison)
- **DPT** (Baseline pseudotime method run on the identical filtered matrices for score comparison)
- **reCAT** (Supplies mean cell cycle scores used by kmeans_clustering (k = 5) and statistical_test (threshold = 0.001) to flag G0-like cells before subsetting)

## Examples

```
subset_GO_data <- get_GO_exp(llorens_exprs, gene_type="gene_names", organism="mouse", takelog=F); filtered_GO_data <- filter_exp(subset_GO_data, dispersion_threshold=10, threads=2); redpath_pseudotime <- redpath(filtered_GO_data, threadnum=4, base_path_range=c(4:8))
```

## Evaluation signals

- redPATH scores should meet or exceed all baselines on CI, BI, KC, and POS across the neuronal and hematopoietic datasets
- On the 56 cells remaining after G0 removal from NSC-Llorens, redPATH should reproduce CI: 0.868, BI: 0.973, KC: 0.832 versus SCORPIUS CI: 0.849, BI: 0.768, KC: 0.566
- G0 removal step should leave exactly 56 cells in the NSC-Llorens expression matrix
- statistical_test on reCAT mean scores with threshold = 0.001 should output a flagged G0-like cluster (README example: 'Possible group of G0-like cells is: 2')
- All methods must be evaluated on the same filtered_GO_data matrix so scores are directly comparable

## Limitations

- This label-based evaluation approach may fail when the pseudotime results perform poorly, since cell type labels are not required as input to the method itself
- redPATH performance on very small datasets (e.g., 56 cells) is a specific stress case validated in the benchmark; behavior outside tested datasets is not guaranteed
- base_path_range must be set to the estimated number of cell types k0 to k0 + 4; a wrong estimate can degrade pseudotime quality

## Evidence

- [results] On the 56 cells remaining after G0 removal from NSC-Llorens, redPATH achieved CI: 0.868, BI: 0.973, KC: 0.832, outperforming SCORPIUS (CI: 0.849, BI: 0.768, KC: 0.566): [evidence span withheld — anchor status: not_found]
- [methods] Run the four baselines, SCORPIUS, TSCAN, Monocle2, and DPT, on the identical filtered matrices to obtain comparable pseudotime orderings.: [evidence span withheld — anchor status: not_found]
- [readme] filtered_GO_data <- filter_exp(subset_GO_data, dispersion_threshold = 10, threads = 2): "filtered_GO_data <- filter_exp(subset_GO_data, dispersion_threshold = 10, threads = 2)"
- [readme] redpath_pseudotime <- redpath(filtered_GO_data, threadnum = 4, base_path_range = c(4:8)): "redpath_pseudotime <- redpath(filtered_GO_data, threadnum = 4, base_path_range = c(4:8))"
- [readme] test_for_g0 <- statistical_test(sample_mean_score$mean_scores, kmeans_clusters, threshold = 0.001): "test_for_g0 <- statistical_test(sample_mean_score$mean_scores, kmeans_clusters, threshold = 0.001)"
