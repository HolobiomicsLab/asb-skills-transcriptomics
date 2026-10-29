---
name: scrnaseq-wasserstein-distance-computation
description: Use when you have a multi-sample scRNA-seq AnnData (e.g. patient biopsies
  with cell-type annotations, PCA embedding, patient and disease-status columns) and
  need sample-level distances to detect disease-associated patient subgroups without
  patient labels.
when_to_use_negative:
- When patients already carry labels required by competing approaches (PhEMD, SCANCell,
  Multi-scale PHATE) and a supervised comparison suffices — PILOT's value here is
  unsupervised sample-level analysis
- When samples were collected non-uniformly across distinct technologies (single nucleus
  vs. single cell) or isolation methods (droplet vs. well based), which the authors
  flag as an open challenge
- When the goal is per-cell rather than per-sample comparison — this skill operates
  on sample-level cluster distributions, not individual cells
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: PILOT
  role: Implements the Wasserstein distance computation between individual single-cell
    samples via pl.tl.wasserstein_distance
  repo: https://github.com/CostaLab/PILOT
- name: POT library
  role: Provides the emd2 optimal-transport solver underlying PILOT's distance computation
- name: scanpy
  role: Used to load and preprocess the AnnData object (sc.read_h5ad) and supply the
    PCA embedding consumed by the distance computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:bcaccd20171da9083a8eb2b2d4da37db28bd7dd740e07321101ccd17b5b79ca3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - PILOT
  - POT library
  - Seurat
  - pandas
  - scanpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# scrnaseq-wasserstein-distance-computation

## Summary

Compute Wasserstein (earth mover's) distances between individual single-cell samples with PILOT's optimal-transport machinery (emd2 from the POT library) so that samples—not cells—become the unit of unsupervised analysis. This yields a sample-by-sample distance matrix W that feeds Leiden clustering or trajectory analysis.

## When to use

Use when you have a multi-sample scRNA-seq AnnData (e.g. patient biopsies with cell-type annotations, PCA embedding, patient and disease-status columns) and need sample-level distances to detect disease-associated patient subgroups without patient labels.

## When NOT to use

- When patients already carry labels required by competing approaches (PhEMD, SCANCell, Multi-scale PHATE) and a supervised comparison suffices — PILOT's value here is unsupervised sample-level analysis
- When samples were collected non-uniformly across distinct technologies (single nucleus vs. single cell) or isolation methods (droplet vs. well based), which the authors flag as an open challenge
- When the goal is per-cell rather than per-sample comparison — this skill operates on sample-level cluster distributions, not individual cells

## Inputs

- Reprocessed single-cell AnnData object (.h5ad) with one sample/patient per biological replicate, containing cell-level data
- PCA embedding of cells (dimensionality-reduced expression matrix)
- Cell-type annotation column per cell
- Patient/sample ID column
- Disease-status column (e.g. control vs. tumor)

## Outputs

- Sample-by-sample Wasserstein distance matrix W suitable for Leiden clustering or trajectory inference
- Optimal transport plan between pairs of sample cluster distributions

## How to apply

Load the reprocessed AnnData (e.g. PDAC data from GSA CRA001160) and run pl.tl.wasserstein_distance, supplying the PCA embedding plus columns for cell type, patient, and disease status. PILOT represents each sample as a probability distribution over cell clusters, robustly estimated with a hierarchical Dirichlet-multinomial model that adds pseudo cell counts weighted by the cell distribution of all samples; the cost matrix is the cosine distance between cluster medians in PCA space. The optimal transport plan between two sample distributions is computed with the emd2 function from the POT library (implementing the Bonneel et al 2011 solver), producing the Wasserstein distance between each pair of individual samples. Collect the pairwise distances into the sample distance matrix W, which is the input for downstream Leiden clustering or diffusion-map/EIPLGraph trajectory analysis. Filter low-quality samples first (e.g. exclude samples with less than 1000 genes on average per cell).

## Related tools

- **PILOT** (Implements the Wasserstein distance computation between individual single-cell samples via pl.tl.wasserstein_distance) — https://github.com/CostaLab/PILOT
- **POT library** (Provides the emd2 optimal-transport solver underlying PILOT's distance computation)
- **scanpy** (Used to load and preprocess the AnnData object (sc.read_h5ad) and supply the PCA embedding consumed by the distance computation)

## Examples

```
import pilotpy as pl; adata = sc.read_h5ad('PDAC.h5ad'); pl.tl.wasserstein_distance(adata, 'clusters', 'disease', 'patient')
```

## Evaluation signals

- The output is a square, symmetric pairwise distance matrix W with one row/column per sample (e.g. 35 PDAC patients for the GSA CRA001160 case study)
- Distances are non-negative and diagonal entries are zero (Wasserstein distance of a distribution to itself)
- Downstream Silhouette-score analysis (pl.pl.select_best_sil) over Leiden resolutions must yield a well-defined optimum (0.3 for the PDAC data) from the computed W
- Clustering W at the optimal resolution must reproduce expected patient groupings (11 control / 14 Tumor 1 / 10 Tumor 2 for PDAC)
- Low-quality samples (less than 1000 genes on average per cell) are excluded before computing distances

## Limitations

- PILOT does not explicitly correct for technical artefacts; it only provides statistical tests to evaluate whether technical factors influence the predictions
- Assumes tissue samples were collected uniformly (similar tissue areas and same technology); analysis across distinct technologies or isolation methods is an open challenge
- Requires a pre-defined cell-type/clustering annotation per sample; the root sample for downstream EIPLGraph trajectories was visually selected rather than chosen by an objective rule

## Evidence

- [abstract] PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples.: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples."
- [discussion] emd2 POT solver: "PILOT is based on the emd2 function from POT library (Flamary et al, 2021), which implements the best solver described in (Bonneel et al, 2011)."
- [discussion] Dirichlet-multinomial robust estimation: "This prior adds “pseudo cell counts”, which are weighted by the cell distribution of all samples."
- [discussion] cosine cost matrix in PCA space: "we generate a median representation for each cluster (in the PCA space) and estimate the cosine distance between the median value of clusters"
- [methods] select_best_sil method call: "pl.pl.select_best_sil(adata, start = 0.2)"
- [readme] README install: "The easiest way to install PILOT and the required packages is using the following way:

```terminal
pip install pilotpy
```"
