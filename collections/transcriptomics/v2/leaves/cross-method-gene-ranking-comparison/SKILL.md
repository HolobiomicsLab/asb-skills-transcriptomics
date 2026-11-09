---
name: cross-method-gene-ranking-comparison
description: Use when you have a spatial transcriptomics dataset (e.g. 10x Visium
  or HDST h5ad with 2D spatial coordinates) and need to compare which top-scoring
  spatially variable genes one method detects that another misses.
when_to_use_negative:
- When you need multi-condition comparisons (e.g. wild-type vs knockout) — singleCellHaystack
  explicitly does not support these yet
- When runtime on atlas-scale spatial data is the binding constraint — SPARK-X alone
  is the faster option (singleCellHaystack has 'relatively longer runtimes compared
  to SPARK-X')
- When your data lacks spatial coordinates or you only have cluster labels — singleCellHaystack
  requires an input space (1D/2D/3D coordinates or latent space), not cluster comparisons
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0203
tools:
- name: singleCellHaystack
  role: Python/R method run on all genes with 2D spatial coordinates as input to produce
    KL-divergence-based SVG p-values and top-features ranking
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: SPARK-X
  role: Fast spatial transcriptomics method run on the same expression matrix and
    coordinates to produce the comparison SVG p-values and ranks
- name: scanpy
  role: Loads the h5ad count matrix and spatial coordinates into an AnnData object
    (sc.read_h5ad) prior to running both methods
- name: R
  role: Language for running SPARK-X (and the R singleCellHaystack package) on the
    shared expression matrix
  repo: https://github.com/alexisvdb/singleCellHaystack
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:159199b1ff6a7fe769c86af2184150ab16851b0ac3421c0f4871ff03be2ec82f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ddiez/singleCellHaystack-py
  tools:
  - Python
  - R
  - SPARK-X
  - scanpy
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# cross-method-gene-ranking-comparison

## Summary

Run two spatially variable gene (SVG) methods — singleCellHaystack and SPARK-X — on the same spatial transcriptomics expression matrix and 2D coordinates, merge per-gene ranks and p-values, and quantify rank discrepancies to reveal sensitivity differences (e.g. singleCellHaystack's higher sensitivity for spatially restricted genes like Ttr).

## When to use

Use when you have a spatial transcriptomics dataset (e.g. 10x Visium or HDST h5ad with 2D spatial coordinates) and need to compare which top-scoring spatially variable genes one method detects that another misses.

## When NOT to use

- When you need multi-condition comparisons (e.g. wild-type vs knockout) — singleCellHaystack explicitly does not support these yet
- When runtime on atlas-scale spatial data is the binding constraint — SPARK-X alone is the faster option (singleCellHaystack has 'relatively longer runtimes compared to SPARK-X')
- When your data lacks spatial coordinates or you only have cluster labels — singleCellHaystack requires an input space (1D/2D/3D coordinates or latent space), not cluster comparisons

## Inputs

- Count matrix and 2D spatial coordinates from a spatial transcriptomics dataset (e.g. 10x Visium posterior1 mouse brain via SeuratData, or mouse olfactory bulb HDST from Broad Single Cell Portal SCP420) as an AnnData h5ad file
- Gene-level expression matrix shared by both methods (all genes, e.g. 16,596 genes in posterior1)

## Outputs

- Per-gene ranked feature lists and p-values from singleCellHaystack (res.top_features plus full ranked table)
- Per-gene spatial variability p-values and ranks from SPARK-X
- Merged rank/p-value comparison CSV (e.g. haystack_sparkx_rank_comparison.csv or svg_rank_comparison.csv) with rank and p-value columns for both methods

## How to apply

Load the count matrix and 2D spatial coordinates into an AnnData object with scanpy (sc.read_h5ad). Run singleCellHaystack on all genes using the 2D spatial coordinates as input positions (hs.haystack on the AnnData object) and extract the ranked feature list with res.top_features(n=10) plus the full ranked table; run SPARK-X on the same expression matrix and coordinates to obtain per-gene spatial variability p-values and ranks. Merge the two methods' rankings and p-values per gene into a side-by-side table and save as CSV (e.g. haystack_sparkx_rank_comparison.csv). Judge correctness by checking that known discrepancy genes match reported ranks: in the Visium posterior1 mouse brain, Ttr ranked 58th by singleCellHaystack (p=1.1e-105) but 11,881st by SPARK-X (p=0.0087); in HDST, Gm42418 was top by both methods while Gm10925 was 13th by singleCellHaystack (p=2.7e-39) vs 105th by SPARK-X (p=0.013). Rationale: singleCellHaystack uses Kullback-Leibler Divergence on continuous activity levels without clustering or strong distributional assumptions, giving better sensitivity despite longer runtimes than SPARK-X, the fastest spatial method.

## Related tools

- **singleCellHaystack** (Python/R method run on all genes with 2D spatial coordinates as input to produce KL-divergence-based SVG p-values and top-features ranking) — https://github.com/ddiez/singleCellHaystack-py
- **SPARK-X** (Fast spatial transcriptomics method run on the same expression matrix and coordinates to produce the comparison SVG p-values and ranks)
- **scanpy** (Loads the h5ad count matrix and spatial coordinates into an AnnData object (sc.read_h5ad) prior to running both methods)
- **R** (Language for running SPARK-X (and the R singleCellHaystack package) on the shared expression matrix) — https://github.com/alexisvdb/singleCellHaystack

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Output CSV contains rank and p-value columns for both singleCellHaystack and SPARK-X
- Ranks of known discrepancy genes match reported values: Ttr 58th (p=1.1e-105) vs 11,881st (p=0.0087) in Visium posterior1; Gm10925 13th (p=2.7e-39) vs 105th (p=0.013) in HDST
- Gm42418 is the top-scoring SVG by both methods in the HDST mouse olfactory bulb dataset
- Top hits in Visium posterior1 include Mbp, Calb2, Plp1, Ttr (singleCellHaystack) and Arpp19 (18th), Ccdc36 (73rd) (SPARK-X)
- Both methods were run on all genes of the same expression matrix with identical 2D spatial coordinates, so rank positions are directly comparable

## Limitations

- singleCellHaystack has relatively longer runtimes compared to SPARK-X for spatial transcriptomics, though its better sensitivity makes it the best alternative
- Comparisons between multiple conditions (e.g. wild-type and knockout) cannot be easily conducted with singleCellHaystack
- The Python singleCellHaystack implementation is in beta: weights.advanced.Q, the seeding method for grid points, and hierarchical clustering for cluster_genes are not yet available
- singleCellHaystack results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses

## Evidence

- [abstract] singleCellHaystack can handle 1D trajectories, 2-3D spatial coordinates, and higher-dimensional latent spaces as input coordinates: "it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates"
- [discussion] Limitation: relatively longer runtimes compared to SPARK-X for spatial transcriptomics: "The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes."
- [methods] Python example invocation of singleCellHaystack on AnnData: "res = hs.haystack(adata, basis="pca")"
- [discussion] SPARK-X is the fastest spatial transcriptomics method: "For spatial transcriptomics the fastest method available is SPARK-X."
- [readme] README example: read h5ad with scanpy and extract top features: "adata = sc.read_h5ad("data.h5ad")

[... process adata object ...]

res = hs.haystack(adata, basis="pca")
res.top_features(n=10)"
