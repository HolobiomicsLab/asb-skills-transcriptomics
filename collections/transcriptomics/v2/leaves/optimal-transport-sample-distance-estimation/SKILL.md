---
name: optimal-transport-sample-distance-estimation
description: Use when you have multiple single-cell/pathomics samples (.h5ad with
  per-cell cluster labels and sample IDs) and need an unsupervised, sample-level distance
  matrix to cluster patients or infer disease trajectories.
when_to_use_negative:
- When the input is already a sample-level feature/summary table rather than per-cell
  annotated data — no cluster distributions exist to define transport marginals
- When samples come from distinct technologies or isolation methods (single nucleus
  vs. single cell, droplet vs. well based), which the authors flag as an open challenge
  for cross-technology comparison
- When labeled, supervised patient stratification is required — competing approaches
  needing patient labels are out of PILOT's unsupervised scope
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3474
tools:
- name: PILOT
  role: Package exposing pl.tl.wasserstein_distance, which robustly estimates sample
    cluster distributions and computes the EMD matrix
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: Provides the emd2 function (Bonneel et al 2011 solver) used to solve the optimal
    transport problem for each sample pair
- name: scanpy
  role: Loads the .h5ad inputs and provides PCA embeddings from which cluster medians
    are taken
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  - task_008
  - task_007
  - task_002
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:e983804a1df0ddc75452fe5ded5714eb508e8f3552ca6e4e1c92d93588b883e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  - SCANCell
  - Seurat
  - pandas
  - scanpy
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# optimal-transport-sample-distance-estimation

## Summary

PILOT-style computation of patient-level Wasserstein distances between single-cell or pathomics samples using optimal transport over cell-cluster distributions, with the cost matrix derived from cosine distances between cluster medians in PCA space.

## When to use

Use when you have multiple single-cell/pathomics samples (.h5ad with per-cell cluster labels and sample IDs) and need an unsupervised, sample-level distance matrix to cluster patients or infer disease trajectories.

## When NOT to use

- When the input is already a sample-level feature/summary table rather than per-cell annotated data — no cluster distributions exist to define transport marginals
- When samples come from distinct technologies or isolation methods (single nucleus vs. single cell, droplet vs. well based), which the authors flag as an open challenge for cross-technology comparison
- When labeled, supervised patient stratification is required — competing approaches needing patient labels are out of PILOT's unsupervised scope

## Inputs

- AnnData .h5ad single-cell or pathomics object with per-cell cluster annotations (e.g. clusters_col='cell_subtype' or 'Cell_type'), sample IDs (sample_col='sampleID'), and disease status (status='Status')
- Per-sample cluster-distribution matrix (Dirichlet-multinomial MAP estimates)
- Cluster median representations in PCA space
- Pathomics data: morphometric AnnData such as Kidney_IgAN_T.h5ad / Kidney_IgAN_G.h5ad with data_type='Pathomics'

## Outputs

- Symmetric n_samples x n_samples Wasserstein (EMD) distance matrix with zero diagonal, stored as adata.uns['EMD'] and exportable as wasserstein_distance_matrix.csv
- Optimal transport plan T per sample pair, whose total cost is the sample distance
- Downstream inputs: sample-level distance matrix for Leiden clustering, diffusion maps, and EIPLGraph trajectory fitting

## How to apply

Load each sample's cells with scanpy and, per the PILOT methods, robustly estimate each sample's cluster-distribution vector using a hierarchical Dirichlet-multinomial model with pseudo cell counts. Build the cost matrix C as pairwise cosine distances between cluster median representations in PCA space (Eq. 7). For every sample pair, solve the earth mover's distance with POT's emd2 solver (Bonneel et al 2011) using the two cluster-distribution vectors as marginals and C as the ground cost, then assemble the symmetric, zero-diagonal Wasserstein distance matrix W (stored by PILOT in adata.uns['EMD']). W then feeds downstream Leiden clustering of samples or diffusion-map + EIPLGraph trajectory estimation; cosine similarity was the top-ranked cost metric in the sensitivity analysis (Appendix Fig. S5).

## Related tools

- **PILOT** (Package exposing pl.tl.wasserstein_distance, which robustly estimates sample cluster distributions and computes the EMD matrix) — https://github.com/CostaLab/PILOT
- **POT library** (Provides the emd2 function (Bonneel et al 2011 solver) used to solve the optimal transport problem for each sample pair)
- **scanpy** (Loads the .h5ad inputs and provides PCA embeddings from which cluster medians are taken)

## Examples

```
import pilotpy as pl; adata = sc.read_h5ad('myocardial_infarction.h5ad'); pl.tl.wasserstein_distance(adata, emb_matrix='PCA', clusters_col='cell_subtype', sample_col='sampleID', status='Status')
```

## Evaluation signals

- Output matrix is square (n_samples x n_samples), symmetric, non-negative, with zero diagonal
- EMD values are consistent with the matrix stored in adata.uns['EMD']
- Leiden clustering of W reproduces disease-status groupings (validated by ARI/Silhouette against disease labels in the 12-dataset benchmark, Friedman–Nemenyi p<0.05)
- Trajectory built from W orders control samples at the start and case samples toward the end (e.g. normal-eGFR samples first in the kidney IgAN pathomics run)
- Sensitivity check: cosine cost matrix should rank first versus Euclidean/Manhattan/Chebyshev alternatives (Appendix Fig. S5)

## Limitations

- PILOT assumes tissue samples were collected uniformly (similar tissue areas, same technology); analysis across distinct technologies or isolation methods is an open challenge
- PILOT does not explicitly correct for technical artefacts; it only provides statistical tests to evaluate their influence
- The cost-matrix choice matters: cosine similarity was the only metric outperforming Euclidean, Manhattan, and Chebyshev in most scenarios, so alternative metrics may degrade performance

## Evidence

- [abstract] PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples.: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples."
- [discussion] we generate a median representation for each cluster (in the PCA space) and estimate the cosine distance between the median value of clusters: "we generate a median representation for each cluster (in the PCA space) and estimate the cosine distance between the median value of clusters"
- [discussion] PILOT is based on the emd2 function from POT library (Flamary et al, 2021), which implements the best solver described in (Bonneel et al, 2011).: "PILOT is based on the emd2 function from POT library (Flamary et al, 2021), which implements the best solver described in (Bonneel et al, 2011)."
- [results] Cosine similarity ranked first and was the only metric outperforming other approaches: [evidence span withheld — anchor status: not_found]
- [readme] This allows us to perform unsupervised analysis at the sample level and uncover trajectories or cellular clusters associated with disease progression.: "This allows us to perform unsupervised analysis at the sample level and uncover trajectories or cellular clusters associated with disease progression."
