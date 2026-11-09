---
name: tsne-cluster-boundary-visualization
description: Use when you have BAMM-SC posterior cluster probabilities per cell and
  need to quantify and spatially validate clustering uncertainty, e.g. confirming
  ambiguous cells sit at cluster boundaries on a t-SNE plot.
when_to_use_negative:
- You need posterior probabilities to represent continuous biological states (e.g.
  cell cycle or developmental stage) — BAMM-SC assumes each cell belongs to one distinct
  cluster and the posterior measures only clustering uncertainty.
- The data come from a single individual with no multi-individual batch heterogeneity,
  where the posterior-uncertainty workflow of BAMMSC (rather than DIMMSC) is not the
  intended mode.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BAMM-SC
  role: Bayesian hierarchical Dirichlet-multinomial mixture model producing the per-cell
    posterior cluster probabilities (delta) used to flag vague cells
  repo: https://github.com/lichen-lab/BAMMSC
- name: t-SNE
  role: Dimensionality reduction (as in the paper's t-SNE on the first 50 principal
    components) used to spatially overlay vague cells and inspect cluster-boundary
    localization
- name: 10x Genomics Chromium system
  role: Droplet-based platform that generated the human PBMC scRNA-seq count matrices
    analyzed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:da9c8560f9953fdd79b43cd51767fd129eaa6f90bca9657084e9abbfae265b5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/BAMMSC
  tools:
  - 10x Genomics Chromium system
  - BAMM-SC
  - t-SNE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# tsne-cluster-boundary-visualization

## Summary

Localizes uncertain cell-type assignments by flagging 'vague' cells (maximum BAMM-SC posterior cluster probability <0.95) and overlaying them on a t-SNE projection to verify they fall at cluster boundaries rather than cluster cores, as done in the human PBMC analysis.

## When to use

Use when you have BAMM-SC posterior cluster probabilities per cell and need to quantify and spatially validate clustering uncertainty, e.g. confirming ambiguous cells sit at cluster boundaries on a t-SNE plot.

## When NOT to use

- You need posterior probabilities to represent continuous biological states (e.g. cell cycle or developmental stage) — BAMM-SC assumes each cell belongs to one distinct cluster and the posterior measures only clustering uncertainty.
- The data come from a single individual with no multi-individual batch heterogeneity, where the posterior-uncertainty workflow of BAMMSC (rather than DIMMSC) is not the intended mode.

## Inputs

- List of per-individual UMI count matrices (genes x cells) from human PBMC samples processed via the 10x Genomics Chromium system
- BAMMSC result object containing delta (posterior probabilities [#cell, #cluster, #iteration]) and mem (cluster membership vectors)

## Outputs

- pbmc_vague_cells.csv — per-cell table with cluster label, maximum posterior probability, and vague flag
- pbmc_vague_cells_tsne.png — t-SNE projection annotated with vague-cell positions
- Fraction of cells flagged as vague (expected to be a small subset of all cells)

## How to apply

Run BAMMSC on the list of per-donor human PBMC UMI count matrices (option='BAMMSC', K clusters, defaults nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10) and extract the delta output, a per-individual 3-D posterior probability matrix [#cell, #cluster, #iteration], along with cluster memberships from mem. For every cell take the maximum posterior probability from the final MCMC iteration and flag cells as 'vague' when this maximum is <0.95, recording the fraction of all cells they comprise. Compute a t-SNE projection analogous to the paper's t-SNE on the first 50 principal components and overlay the vague cells to confirm they map to cluster boundaries rather than cluster cores. Write a per-cell table (cluster label, max posterior, vague flag) to CSV and the annotated projection to PNG. Rationale: the paper argues that vague cells concentrating at boundaries rather than cores supports the validity of the clustering results, since the posterior probability directly measures clustering uncertainty.

## Related tools

- **BAMM-SC** (Bayesian hierarchical Dirichlet-multinomial mixture model producing the per-cell posterior cluster probabilities (delta) used to flag vague cells) — https://github.com/lichen-lab/BAMMSC
- **t-SNE** (Dimensionality reduction (as in the paper's t-SNE on the first 50 principal components) used to spatially overlay vague cells and inspect cluster-boundary localization)
- **10x Genomics Chromium system** (Droplet-based platform that generated the human PBMC scRNA-seq count matrices analyzed)

## Examples

```
result=BAMMSC(data_BAMMSC,K=4); maxpost=apply(result$delta[[1]][,,result$delta[[1]][1,1,] |> length(), drop=FALSE][,,1],1,max); vague=maxpost<0.95
```

## Evaluation signals

- The output CSV contains a max-posterior column and a vague flag for every cell across all individuals.
- The fraction of flagged vague cells (max posterior <0.95) is small relative to the total cell count.
- On the t-SNE plot, vague cells are concentrated at cluster boundaries rather than in cluster cores, matching the paper's reported uncertainty assessment.
- The max-posterior values are consistent with cluster memberships in the mem output for the final MCMC iteration.
- MCMC log-likelihood trace (loglik output) shows convergence with likTol.BAMMSC=1e-10 before posteriors are extracted.

## Limitations

- Genes with excessive zeros are filtered out prior to analysis under the assumption that lowly-expressed genes do not contribute much to clustering, which may be problematic for rare cell type identification.
- Zero-inflation is not explicitly modeled, which may or may not affect clustering accuracy and hence the posterior probabilities.
- Each cell is assumed to belong to one distinct cluster; posterior probabilities cannot be interpreted as cell cycle or developmental stage quantification.
- The number of clusters K is treated as a continuum, so the authors recommend trying a range of cluster numbers rather than a single value.

## Evidence

- [other] Vague cells in the human PBMC samples are mostly located at the boundary of different clusters, supporting the validity of the clustering results.: [evidence span withheld — anchor status: not_found]
- [readme] delta output description: "delta: a list of 3-D matrix. Each matrix [#cell, #cluster, #iteration] is the poster probability estimates for each individual across all iterations."
- [discussion] posterior measures clustering uncertainty: "we assume that each cell belongs to one distinct cluster. The posterior probability measures the clustering uncertainty"
- [intro] t-SNE on first 50 PCs: "we produced a t-SNE plot based on the first 50 principal components (Supplementary Fig. 1) of all cells from these 5 donors"
- [readme] BAMMSC default MCMC parameters: "nBurn: number of iterations discarded at the initial portion of the Markov chain, default is 400"
