---
name: diffusion-map-trajectory-embedding
description: Use when sample-level distances (e.g. PILOT Wasserstein distances between
  scRNA-seq samples) are available and the goal is to order patients along a disease
  progression trajectory for downstream cell/gene importance testing.
when_to_use_negative:
- When patients are already labeled by disease group and only group-level comparison
  (not a continuous trajectory) is needed — PILOT also supports Leiden clustering
  of W instead of trajectory estimation
- When the input is a raw cell-by-gene matrix without a computed sample-level distance
  matrix — the Wasserstein distances must be estimated first
- When samples come from distinct technologies or isolation methods (single nucleus
  vs single cell, droplet vs well based), which the article flags as an open challenge
  for this analysis
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: PILOT
  role: Performs the diffusion map embedding of the sample distance matrix (pl.pl.trajectory)
    and the EIPLGraph principal graph fit (pl.pl.fit_pricipla_graph) to define pseudotime
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: Provides the emd2 optimal transport solver used to compute the Wasserstein
    distances that form the trajectory input matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:c8527953dc21628d4ac7be21429a98a1aad6d3b583a04ddea9037c64c63c0c32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  - scanpy
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# diffusion-map-trajectory-embedding

## Summary

Embeds a PILOT patient-level Wasserstein distance matrix into a low-dimensional space with diffusion maps and fits an EIPLGraph principal graph to define a disease-progression pseudotime across single-cell samples.

## When to use

Use when sample-level distances (e.g. PILOT Wasserstein distances between scRNA-seq samples) are available and the goal is to order patients along a disease progression trajectory for downstream cell/gene importance testing.

## When NOT to use

- When patients are already labeled by disease group and only group-level comparison (not a continuous trajectory) is needed — PILOT also supports Leiden clustering of W instead of trajectory estimation
- When the input is a raw cell-by-gene matrix without a computed sample-level distance matrix — the Wasserstein distances must be estimated first
- When samples come from distinct technologies or isolation methods (single nucleus vs single cell, droplet vs well based), which the article flags as an open challenge for this analysis

## Inputs

- PILOT Wasserstein distance matrix W between patient samples (from pl.tl.wasserstein_distance)
- AnnData object (e.g. myocardial_infarction.h5ad) with sampleID, Status, and cell_subtype/cell cluster annotations
- Root/source sample node identifier for EIPLGraph (a control sample, e.g. source_node=7 in the MI tutorial)

## Outputs

- Diffusion-map embedding of the sample distance matrix
- EIPLGraph principal graph defining the disease progression trajectory
- Per-sample disease progression pseudotime values for downstream cell_importance and genes_importance analyses

## How to apply

First compute the sample-by-sample Wasserstein distance matrix with PILOT's optimal transport (emd2 from the POT library, e.g. pl.tl.wasserstein_distance on the myocardial_infarction.h5ad AnnData with emb_matrix='PCA', clusters_col='cell_subtype', sample_col='sampleID', status='Status'). Embed this distance matrix with diffusion maps (Coifman et al) via pl.pl.trajectory, then fit the EIPLGraph principal graph with pl.pl.fit_pricipla_graph, supplying a source_node that is a control sample (in the MI tutorial, source_node=7) to root the disease progression pseudotime. Samples that are trajectory outliers (as in the pancreas data, where embryos E12-E15 were excluded) should be filtered before embedding. The resulting pseudotime is the input for PILOT's downstream cell_importance (Huber regression with linear, quadratic, and linear-quadratic terms) and genes_importance / Wald test analyses.

## Related tools

- **PILOT** (Performs the diffusion map embedding of the sample distance matrix (pl.pl.trajectory) and the EIPLGraph principal graph fit (pl.pl.fit_pricipla_graph) to define pseudotime) — https://github.com/CostaLab/PILOT
- **POT library** (Provides the emd2 optimal transport solver used to compute the Wasserstein distances that form the trajectory input matrix)

## Examples

```
import pilotpy as pl; pl.pl.trajectory(adata); pl.pl.fit_pricipla_graph(adata, source_node=7)
```

## Evaluation signals

- The diffusion map embedding is computed on the sample-by-sample Wasserstein distance matrix, not on raw cell data
- EIPLGraph is rooted at a control sample (source_node=7 in the MI tutorial); the trajectory should start among control samples
- Downstream pl.tl.cell_importance on the inferred pseudotime recovers the reported MI hallmarks: quadratic increase of SPP1+ macrophages and myofibroblasts, quadratic decrease of healthy cardiomyocytes
- Trajectory outlier samples (e.g. embryos E12-E15 in the pancreas dataset) are identified and excluded before embedding
- The EIPLGraph fit accepts source_node, clusters_col, and sample_col parameters consistent with the AnnData schema (sampleID, Status, cell_subtype)

## Limitations

- The EIPLGraph root sample was visually selected to reflect control samples rather than chosen by an objective rule, a reproducibility gap acknowledged in the article
- PILOT does not explicitly correct for technical artefacts; it only provides statistical tests to evaluate their influence on trajectory predictions
- Trajectory outliers (e.g. four pancreas samples E12-E15) must be excluded manually, as they distort the embedding

## Evidence

- [discussion] PILOT also performs trajectory analysis by the use of diffusion maps (Coifman et al, 2005) followed by a trajectory estimation with EIPLGraph: "PILOT also performs trajectory analysis by the use of diffusion maps (Coifman et al, 2005) followed by a trajectory estimation with EIPLGraph"
- [discussion] root sample visually selected: "EIPLGraph also requires a root sample, which was visually selected to reflect parts of the trajectory with control samples."
- [readme] Wasserstein distance as trajectory input: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples."
- [discussion] step-wise regression on trajectory: "it fits regression models with linear, quadratic, and linear-quadratic terms for each feature and uses statistical tests to determine the goodness of fit."
