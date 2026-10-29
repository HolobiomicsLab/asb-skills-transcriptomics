---
name: g0-like-cell-detection
description: Use when a scRNA-seq dataset may contain quiescent G0-like cells that
  confound trajectory inference, and reCAT cell-cycle scores (G1, S, G1/S, G2, M,
  G2/M) are available for the cells.
when_to_use_negative:
- The dataset contains no cycling/quiescent heterogeneity and no reCAT cell-cycle
  scores are available, so k-means on mean scores cannot be computed
- The goal is to preserve quiescent cells in the trajectory (e.g. a study specifically
  about quiescence states) rather than removing them before pseudotime inference
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: redPATH
  role: Provides kmeans_clustering, statistical_test, distribution_plot, and mean_score_plot
    functions used to cluster cells on cell-cycle scores and flag the G0-like group
  repo: https://github.com/tinglabs/redPATH
- name: reCAT
  role: Supplies the mean cell-cycle scores (G1, S, G1/S, G2, M, G2/M) computed from
    Cyclebase annotations that serve as the clustering input
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:e4260ea2b6d614ffaf735b6be474992cc33b6e3bb477971ca2ea4b5a6cb54db9
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# g0-like-cell-detection

## Summary

Identification and removal of G0-like quiescent cells from single-cell RNA-seq datasets using reCAT-derived mean cell-cycle scores, k-means clustering, and pairwise ANOVA testing, as implemented in the redPATH package prior to pseudotime inference.

## When to use

Use when a scRNA-seq dataset may contain quiescent G0-like cells that confound trajectory inference, and reCAT cell-cycle scores (G1, S, G1/S, G2, M, G2/M) are available for the cells.

## When NOT to use

- The dataset contains no cycling/quiescent heterogeneity and no reCAT cell-cycle scores are available, so k-means on mean scores cannot be computed
- The goal is to preserve quiescent cells in the trajectory (e.g. a study specifically about quiescence states) rather than removing them before pseudotime inference

## Inputs

- reCAT-derived mean cell-cycle scores matrix (six scores per cell: G1, S, G1/S, G2, M, G2/M), e.g. llorens_reCAT_sample_data
- genes-by-cells expression matrix, e.g. llorens_exprs (145 cells for the Llorens-Bobadilla NSC dataset)

## Outputs

- k-means cluster assignments for each cell (k = 5)
- flagged G0-like cell identifiers (the cluster significantly lowest at P < 0.001)
- G0-removed expression matrix (e.g. 56 cells remaining from 145 after removing 89 G0-like cells)
- diagnostic plots: distribution_plot and mean_score_plot outputs

## How to apply

Load the reCAT-derived mean cell-cycle scores (e.g. llorens_reCAT_sample_data for the 145-cell Llorens-Bobadilla neural stem cell dataset) and run redPATH's kmeans_clustering with k = 5 to partition cells into cell-cycle groups. Apply statistical_test on the mean scores and cluster assignments with threshold = 0.001 to flag the group that is significantly lowest (P < 0.001) across all six phase scores as G0-like (e.g. 'Possible group of G0-like cells is: 2'). Remove the flagged cluster from the expression matrix (new_input_data <- input_data[,-which(kmeans_clusters == 2)]), leaving 56 cells for the NSC-Llorens example. Generate distribution_plot and mean_score_plot diagnostics to verify the flagged group's score profiles. The rationale is that quiescent cells lack cycling signatures across all six cell-cycle phases, so a cluster low in every score identifies G0-like cells without prior cell-type labels.

## Related tools

- **redPATH** (Provides kmeans_clustering, statistical_test, distribution_plot, and mean_score_plot functions used to cluster cells on cell-cycle scores and flag the G0-like group) — https://github.com/tinglabs/redPATH
- **reCAT** (Supplies the mean cell-cycle scores (G1, S, G1/S, G2, M, G2/M) computed from Cyclebase annotations that serve as the clustering input)

## Examples

```
kmeans_clusters <- kmeans_clustering(sample_mean_score$mean_scores, k = 5); test_for_g0 <- statistical_test(sample_mean_score$mean_scores, kmeans_clusters, threshold = 0.001); new_input_data <- input_data[,-which(kmeans_clusters == 2)]
```

## Evaluation signals

- statistical_test returns exactly one flagged cluster, announced as e.g. 'Possible group of G0-like cells is: 2'
- On the Llorens dataset, the flagged set matches the 89 reported G0-like cells removed from the 145-cell input (56 cells remain)
- The flagged cluster shows the lowest mean score across all six cell-cycle phase scores in mean_score_plot
- distribution_plot shows the flagged cluster separated from cycling clusters in cell-cycle score distributions
- Downstream redPATH pseudotime on the G0-removed matrix reproduces the reported NSC-Llorens metrics (CI: 0.868, BI: 0.973, KC: 0.832), outperforming SCORPIUS

## Limitations

- No known algorithm for G0-like detection existed prior to this cell-cycle-score approach, so validation is largely empirical against reported removal counts
- The pairwise ANOVA threshold (P < 0.001) controls which cluster is flagged; a poor clustering (k = 5) could misassign quiescent cells
- The paper notes that cell-type-label-free approaches 'may fail when the pseudotime results perform poorly', so downstream metrics should still be checked

## Evidence

- [methods] kmeans_clustering with k=5 on the mean scores, then uses pairwise ANOVA tests, flagging as G0-like the group that is significantly lowest (P < 0.001) in all six scores: [evidence span withheld — anchor status: not_found]
- [readme] README G0 detection code block: "test_for_g0 <- statistical_test(sample_mean_score$mean_scores, kmeans_clusters, threshold = 0.001)"
- [readme] README G0 removal: "new_input_data <- input_data[,-which(kmeans_clusters == 2)]"
- [intro] Article intro on cell cycle removal strategies: "Traditional trajectory inference analyses would remove cell cycle effects through the removal of cycling genes."
- [discussion] Source card finding on 89 cells removed: [evidence span withheld — anchor status: not_found]
