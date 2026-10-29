---
name: louvain-graph-clustering
description: Use when clustering cells from a log1p-normalized AnnData object into
  graph communities — e.g., defining control cell states prior to NCFS gene weighting,
  running sslouvain with fixed control labels, or clustering integrated embeddings
  to test whether a method detects perturbation-unique responses.
when_to_use_negative:
- When the goal is batch or condition integration/correction — Louvain only partitions
  an existing neighbor graph; Seurat, Scanorama, or Harmony are the appropriate integration
  methods
- When no neighbor graph or dimensionality-reduced embedding has been computed — raw
  count matrices must first be normalized (log1p) and PCA-projected before Louvain
  is meaningful
- When a priori known cell-type labels exist and a supervised classifier is desired
  rather than unsupervised community detection
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scanpy
  role: provides sc.pp.neighbors and sc.tl.louvain for kNN graph construction and
    Louvain community detection on control, semi-supervised, and integrated embeddings
- name: icat-sc
  role: wraps Louvain and semi-supervised Louvain (sslouvain) clustering inside models.icat.cluster
    and optimize.optimize_louvain / optimize_sslouvain grid searches
  repo: https://github.com/BradhamLab/icat
- name: AnnData
  role: data structure holding the count matrix, treatment labels, and resulting louvain/sslouvain
    cluster columns
- name: Seurat
  role: produces an integrated embedding whose Louvain clustering is compared against
    ICAT sslouvain assignments
- name: Scanorama
  role: produces an integrated embedding whose Louvain clustering is compared against
    ICAT sslouvain assignments
- name: Harmony
  role: produces an integrated embedding whose Louvain clustering is compared against
    ICAT sslouvain assignments
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:bb3a10adfa7f0a000dc84ac6073348dc38a4f35e28fd0bb1719af5442a1e8cb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - AnnData
  - Harmony
  - Python
  - Scanorama
  - Scanpy
  - Seurat
  - icat-sc
  - numpy
  - scDrug
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- louvain-graph-based-clustering
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# louvain-graph-clustering

## Summary

Louvain modularity-based community detection on a single-cell kNN graph, used in the ICAT pipeline to define control-defined cell states, to drive semi-supervised (sslouvain) clustering of perturbed cells, and to partition Seurat/Scanorama/Harmony integrated embeddings for perturbation-response comparison.

## When to use

Use when clustering cells from a log1p-normalized AnnData object into graph communities — e.g., defining control cell states prior to NCFS gene weighting, running sslouvain with fixed control labels, or clustering integrated embeddings to test whether a method detects perturbation-unique responses.

## When NOT to use

- When the goal is batch or condition integration/correction — Louvain only partitions an existing neighbor graph; Seurat, Scanorama, or Harmony are the appropriate integration methods
- When no neighbor graph or dimensionality-reduced embedding has been computed — raw count matrices must first be normalized (log1p) and PCA-projected before Louvain is meaningful
- When a priori known cell-type labels exist and a supervised classifier is desired rather than unsupervised community detection

## Inputs

- log1p-normalized AnnData object of control and perturbed scRNA-seq counts (e.g., processed GSE164240 PMC dataset)
- PCA embedding of the expression matrix (sc.pp.pca) for neighbor graph construction
- control Louvain cluster labels (adata.obs.louvain / control_clusters) for the semi-supervised step
- integrated embeddings from Seurat, Scanorama, or Harmony for comparison clustering

## Outputs

- adata.obs['louvain'] cluster labels for control-defined cell states
- adata.obs['sslouvain'] semi-supervised cluster labels assigning perturbed cells to control states or new states
- per-cluster control vs. perturbed cell count tables for G-test / Fisher's exact enrichment analysis

## How to apply

