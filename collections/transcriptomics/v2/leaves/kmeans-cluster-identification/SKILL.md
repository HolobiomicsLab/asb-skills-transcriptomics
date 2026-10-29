---
name: kmeans-cluster-identification
description: Use when single-cell RNA-seq expression data is accompanied by reCAT
  mean cell-cycle scores and quiescent/G0-like cells must be removed before pseudotime
  reconstruction, or when no cell-type labels are available for gating non-cycling
  cells.
when_to_use_negative:
- No reCAT (or equivalent) cell-cycle score matrix exists for the dataset — the clustering
  operates on mean phase scores, not expression values
- The goal is to study cycling/proliferating subpopulations rather than remove quiescent
  cells before trajectory inference
- Cell-cycle scores are unreliable or the pseudotime/phase decomposition performs
  poorly, which the paper notes can make this labeling approach fail
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: redPATH
  role: Provides kmeans_clustering, statistical_test, distribution_plot, and mean_score_plot
    functions that cluster mean cell-cycle scores and flag the G0-like group
  repo: https://github.com/tinglabs/redPATH
- name: reCAT
  role: Produces the normalized cell-cycle scoring output from which the mean_scores
    matrix (G1, S, G1/S, G2, M, G2/M) is computed for clustering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:61e2c0878f497072e794ed520b8e4d97e6b975511d66dea2ad4f2469f5f9ccf8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - reCAT
  - redPATH
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# kmeans-cluster-identification

## Summary

Identify G0-like (quiescent) cells by running k-means (k = 5) on reCAT-derived mean cell-cycle scores (G1, S, G1/S, G2, M, G2/M) and flagging the cluster that is significantly lowest (pairwise ANOVA, P < 0.001) across all six phase scores. This is the redPATH approach for detecting quiescent cells prior to trajectory inference.

## When to use

Use when single-cell RNA-seq expression data is accompanied by reCAT mean cell-cycle scores and quiescent/G0-like cells must be removed before pseudotime reconstruction, or when no cell-type labels are available for gating non-cycling cells.

## When NOT to use

- No reCAT (or equivalent) cell-cycle score matrix exists for the dataset — the clustering operates on mean phase scores, not expression values
- The goal is to study cycling/proliferating subpopulations rather than remove quiescent cells before trajectory inference
- Cell-cycle scores are unreliable or the pseudotime/phase decomposition performs poorly, which the paper notes can make this labeling approach fail

## Inputs

- reCAT mean cell-cycle score matrix (cells x 6 phases: G1, S, G1/S, G2, M, G2/M), e.g. llorens_reCAT_scores$mean_scores
- raw gene-by-cell expression matrix (e.g. llorens_exprs, 145 neural stem cells) from which flagged cells will be removed
- k for k-means (default k = 5 in redPATH)
- ANOVA significance threshold (default 0.001)

## Outputs

- k-means cluster assignment per cell (kmeans_clusters)
- G0-like group identifier from statistical_test (e.g. 'Possible group of G0-like cells is: 2')
- filtered expression matrix with G0-like cells removed (new_input_data)
- diagnostic plots from distribution_plot and mean_score_plot
- flagged G0-like cell identifiers and overlap summary (e.g. g0_like_cells.csv)

## How to apply

Load the reCAT-derived mean cell-cycle score matrix (e.g. llorens_reCAT_scores for the 145-cell Llorens-Bobadilla neural stem cell dataset) and run redPATH's kmeans_clustering on the mean_scores with k = 5 to partition cells into cell-cycle groups. Apply redPATH's statistical_test on the same mean_scores and cluster assignments with threshold = 0.001; the function performs pairwise ANOVA tests across the six phase scores (G1, S, G1/S, G2, M, G2/M from Cyclebase annotations) and reports which group is significantly lowest. Cells in the reported group (e.g. group 2 in the README example) are flagged as G0-like and removed via input_data[,-which(kmeans_clusters == g)]. Generate diagnostic plots with distribution_plot and mean_score_plot, and compare the flagged set against known counts (89 G0-like cells in the Llorens dataset) to validate. Because cell-cycle clustering indices can shift between runs, always use the statistical_test output — not a hard-coded group number — to select the G0-like cluster.

## Related tools

- **redPATH** (Provides kmeans_clustering, statistical_test, distribution_plot, and mean_score_plot functions that cluster mean cell-cycle scores and flag the G0-like group) — https://github.com/tinglabs/redPATH
- **reCAT** (Produces the normalized cell-cycle scoring output from which the mean_scores matrix (G1, S, G1/S, G2, M, G2/M) is computed for clustering)

## Examples

```
kmeans_clusters <- kmeans_clustering(sample_mean_score$mean_scores, k = 5); test_for_g0 <- statistical_test(sample_mean_score$mean_scores, kmeans_clusters, threshold = 0.001)
```

## Evaluation signals

- statistical_test prints exactly one G0-like group whose cells are significantly lowest (P < 0.001) across all six phase scores
- Mean cell-cycle scores of the flagged group should be lower than all other k-means groups in the mean_score_plot
- On the Llorens-Bobadilla neural stem cell dataset, the flagged set should closely overlap the 89 reported G0-like cells
- The filtered matrix (new_input_data) should have fewer cells than the input (145 minus flagged cells) with column indices consistent with kmeans_clusters == g0_group

## Limitations

- Cell-cycle clustering is computed without cell-type labels; the paper notes this label-free approach 'may fail when the pseudotime results perform poorly'
- The G0-like group number is data-dependent (k-means label switching), so a hard-coded group index from one run is not transferable

## Evidence

- [methods] kmeans_clustering on mean scores with k=5: "kmeans_clusters <- kmeans_clustering(sample_mean_score$mean_scores, k = 5)"
- [methods] ANOVA-based G0 test with threshold 0.001: "test_for_g0 <- statistical_test(sample_mean_score$mean_scores, kmeans_clusters, threshold = 0.001)"
- [readme] README documents the optional G0-like detection workflow: "An optional function is also available to identify the G0-like cells"
- [discussion] Label-free approach can fail with poor pseudotime: "Although the advantage is that cell type labeling is not required here, this approach may fail when the pseudotime results perform poorly."
