---
name: single-cell-cell-cycle-scoring
description: Use when a single-cell RNA-seq dataset (e.g., the 145-cell Llorens-Bobadilla
  neural stem cell data) contains quiescent G0-like cells that standard cell-cycle
  analysis does not detect and must be removed before pseudotime inference.
when_to_use_negative:
- When the goal is to retain cell-cycle signal for trajectory reconstruction — redPATH
  instead contrasts this approach with the traditional strategy of removing cycling
  genes to remove cell cycle effects
- When pseudotime results already perform poorly, since redPATH notes downstream cell-type-label-free
  approaches may fail when pseudotime performs poorly and G0 filtering is a preprocessing
  step feeding redPATH
- When data lack per-cell cycle-phase scores (no reCAT/Cyclebase-annotated scores),
  since k-means and ANOVA operate on the six mean_scores columns
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: redPATH
  role: Provides kmeans_clustering, statistical_test, distribution_plot, and mean_score_plot
    functions that implement the G0-like cell detection workflow
  repo: https://github.com/tinglabs/redPATH
- name: reCAT
  role: Produces the per-cell mean cell-cycle scores (mean_scores) over six Cyclebase-annotated
    phases that serve as clustering input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:b1ff17c69499a39778e428babde8a715548f96b5670a5c408feaab7506227877
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - reCAT
  - redPATH
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-cell-cycle-scoring

## Summary

Identifies quiescent (G0-like) cells in single-cell RNA-seq data by clustering reCAT-derived mean cell-cycle scores (G1, S, G1/S, G2, M, G2/M) with k-means and flagging the cluster that is significantly lowest across all six scores via pairwise ANOVA, as implemented in redPATH's optional G0 detection module.

## When to use

Use when a single-cell RNA-seq dataset (e.g., the 145-cell Llorens-Bobadilla neural stem cell data) contains quiescent G0-like cells that standard cell-cycle analysis does not detect and must be removed before pseudotime inference.

## When NOT to use

- When the goal is to retain cell-cycle signal for trajectory reconstruction — redPATH instead contrasts this approach with the traditional strategy of removing cycling genes to remove cell cycle effects
- When pseudotime results already perform poorly, since redPATH notes downstream cell-type-label-free approaches may fail when pseudotime performs poorly and G0 filtering is a preprocessing step feeding redPATH
- When data lack per-cell cycle-phase scores (no reCAT/Cyclebase-annotated scores), since k-means and ANOVA operate on the six mean_scores columns

## Inputs

- reCAT-derived mean cell-cycle scores matrix (mean_score/mean_scores covering G1, S, G1/S, G2, M, G2/M phases), e.g. llorens_reCAT_scores
- m genes x n cells expression matrix whose columns correspond to the scored cells, e.g. llorens_exprs (145 cells)
- k-means cluster assignments from kmeans_clustering(mean_scores, k = 5)

## Outputs

- Flagged G0-like cluster identifier (e.g. "Possible group of G0-like cells is: 2")
- Filtered expression matrix with G0-like cells removed (89 cells removed on NSC-Llorens)
- Diagnostic plots: distribution_plot and mean_score_plot over mean_scores and cluster assignments

## How to apply

Load the reCAT-derived mean cell-cycle scores (e.g., llorens_reCAT_scores from the packaged Llorens-Bobadilla dataset) and run redPATH's kmeans_clustering on the mean_scores matrix with k = 5 to partition cells into cell-cycle groups. Apply statistical_test with threshold = 0.001, which uses pairwise ANOVA to flag the k-means group significantly lowest (P < 0.001) across all six Cyclebase-annotated phase scores as the G0-like population; the test prints the flagged group, e.g. "Possible group of G0-like cells is: 2". Remove the flagged cells (e.g., new_input_data <- input_data[,-which(kmeans_clusters == 2)]) — on the NSC-Llorens dataset this removes 89 cells. Generate diagnostic distribution_plot and mean_score_plot (with reCAT_ordIndex) to visually verify that the flagged group shows depressed scores across all phases before proceeding to redPATH pseudotime inference.

## Related tools

- **redPATH** (Provides kmeans_clustering, statistical_test, distribution_plot, and mean_score_plot functions that implement the G0-like cell detection workflow) — https://github.com/tinglabs/redPATH
- **reCAT** (Produces the per-cell mean cell-cycle scores (mean_scores) over six Cyclebase-annotated phases that serve as clustering input)

## Examples

```
kmeans_clusters <- kmeans_clustering(sample_mean_score$mean_scores, k = 5); test_for_g0 <- statistical_test(sample_mean_score$mean_scores, kmeans_clusters, threshold = 0.001); new_input_data <- input_data[,-which(kmeans_clusters == 2)]
```

## Evaluation signals

- statistical_test prints a named group, e.g. "Possible group of G0-like cells is: 2"; the output must name exactly one cluster passing P < 0.001 across all six scores
- On the NSC-Llorens dataset, the flagged set should overlap the 89 reported G0-like cells out of 145 cells
- distribution_plot and mean_score_plot must show the flagged cluster is lowest in all six phase scores (G1, S, G1/S, G2, M, G2/M)
- Filtered expression matrix column count must equal original n cells minus the flagged G0-like cells (145 - 89 = 56 for Llorens)
- k-means must run with k = 5 on the mean_scores matrix, matching the documented redPATH parameters

## Limitations

- G0 detection is an optional preprocessing function; no prior algorithm existed for G0-like detection before this cell-cycle-score approach, so validation relies on dataset-specific reported counts (89 cells in Llorens)
- redPATH's downstream label-free approaches may fail when pseudotime results perform poorly, and G0 filtering only partially de-risks this
- Threshold and k = 5 were tuned on the neural stem cell example; behavior on datasets with different cell-cycle composition is not established

## Evidence

- [readme] kmeans clustering then ANOVA flagging: [evidence span withheld — anchor status: external_doc]
- [readme] optional G0 detection function: "An optional function is also available to identify the G0-like cells"
- [methods] documented k parameter: "kmeans_clusters <- kmeans_clustering(sample_mean_score$mean_scores, k = 5)"
- [discussion] G0 detection as redPATH analysis capability: "1) detection of G0-like cells"
- [intro] contrast with traditional cycling-gene removal: "Traditional trajectory inference analyses would remove cell cycle effects through the removal of cycling genes."
