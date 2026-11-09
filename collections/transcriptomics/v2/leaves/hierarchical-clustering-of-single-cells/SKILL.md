---
name: hierarchical-clustering-of-single-cells
description: Use when you already have SClineager MCMC output (results.RData with
  inferred `genotype_mat` and `sigma`) and need to assign each cell to a discrete
  lineage cluster.
when_to_use_negative:
- Upstream SClineager genotype inference (run_sclineager) has not been run yet — this
  skill operates on its results.RData, not on raw coverage.txt or cleaned.RData inputs
- 'Cells come from different samples: run_sclineager must be performed per sample
  independently, so clustering should also be done within a sample, not pooled across
  samples'
edam_operation: http://edamontology.org/operation_3432
tools:
- name: SClineager
  role: provides the inferred VAF matrix (`genotype_mat`) whose drop-out-corrected
    values define the cell correlation matrix used for clustering
  repo: https://github.com/inmybrain/SClineager
- name: R
  role: runtime for stats::hclust/cutree hierarchical clustering and for saving the
    lineage assignments
  repo: https://github.com/inmybrain/SClineager
- name: gplots
  role: heatmap.2 visualization of the cell-to-cell correlation/clustering structure
    as a PNG
  repo: https://github.com/inmybrain/SClineager
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.celrep.2020.108589
    title: Overcoming Expressional Drop-outs in Lineage Reconstruction from Single-Cell
      RNA-Sequencing Data
  source_content_hash: sha256:cd3e2a15dee11f6d04f8ff349da1c2c03785b1edb51c14667dc0659529d42fca
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/inmybrain/SClineager
  repo_url: https://github.com/inmybrain/SClineager
  tools:
  - R
  - SClineager
  - gplots
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# hierarchical-clustering-of-single-cells

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns single cells to lineages by hierarchical clustering of the cell-to-cell correlation matrix computed from SClineager's inferred VAFs (results.RData's `genotype_mat`), converting correlation to distance as 1 - correlation. This post-processing step turns SClineager's drop-out-corrected genotype estimates into explicit per-cell lineage labels.

## When to use

Use when you already have SClineager MCMC output (results.RData with inferred `genotype_mat` and `sigma`) and need to assign each cell to a discrete lineage cluster.

## When NOT to use

- Upstream SClineager genotype inference (run_sclineager) has not been run yet — this skill operates on its results.RData, not on raw coverage.txt or cleaned.RData inputs
- Cells come from different samples: run_sclineager must be performed per sample independently, so clustering should also be done within a sample, not pooled across samples

## Inputs

- results.RData from run_sclineager containing the R object `results` with inferred VAF matrix `genotype_mat` and inferred covariance matrix `sigma`
- cluster count / dendrogram cut height for cutree

## Outputs

- cell_lineage_assignments.csv: one row per cell with a non-null lineage label
- PNG heatmap of the cell-to-cell correlation/clustering structure produced with gplots::heatmap.2

## How to apply

Load the results.RData produced by run_sclineager and extract `genotype_mat` (the inferred VAF matrix) and `sigma` (the inferred covariance matrix). Compute the cell-to-cell Pearson correlation across the variant columns of the inferred VAFs — SClineager's Bayesian MCMC imputation mitigates expressional drop-outs, making the correlation matrix more informative than one from the raw VAF matrix (`genotype_mat_orig`). Convert correlation to a distance with 1 - correlation and run hierarchical clustering with stats::hclust, then cut the dendrogram (stats::cutree) to obtain one lineage label per cell. Visualize the correlation/clustering structure with gplots::heatmap.2, saving the result as a PNG. Validate that the assignment table has exactly one non-null lineage label per cell, with the cell count matching nrow(genotype_mat).

## Related tools

- **SClineager** (provides the inferred VAF matrix (`genotype_mat`) whose drop-out-corrected values define the cell correlation matrix used for clustering) — https://github.com/inmybrain/SClineager
- **R** (runtime for stats::hclust/cutree hierarchical clustering and for saving the lineage assignments) — https://github.com/inmybrain/SClineager
- **gplots** (heatmap.2 visualization of the cell-to-cell correlation/clustering structure as a PNG) — https://github.com/inmybrain/SClineager

## Examples

```
hc <- hclust(as.dist(1 - cor(t(results$genotype_mat)))); lab <- cutree(hc, k = n_lineages); gplots::heatmap.2(cor(t(results$genotype_mat)))
```

## Evaluation signals

- cell_lineage_assignments.csv contains exactly one row per cell and one non-null lineage label per cell
- number of cells in the assignment table equals nrow(results$genotype_mat)
- distance matrix is well-defined (all pairwise 1 - correlation values in [0, 2], diagonal 0)
- heatmap PNG renders the correlation/clustering structure and matches the cell ordering in the assignments
- cells grouped by the cut dendrogram show consistent (high-correlation) inferred VAF profiles within each lineage

## Limitations

- Lineage assignments inherit drop-out issues from the raw data: low coverage on many positions in many cells and allelic bias degrade correlations if raw VAFs are used instead of SClineager's imputed `genotype_mat`
- The number of lineages depends on where the dendrogram is cut, which is a user decision not fixed by the paper

## Evidence

- [abstract] SClineager infers lineages from drop-out-corrected inferred genotypes via correlation-based clustering: "We introduce SClineager, which infers accurate evolutionary lineages from scRNA-seq data by borrowing information from related cells to overcome expressional drop-outs."
- [readme] results.RData contains the inferred VAF matrix and covariance matrix that feed the clustering: "contains an R object named `results` with the following attributes: - `genotype_mat` : inferred VAF matrix - `genotype_mat_orig` : the raw VAF matrix - `sigma` : inferred covariance matrix"
- [methods] run_sclineager MCMC output is the per-sample upstream input to this clustering skill: "The main function `run_sclineager` performs the MCMC sampling and returns estimation results. The analysis has to be performed for each sample independently."
- [readme] gplots is the visualization dependency used for heatmap rendering: "To install *SClineager*, you will need the following R packages (and their dependencies): *Rcpp* (\>= 1.0.2), *MCMCpack*, *vioplot*, *gplots*."
