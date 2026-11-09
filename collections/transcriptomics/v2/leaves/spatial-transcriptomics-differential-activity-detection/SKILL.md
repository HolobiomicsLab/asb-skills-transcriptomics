---
name: spatial-transcriptomics-differential-activity-detection
description: Use when you have a spatial transcriptomics dataset (e.g., 10x Visium)
  with an expression matrix and 2D spatial coordinates, and need to rank spatially
  variable genes cluster-independently or compare rankings across SVG methods.
when_to_use_negative:
- You need multi-condition comparisons (e.g., wild-type vs knockout) — singleCellHaystack
  cannot easily conduct comparisons between multiple conditions
- You need to integrate results into a cluster-oriented scRNA-seq analysis — the authors
  note results are not straightforward to incorporate into cluster-based DEG workflows
- Runtime is the dominant constraint for large spatial data — SPARK-X is faster (singleCellHaystack
  has relatively longer runtimes despite better sensitivity)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: singleCellHaystack (Python package)
  role: 'Primary method: computes KLD-based differential activity scores for all genes
    against 2D spatial coordinates; run via hs.haystack on the AnnData object'
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: singleCellHaystack (R package)
  role: R implementation available from CRAN (version >= 1.0.0) for the same differential
    activity detection, with spatial Visium and Slide-seq V2 vignettes
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: scanpy
  role: Loads the spatial dataset as an AnnData object from h5ad and prepares the
    expression matrix and spatial coordinates as haystack input
- name: SPARK-X
  role: 'Comparison method: fastest available spatial transcriptomics method, run
    on the same matrix/coordinates to obtain SVG p-values and ranks for benchmarking
    sensitivity'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:1be816851cb6157ea481c2191e0ea0f7338cf13bf6987cec0f303726b7c0c833
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ddiez/singleCellHaystack-py
  tools:
  - Python
  - SPARK-X
  - scanpy
  - singleCellHaystack (Python package)
  - singleCellHaystack (R package)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# spatial-transcriptomics-differential-activity-detection

## Summary

Detect spatially variable genes (differentially active features) in spatial transcriptomics data using singleCellHaystack, which scores features via Kullback-Leibler Divergence against 2D/3D spatial coordinates without clustering cells, and benchmark its sensitivity against SPARK-X.

## When to use

Use when you have a spatial transcriptomics dataset (e.g., 10x Visium) with an expression matrix and 2D spatial coordinates, and need to rank spatially variable genes cluster-independently or compare rankings across SVG methods.

## When NOT to use

- You need multi-condition comparisons (e.g., wild-type vs knockout) — singleCellHaystack cannot easily conduct comparisons between multiple conditions
- You need to integrate results into a cluster-oriented scRNA-seq analysis — the authors note results are not straightforward to incorporate into cluster-based DEG workflows
- Runtime is the dominant constraint for large spatial data — SPARK-X is faster (singleCellHaystack has relatively longer runtimes despite better sensitivity)

## Inputs

- AnnData object (.h5ad) with spatial transcriptomics count matrix (e.g., 10x Visium posterior1 mouse brain)
- 2D spatial coordinates of spots/cells (e.g., tissue_positions spatial coords)
- Optionally a PCA representation (basis="pca") of the expression data

## Outputs

- Ranked list of spatially variable genes with p-values (res.top_features(n=10) and full ranked table)
- Merged per-gene rank/p-value comparison table across methods (e.g., haystack_sparkx_rank_comparison.csv)

## How to apply

Load the spatial dataset as an AnnData object with scanpy (sc.read_h5ad) so it contains the count/expression matrix and the 2D spatial coordinates. Run the Python singleCellHaystack implementation with hs.haystack on the AnnData object, supplying the 2D spatial coordinates as the input space (the method uses Kullback-Leibler Divergence to find features with non-random activity patterns and accepts continuous features, so no binarization threshold is needed); optionally pass basis="pca" to run in a reduced space. Extract ranked features with res.top_features(n=10) and the full ranked table of p-values per gene. For method comparison, run SPARK-X on the same expression matrix and coordinates, merge per-gene ranks and p-values from both methods, and inspect discrepancies (e.g., Ttr ranked 58th by singleCellHaystack at p=1.1e-105 vs 11,881st by SPARK-X at p=0.0087 in Visium posterior1 mouse brain). Validate that the merged table contains rank and p-value columns for both methods and that known hit genes match their reported ranks.

## Related tools

- **singleCellHaystack (Python package)** (Primary method: computes KLD-based differential activity scores for all genes against 2D spatial coordinates; run via hs.haystack on the AnnData object) — https://github.com/ddiez/singleCellHaystack-py
- **singleCellHaystack (R package)** (R implementation available from CRAN (version >= 1.0.0) for the same differential activity detection, with spatial Visium and Slide-seq V2 vignettes) — https://github.com/alexisvdb/singleCellHaystack
- **scanpy** (Loads the spatial dataset as an AnnData object from h5ad and prepares the expression matrix and spatial coordinates as haystack input)
- **SPARK-X** (Comparison method: fastest available spatial transcriptomics method, run on the same matrix/coordinates to obtain SVG p-values and ranks for benchmarking sensitivity)

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- res.top_features(n=10) returns a ranked feature table and the full ranked table includes p-values for all genes (16,596 genes in the posterior1 benchmark)
- Known spatial marker genes are recovered: Ttr, Mbp, Calb2, Plp1 appear among top hits
- Merged comparison CSV contains rank and p-value columns for both singleCellHaystack and SPARK-X
- Specific rank checks pass: Ttr 58th (p=1.1e-105) by singleCellHaystack vs 11,881st (p=0.0087) by SPARK-X; Arpp19 18th SPARK-X vs 983rd singleCellHaystack; Ccdc36 73rd vs 1585th

## Limitations

- Comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses
- Relatively longer runtimes compared to SPARK-X for spatial transcriptomics, though singleCellHaystack has better sensitivity
- The Python package is in beta: weights.advanced.Q, the seeding method for grid points, and the hierarchical clustering method for cluster_genes are not yet available

## Evidence

- [readme] singleCellHaystack uses Kullback-Leibler Divergence to find features that have patterns of activity in subsets of samples that are non-randomly positioned inside any input space (1D, 2D, or >2D).: [evidence span withheld — anchor status: not_found]
- [abstract] it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates: "it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates"
- [discussion] For spatial transcriptomics the fastest method available is SPARK-X.: "For spatial transcriptomics the fastest method available is SPARK-X."
- [discussion] The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes.: "The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes."
- [readme] adata = sc.read_h5ad("data.h5ad") ... res = hs.haystack(adata, basis="pca") ... res.top_features(n=10): [evidence span withheld — anchor status: not_found]
