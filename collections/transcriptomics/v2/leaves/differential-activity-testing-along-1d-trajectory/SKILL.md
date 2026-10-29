---
name: differential-activity-testing-along-1d-trajectory
description: Use when single-cell RNA-seq data have been ordered along a 1D pseudotime
  trajectory (e.g. monocle3 learn_graph/order_cells from DN to SP thymocytes) and
  you need genes with biased expression along that axis rather than cluster-marker
  DEGs.
when_to_use_negative:
- 'Comparing multiple conditions (e.g. wild-type vs knockout): the authors note such
  comparisons cannot easily be conducted with singleCellHaystack.'
- You need results integrated into a cluster-oriented scRNA-seq analysis — cluster-based
  DEG prediction approaches are more straightforward for that.
- Spatial transcriptomics where raw speed matters most — SPARK-X is faster, though
  singleCellHaystack is more sensitive.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack (R)
  role: 'Core method: predicts differentially active features along the 1D pseudotime
    input space via Kullback-Leibler Divergence; provides haystack, top_features and
    cluster_genes'
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: singleCellHaystack (Python)
  role: Python implementation of singleCellHaystack >= 1.0.0 for atlas-scale datasets
    (hs.haystack, res.top_features); hierarchical cluster_genes not yet implemented
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: Seurat
  role: 'Preprocessing: normalize counts, identify highly variable genes, scale data,
    run PCA; subset DN/SP T cell clusters before monocle3 conversion'
- name: monocle3
  role: 'Trajectory inference: learn_graph and order_cells to infer DN-to-SP pseudotime
    used as the 1D coordinate input to singleCellHaystack'
  repo: https://cole-trapnell-lab.github.io/monocle3
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:00c8d75c8f59a3aa7eaef9160e6a74338af6479cefb2a00c1801adc4cb64e6d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - Seurat
  - monocle3
  - singleCellHaystack (Python)
  - singleCellHaystack (R)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# differential-activity-testing-along-1d-trajectory

## Summary

Apply singleCellHaystack (R/CRAN or Python) to a single-cell expression matrix with 1D pseudotime coordinates to predict genes (or other features) whose activity is non-randomly positioned along the trajectory, without cluster-based DE comparisons. Top-ranked DEGs can then be partitioned into expression-pattern modules.

## When to use

Use when single-cell RNA-seq data have been ordered along a 1D pseudotime trajectory (e.g. monocle3 learn_graph/order_cells from DN to SP thymocytes) and you need genes with biased expression along that axis rather than cluster-marker DEGs.

## When NOT to use

- Comparing multiple conditions (e.g. wild-type vs knockout): the authors note such comparisons cannot easily be conducted with singleCellHaystack.
- You need results integrated into a cluster-oriented scRNA-seq analysis — cluster-based DEG prediction approaches are more straightforward for that.
- Spatial transcriptomics where raw speed matters most — SPARK-X is faster, though singleCellHaystack is more sensitive.

## Inputs

- Processed scRNA-seq expression matrix (e.g. Seurat-normalized/scaled counts from a Tabula Muris thymus count matrix)
- 1D pseudotime coordinates per cell (from monocle3 learn_graph/order_cells)
- Optionally a Seurat or Scanpy/h5ad object carrying the processed data and pseudotime

## Outputs

- Haystack result table of DEGs ranked by significance for differential activity along the trajectory
- Top-N feature list (e.g. res.top_features(n=10); top 1000 DEGs via top_features)
- Gene-to-module assignment table (thymus_deg_modules.csv) from k-means clustering (k=6)
- Per-module mean expression-vs-pseudotime plots

## How to apply

First preprocess counts in Seurat (normalize, identify highly variable genes, scale, PCA) and infer pseudotime with monocle3 (learn_graph then order_cells from the DN to SP trajectory in the Tabula Muris thymus case). Run singleCellHaystack v1.0.0+ on the processed expression matrix using the 1D pseudotime values as the input coordinates — the method uses Kullback-Leibler Divergence to detect features non-randomly positioned in the input space, uses continuous activity levels instead of a hard detection threshold, and cross-validates spline flexibility. Extract the top 1000 DEGs with top_features, then cluster their scaled expression profiles with k-means (k=6, fixed seed, e.g. via cluster_genes in R) and save the gene-to-module table (thymus_deg_modules.csv). Plot each module's mean expression along pseudotime to confirm modules qualitatively match the expected expression patterns (Fig 3B-C).

## Related tools

- **singleCellHaystack (R)** (Core method: predicts differentially active features along the 1D pseudotime input space via Kullback-Leibler Divergence; provides haystack, top_features and cluster_genes) — https://github.com/alexisvdb/singleCellHaystack
- **singleCellHaystack (Python)** (Python implementation of singleCellHaystack >= 1.0.0 for atlas-scale datasets (hs.haystack, res.top_features); hierarchical cluster_genes not yet implemented) — https://github.com/ddiez/singleCellHaystack-py
- **Seurat** (Preprocessing: normalize counts, identify highly variable genes, scale data, run PCA; subset DN/SP T cell clusters before monocle3 conversion)
- **monocle3** (Trajectory inference: learn_graph and order_cells to infer DN-to-SP pseudotime used as the 1D coordinate input to singleCellHaystack) — https://cole-trapnell-lab.github.io/monocle3

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- haystack result returns a ranked table of DEGs with significance scores for the 1D pseudotime input; top_features(n) extracts the top-ranked genes (top 1000 in the thymus workflow).
- k-means clustering (k=6, fixed seed) of the top 1000 DEGs yields a gene-to-module table that qualitatively reproduces the 6 expression-pattern modules of Fig 3B-C.
- Per-module mean-expression-vs-pseudotime plots show distinct, biologically coherent patterns (DN to CD4+/CD8+ SP transition).
- Input sanity check: coordinates passed to haystack are 1D pseudotime values covering the DN-to-SP trajectory, not a 2D embedding.
- Scaling/efficiency check: on large datasets (e.g. MOCA ~100k cells) the run should complete in minutes-to-tens-of-minutes given the reported up to tenfold speedup.

## Limitations

- Comparisons between multiple conditions (e.g. wild-type and knockout) cannot be easily conducted.
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses, compared to cluster-based DEG prediction approaches.
- The Python implementation is in beta: weights.advanced.Q, the seeding grid method, and hierarchical clustering for cluster_genes are not yet available.
- For spatial data, SPARK-X is faster despite singleCellHaystack's better sensitivity.

## Evidence

- [abstract] Handles 1D trajectories as coordinate input: "it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates"
- [readme] Method basis: KLD over non-random feature positioning: [evidence span withheld — anchor status: external_doc]
- [discussion] Rationale for continuous activity levels over binary detection: "the use of a hard threshold can result in an overestimation of the significance of relatively modest differences in expression"
- [methods] Python API for running haystack and extracting top features: "res = hs.haystack(adata, basis="pca")
res.top_features(n=10)"
- [readme] Python package targets atlas-scale datasets and documents missing features: "This package is currently in beta. The most important functionality in the R package works, but some features are not yet available."
