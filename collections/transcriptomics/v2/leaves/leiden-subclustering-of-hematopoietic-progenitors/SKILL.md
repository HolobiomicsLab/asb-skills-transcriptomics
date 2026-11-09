---
name: leiden-subclustering-of-hematopoietic-progenitors
description: Use when a batch-integrated single-cell atlas (e.g. wild-type + Tal1-/-
  chimeric E6.5-E8.5 embryos in a CODAL topic latent space) contains a coarse Leiden
  cluster enriched for hemato-endothelial progenitors that must be split into finer
  perturbation-affected subtypes.
when_to_use_negative:
- Input data has not been batch-integrated — subclustering raw scanpy PCA space will
  confound batch effects with cell state when chimeric status is batch-confounded
- The dataset is a single-batch atlas without a perturbation or batch-confounded cell
  states, so CODAL integration and chimera-vs-wildtype subclustering add no value
- The target cluster is already well resolved at coarse resolution and contains no
  rare or perturbation-induced populations to discover
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CODAL/MIRA
  role: Variational topic model that disentangles the four technical covariates from
    biological effects, producing the latent space (ILR-transformed topic compositions)
    in which Leiden clustering is run
  repo: https://github.com/cistrome/MIRA
- name: scanpy
  role: Preprocessing (concatenate batches, 400/30-count filters, normalize_total
    target_sum 10000, log1p, HVG dispersion 0.5), neighbor graph, Leiden clustering,
    and UMAP
  repo: https://scanpy.readthedocs.io/en/stable/
- name: Leiden algorithm (leidenalg)
  role: Graph community detection used both for rough clustering of the integrated
    atlas and high-resolution subclustering of Tom+ Tal1-/- HE-prog cells
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:32ba987796d2cf048b745ec0e6613cc4561962c19a26574b6d81eddbb6947d08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cistrome/MIRA
  tools:
  - CODAL/MIRA
  - Leiden algorithm
  - Leiden algorithm (leidenalg)
  - MIRA
  - UMAP-learn
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# leiden-subclustering-of-hematopoietic-progenitors

## Summary

High-resolution Leiden subclustering of hemato-endothelial progenitor (HE-prog) cells within the CODAL topic-model latent space of the mouse gastrulation atlas, isolating perturbation-induced Tal1-/- subclusters (A-E) and annotating them by mesodermal marker gene expression.

## When to use

Use when a batch-integrated single-cell atlas (e.g. wild-type + Tal1-/- chimeric E6.5-E8.5 embryos in a CODAL topic latent space) contains a coarse Leiden cluster enriched for hemato-endothelial progenitors that must be split into finer perturbation-affected subtypes.

## When NOT to use

- Input data has not been batch-integrated — subclustering raw scanpy PCA space will confound batch effects with cell state when chimeric status is batch-confounded
- The dataset is a single-batch atlas without a perturbation or batch-confounded cell states, so CODAL integration and chimera-vs-wildtype subclustering add no value
- The target cluster is already well resolved at coarse resolution and contains no rare or perturbation-induced populations to discover

## Inputs

- AnnData (h5ad) object with the batch-integrated CODAL topic compositions (ILR-transformed) for all wild-type and Tal1-/- chimeric batches
- Cell-level covariate metadata (sequencing batch, chimeric status, embryo mixing, collection day) and Tom+ reporter status
- Rough Leiden cluster assignment identifying the HE-prog-enriched cluster
- Marker gene list for mesodermal cell type matching: Emcn, Hoxb9, Hoxc8, Cdx4, Kdr, Sox17, Esam

## Outputs

- Leiden subcluster labels (A-E) for the Tom+ Tal1-/- HE-prog cells
- Marker expression table fig6_heprog_marker_expression_table.csv mapping subclusters to mesodermal cell types
- UMAP embeddings computed from ILR-transformed topic compositions (min_dist 0.1, negative_sample_rate 3)

## How to apply

