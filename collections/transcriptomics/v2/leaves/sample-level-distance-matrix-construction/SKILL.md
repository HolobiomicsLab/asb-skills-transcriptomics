---
name: sample-level-distance-matrix-construction
description: Use when single-cell or pathomics data are summarized per sample as cell-cluster/structure
  proportions (plus cluster medians in PCA space) and you need an unsupervised sample-level
  distance matrix W to cluster samples or infer disease trajectories.
when_to_use_negative:
- Samples are not represented as distributions over a common cluster space (e.g. input
  is already a feature table or raw cell-level data without prior clustering)
- Data combine samples from distinct technologies or isolation methods (e.g. single
  nucleus vs single cell, droplet-based vs well-based); PILOT's authors flag such
  cross-technology comparison as an open challenge
- A patient label-supervised method is required — one motivation for PILOT was that
  competing approaches require labels of patients and cannot be used in unsupervised
  analysis
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PILOT
  role: Pipeline that estimates sample cluster distributions, builds the cosine-distance
    cost matrix, and computes the Wasserstein distances between samples
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: Provides the emd2 earth-mover-distance solver used to compute each pairwise
    Wasserstein distance
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:a49cb05122b96907890bc0af18a2349c459671533edad5cde9f71974b2fbf18a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# sample-level-distance-matrix-construction

## Summary

Constructs a symmetric patient/sample-level Wasserstein distance matrix by solving optimal transport (earth mover's distance) problems between the cluster distributions of every sample pair, as done in PILOT. This turns multi-scale single-cell or pathomics data into a sample-level distance representation for downstream clustering and trajectory analysis.

## When to use

Use when single-cell or pathomics data are summarized per sample as cell-cluster/structure proportions (plus cluster medians in PCA space) and you need an unsupervised sample-level distance matrix W to cluster samples or infer disease trajectories.

## When NOT to use

- Samples are not represented as distributions over a common cluster space (e.g. input is already a feature table or raw cell-level data without prior clustering)
- Data combine samples from distinct technologies or isolation methods (e.g. single nucleus vs single cell, droplet-based vs well-based); PILOT's authors flag such cross-technology comparison as an open challenge
- A patient label-supervised method is required — one motivation for PILOT was that competing approaches require labels of patients and cannot be used in unsupervised analysis

## Inputs

- Per-sample cluster-distribution vectors (proportions of cells/tissue structures per cluster per sample)
- Cluster median representations in PCA space (used to compute the cosine-distance cost matrix)
- Number of clusters defining the dimension of the marginals

## Outputs

- Symmetric n_samples x n_samples Wasserstein distance matrix W with zero diagonal (e.g. wasserstein_distance_matrix.csv, stored by PILOT in adata.uns['EMD'])

## How to apply

First, load the per-sample cluster-distribution vectors (PILOT uses Dirichlet-multinomial MAP estimates with 'pseudo cell counts' added by a prior weighted by the cell distribution of all samples) and the median representation of each cluster in PCA space. Build the cost matrix C as the pairwise cosine distance between cluster medians (Eq. 7 of the PILOT methods), so that transport between clusters reflects similarity of their expression/structure profiles. For each pair of samples, solve the earth-mover-distance problem with POT's emd2 function, which implements the solver of Bonneel et al 2011, using the two samples' cluster-distribution vectors as marginals and C as the ground cost. Assemble the resulting distances into a symmetric Wasserstein distance matrix W with a zero diagonal (PILOT stores it in adata.uns['EMD']); W can then be fed directly to Leiden clustering or used as input to diffusion maps plus EIPLGraph trajectory inference.

## Related tools

- **PILOT** (Pipeline that estimates sample cluster distributions, builds the cosine-distance cost matrix, and computes the Wasserstein distances between samples) — https://github.com/CostaLab/PILOT
- **POT library** (Provides the emd2 earth-mover-distance solver used to compute each pairwise Wasserstein distance)

## Examples

```
import ot; w = ot.emd2(a, b, C)  # a, b: sample cluster-distribution marginals; C: cosine-distance cost matrix from cluster PCA medians
```

## Evaluation signals

- Output matrix W is square with dimensions n_samples x n_samples
- W is symmetric and non-negative, with a zero diagonal (distance of each sample to itself)
- Off-diagonal values are consistent with the EMD matrix stored by PILOT in adata.uns['EMD']
- Each pairwise distance is finite and achievable given the marginals (the cluster-distribution vectors sum to 1 per sample, as expected for Dirichlet-multinomial MAP estimates)

## Limitations

- Assumes tissue samples were collected uniformly (similar tissue areas, same technology); analysis across distinct technologies (single nucleus vs single cell) or isolation methods (droplet vs well based) is an open challenge
- PILOT does not explicitly correct for potential technical artefacts; it only provides statistical tests to evaluate their influence
- Cost-matrix quality depends on the upstream cluster medians and the number of PCA components kept (e.g. 10 components for glomeruli and 2 for tubules in the kidney pathomics benchmark)

## Evidence

- [abstract] PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples.: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples."
- [discussion] PILOT is based on the emd2 function from POT library (Flamary et al, 2021), which implements the best solver described in (Bonneel et al, 2011).: "PILOT is based on the emd2 function from POT library (Flamary et al, 2021), which implements the best solver described in (Bonneel et al, 2011)."
- [discussion] we generate a median representation for each cluster (in the PCA space) and estimate the cosine distance between the median value of clusters: "we generate a median representation for each cluster (in the PCA space) and estimate the cosine distance between the median value of clusters"
- [discussion] Clustering analysis can be performed by providing W as input to a Leiden clustering algorithm (Traag et al, 2019).: "Clustering analysis can be performed by providing W as input to a Leiden clustering algorithm (Traag et al, 2019)."
- [readme] This remains challenging as single-cell/pathomics data are multi-scale, i.e., a sample is represented by clusters of cells/structures, and samples cannot be easily compared with each other.: "This remains challenging as single-cell/pathomics data are multi-scale, i.e., a sample is represented by clusters of cells/structures, and samples cannot be easily compared with each other."
