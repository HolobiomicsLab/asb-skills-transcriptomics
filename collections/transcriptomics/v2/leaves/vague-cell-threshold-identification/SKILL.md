---
name: vague-cell-threshold-identification
description: Use when a BAMM-SC clustering run on multi-individual droplet scRNA-seq
  data (e.g., human PBMC) needs per-cell uncertainty quantification or localization
  of cells with ambiguous cell-type assignments.
when_to_use_negative:
- Analysis uses a single individual — BAMMSC falls back to DIMMSC and the posterior-uncertainty
  workflow on multi-individual delta matrices does not directly apply
- The goal is interpreting posterior probabilities as continuous quantities (e.g.,
  cell cycle or developmental stage) rather than discrete cluster-assignment uncertainty
  — the authors assume each cell belongs to one distinct cluster
- Input is already-reduced embeddings or cluster labels from another tool (SIMLR,
  SC3, Seurat) that do not produce BAMM-SC posterior probability matrices
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: BAMM-SC
  role: Produces the per-cell posterior cluster-probability matrices (delta) used
    to identify vague cells
  repo: github.com/lichen-lab/BAMMSC
- name: t-SNE
  role: 2-D projection used to visualize vague cells and confirm their localization
    at cluster boundaries
- name: 10x Genomics Chromium system
  role: Source platform for the droplet-based UMI count matrices of PBMC donors analyzed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:0ab2894c918eaba8822d4a0d4bcdf10fcc009aca25ac13d628a2c2d6a98fe6d5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - 10x Genomics Chromium system
  - BAMM-SC
  - t-SNE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# vague-cell-threshold-identification

## Summary

Quantifies per-cell clustering uncertainty from BAMM-SC by extracting final-iteration posterior probabilities (delta) and flagging 'vague' cells whose maximum posterior cluster-specific probability is below 0.95, then verifying via t-SNE that such cells sit at cluster boundaries.

## When to use

Use when a BAMM-SC clustering run on multi-individual droplet scRNA-seq data (e.g., human PBMC) needs per-cell uncertainty quantification or localization of cells with ambiguous cell-type assignments.

## When NOT to use

- Analysis uses a single individual — BAMMSC falls back to DIMMSC and the posterior-uncertainty workflow on multi-individual delta matrices does not directly apply
- The goal is interpreting posterior probabilities as continuous quantities (e.g., cell cycle or developmental stage) rather than discrete cluster-assignment uncertainty — the authors assume each cell belongs to one distinct cluster
- Input is already-reduced embeddings or cluster labels from another tool (SIMLR, SC3, Seurat) that do not produce BAMM-SC posterior probability matrices

## Inputs

- List of per-individual UMI count matrices (genes x cells) from droplet-based scRNA-seq, e.g., human PBMC data from multiple donors
- BAMMSC run outputs: delta (per-individual [#cell, #cluster, #iteration] posterior probability matrices) and mem (per-individual cluster membership vectors)

## Outputs

- Per-cell CSV table with cluster label, maximum posterior probability, and vague flag (e.g., pbmc_vague_cells.csv)
- Annotated t-SNE projection highlighting vague cells at cluster boundaries (e.g., pbmc_vague_cells_tsne.png)
- Fraction of cells flagged as vague (max posterior < 0.95) relative to total cell count

## How to apply

Run the BAMMSC Bayesian hierarchical Dirichlet-multinomial mixture clustering (option="BAMMSC", nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10) on the list of per-individual UMI count matrices with K clusters, and retrieve the delta output — a per-individual 3-D posterior probability matrix [#cell, #cluster, #iteration]. Take the final-iteration maximum posterior probability for every cell along with its membership from mem. Flag cells as vague when their maximum posterior probability is below 0.95 and record the fraction of all cells they comprise, verifying it is a small subset. Compute a t-SNE projection (analogous to the paper's t-SNE on the first 50 principal components) and overlay the vague cells to confirm they map to cluster boundaries rather than cluster cores, supporting the validity of the clustering. Write the per-cell table (cluster label, max posterior, vague flag) and the annotated projection as deliverables.

## Related tools

- **BAMM-SC** (Produces the per-cell posterior cluster-probability matrices (delta) used to identify vague cells) — github.com/lichen-lab/BAMMSC
- **t-SNE** (2-D projection used to visualize vague cells and confirm their localization at cluster boundaries)
- **10x Genomics Chromium system** (Source platform for the droplet-based UMI count matrices of PBMC donors analyzed)

## Examples

```
library(devtools); install_github("lichen-lab/BAMMSC"); result <- BAMMSC(data_BAMMSC, K=4, option="BAMMSC", nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10); maxpost <- apply(result$delta[[1]][,,dim(result$delta[[1]])[3]], 1, max); vague <- maxpost < 0.95
```

## Evaluation signals

- Output CSV contains a maximum-posterior column and a vague flag for every cell, keyed by cluster membership from mem
- Fraction of flagged vague cells (max posterior < 0.95) is small relative to the total cell count
- t-SNE overlay shows vague cells concentrated at cluster boundaries rather than cluster cores
- delta matrices have the documented shape [#cell, #cluster, #iteration] for each individual, with final-iteration values used for thresholds
- Convergence of the MCMC run is supported by the log-likelihood trace with likTol.BAMMSC=1e-10

## Limitations

- The 0.95 threshold is a heuristic; the authors recommend trying a range of cluster numbers in practice since K is treated as a continuum
- Posterior probabilities measure only clustering uncertainty — each cell is assumed to belong to one distinct cluster, so they cannot be interpreted as cell cycle or developmental stage quantification
- Filtering genes with excessive zeros prior to clustering may be problematic for rare cell type identification, potentially biasing which cells appear vague

## Evidence

- [results] Vague cells definition and boundary localization: [evidence span withheld — anchor status: not_found]
- [readme] Posterior probability output structure: "delta: a list of 3-D matrix. Each matrix [#cell, #cluster, #iteration] is the poster probability estimates for each individual across all iterations."
- [readme] BAMMSC default parameters: "nBurn: number of iterations discarded at the initial portion of the Markov chain, default is 400"
- [discussion] Posterior as clustering uncertainty: "we assume that each cell belongs to one distinct cluster. The posterior probability measures the clustering uncertainty"
- [methods] Installation command: "install_github("lichen-lab/BAMMSC")"