Build a k-nearest-neighbor graph with scanpy's sc.pp.neighbors on the PCA embedding, then run sc.tl.louvain to partition cells by modularity. Optimize n_neighbors and resolution by grid search (min_neighbors=3, max_neighbors=50, neighbor_step=2; resolution 0.3–1.2 in steps of 0.02) maximizing the Calinski-Harabasz index on control cells only. In the ICAT workflow, the optimized control Louvain labels feed NCFS gene weighting, and the same (or separately optimized) n_neighbors and resolution parameters are reused for semi-supervised Louvain clustering, where control labels are held constant and perturbed cells are free to join any cluster; the final assignment lands in adata.obs['sslouvain']. Louvain is also run independently on Seurat, Scanorama, and Harmony integrated embeddings so per-cluster control-vs-perturbed counts can be tabulated and tested (G-test, post hoc Fisher's exact) against the ICAT sslouvain assignments.

## Related tools

- **scanpy** (provides sc.pp.neighbors and sc.tl.louvain for kNN graph construction and Louvain community detection on control, semi-supervised, and integrated embeddings)
- **icat-sc** (wraps Louvain and semi-supervised Louvain (sslouvain) clustering inside models.icat.cluster and optimize.optimize_louvain / optimize_sslouvain grid searches) — https://github.com/BradhamLab/icat
- **AnnData** (data structure holding the count matrix, treatment labels, and resulting louvain/sslouvain cluster columns)
- **Seurat** (produces an integrated embedding whose Louvain clustering is compared against ICAT sslouvain assignments)
- **Scanorama** (produces an integrated embedding whose Louvain clustering is compared against ICAT sslouvain assignments)
- **Harmony** (produces an integrated embedding whose Louvain clustering is compared against ICAT sslouvain assignments)

## Examples

```
sc.pp.neighbors(controls, n_neighbors=louvain_n); sc.tl.louvain(controls, resolution=louvain_r)
```

## Evaluation signals

- Optimized parameters maximize the Calinski-Harabasz index over the searched neighbor/resolution grid on control cells
- In sslouvain output, originally identified control labels remain constant while perturbed cells are reassigned; out.obs['sslouvain'].unique() returns the expected cluster set
- On the PMC dataset, only the ICAT sslouvain clustering resolves both a depleted and an enriched perturbation-response cluster (sm50/pks2 states) under G-test / Fisher's exact testing, whereas Seurat/Scanorama/Harmony Louvain clusterings do not
- Cluster assignments are reproducible when rerun with the same optimized n_neighbors and resolution values

## Limitations

- Standard Louvain clustering on integrated embeddings from Seurat, Scanorama, and Harmony failed to identify both sm50/pks2 PMC responses to MK-886 — only ICAT's control-guided semi-supervised variant resolved the perturbation-unique response
- Louvain results are sensitive to n_neighbors and resolution; the README recommends grid-search optimization of these hyperparameters before running the full ICAT workflow
- Louvain alone does not compute UMAP/tSNE or integrate conditions; visualization requires passing the upweighted matrix (adata.obsm['X_icat']) to downstream embedding algorithms

## Evidence

- [intro] The method works by first identifying a set of conrol-defined cell states by performing unsupervised clustering.: "The method works by first identifying a set of conrol-defined cell states by performing unsupervised clustering."
- [intro] Semi-supervised clustering fixes control labels and frees experimental cells: "perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster"
- [methods] scanpy/AnnData dependency: "ICAT makes heavy use of the excellent `scanpy` library along with the associated `AnnData` data structure."
- [readme] Calinski-Harabasz grid optimization of Louvain parameters: "optimizing the Calinski-Harabasz Index over a grid of `n` and `r` values"
- [readme] Concrete scanpy Louvain invocation: "sc.pp.neighbors(controls, n_neighbors=louvain_n)
sc.tl.louvain(controls, resolution=louvain_r)"
- [readme] ICAT cluster call returning sslouvain: "out = model.cluster(adata, adata.obs['treatment'])
    print(out.obs['sslouvain'].unique())"
