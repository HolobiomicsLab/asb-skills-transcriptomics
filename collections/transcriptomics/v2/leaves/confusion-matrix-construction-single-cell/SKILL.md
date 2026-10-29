---
name: confusion-matrix-construction-single-cell
description: Use when you have hybrid clustering assignments from ClonoCluster (over
  alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1) plus a barcode table, and need
  to quantify how well clusters recapitulate clonal origin.
when_to_use_negative:
- Your goal is transcriptome-only cluster validation with no barcode/clonal-identity
  reference available to cross-tabulate against
- Clustering assignments were produced at a single alpha value only, so no kappa-versus-alpha
  transition can be characterized
- You need marker fidelity or marker AUC analysis (use Find_Markers_ROC / Plot_alluvia_track
  instead of cast_confusion)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3630
tools:
- name: ClonoCluster
  role: Provides the clonocluster clustering call that generates the hybrid cluster
    assignments and the cast_confusion function used to cross-tabulate barcode versus
    cluster and compute concordance
  repo: https://github.com/leeprichman/ClonoCluster
- name: R (version >= 3.6)
  role: Runtime environment for the concordance analysis; devtools::install_github("leeprichman/ClonoCluster")
- name: data.table
  role: CRAN dependency of the ClonoCluster_paper workflow used to load and manipulate
    the assignment and barcode tables
  repo: https://github.com/arjunrajlaboratory/ClonoCluster_paper
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:5c62e428fa883455123cdcdf1868fe395a80ca7d82adde7131c236dd3d06e8ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - R (version >= 3.6)
  - R (version >=)
  - data.table
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3630
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# confusion-matrix-construction-single-cell

## Summary

Cross-tabulates single-cell hybrid cluster assignments against clone barcode membership across a stepwise alpha grid, then computes Cohen's kappa from each confusion table to quantify clone-to-cluster concordance as a function of the transcriptome-vs-clone weighting parameter.

## When to use

Use when you have hybrid clustering assignments from ClonoCluster (over alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1) plus a barcode table, and need to quantify how well clusters recapitulate clonal origin.

## When NOT to use

- Your goal is transcriptome-only cluster validation with no barcode/clonal-identity reference available to cross-tabulate against
- Clustering assignments were produced at a single alpha value only, so no kappa-versus-alpha transition can be characterized
- You need marker fidelity or marker AUC analysis (use Find_Markers_ROC / Plot_alluvia_track instead of cast_confusion)

## Inputs

- Hybrid cluster assignment table per dataset (e.g. *_cluster_assignments.txt) covering alpha values 0 to 1 by 0.1, from clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)
- Two-column barcode assignment table with columns 'rn' (cell ID) and 'Barcode'

## Outputs

- Per-dataset table of Cohen's kappa versus alpha (11 alpha values, 0 to 1 by 0.1), written as CSV
- Kappa-versus-alpha plots (Confusion_stats.R output, as in Figure S2B)

## How to apply

For each dataset, load the hybrid cluster assignment table produced by clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1) together with the two-column barcode table (columns 'rn' and 'Barcode'). At each of the 11 alpha values, cross-tabulate cluster assignment versus clone barcode membership using the ClonoCluster package's cast_confusion function. From each confusion table compute Cohen's kappa, the chance-corrected agreement statistic between hybrid clusters and clone identity, which should approach 1 as alpha approaches 1 (purely clone weights) and be near 0 at alpha = 0 (purely transcriptome weights). Assemble kappa versus alpha into a per-dataset table and write it out as CSV, then plot kappa versus alpha (as in Confusion_stats.R, Figure S2B) to inspect the transition: most datasets show a graded rise, while the cardiomyocyte-directed iPSC dataset (CJ) shows an abrupt switch from low to high kappa, indicating few meaningful intermediate hybrid clusters.

## Related tools

- **ClonoCluster** (Provides the clonocluster clustering call that generates the hybrid cluster assignments and the cast_confusion function used to cross-tabulate barcode versus cluster and compute concordance) — https://github.com/leeprichman/ClonoCluster
- **R (version >= 3.6)** (Runtime environment for the concordance analysis; devtools::install_github("leeprichman/ClonoCluster"))
- **data.table** (CRAN dependency of the ClonoCluster_paper workflow used to load and manipulate the assignment and barcode tables) — https://github.com/arjunrajlaboratory/ClonoCluster_paper

## Examples

```
clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)
```

## Evaluation signals

- Each per-dataset table contains exactly 11 alpha values (0 to 1 by 0.1)
- Kappa approaches perfect agreement (kappa ~ 1) as alpha approaches 1 (purely clone weights)
- Kappa is near chance-corrected disagreement (~ 0) at alpha = 0 (purely transcriptome weights)
- Most datasets show a gradual, monotonic kappa rise across alpha, while the CJ (cardiomyocyte-directed iPSC) dataset shows an abrupt low-to-high kappa switch
- Confusion tables are cross-tabulated on cell IDs ('rn'), so row counts match the barcode table exactly

## Limitations

- No reliable method was found to identify which experimental systems have intrinsic determinants of cell fate, so a sudden versus graded kappa transition must be interpreted per dataset (e.g. CJ's fate is largely determined by extrinsic factors)
- High kappa at intermediate alpha does not by itself establish that hybrid clusters are biologically meaningful; marker fidelity analysis or ground-truth comparison is recommended to confirm the chosen alpha

## Evidence

- [methods] cast_confusion: "Barcode to cluster concordance plots with `cast_confusion`"
- [methods] clonocluster call: "clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)"
- [intro] alpha weighting definition: "We linearly combined transcriptome and clone weights using α such that it returned purely transcriptome weights for α = 0 and purely clone weights for α = 1"
- [readme] kappa vs alpha plots: [evidence span withheld — anchor status: external_doc]
- [discussion] CJ dataset abrupt transition: "in the case of cardiomyocyte-directed iPSCs from Jiang et al.,18 in which we know that cell fate is largely determined by extrinsic factors"