First fit the CODAL topic model on the jointly modeled batches (63-topic model, 60 Bayesian hyperparameter tuning iterations over N_topics 35-80 with four technical covariates: sequencing batch, chimeric status, embryo mixing, collection day), then build a k-NN graph from the isometric log-ratio (ILR)-transformed topic compositions using Manhattan distance. Roughly Leiden-cluster the full integrated atlas and select the cluster that primarily contains HE-prog cells. Subset that cluster to Tom+ Tal1-/- cells and re-run the Leiden algorithm at high resolution to obtain subclusters A-E. Match each subcluster to mesodermal cell type populations via marker gene expression and shared topic latent-variable composition, tabulating markers (Emcn; Hoxb9/Hoxc8/Cdx4; Kdr/Sox17/Esam) into fig6_heprog_marker_expression_table.csv. Preprocessing upstream uses the scanpy standard workflow: filter cells with fewer than 400 counts and genes with fewer than 30 counts, normalize_total to target_sum 10000, log1p-transform, and select highly variable genes with a minimum dispersion threshold of 0.5.

## Related tools

- **CODAL/MIRA** (Variational topic model that disentangles the four technical covariates from biological effects, producing the latent space (ILR-transformed topic compositions) in which Leiden clustering is run) — https://github.com/cistrome/MIRA
- **scanpy** (Preprocessing (concatenate batches, 400/30-count filters, normalize_total target_sum 10000, log1p, HVG dispersion 0.5), neighbor graph, Leiden clustering, and UMAP) — https://scanpy.readthedocs.io/en/stable/
- **Leiden algorithm (leidenalg)** (Graph community detection used both for rough clustering of the integrated atlas and high-resolution subclustering of Tom+ Tal1-/- HE-prog cells)

## Examples

```
import scanpy as sc; sc.pp.neighbors(adata_heprog, use_rep='X_ilr_topics', metric='manhattan'); sc.tl.leiden(adata_heprog, resolution=1.5, key_added='heprog_subcluster')
```

## Evaluation signals

- The CODAL UMAP shows batch intermixing and cell-type separation comparable to or better than the scanpy PCA baseline
- High-resolution Leiden on the Tom+ Tal1-/- subset yields five distinct subclusters (A-E) with differing marker-expression profiles
- Each subcluster's marker profile (Emcn; Hoxb9/Hoxc8/Cdx4; Kdr/Sox17/Esam) is consistent with the mesodermal cell type annotations reported in Pijuan-Sala et al.
- Marker expression table fig6_heprog_marker_expression_table.csv contains expression values for all listed marker genes across subclusters A-E
- Subclustering is performed on the ILR-transformed CODAL topic compositions with Manhattan-distance k-NN graph, not on the scanpy PCA space

## Limitations

- Perturbation-induced or batch-confounded cell states found by subclustering should be validated using orthogonal biological evidence to rule out under-correction of technical effects
- CODAL does not remove the need for good experimental design; experimentalists must include appropriate controls (e.g. a control group of cells unaffected by the perturbation in both treatment and control batches)
- Further improvements may require better characterization of technical sources of bias in conjunction with the CODAL model

## Evidence

- [methods] ran the Leiden algorithm to obtain high-resolution subclusters: "ran the Leiden algorithm to obtain high-resolution subclusters"
- [methods] We then calculate distances between transformed cell latent variables using the Manhattan distance to create a k-nearest neighbors (k-NN) graph: "We then calculate distances between transformed cell latent variables using the Manhattan distance to create a k-nearest neighbors (k-NN) graph"
- [methods] cells with fewer than 400 counts and genes with fewer than 30 counts filtered out: "cells with fewer than 400 counts and genes with fewer than 30 counts filtered out"
- [discussion] the existence of new batch-confounded or perturbation-induced cell states should be validated using orthogonal biological evidence: "the existence of new batch-confounded or perturbation-induced cell states should be validated using orthogonal biological evidence"
- [readme] All MIRA topic models use the new CODAL algorithm for inference.: [evidence span withheld — anchor status: external_doc]
