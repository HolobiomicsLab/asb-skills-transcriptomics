---
name: earth-mover-distance-solving-single-cell-samples
description: Use when each sample is represented as a vector of cell-cluster (or tissue-structure)
  proportions and you need a pairwise, unsupervised sample-level distance matrix (e.g.
  for downstream Leiden clustering or diffusion-map trajectories).
when_to_use_negative:
- Samples have no cluster-level representation (raw per-cell data not yet clustered
  into cell populations or tissue structures) — cluster proportions are required marginals
- You need label-supervised sample grouping — prior methods like PhEMD/SCANCell/Multi-scale
  PHATE require patient labels, but if labels are the analysis goal a supervised approach
  fits better
- Data were produced with distinct technologies or cell-isolation methods (e.g. single-nucleus
  vs single-cell, droplet vs well based) — PILOT assumes samples were collected uniformly
  and such cross-technology comparison is an open challenge
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0091
tools:
- name: PILOT
  role: Framework that uses optimal transport to compute the Wasserstein distance
    between two individual single-cell samples; stores the resulting EMD matrix in
    adata.uns['EMD']
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: Provides the emd2 function that solves the optimal transport problem between
    two samples' cluster distributions using the Bonneel et al 2011 solver
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:cbcceba6c2bb7b9888b5332b6e7e49bc239936da0510c3008b9175a704a73076
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# earth-mover-distance-solving-single-cell-samples

## Summary

Compute a Wasserstein (earth mover's) distance between two individual single-cell samples by solving an optimal transport problem over their cluster distributions, with a cost matrix built from cosine distances between cluster medians in PCA space. This is the core sample-level distance of PILOT.

## When to use

Use when each sample is represented as a vector of cell-cluster (or tissue-structure) proportions and you need a pairwise, unsupervised sample-level distance matrix (e.g. for downstream Leiden clustering or diffusion-map trajectories).

## When NOT to use

- Samples have no cluster-level representation (raw per-cell data not yet clustered into cell populations or tissue structures) — cluster proportions are required marginals
- You need label-supervised sample grouping — prior methods like PhEMD/SCANCell/Multi-scale PHATE require patient labels, but if labels are the analysis goal a supervised approach fits better
- Data were produced with distinct technologies or cell-isolation methods (e.g. single-nucleus vs single-cell, droplet vs well based) — PILOT assumes samples were collected uniformly and such cross-technology comparison is an open challenge

## Inputs

- Per-sample cluster-distribution matrix (Dirichlet-multinomial MAP estimates of cluster proportions per sample)
- Cluster median representations in PCA space (one median vector per cell cluster)
- AnnData object holding per-sample cluster distributions and PCA embeddings

## Outputs

- Symmetric pairwise Wasserstein distance matrix W (n_samples x n_samples, non-negative, zero diagonal)
- wasserstein_distance_matrix.csv
- EMD matrix stored in adata.uns['EMD']

## How to apply

Load the per-sample cluster-distribution vectors (Dirichlet-multinomial MAP estimates of cluster proportions) and the cluster median representations in PCA space. Build the cost matrix C as the pairwise cosine distance between the median value of each cluster, following Eq. 7 of the PILOT methods. For each pair of samples, solve the earth mover's distance problem with POT's emd2 function (Bonneel et al 2011 solver), using the two samples' cluster-distribution vectors as marginals and C as the ground cost, then assemble the resulting transport costs into a symmetric Wasserstein distance matrix W with zero diagonal. Save W (e.g. as wasserstein_distance_matrix.csv) and verify consistency with the EMD matrix stored by PILOT in adata.uns['EMD'].

## Related tools

- **PILOT** (Framework that uses optimal transport to compute the Wasserstein distance between two individual single-cell samples; stores the resulting EMD matrix in adata.uns['EMD']) — https://github.com/CostaLab/PILOT
- **POT library** (Provides the emd2 function that solves the optimal transport problem between two samples' cluster distributions using the Bonneel et al 2011 solver)

## Examples

```
import ot; W = ot.emd2(proportions_l, proportions_q, C)  # C = cosine-distance cost matrix between cluster medians in PCA space
```

## Evaluation signals

- Output matrix is square (n_samples x n_samples) with zero diagonal
- Matrix is symmetric and non-negative
- Values are consistent with the EMD matrix stored by PILOT in adata.uns['EMD']
- Each row/column of the input cluster-distribution matrix is a valid probability vector whose marginals match the emd2 constraints
- Cost matrix C equals the pairwise cosine distances between cluster medians in PCA space (recomputable directly)

## Limitations

- PILOT does not explicitly correct for potential technical artefacts; it only provides statistical tests to evaluate their influence
- Assumes tissue samples were collected uniformly (similar tissue areas, same technology); comparing data across distinct technologies or isolation methods is an open challenge
- Distance quality depends on upstream clustering granularity (e.g. number of Leiden clusters) and the Dirichlet-multinomial MAP estimates of cluster proportions

## Evidence

- [discussion] PILOT is based on the emd2 function from POT library (Flamary et al, 2021), which implements the best solver described in (Bonneel et al, 2011).: "PILOT is based on the emd2 function from POT library (Flamary et al, 2021), which implements the best solver described in (Bonneel et al, 2011)."
- [discussion] we generate a median representation for each cluster (in the PCA space) and estimate the cosine distance between the median value of clusters: "we generate a median representation for each cluster (in the PCA space) and estimate the cosine distance between the median value of clusters"
- [readme] PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples.: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples."
- [discussion] Robust estimation of sample probability distributions using a hierarchical Dirichlet-multinomial model with pseudo cell counts: [evidence span withheld — anchor status: not_found]
