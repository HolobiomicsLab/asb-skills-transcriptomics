---
name: cell-to-cell-correlation-analysis
description: Use when you have SClineager genotype-inference outputs (inferred VAF
  matrix and covariance from run_sclineager) and need to partition cells into lineages
  based on similarity of inferred genotypes.
when_to_use_negative:
- The input is raw VAF data before SClineager's Bayesian imputation — expressional
  drop-outs (low coverage, allelic bias) make raw correlations unreliable; run genotype
  inference first.
- You need mutation/variant calling rather than lineage assignment — use the QBRC
  Somatic Pipeline on scSplitter-split single-cell files instead.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: SClineager
  role: Upstream Bayesian MCMC inference of per-cell genotypes/VAFs whose imputed
    matrix feeds the correlation and clustering
  repo: https://github.com/inmybrain/SClineager
- name: R
  role: Environment (R 4.0.3 in the paper) for computing correlations, distances,
    hclust, and cutree
  repo: https://github.com/inmybrain/SClineager
- name: gplots
  role: heatmap.2 visualization of raw and fitted VAF / cell-correlation clustering
    structure (gplots 3.0.3)
  repo: https://github.com/inmybrain/SClineager
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.celrep.2020.108589
    title: Overcoming Expressional Drop-outs in Lineage Reconstruction from Single-Cell
      RNA-Sequencing Data
  source_content_hash: sha256:13f2bae60e91dc04cb24e262ff8c876fadaf87205a5d17e578ec93f3225083ea
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
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# cell-to-cell-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns single cells to evolutionary lineages by computing a cell-to-cell Pearson correlation matrix over SClineager's imputed/inferred VAF genotypes, converting correlations to distances, and detecting lineages via hierarchical clustering. This exploits SClineager's Bayesian MCMC imputation, which mitigates expressional drop-outs before clustering.

## When to use

Use when you have SClineager genotype-inference outputs (inferred VAF matrix and covariance from run_sclineager) and need to partition cells into lineages based on similarity of inferred genotypes.

## When NOT to use

- The input is raw VAF data before SClineager's Bayesian imputation — expressional drop-outs (low coverage, allelic bias) make raw correlations unreliable; run genotype inference first.
- You need mutation/variant calling rather than lineage assignment — use the QBRC Somatic Pipeline on scSplitter-split single-cell files instead.

## Inputs

- results.RData from SClineager run_sclineager containing inferred VAF matrix genotype_mat and inferred covariance matrix sigma
- Number of lineages (k) or dendrogram cut height for cutree

## Outputs

- cell_lineage_assignments.csv: one row per cell with a non-null lineage label
- PNG heatmap of the cell-to-cell correlation/clustering structure (gplots::heatmap.2)

## How to apply

Load the results.RData from the upstream run_sclineager task and extract the inferred VAF matrix (genotype_mat) and the inferred covariance matrix (sigma). Compute the Pearson correlation between cells across the variant columns of the imputed VAF matrix — imputation by SClineager's Bayesian MCMC is what mitigates the expressional drop-outs (low coverage and allelic bias) that would otherwise distort correlations. Convert the correlation matrix to a distance with 1 - correlation and run R's stats::hclust on it, then cut the dendrogram (e.g. cutree) to assign one lineage label per cell. Write the assignments to cell_lineage_assignments.csv and visualize the correlation/clustering structure as a heatmap with gplots::heatmap.2 (gplots 3.0.3 was used in the paper's configuration with R 4.0.3), exported as PNG. The rationale, per the paper, is that borrowing information from related cells yields accurate evolutionary lineages even where single-cell variant calling alone fails.

## Related tools

- **SClineager** (Upstream Bayesian MCMC inference of per-cell genotypes/VAFs whose imputed matrix feeds the correlation and clustering) — https://github.com/inmybrain/SClineager
- **R** (Environment (R 4.0.3 in the paper) for computing correlations, distances, hclust, and cutree) — https://github.com/inmybrain/SClineager
- **gplots** (heatmap.2 visualization of raw and fitted VAF / cell-correlation clustering structure (gplots 3.0.3)) — https://github.com/inmybrain/SClineager

## Examples

```
cor_mat <- cor(t(genotype_mat)); hc <- hclust(as.dist(1 - cor_mat)); lineage <- cutree(hc, k = n_lineages); write.csv(data.frame(cell = rownames(genotype_mat), lineage), "cell_lineage_assignments.csv", row.names = FALSE)
```

## Evaluation signals

- cell_lineage_assignments.csv contains exactly one row and one non-null lineage label per cell in genotype_mat
- Number of assigned cells equals the number of rows of the inferred VAF matrix
- Every lineage label corresponds to a valid cluster produced by cutting the hclust dendrogram (cluster IDs within 1..k)
- Distance matrix values fall in [0, 2] since distance = 1 - Pearson correlation
- heatmap.2 output renders and orders rows/columns consistent with the dendrogram from the correlation-based clustering

## Limitations

- Requires SClineager inference to have been run per sample independently; results from different samples cannot be pooled directly.
- Quality depends on upstream imputation: low coverage on many positions and allelic bias are the inherent issues SClineager corrects; if coverage/filter thresholds (e.g. coverage_cutoff) were too permissive, correlations will be noise-dominated.
- The number of lineages (dendrogram cut) must be chosen by the analyst; the method does not by itself determine k.

## Evidence

- [abstract] SClineager infers lineages from correlation clustering: "We introduce SClineager, which infers accurate evolutionary lineages from scRNA-seq data by borrowing information from related cells to overcome expressional drop-outs."
- [readme] Bayesian model takes drop-out issues into account: "This algorithm infers genetic trajectories of cells by taking these two issues into account."
- [methods] Heatmap generation for VAFs: "The following code block is to generate heatmaps for raw and fitted VAFs."
- [readme] Software configuration used: "*gplots* ver 3.0.3 are used."
