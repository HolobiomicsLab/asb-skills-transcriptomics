---
name: spatial-variable-gene-pvalue-interpretation
description: Use when comparing SVG calls from two methods (e.g., singleCellHaystack
  vs SPARK-X) on the same 2D spatial coordinates and count matrix, to identify high-sensitivity
  hits one method ranks poorly and produce a merged rank/p-value comparison table.
when_to_use_negative:
- The goal is multi-condition comparison (e.g., wild-type vs knockout) — the article
  states such comparisons 'cannot be easily conducted' with singleCellHaystack
- Results must feed a cluster-oriented scRNA-seq analysis — the article notes singleCellHaystack
  results are 'not as straightforward to incorporate' into cluster-based workflows
- Runtime is the binding constraint on large spatial data — SPARK-X alone is the faster
  method, though at lower sensitivity
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: Python implementation (hs.haystack) run on the AnnData object with 2D spatial
    coordinates to produce SVG p-values and rankings; the sensitive of the two methods
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: SPARK-X
  role: Second SVG method run on the same expression matrix and coordinates to provide
    the comparison ranking; noted as the fastest method for spatial transcriptomics
- name: scanpy
  role: Loads the h5ad count matrix and spatial coordinates into the AnnData object
    consumed by hs.haystack
- name: singleCellHaystack (R)
  role: R/CRAN package alternative implementation (version 1.0.0+) of the same KL-divergence-based
    SVG prediction
  repo: https://github.com/alexisvdb/singleCellHaystack
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:8807800f76a55e202917f1ec960024b2535606dd1175af088d9a1f286ce9d701
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ddiez/singleCellHaystack-py
  tools:
  - Python
  - SPARK-X
  - scanpy
  - singleCellHaystack
  - singleCellHaystack (R)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# spatial-variable-gene-pvalue-interpretation

## Summary

Run and cross-compare spatially variable gene (SVG) rankings from singleCellHaystack and SPARK-X on the same spatial transcriptomics dataset, then interpret rank and p-value discrepancies (e.g., Ttr at 58th, p=1.1e-105 in singleCellHaystack vs 11,881st, p=0.0087 in SPARK-X). singleCellHaystack is more sensitive but slower; SPARK-X is fastest.

## When to use

Use when comparing SVG calls from two methods (e.g., singleCellHaystack vs SPARK-X) on the same 2D spatial coordinates and count matrix, to identify high-sensitivity hits one method ranks poorly and produce a merged rank/p-value comparison table.

## When NOT to use

- The goal is multi-condition comparison (e.g., wild-type vs knockout) — the article states such comparisons 'cannot be easily conducted' with singleCellHaystack
- Results must feed a cluster-oriented scRNA-seq analysis — the article notes singleCellHaystack results are 'not as straightforward to incorporate' into cluster-based workflows
- Runtime is the binding constraint on large spatial data — SPARK-X alone is the faster method, though at lower sensitivity

## Inputs

- AnnData object (.h5ad) with a gene-by-spot count matrix (e.g., 10x Visium posterior1 mouse brain, ~16,596 genes)
- 2D spatial coordinates array (spot positions) for the same tissue section
- Reference list of expected top SVGs for validation (e.g., Ttr, Mbp, Calb2, Plp1, Arpp19, Ccdc36)

## Outputs

- Full per-gene rank and p-value table for singleCellHaystack (res.top_features plus full ranked table)
- Per-gene p-values and ranks from SPARK-X on the same matrix and coordinates
- Merged rank comparison table (e.g., haystack_sparkx_rank_comparison.csv) with rank and p-value columns for both methods

## How to apply

Load the spatial dataset (e.g., 10x Visium posterior1 mouse brain) as an AnnData object via sc.read_h5ad, using the h5ad count matrix and 2D spatial coordinates. Run the Python singleCellHaystack implementation with hs.haystack on the AnnData (it uses Kullback-Leibler Divergence to find features whose activity is non-randomly positioned in the 2D coordinate space, accepting 2-3D spatial coordinates as input), and extract rankings with res.top_features(n=10) plus the full ranked table. Run SPARK-X on the same expression matrix and spatial coordinates to obtain per-gene p-values and ranks. Merge both methods' rank and p-value columns per gene into a single table (e.g., haystack_sparkx_rank_comparison.csv) and check reported discrepancies such as Ttr (58th, p=1.1e-105 vs 11,881st, p=0.0087), Arpp19 (18th SPARK-X vs 983rd singleCellHaystack), and Ccdc36 (73rd vs 1,585th). Interpretation: singleCellHaystack has better sensitivity for spatial data 'despite relatively longer runtimes', while SPARK-X is the fastest available method, so large rank gaps should be read as sensitivity differences rather than errors by default.

## Related tools

- **singleCellHaystack** (Python implementation (hs.haystack) run on the AnnData object with 2D spatial coordinates to produce SVG p-values and rankings; the sensitive of the two methods) — https://github.com/ddiez/singleCellHaystack-py
- **SPARK-X** (Second SVG method run on the same expression matrix and coordinates to provide the comparison ranking; noted as the fastest method for spatial transcriptomics)
- **scanpy** (Loads the h5ad count matrix and spatial coordinates into the AnnData object consumed by hs.haystack)
- **singleCellHaystack (R)** (R/CRAN package alternative implementation (version 1.0.0+) of the same KL-divergence-based SVG prediction) — https://github.com/alexisvdb/singleCellHaystack

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- The merged CSV contains rank and p-value columns for both singleCellHaystack and SPARK-X across all genes (e.g., 16,596 genes in posterior1)
- Ttr's ranks match the reported values: 58th with p=1.1e-105 in singleCellHaystack and 11,881st with p=0.0087 in SPARK-X
- Arpp19 ranks as 18th in SPARK-X vs 983rd in singleCellHaystack, and Ccdc36 as 73rd vs 1,585th
- Known top SVG biology hits (Mbp, Calb2, Plp1) appear among top-ranked genes in at least the singleCellHaystack ranking
- Both methods were run on the identical expression matrix and 2D spatial coordinates, so rank differences are attributable to the methods rather than input mismatch

## Limitations

- singleCellHaystack has 'relatively longer runtimes' than SPARK-X for spatial transcriptomics
- Comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses; multi-condition and cluster-integration features are planned future work
- The Python package is in beta: weights.advanced.Q, the seeding grid method, and hierarchical clustering for cluster_genes are not yet available

## Evidence

- [discussion] For spatial transcriptomics the fastest method available is SPARK-X.: "For spatial transcriptomics the fastest method available is SPARK-X."
- [discussion] The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes.: "The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes."
- [readme] singleCellHaystack uses Kullback-Leibler Divergence to find features that have patterns of activity in subsets of samples that are non-randomly positioned inside any input space.: [evidence span withheld — anchor status: external_doc]
- [abstract] it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates: "it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates"
- [methods] res = hs.haystack(adata, basis="pca"): [evidence span withheld — anchor status: not_found]
