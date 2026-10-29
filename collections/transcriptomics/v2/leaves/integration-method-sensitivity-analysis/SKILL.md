---
name: integration-method-sensitivity-analysis
description: 'Use when you must justify or tune PILOT''s optimal-transport cost matrix:
  i.e., before reporting sample-level Wasserstein distances/Leiden clusters, to test
  whether the cosine similarity and integration choice materially affect clustering
  or trajectory concordance.'
when_to_use_negative:
- The goal is cell-level integration or batch correction quality itself rather than
  how integration choices affect PILOT's sample-level Wasserstein distances
- Only a single pre-integrated dataset with a fixed cost matrix is available and no
  alternative similarity metrics can be recomputed from cluster medians in PCA space
- The dataset is a pathomics/structural dataset without cluster-median feature representations
  from which cosine or alternative cost matrices can be estimated
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PILOT
  role: computes sample-level Wasserstein distances (wasserstein_distance), Leiden
    clustering of W, select_best_sil, and EIPLGraph trajectory estimation for each
    configuration
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: provides the emd2 optimal-transport solver underlying PILOT's Wasserstein
    distance computation
- name: scanpy
  role: PCA embedding of cells in the Lupus PBMC data prior to cost-matrix estimation
- name: Leiden clustering
  role: clusters the sample distance matrix W per configuration, with resolution picked
    via silhouette score
- name: pandas
  role: tabulates integration-method × similarity-metric performance into the CSV
    table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:c2615195705edc250a0fc6cdca717ecb10e7fb6b3639804db0375f0cad178e01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - Leiden clustering
  - PILOT
  - POT library
  - pandas
  - scanpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# integration-method-sensitivity-analysis

## Summary

Benchmark sensitivity of PILOT's sample-level Wasserstein distances to the cell-integration method (e.g., harmony/bknn/scanorama) and to the cost-matrix similarity metric (cosine vs. Euclidean, Manhattan, Chebyshev), to justify the default cosine cost function.

## When to use

Use when you must justify or tune PILOT's optimal-transport cost matrix: i.e., before reporting sample-level Wasserstein distances/Leiden clusters, to test whether the cosine similarity and integration choice materially affect clustering or trajectory concordance.

## When NOT to use

- The goal is cell-level integration or batch correction quality itself rather than how integration choices affect PILOT's sample-level Wasserstein distances
- Only a single pre-integrated dataset with a fixed cost matrix is available and no alternative similarity metrics can be recomputed from cluster medians in PCA space
- The dataset is a pathomics/structural dataset without cluster-median feature representations from which cosine or alternative cost matrices can be estimated

## Inputs

- Lupus PBMC scRNA-seq matrix GSE174188_CLUES1_adjusted.h5ad.gz from GEO (scverse/AnnData .h5ad)
- PCA embedding of cells computed with scanpy
- Per-sample disease-status labels (e.g., control vs. SLE) for scoring ARI/NMI and trajectory concordance

## Outputs

- CSV table of performance for all integration-method × similarity-metric combinations (Appendix Fig. S5 layout)
- Per-configuration sample-level Wasserstein distance matrix W from PILOT's wasserstein_distance (POT emd2)
- Per-configuration Leiden sample clusters (resolution chosen by select_best_sil silhouette curve) and EIPLGraph trajectory orderings
- ARI/NMI and trajectory label-concordance scores per configuration

## How to apply

Download the Lupus PBMC scRNA data (GSE174188_CLUES1_adjusted.h5ad.gz from GEO) and embed cells in PCA space with scanpy. For every combination of cell-integration method (harmony, bknn, scanorama) and cost-matrix similarity metric (cosine as PILOT's default, plus Euclidean, Manhattan, Chebyshev), compute sample-level Wasserstein distances with PILOT's wasserstein_distance, which calls the emd2 solver of the POT library. Cluster each Wasserstein distance matrix with Leiden, choosing the resolution via the silhouette score curve with pl.pl.select_best_sil(adata, start = 0.2). Estimate the disease trajectory per configuration using diffusion maps followed by EIPLGraph backbone fitting with a control sample as the source node. Score each configuration by agreement of predicted sample clusters with true disease-status labels (ARI/NMI) and by label concordance of trajectory ordering, then tabulate all integration-method × metric combinations in a CSV table reproducing Appendix Fig. S5. Accept the table only if it confirms cosine as the preferred similarity metric and a limited impact of the integration-method choice, consistent with Fig. S5.

## Related tools

- **PILOT** (computes sample-level Wasserstein distances (wasserstein_distance), Leiden clustering of W, select_best_sil, and EIPLGraph trajectory estimation for each configuration) — https://github.com/CostaLab/PILOT
- **POT library** (provides the emd2 optimal-transport solver underlying PILOT's Wasserstein distance computation)
- **scanpy** (PCA embedding of cells in the Lupus PBMC data prior to cost-matrix estimation)
- **Leiden clustering** (clusters the sample distance matrix W per configuration, with resolution picked via silhouette score)
- **pandas** (tabulates integration-method × similarity-metric performance into the CSV table)

## Examples

```
import pilotpy as pl; pl.pl.select_best_sil(adata, start = 0.2)
```

## Evaluation signals

- The CSV table confirms cosine similarity ranks first and is the only metric outperforming Euclidean, Manhattan and Chebyshev in most scenarios (Fig. S5)
- Changing the integration method (harmony/bknn/scanorama) has limited impact on sample-level clustering/trajectory performance
- For each configuration, Leiden resolution is selected from the silhouette score curve via select_best_sil with start = 0.2, not fixed arbitrarily
- Predicted sample clusters/trajectory orderings agree with true disease-status labels as quantified by ARI/NMI and label concordance
- Each configuration's W matrix is reproducible from PILOT's wasserstein_distance using the emd2 solver of the POT library

## Limitations

- PILOT estimates the cost matrix as cosine distance between cluster medians in PCA space; the benchmark does not cover similarity metrics beyond Euclidean, Manhattan and Chebyshev
- The EIPLGraph trajectory root sample is visually selected rather than chosen by an objective rule, which can affect trajectory-concordance scoring
- Analysis of data from distinct technologies (single nucleus vs. single cell) or isolation methods is still an open challenge, so integration-method conclusions may not transfer across such datasets
- PILOT does not explicitly correct for potential technical artefacts; it only provides statistical tests to evaluate their influence

## Evidence

- [discussion] cost matrix as cosine distance between cluster medians in PCA space: "we generate a median representation for each cluster (in the PCA space) and estimate the cosine distance between the median value of clusters"
- [discussion] emd2 solver from POT underlies PILOT distances: "PILOT is based on the emd2 function from POT library (Flamary et al, 2021), which implements the best solver described in (Bonneel et al, 2011)."
- [methods] silhouette-based resolution selection used per configuration: "pl.pl.select_best_sil(adata, start = 0.2)"
- [readme] optimal transport computes sample-level Wasserstein distances: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples."
