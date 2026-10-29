---
name: cohen-kappa-agreement-statistics
description: Use when you need to measure how well clustering assignments (e.g., hybrid
  ClonoCluster clusters at stepwise α values) agree with a known categorical labeling
  such as clone barcode membership, especially to characterize transition sharpness
  across a parameter sweep.
when_to_use_negative:
- When comparing two clusterings that do not share a one-to-one label semantics with
  clone barcodes (kappa requires matched categorical levels; use adjusted Rand index
  or mutual information instead)
- When the goal is marker identification rather than clustering concordance — use
  Find_Markers_ROC or reorganization marker AUC analysis
- When clone barcode data are absent or unreliable, since kappa against barcode membership
  is undefined or meaningless
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3630
tools:
- name: ClonoCluster
  role: Generates hybrid cluster assignments across the alpha sweep whose concordance
    with clone barcodes is scored by Cohen's kappa
  repo: https://github.com/leeprichman/ClonoCluster
- name: R (version >= 3.6)
  role: Analysis environment in which cast_confusion confusion tables and kappa statistics
    are computed and exported
  repo: https://github.com/leeprichman/ClonoCluster
- name: data.table
  role: CRAN dependency used in the ClonoCluster_paper workflow for handling barcode
    and cluster assignment tables
  repo: https://github.com/arjunrajlaboratory/ClonoCluster_paper
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:ba4948a0a60d74c43303b20d72d5575e0cc7811a8c0498ee66769b53db934ce8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - R (version >= 3.6)
  - R (version >=)
  - data.table
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3630
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cohen-kappa-agreement-statistics

## Summary

Quantifies chance-corrected agreement between hybrid cluster assignments (from ClonoCluster across α = 0 to 1) and clone barcode membership using Cohen's κ, revealing how gradually or abruptly clones conglomerate into hybrid clusters in each single-cell dataset.

## When to use

Use when you need to measure how well clustering assignments (e.g., hybrid ClonoCluster clusters at stepwise α values) agree with a known categorical labeling such as clone barcode membership, especially to characterize transition sharpness across a parameter sweep.

## When NOT to use

- When comparing two clusterings that do not share a one-to-one label semantics with clone barcodes (kappa requires matched categorical levels; use adjusted Rand index or mutual information instead)
- When the goal is marker identification rather than clustering concordance — use Find_Markers_ROC or reorganization marker AUC analysis
- When clone barcode data are absent or unreliable, since kappa against barcode membership is undefined or meaningless

## Inputs

- Hybrid cluster assignment table from clonocluster run over alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1
- Two-column barcode assignment table with columns 'rn' (cell ID) and 'Barcode' (e.g., from Data_barcodes/)

## Outputs

- Per-dataset table of Cohen's kappa versus alpha (11 alpha values, 0 to 1 by 0.1), written as CSV
- Confusion (cross-tabulation) tables of cluster assignment vs clone barcode at each alpha
- Kappa vs alpha plots (as in Confusion_stats.R / Figure S2B boxplots)

## How to apply

For each dataset, run clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1) on the PCA embedding (25 PCs via irlba_wrap) and the two-column barcode table ('rn', 'Barcode'). At each of the 11 alpha values, cross-tabulate hybrid cluster assignment versus clone barcode identity with the cast_confusion concordance function, then compute Cohen's kappa from each confusion table as the chance-corrected agreement statistic. Assemble kappa versus alpha into a per-dataset table and write it as CSV. Interpretation: κ approaches perfect agreement (κ ≈ 1) as α → 1 (purely clone weights) and drops toward 0 at α = 0 (purely transcriptome weights); most datasets show a gradual rise, so a sudden low-to-high κ jump indicates few meaningful intermediate hybrid clusters, as observed for the cardiomyocyte-directed iPSC (CJ) dataset.

## Related tools

- **ClonoCluster** (Generates hybrid cluster assignments across the alpha sweep whose concordance with clone barcodes is scored by Cohen's kappa) — https://github.com/leeprichman/ClonoCluster
- **R (version >= 3.6)** (Analysis environment in which cast_confusion confusion tables and kappa statistics are computed and exported) — https://github.com/leeprichman/ClonoCluster
- **data.table** (CRAN dependency used in the ClonoCluster_paper workflow for handling barcode and cluster assignment tables) — https://github.com/arjunrajlaboratory/ClonoCluster_paper

## Examples

```
clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)
```

## Evaluation signals

- Each per-dataset table contains exactly 11 alpha values (0 to 1 by 0.1)
- Kappa approaches perfect agreement (κ ≈ 1) as alpha approaches 1 (purely clone weights)
- Kappa is near 0 at alpha = 0 (purely transcriptome weights), since clusters are uncorrelated with clone identity
- Most datasets show a gradual monotonic rise in kappa across alpha; the CJ (cardiomyocyte-directed iPSC) dataset should show an abrupt low-to-high jump
- Each confusion table dimensions match the number of hybrid clusters × number of unique barcodes at that alpha

## Limitations

- A sudden kappa transition (as in the cardiomyocyte-directed iPSC dataset) indicates few meaningful intermediate hybrid clusters, limiting fine-grained alpha tuning for such systems
- The study did not reveal a reliable method to identify which experimental systems have intrinsic determinants of cell fate, so kappa trajectories must be assessed case by case
- Full-lineage tracing data (CRISPR-Cas9 or imaging-based family trees) were not addressed; kappa against flat barcode membership does not capture phylogenetic distance

## Evidence

- [methods] clonocluster over alpha sweep: "clust <- clonocluster(pca, bt, alpha = seq(0, 1, by = 0.1), beta = 0.1, res = 1)"
- [methods] cast_confusion concordance: "* Barcode to cluster concordance plots with `cast_confusion`"
- [intro] alpha = 1 purely clone weights: "We linearly combined transcriptome and clone weights using α such that it returned purely transcriptome weights for α = 0 and purely clone weights for α = 1"
- [readme] kappa vs alpha figure script: [evidence span withheld — anchor status: external_doc]
- [readme] raw data availability for six datasets: [evidence span withheld — anchor status: external_doc]
