---
name: cell-cycle-score-clustering
description: Use when a single-cell RNA-seq dataset may contain quiescent G0-like
  cells that confound pseudotime reconstruction, and cell-cycle scores (e.g. reCAT
  mean scores) are available to identify and remove them.
when_to_use_negative:
- The dataset contains no reCAT (or equivalent) cell-cycle scores, since kmeans_clustering
  and statistical_test operate on the mean score matrix
- Trajectory inference is planned on the full proliferative population where G0-like
  quiescent cells are biologically relevant and should be retained
- The number of expected cell-cycle states is far from the k = 5 assumption used in
  the redPATH NSC example
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: redPATH
  role: provides kmeans_clustering and statistical_test for G0-like cell detection
    on reCAT mean scores, plus redpath pseudotime used to evaluate the filtered dataset
  repo: https://github.com/tinglabs/redPATH
- name: reCAT
  role: supplies the mean cell-cycle scores that are clustered to flag G0-like quiescent
    cells
- name: SCORPIUS
  role: comparison pseudotime method used to benchmark redPATH's reliability on the
    G0-removed 56-cell dataset
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:cfee5b04dda583d1ff1e59fd9608a4070a7b0b31141f708894dfeca2e0a2294f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - SCORPIUS
  - reCAT
  - redPATH
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cell-cycle-score-clustering

## Summary

Detects G0-like quiescent cells by k-means clustering (k = 5) of reCAT mean cell-cycle scores followed by a statistical test (threshold = 0.001), so the flagged G0 cluster can be removed before pseudotime inference with redPATH.

## When to use

Use when a single-cell RNA-seq dataset may contain quiescent G0-like cells that confound pseudotime reconstruction, and cell-cycle scores (e.g. reCAT mean scores) are available to identify and remove them.

## When NOT to use

- The dataset contains no reCAT (or equivalent) cell-cycle scores, since kmeans_clustering and statistical_test operate on the mean score matrix
- Trajectory inference is planned on the full proliferative population where G0-like quiescent cells are biologically relevant and should be retained
- The number of expected cell-cycle states is far from the k = 5 assumption used in the redPATH NSC example

## Inputs

- reCAT mean cell-cycle scores (e.g. llorens_reCAT_sample_data / llorens_reCAT_scores with $mean_scores)
- genes-by-cells expression matrix (e.g. llorens_exprs, 145 NSC cells)
- known cell type/stage labels for downstream evaluation (e.g. llorens_labels)

## Outputs

- kmeans cluster assignments of cells (k = 5)
- identified G0-like cluster index from statistical_test
- G0-removed expression matrix (56 cells for NSC-Llorens)
- diagnostic plots (distribution_plot, mean_score_plot)
- downstream pseudotime and CI/BI/KC evaluation metrics (e.g. pseudotime_evaluation_metrics.csv)

## How to apply

Load reCAT sample mean cell-cycle scores (e.g. llorens_reCAT_sample_data) alongside the expression matrix (llorens_exprs, 145 neural stem cells). Run kmeans_clustering on the mean scores with k = 5, then apply statistical_test with threshold = 0.001, which reports the possible G0-like cluster (e.g. 'Possible group of G0-like cells is: 2'). Remove the flagged cluster's columns from the expression matrix — for NSC-Llorens this leaves 56 cells. Optionally inspect distribution_plot and mean_score_plot to verify cluster separation before removal. After removal, subset to GO development genes (get_GO_exp, organism = 'mouse', takelog = FALSE), filter by dispersion_threshold = 10, and run redpath with threadnum = 4 and base_path_range = c(4:8); reliability is judged by comparing CI, BI, and KC metrics against SCORPIUS (redPATH: CI 0.868, BI 0.973, KC 0.832 vs SCORPIUS: CI 0.849, BI 0.768, KC 0.566 on the 56-cell set).

## Related tools

- **redPATH** (provides kmeans_clustering and statistical_test for G0-like cell detection on reCAT mean scores, plus redpath pseudotime used to evaluate the filtered dataset) — https://github.com/tinglabs/redPATH
- **reCAT** (supplies the mean cell-cycle scores that are clustered to flag G0-like quiescent cells)
- **SCORPIUS** (comparison pseudotime method used to benchmark redPATH's reliability on the G0-removed 56-cell dataset)

## Examples

```
kmeans_clusters <- kmeans_clustering(sample_mean_score$mean_scores, k = 5); test_for_g0 <- statistical_test(sample_mean_score$mean_scores, kmeans_clusters, threshold = 0.001); new_input_data <- input_data[,-which(kmeans_clusters == 2)]
```

## Evaluation signals

- statistical_test prints the G0-like cluster (e.g. 'Possible group of G0-like cells is: 2') at threshold = 0.001
- After removing the flagged cluster, the NSC-Llorens expression matrix retains exactly 56 cells
- distribution_plot and mean_score_plot show separable clusters consistent with the k = 5 assignment
- On the 56-cell G0-removed set, redPATH metrics should reproduce approximately CI 0.868, BI 0.973, KC 0.832, exceeding SCORPIUS (CI 0.849, BI 0.768, KC 0.566)
- Results written to pseudotime_evaluation_metrics.csv match the reported benchmark values

## Limitations

- The G0-detection approach relies on k-means cluster quality; it may fail when the pseudotime results perform poorly, as noted in the article's discussion
- Removal of G0-like cells reduces the dataset to very small sample sizes (56 cells in the NSC-Llorens example), which stresses downstream tools
- The choice k = 5 and threshold 0.001 are example-specific parameters and may need tuning for other datasets

## Evidence

- [readme] kmeans clusters from mean reCAT scores: "kmeans_clusters <- kmeans_clustering(sample_mean_score$mean_scores, k = 5)"
- [readme] statistical test flags G0-like group: "test_for_g0 <- statistical_test(sample_mean_score$mean_scores, kmeans_clusters, threshold = 0.001)"
- [results] G0-like cells removed leaving 56 cells, redPATH metrics reported: [evidence span withheld — anchor status: not_found]
- [discussion] failure mode caveat: "Although the advantage is that cell type labeling is not required here, this approach may fail when the pseudotime results perform poorly."
