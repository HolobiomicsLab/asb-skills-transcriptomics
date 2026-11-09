---
name: high-definition-spatial-transcriptomics-analysis
description: Use when you have a spatial transcriptomics count matrix (e.g., mouse
  olfactory bulb HDST from the Broad Single Cell Portal, SCP420) with 2D spatial coordinates
  and need to detect spatially variable genes or reconcile rank disagreements between
  SVG callers.
when_to_use_negative:
- Comparisons between multiple conditions (e.g., wild-type and knockout) are needed
  — singleCellHaystack cannot easily conduct multi-condition comparisons
- Results must integrate with cluster-oriented scRNA-seq analysis — singleCellHaystack
  output is not straightforward to incorporate into cluster-based DEG workflows
- Input genes are expected to be identified by cluster-vs-cluster differential expression
  rather than non-random spatial patterning of activity
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: 'Primary SVG caller: runs on continuous expression with 2D spatial coordinates
    as input positions, without detection threshold or clustering, using Kullback-Leibler
    Divergence'
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: singleCellHaystack-py
  role: Python implementation used to run haystack on AnnData objects and extract
    top_features rankings
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: SPARK-X
  role: Fastest spatial transcriptomics method used as the comparison caller to obtain
    per-gene spatial variability p-values and ranks for cross-validation
- name: Python
  role: Environment for loading HDST data with scanpy, running both callers, and writing
    the comparison CSV
- name: R
  role: Environment for the R package implementation of singleCellHaystack (CRAN version
    1.0.0 and higher)
  repo: https://github.com/alexisvdb/singleCellHaystack
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:d63ff67faa513a6dfe0f7784f79e6efc58a57ad7d16c23c5e77db65b5ca9f587
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - Python
  - R
  - SPARK-X
  - singleCellHaystack
  - singleCellHaystack-py
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# high-definition-spatial-transcriptomics-analysis

## Summary

A workflow for identifying and cross-validating spatially variable genes (SVGs) in high-definition spatial transcriptomics data by running singleCellHaystack and SPARK-X on the same count matrix and 2D spatial coordinates, then comparing per-gene ranks and p-values side by side.

## When to use

Use when you have a spatial transcriptomics count matrix (e.g., mouse olfactory bulb HDST from the Broad Single Cell Portal, SCP420) with 2D spatial coordinates and need to detect spatially variable genes or reconcile rank disagreements between SVG callers.

## When NOT to use

- Comparisons between multiple conditions (e.g., wild-type and knockout) are needed — singleCellHaystack cannot easily conduct multi-condition comparisons
- Results must integrate with cluster-oriented scRNA-seq analysis — singleCellHaystack output is not straightforward to incorporate into cluster-based DEG workflows
- Input genes are expected to be identified by cluster-vs-cluster differential expression rather than non-random spatial patterning of activity

## Inputs

- HDST spatial transcriptomics count matrix from the Broad Single Cell Portal (SCP420), mouse olfactory bulb
- 2D spatial coordinates for each spot
- AnnData (.h5ad) object processed with scanpy

## Outputs

- Per-gene SVG p-values and rankings from singleCellHaystack (res.top_features)
- Per-gene spatial variability p-values and rankings from SPARK-X
- Side-by-side rank and p-value comparison table (svg_rank_comparison.csv)

## How to apply

Load the HDST count matrix and spatial coordinates into an AnnData object with scanpy (sc.read_h5ad). Run singleCellHaystack on all genes using the 2D spatial coordinates as input positions — the method uses Kullback-Leibler Divergence to find features with non-random activity in the input space and does not require a detection threshold or prior clustering of cells. Run SPARK-X on the same count matrix and coordinates to obtain per-gene spatial variability p-values and ranks (SPARK-X is the faster method, but singleCellHaystack has better sensitivity, making it the best alternative despite longer runtimes). Extract each method's top-features ranking (res.top_features), locate key genes such as Gm42418 and Gm10925 in both ranked lists, and write a side-by-side rank and p-value comparison (e.g., to svg_rank_comparison.csv). In the HDST reference result, Gm42418 is the top-scoring SVG by both methods, while Gm10925 is the only discrepant gene: ranked 13th by singleCellHaystack (p=2.7e-39) but only 105th by SPARK-X (p=0.013).

## Related tools

- **singleCellHaystack** (Primary SVG caller: runs on continuous expression with 2D spatial coordinates as input positions, without detection threshold or clustering, using Kullback-Leibler Divergence) — https://github.com/alexisvdb/singleCellHaystack
- **singleCellHaystack-py** (Python implementation used to run haystack on AnnData objects and extract top_features rankings) — https://github.com/ddiez/singleCellHaystack-py
- **SPARK-X** (Fastest spatial transcriptomics method used as the comparison caller to obtain per-gene spatial variability p-values and ranks for cross-validation)
- **Python** (Environment for loading HDST data with scanpy, running both callers, and writing the comparison CSV)
- **R** (Environment for the R package implementation of singleCellHaystack (CRAN version 1.0.0 and higher)) — https://github.com/alexisvdb/singleCellHaystack

## Examples

```
import scanpy as sc; import singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Gm42418 appears as the top-scoring spatially variable gene in BOTH singleCellHaystack and SPARK-X ranked lists
- Gm10925 is ranked 13th by singleCellHaystack (p=2.7e-39) and 105th by SPARK-X (p=0.013) — the only discrepant gene between the two methods
- singleCellHaystack was run on all genes using 2D spatial coordinates without applying a detection threshold or clustering cells
- svg_rank_comparison.csv contains side-by-side rank and p-value columns for both methods over the same gene set
- top_features rankings are extractable from each result object (res.top_features(n=10))

## Limitations

- singleCellHaystack has relatively longer runtimes compared to SPARK-X for spatial transcriptomics; SPARK-X is the fastest available method
- Comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted with singleCellHaystack
- singleCellHaystack results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses
- The Python implementation (singleCellHaystack-py) is in beta: weights.advanced.Q, the seeding grid method, and hierarchical clustering for cluster_genes are not yet available
- Using a hard binary detection threshold (as in the original version) can overestimate significance of relatively modest expression differences — the updated method uses continuous activity levels instead

## Evidence

- [readme] singleCellHaystack uses Kullback-Leibler Divergence to find features that have patterns of activity in subsets of samples that are non-randomly positioned inside any input space.: [evidence span withheld — anchor status: external_doc]
- [discussion] SPARK-X is fastest for spatial transcriptomics: "For spatial transcriptomics the fastest method available is SPARK-X."
- [discussion] sensitivity vs runtime trade-off in spatial data: "The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes."
- [discussion] multi-condition comparison limitation: "One is that comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted."
- [methods] Python invocation of haystack on AnnData: "res = hs.haystack(adata, basis="pca")"
- [methods] top features extraction: "res.top_features(n=10)"
