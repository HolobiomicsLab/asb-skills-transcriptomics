---
name: pca-latent-factor-detection
description: Use when a scRNA-Seq expression matrix shows within-cell-type heterogeneity
  suspected to be a cell-cycle effect, and you need to test which principal components
  are cell-cycle driven before removing them while preserving cell-type signal.
when_to_use_negative:
- When the dominant unwanted confounder is not the cell cycle — ccRemover is designed
  for removing the cell-cycle effect exclusively and does not remove other confounding
  effects that may dominate after removal
- When the cell-cycle effect influences very few control genes (below roughly 8% affected),
  since it may not be captured by the principal components of the control-gene matrix
  and will be missed
- When no annotated cell-cycle gene set exists or the annotation is too unreliable,
  since the method relies on a known set of cell-cycle genes from annotation databases
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ccRemover
  role: Implements the PCA-on-control-genes plus bootstrap t-test detection and removal
    of cell-cycle components; called on the if_cc-annotated matrix with cutoff = 3,
    max_it = 4, nboot = 200, ntop = 10
  repo: cran/ccRemover
- name: gene_indexer
  role: Maps gene names to cell-cycle annotations by species (mouse/human) to build
    the if_cc indicator used to split genes into sets A and B
- name: scLVM
  role: 'Comparison baseline: applied via its Python script with K = 1 (default) and
    scree-plot-derived K = 3 to demonstrate removal of the wrong effect on simulated
    data'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:2f830c0c9cdf8fc0e98e158d44a295b65b5b204b69082cdf4ac7c81a218cfcd4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - Python
  - ccRemover
  - gene_indexer
  - scLVM
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pca-latent-factor-detection

## Summary

Detects cell-cycle latent factors in scRNA-Seq data by running PCA on control genes and applying a two-class bootstrap t-test to component loadings, flagging directions whose loading difference between cell-cycle and control gene sets exceeds a cutoff. It is the core detection step inside ccRemover, distinguishing true cell-cycle components from other biological signals such as cell-type effects.

## When to use

Use when a scRNA-Seq expression matrix shows within-cell-type heterogeneity suspected to be a cell-cycle effect, and you need to test which leading principal components are cell-cycle driven before removing them while preserving cell-type signal.

## When NOT to use

- When the dominant unwanted confounder is not the cell cycle — ccRemover is designed for removing the cell-cycle effect exclusively and does not remove other confounding effects that may dominate after removal
- When the cell-cycle effect influences very few control genes (below roughly 8% affected), since it may not be captured by the principal components of the control-gene matrix and will be missed
- When no annotated cell-cycle gene set exists or the annotation is too unreliable, since the method relies on a known set of cell-cycle genes from annotation databases

## Inputs

- Log-transformed, gene-by-gene centered scRNA-Seq expression matrix (cells x genes)
- if_cc indicator vector annotating which genes are cell-cycle genes (e.g. from org.Mm.eg.db / org.Hs.eg.db or Cyclebase)
- Control-gene matrix XB (all genes not annotated as cell-cycle genes)
- ccRemover parameters: cutoff C_delta = 3, max_it = 4, nboot = 100–200, ntop = 10

## Outputs

- Corrected expression matrix (e.g. xhat) with projections onto flagged cell-cycle directions subtracted
- Set of principal components flagged as cell-cycle effects (e.g. the 2nd and 3rd PCs on the 50-cell, 2,000-gene simulation)
- Per-iteration bootstrap t-statistics on the loading difference Delta_j for tested components

## How to apply

First log-transform the expression data and center it on a gene-by-gene basis, then split genes into a cell-cycle set A (via the if_cc indicator, e.g. from gene_indexer using mouse or human annotation) and a control set B (all genes not annotated to the cell cycle). Perform a PCA on the control-gene matrix XB and test the top components (ntop = 10) using a two-class bootstrap t-test on the loading difference Delta_j: take random samples with replacement of |A| rows from XA and |B| rows from XB to build bootstrap replicates of Delta_j. Flag components whose bootstrap t-statistic exceeds the cutoff C_delta = 3 (use C_delta = 2 for data with very low cell-cycle activity such as the glioblastoma data) and subtract the projections of the data onto those flagged directions. Repeat for at most max_it = 4 iterations (nboot = 100–200 bootstrap replicates) until no further components are flagged; the rationale is that a formal statistical test on loadings prevents removing biological components of interest, unlike scLVM which does not inspect whether a leading factor represents the cell cycle.

## Related tools

- **ccRemover** (Implements the PCA-on-control-genes plus bootstrap t-test detection and removal of cell-cycle components; called on the if_cc-annotated matrix with cutoff = 3, max_it = 4, nboot = 200, ntop = 10) — cran/ccRemover
- **gene_indexer** (Maps gene names to cell-cycle annotations by species (mouse/human) to build the if_cc indicator used to split genes into sets A and B)
- **scLVM** (Comparison baseline: applied via its Python script with K = 1 (default) and scree-plot-derived K = 3 to demonstrate removal of the wrong effect on simulated data)

## Examples

```
xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)
```

## Evaluation signals

- On the 50-cell, 2,000-gene (400 cell-cycle genes) simulation, the detector must flag the 2nd and 3rd principal components — not the 1st, which carries the cell-type effect
- After correction, cells should cluster by cell type with cell-cycle stages mixed within clusters (six original cell-type-by-cell-cycle-stage groups collapse to cell-type separation)
- Cross-check against scLVM: scLVM with K = 1 mistakenly removes the cell-type effect and with K = 3 removes both effects leaving noise, whereas this detection step preserves cell-type structure
- Convergence check: no further components flagged after at most 4 iterations (on the T helper data the first PC is flagged on iteration 1 and no other effects after removal)
- Bootstrap stability: with nboot = 100–200 replicates, the two-class t-statistic on Delta_j must exceed C_delta = 3 for flagged components and not for cell-type components

## Limitations

- Relies on annotation databases that are always incomplete and inaccurate, so the if_cc gene set quality bounds detection sensitivity
- If the cell-cycle effect only influences a few of the control genes, it may not be captured by the principal components and hence missed for detection (below ~8% of control genes affected)
- Removes only the cell-cycle effect, not other confounding effects that may dominate after removal
- Cutoff may need adjustment: C_delta = 2 was used for glioblastoma data where cell-cycle activity is at a very low level, instead of the default C_delta = 3

## Evidence

- [methods] The ccRemover algorithm follows these steps: 1. Perform a PCA on the data matrix of control genes XB.: "The ccRemover algorithm follows these steps: 1. Perform a PCA on the data matrix of control genes XB."
- [methods] xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE): "xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)"
- [results] ccRemover is able to correctly identify the second and third principal components as cell-cycle effects and removes them: "ccRemover is able to correctly identify the second and third principal components as cell-cycle effects and removes them"
- [discussion] if the cell-cycle effect only influences a few of the control genes, it may not be captured by the principal components and hence missed for detection: "if the cell-cycle effect only influences a few of the control genes, it may not be captured by the principal components and hence missed for detection"
