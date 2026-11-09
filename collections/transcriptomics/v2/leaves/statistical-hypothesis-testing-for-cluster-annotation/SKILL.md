---
name: statistical-hypothesis-testing-for-cluster-annotation
description: Use when single-cell RNA-seq data has reCAT-derived mean cell-cycle scores
  and you need to flag and remove a quiescent/G0-like cell group (no prior algorithm
  existed for G0-like detection) before pseudotime reconstruction.
when_to_use_negative:
- No cell-cycle scores are available and reCAT has not been run — the ANOVA test requires
  the six mean cell-cycle phase scores as input.
- You want to remove cell-cycle effects by simply dropping cycling genes (the traditional
  trajectory-inference route) rather than identifying quiescent G0-like cells.
- The dataset is not expected to contain a quiescent subpopulation and removing cells
  would distort the lineage being reconstructed.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: redPATH
  role: provides kmeans_clustering, statistical_test, distribution_plot, and mean_score_plot
    functions to cluster cells by cell-cycle scores and flag the G0-like group via
    pairwise ANOVA
  repo: https://github.com/tinglabs/redPATH
- name: reCAT
  role: produces the mean cell-cycle scores (G1, S, G1/S, G2, M, G2/M) that are the
    input to the k-means clustering and ANOVA test
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:4aa15336344ff4c0dea3a12bef0287fb53763fbeee6fda8736a8aae0d4ef27f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - reCAT
  - redPATH
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# statistical-hypothesis-testing-for-cluster-annotation

## Summary

A redPATH procedure that annotates a cell-cycle k-means cluster as G0-like (quiescent) by testing whether it is significantly lowest across all six mean cell-cycle scores (G1, S, G1/S, G2, M, G2/M) using pairwise ANOVA at P < 0.001, enabling removal of quiescent cells (89 removed in the Llorens NSC dataset) before trajectory inference.

## When to use

Use when single-cell RNA-seq data has reCAT-derived mean cell-cycle scores and you need to flag and remove a quiescent/G0-like cell group (no prior algorithm existed for G0-like detection) before pseudotime reconstruction.

## When NOT to use

- No cell-cycle scores are available and reCAT has not been run — the ANOVA test requires the six mean cell-cycle phase scores as input.
- You want to remove cell-cycle effects by simply dropping cycling genes (the traditional trajectory-inference route) rather than identifying quiescent G0-like cells.
- The dataset is not expected to contain a quiescent subpopulation and removing cells would distort the lineage being reconstructed.

## Inputs

- reCAT-derived mean cell-cycle scores matrix (mean_scores: cells x 6 phases G1, S, G1/S, G2, M, G2/M, e.g., llorens_reCAT_scores)
- k value for k-means clustering (k = 5 in the reference workflow)
- significance threshold for the ANOVA test (threshold = 0.001)
- raw gene-by-cell expression matrix (e.g., llorens_exprs) from which flagged cells will be removed

## Outputs

- k-means cluster assignment per cell (kmeans_clusters)
- flagged G0-like group identifier from statistical_test (e.g., 'Possible group of G0-like cells is: 2')
- filtered expression matrix excluding G0-like cells (new_input_data)
- diagnostic plots: distribution_plot and mean_score_plot of mean_scores by cluster
- G0-like cell identifiers and comparison summary (e.g., g0_like_cells.csv; 89 cells flagged on the Llorens dataset)

## How to apply

Load the reCAT-derived mean_scores matrix (e.g., llorens_reCAT_scores from the 145-cell Llorens-Bobadilla NSC dataset) covering six cell-cycle phases (G1, S, G1/S, G2, M, G2/M) computed from Cyclebase annotations. Partition cells with redPATH's kmeans_clustering using k = 5, then call statistical_test on the mean_scores and cluster assignments with threshold = 0.001; the function reports the group significantly lowest in all six phase scores via pairwise ANOVA (e.g., 'Possible group of G0-like cells is: 2'). Remove the flagged cluster's cells from the expression matrix with input_data[,-which(kmeans_clusters == flagged_group)] and generate distribution_plot and mean_score_plot diagnostics. Judge success by comparing the flagged cell set against the 89 G0-like cells reported for the Llorens dataset and confirming the flagged group is consistently lowest across all six scores.

## Related tools

- **redPATH** (provides kmeans_clustering, statistical_test, distribution_plot, and mean_score_plot functions to cluster cells by cell-cycle scores and flag the G0-like group via pairwise ANOVA) — https://github.com/tinglabs/redPATH
- **reCAT** (produces the mean cell-cycle scores (G1, S, G1/S, G2, M, G2/M) that are the input to the k-means clustering and ANOVA test)

## Examples

```
kmeans_clusters <- kmeans_clustering(sample_mean_score$mean_scores, k = 5); test_for_g0 <- statistical_test(sample_mean_score$mean_scores, kmeans_clusters, threshold = 0.001); new_input_data <- input_data[,-which(kmeans_clusters == 2)]
```

## Evaluation signals

- statistical_test returns a single flagged group index with the message 'Possible group of G0-like cells is: <k>' and the flagged group is significantly lowest (P < 0.001) in all six phase scores
- Flagged cell set overlaps the 89 G0-like cells reported for the 145-cell Llorens-Bobadilla NSC dataset
- distribution_plot and mean_score_plot show the flagged cluster with consistently lower mean scores across G1, S, G1/S, G2, M, and G2/M
- After removal, new_input_data contains 145 - 89 = 56 fewer cells for the Llorens example (cluster membership counts match the k-means assignment)

## Limitations

- The procedure requires prior reCAT cell-cycle scoring, so it is only applicable when cell-cycle scores can be computed from the expression data
- Flagging depends on a single threshold (P < 0.001) and the chosen k = 5 clustering; a different k or threshold may flag a different group
- No known algorithm previously existed for G0-like detection prior to this cell-cycle-analysis-based approach, so validation is largely limited to the reported NSC dataset result (89 cells removed)

## Evidence

- [methods] applies k-means with k=5 to the mean scores, then uses pairwise ANOVA tests, flagging as G0-like the group that is significantly lowest (P < 0.001) in all six scores; on the NSC-Llorens dataset this procedure removed 89 cells: "kmeans_clustering(sample_mean_score$mean_scores, k = 5)"
- [readme] Example output: [1] "Possible group of G0-like cells is: 2" and removal via new_input_data <- input_data[,-which(kmeans_clusters == 2)]: "test_for_g0 <- statistical_test(sample_mean_score$mean_scores, kmeans_clusters, threshold = 0.001)"
- [supplementary] redPATH computes six mean cell-cycle scores (G1, S, G1/S, G2, M, G2/M) from Cyclebase annotations: "1) detection of G0-like cells"
- [readme] Evaluation plots d1 <- distribution_plot(...) and d2 <- mean_score_plot(...) on mean_scores and kmeans_clusters: "d2 <- mean_score_plot(sample_mean_score$mean_scores, kmeans_clusters, reCAT_ordIndex)"
