---
name: posterior-probability-uncertainty-quantification
description: Use when BAMM-SC (or DIMM-SC) clustering of droplet-based scRNA-seq data
  must report which individual cells have uncertain cluster-type assignments, e.g.
  per-cell posterior probabilities rather than only hard labels.
when_to_use_negative:
- When the posterior probabilities are expected to quantify continuous properties
  such as cell cycle state or developmental stage — the paper states each cell is
  assumed to belong to one distinct cluster and the posterior measures only clustering
  uncertainty
- For single-individual analysis where DIMMSC is called directly — still valid, but
  the multi-individual delta list structure (one matrix per individual) does not apply
- When a hard cluster label alone suffices and per-cell uncertainty is not required
  — running the posterior extraction adds no value over reading $mem
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BAMM-SC
  role: Bayesian hierarchical Dirichlet-multinomial mixture model whose MCMC output
    provides the per-cell posterior cluster-probability matrices (delta) used to flag
    vague cells
  repo: github.com/lichen-lab/BAMMSC
- name: t-SNE
  role: 2-D projection (computed on the first 50 principal components in the paper)
    used to overlay vague cells and verify they lie at cluster boundaries
- name: 10x Genomics Chromium system
  role: Droplet-based platform that generated the human PBMC UMI count matrices the
    uncertainty assessment is applied to
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:9ecfd0d0d8941dd0a535dde1d6d04a7d4defbe3986bd4850e0b84ca94fb7f2e6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Genomics Chromium system
  - BAMM-SC
  - t-SNE
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# posterior-probability-uncertainty-quantification

## Summary

Quantifies per-cell clustering uncertainty in BAMM-SC by extracting the posterior cluster-membership probabilities (delta output) and flagging 'vague' cells whose maximum posterior probability falls below 0.95, then localizing them on a t-SNE projection to verify they sit at cluster boundaries.

## When to use

Use when BAMM-SC (or DIMM-SC) clustering of droplet-based scRNA-seq data must report which individual cells have uncertain cluster-type assignments, e.g. per-cell posterior probabilities rather than only hard labels.

## When NOT to use

- When the posterior probabilities are expected to quantify continuous properties such as cell cycle state or developmental stage — the paper states each cell is assumed to belong to one distinct cluster and the posterior measures only clustering uncertainty
- For single-individual analysis where DIMMSC is called directly — still valid, but the multi-individual delta list structure (one matrix per individual) does not apply
- When a hard cluster label alone suffices and per-cell uncertainty is not required — running the posterior extraction adds no value over reading $mem

## Inputs

- BAMMSC result object with $delta (list of per-individual [#cell, #cluster, #iteration] posterior probability matrices) and $mem (list of per-cell cluster memberships)
- Human PBMC UMI count matrices for multiple donors (list of gene-by-cell count matrices, same gene set across individuals)
- Optionally the first 50 principal components of the cells for t-SNE projection

## Outputs

- Per-cell table (e.g. pbmc_vague_cells.csv) with cluster label, maximum posterior probability, and vague flag (<0.95)
- Annotated t-SNE projection (e.g. pbmc_vague_cells_tsne.png) showing vague cells localized at cluster boundaries
- Fraction of all cells flagged as vague

## How to apply

After running the Bayesian hierarchical Dirichlet-multinomial mixture (BAMMSC(X, K) with defaults nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10), extract the delta output — a list of per-individual 3-D posterior probability matrices [#cell, #cluster, #iteration] — and take the final-iteration maximum posterior probability for every cell, pairing it with the cluster membership from mem. Flag cells as 'vague' when the largest cluster-specific posterior probability is <0.95 (the paper's threshold used in the human PBMC analysis) and record the fraction of cells flagged, which should be a small subset. Project cells with t-SNE (the paper computed t-SNE on the first 50 principal components) and overlay the vague cells to confirm they map to cluster boundaries rather than cluster cores, supporting the validity of the clustering. Rationale: the posterior probability directly measures clustering uncertainty because BAMM-SC assumes each cell belongs to one distinct cluster.

## Related tools

- **BAMM-SC** (Bayesian hierarchical Dirichlet-multinomial mixture model whose MCMC output provides the per-cell posterior cluster-probability matrices (delta) used to flag vague cells) — github.com/lichen-lab/BAMMSC
- **t-SNE** (2-D projection (computed on the first 50 principal components in the paper) used to overlay vague cells and verify they lie at cluster boundaries)
- **10x Genomics Chromium system** (Droplet-based platform that generated the human PBMC UMI count matrices the uncertainty assessment is applied to)

## Examples

```
library(BAMMSC); result <- BAMMSC(data_BAMMSC, K=4); maxpost <- apply(result$delta[[1]][, , dim(result$delta[[1]])[3]], 1, max); vague <- maxpost < 0.95
```

## Evaluation signals

- The per-cell CSV contains a maximum-posterior column and a vague flag for every cell in every individual
- The fraction of cells flagged vague (max posterior <0.95) is small relative to the total cell count
- The t-SNE overlay shows vague cells concentrated at cluster boundaries rather than cluster cores, as reported in the paper's PBMC uncertainty assessment
- The extracted posterior values come from the final MCMC iteration of delta and are consistent with cluster memberships in mem
- Log-likelihood (loglik output) has converged per likTol.BAMMSC=1e-10 before posterior probabilities are interpreted

## Limitations

- Posterior probabilities cannot be interpreted as cell cycle or developmental stage quantification; they only measure clustering uncertainty because each cell is assumed to belong to one distinct cluster
- The 0.95 vague-cell threshold is a heuristic; results may vary with the choice of threshold and with the number of clusters K
- Filtering genes with excessive zeros prior to clustering may be problematic for rare cell type identification, which could bias which cells appear vague
- Zero-inflation is not explicitly modeled by BAMM-SC, which may or may not affect clustering accuracy and hence posterior confidence

## Evidence

- [readme] delta output: "delta: a list of 3-D matrix. Each matrix [#cell, #cluster, #iteration] is the poster probability estimates for each individual across all iterations."
- [discussion] posterior as clustering uncertainty: "we assume that each cell belongs to one distinct cluster. The posterior probability measures the clustering uncertainty"
- [results] vague cells at boundaries: [evidence span withheld — anchor status: not_found]
- [readme] BAMMSC defaults: "nBurn: number of iterations discarded at the initial portion of the Markov chain, default is 400"
- [intro] t-SNE on PCs: "we produced a t-SNE plot based on the first 50 principal components (Supplementary Fig. 1) of all cells from these 5 donors"
