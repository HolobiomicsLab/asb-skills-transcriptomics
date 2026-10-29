---
name: clustering-robustness-subset-resampling
description: 'Use when you need to quantify how sensitive a scRNA-seq clustering method
  is to cell subsampling: run each method on 10 random 95%-cell subsets, compute ARI/NMI
  per repeat against ground truth, and compare per-method medians.'
when_to_use_negative:
- The input is not labeled single-cell expression data (no ground-truth cell-type
  labels), since ARI/NMI against ground truth cannot be computed
- The goal is a single clustering run on the full dataset rather than a subsampling-based
  robustness comparison across multiple methods
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scanpy
  role: AnnData container and pre-processing (filter_genes min_cells=3, filter_cells
    min_genes=200, normalize_per_cell, log1p, scale) before each subset clustering
    run
  repo: https://github.com/sldyns/scAce
- name: scAce
  role: Benchmarked clustering method whose robustness is assessed; run via run_scace
    to obtain cluster labels and cell embeddings
  repo: https://github.com/sldyns/scAce
- name: Leiden
  role: Baseline clustering method in the ten-method robustness comparison; also used
    inside scAce for de novo cluster initialization
  repo: https://github.com/sldyns/scAce
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:42687393977f928c73c49ce80b9b617c76627bef27c5143666021b2eeaa9f8be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Leiden
  - Python 3.8
  - scAce
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# clustering-robustness-subset-resampling

## Summary

Evaluates scRNA-seq clustering robustness by repeatedly clustering randomly sampled 95% cell subsets (10 repeats) and comparing median ARI/NMI across methods. In the scAce paper this demonstrated scAce attains the highest median ARI and NMI among ten clustering methods on simulated data.

## When to use

Use when you need to quantify how sensitive a scRNA-seq clustering method is to cell subsampling: run each method on 10 random 95%-cell subsets, compute ARI/NMI per repeat against ground truth, and compare per-method medians.

## When NOT to use

- The input is not labeled single-cell expression data (no ground-truth cell-type labels), since ARI/NMI against ground truth cannot be computed
- The goal is a single clustering run on the full dataset rather than a subsampling-based robustness comparison across multiple methods

## Inputs

- Simulated scRNA-seq gene expression matrix with ground-truth cell-type labels (loadable into a scanpy AnnData object)
- Set of ten clustering methods to benchmark, including scAce and a Leiden clustering baseline

## Outputs

- Per-repeat ARI/NMI table for every method across the 10 repeats of 95%-subset clustering
- Per-method median ARI and median NMI across the ten repeats
- robustness_ari_nmi.csv containing the full per-repeat table and per-method medians

## How to apply

Load the simulated single-cell gene expression matrix with ground-truth labels into a scanpy AnnData object, then pre-process exactly as the scAce README specifies: filter genes with min_cells=3, filter cells with min_genes=200, store adata.raw, normalize per cell, record the scale factor as n_counts / median n_counts in adata.obs['scale_factor'], then log1p-transform and scale. For each of 10 repeats, randomly subsample 95% of cells and run each of the ten clustering methods — including scAce (ZINB-VAE pre-training, Leiden-based de novo cluster initialization, adaptive cluster merging) and the Leiden baseline — capturing cluster labels from adata.obs['scace_cluster'] for scAce. Compute Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI) against the ground-truth labels for every method on every repeat, tabulate all 10 repeats per method, and compute per-method medians. Write the full per-repeat ARI/NMI table plus per-method medians to robustness_ari_nmi.csv. Validation: scAce should attain both the highest median ARI and the highest median NMI among the ten methods, consistent with its reported better clustering accuracy and robustness.

## Related tools

- **scanpy** (AnnData container and pre-processing (filter_genes min_cells=3, filter_cells min_genes=200, normalize_per_cell, log1p, scale) before each subset clustering run) — https://github.com/sldyns/scAce
- **scAce** (Benchmarked clustering method whose robustness is assessed; run via run_scace to obtain cluster labels and cell embeddings) — https://github.com/sldyns/scAce
- **Leiden** (Baseline clustering method in the ten-method robustness comparison; also used inside scAce for de novo cluster initialization) — https://github.com/sldyns/scAce

## Examples

```
import scanpy as sc; adata = sc.AnnData(data); sc.pp.filter_genes(adata, min_cells=3); sc.pp.filter_cells(adata, min_genes=200); sc.pp.normalize_per_cell(adata); adata.obs['scale_factor'] = adata.obs.n_counts / adata.obs.n_counts.median(); sc.pp.log1p(adata); sc.pp.scale(adata); from scace import run_scace; adata = run_scace(adata)
```

## Evaluation signals

- scAce attains the highest median ARI among the ten methods across the ten 95%-subset repeats
- scAce attains the highest median NMI among the ten methods across the ten repeats
- robustness_ari_nmi.csv contains 10 ARI/NMI rows per method (10 repeats) plus per-method medians
- Every repeat uses a random 95% subset of cells, so ARI/NMI values vary across repeats rather than being identical

## Limitations

- Robustness is assessed on simulated data with ground-truth labels; conclusions depend on the simulation's fidelity to real scRNA-seq characteristics
- Results are summarized by the median over only 10 repeats, which is a small number of resamples for estimating stability
- With de novo initialization scAce depends on Leiden for initial cluster labels, so Leiden-related variability can propagate into scAce results

## Evidence

- [abstract] scAce outperforms state-of-the-art scRNA-seq clustering methods with better accuracy and robustness on simulated and real datasets: "we demonstrate that scAce outperforms state-of-the-art clustering methods for scRNA-seq data, and achieves better clustering accuracy and robustness"
- [readme] Pre-processing filter thresholds: "sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)"
- [readme] Normalize, log-transform, scale workflow: "# Total-count normlize, logarithmize and scale the data  
sc.pp.normalize_per_cell(adata)
adata.obs['scale_factor'] = adata.obs.n_counts / adata.obs.n_counts.median()"
- [readme] scAce three-step pipeline: "a pre-training step based on a variational autoencoder, a cluster initialization step to obtain initial cluster labels, and an adaptive cluster merging step"
- [readme] Running scAce and obtaining outputs: "The output adata contains cluster labels in `adata.obs['scace_cluster']` and the cell embeddings in `adata.obsm['scace_emb']`"
