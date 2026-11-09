---
name: reference-label-overlap-comparison
description: Use when a clustering/statistical flagging step yields a candidate cell
  or feature set and a published reference set (count or labeled IDs) exists for the
  same dataset to verify against.
when_to_use_negative:
- No published reference set or labeled ground truth exists for the dataset — there
  is nothing to compare the flagged set against
- The goal is pseudotime trajectory reconstruction itself (use the redpath cHMT pipeline)
  rather than validating a flagged cell set
- Input data lacks cell-cycle scores (e.g. no reCAT output), so no G0-like flagging
  can be produced for comparison
tools:
- name: redPATH
  role: Provides kmeans_clustering (k = 5), statistical_test (threshold = 0.001),
    distribution_plot, and mean_score_plot functions used to flag the G0-like cell
    set that is then compared against the reference
  repo: https://github.com/tinglabs/redPATH
- name: reCAT
  role: Source of the mean cell-cycle scores (mean_scores) on which the G0-like flagging
    and subsequent overlap validation are based
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:e91f70b8255798fe1a62e34266f659d8b01b7885498ee6a73d33a351e09a5f6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - reCAT
  - redPATH
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Reference label overlap comparison

## Summary

Validates a newly derived cell set (e.g. redPATH's G0-like cells flagged via k-means on reCAT mean cell-cycle scores and pairwise ANOVA at P < 0.001) by computing its overlap against a published reference count or label list (e.g. the 89 G0-like cells removed from the 145-cell Llorens-Bobadilla neural stem cell dataset).

## When to use

Use when a clustering/statistical flagging step yields a candidate cell or feature set and a published reference set (count or labeled IDs) exists for the same dataset to verify against.

## When NOT to use

- No published reference set or labeled ground truth exists for the dataset — there is nothing to compare the flagged set against
- The goal is pseudotime trajectory reconstruction itself (use the redpath cHMT pipeline) rather than validating a flagged cell set
- Input data lacks cell-cycle scores (e.g. no reCAT output), so no G0-like flagging can be produced for comparison

## Inputs

- reCAT-derived mean cell-cycle scores matrix for the dataset (e.g. llorens_reCAT_scores$mean_scores, 145 cells)
- k-means cluster assignments from redPATH kmeans_clustering (k = 5)
- Flagged G0-like cell identifier list from redPATH statistical_test (threshold = 0.001)
- Published reference label list or count (89 G0-like cells reported for the Llorens-Bobadilla NSC dataset)

## Outputs

- Overlap summary between flagged and reference cell sets (intersection size / concordance metrics)
- Cluster assignments per cell with flagged G0-like cell identifiers (e.g. g0_like_cells.csv)
- Optional diagnostic plots from distribution_plot and mean_score_plot

## How to apply

Reproduce the discovery procedure first: load the reCAT-derived mean cell-cycle scores (G1, S, G1/S, G2, M, G2/M) for the dataset — e.g. llorens_reCAT_scores for the 145-cell Llorens-Bobadilla neural stem cell data — then run redPATH's kmeans_clustering with k = 5 on the mean_scores matrix and statistical_test with threshold = 0.001, which flags the cluster significantly lowest across all six phase scores as G0-like. Extract the identifiers of cells in the flagged cluster (README shows the flagged group index, e.g. 'Possible group of G0-like cells is: 2'). Next, obtain the reference set — the 89 G0-like cells reported removed from the Llorens dataset — and compute set overlap (intersection size, and optionally precision/recall or Jaccard index) between the flagged IDs and the reference IDs. Finally, write the cluster assignments, flagged G0-like cell identifiers, and overlap summary (e.g. to g0_like_cells.csv) so the validation is auditable.

## Related tools

- **redPATH** (Provides kmeans_clustering (k = 5), statistical_test (threshold = 0.001), distribution_plot, and mean_score_plot functions used to flag the G0-like cell set that is then compared against the reference) — https://github.com/tinglabs/redPATH
- **reCAT** (Source of the mean cell-cycle scores (mean_scores) on which the G0-like flagging and subsequent overlap validation are based)

## Examples

```
kmeans_clusters <- kmeans_clustering(sample_mean_score$mean_scores, k = 5); test_for_g0 <- statistical_test(sample_mean_score$mean_scores, kmeans_clusters, threshold = 0.001)
```

## Evaluation signals

- The flagged cluster index reported by statistical_test matches the documented example (e.g. 'Possible group of G0-like cells is: 2')
- The size of the flagged G0-like set is consistent with the reference count of 89 cells removed from the 145-cell Llorens dataset
- Overlap between flagged identifiers and the published reference set is computed and reported (intersection size should approach the reference count for a correct reproduction)
- distribution_plot and mean_score_plot show the flagged group with the lowest scores across all six phases (G1, S, G1/S, G2, M, G2/M)
- Output file (e.g. g0_like_cells.csv) contains per-cell cluster assignments, flagged G0-like IDs, and the comparison summary

## Limitations

- The validation only checks set overlap against a reference count/label list; redPATH's authors note that cell type labeling is not required by the approach, so discrepancy resolution is limited without additional annotation
- This procedure removed 89 cells specifically for the Llorens NSC dataset; overlap magnitude for other datasets is unvalidated

## Evidence

- [readme] G0 clustering / statistical test: "test_for_g0 <- statistical_test(sample_mean_score$mean_scores, kmeans_clusters, threshold = 0.001)"
- [readme] README example output of the flagged G0-like group: "# Example output:
# [1] "Possible group of G0-like cells is: 2""
- [readme] Optional G0 identification function availability: "An optional function is also available to identify the G0-like cells:"
- [other] redPATH G0-like detection procedure and 89-cell reference for the Llorens dataset: [evidence span withheld — anchor status: not_found]
