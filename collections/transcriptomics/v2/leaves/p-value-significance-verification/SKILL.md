---
name: p-value-significance-verification
description: Use when two SVG/DEG detection methods have been run on the same spatial
  dataset and you must verify and report which features agree or show discrepant ranks
  and p-values between them.
when_to_use_negative:
- When only one method's output is available — the skill requires matched ranked feature
  lists and p-values from both singleCellHaystack and SPARK-X on the same data.
- When the analysis goal is multi-condition comparison (e.g. wild-type vs knockout)
  — the paper notes such comparisons 'cannot be easily conducted' with singleCellHaystack
  in its current version.
- When a cluster-oriented DEG analysis is required — the paper states singleCellHaystack
  results are 'not as straightforward to incorporate into a cluster-oriented scRNA-seq
  analysis'.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: Produces the per-gene p-values and ranked feature list on 2D spatial coordinates
    (Python implementation hs.haystack, basis='pca'); its higher sensitivity surfaces
    discrepant hits like Gm10925.
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: singleCellHaystack-py
  role: Python implementation (PyPI version 0.0.5) used to run haystack on scanpy
    AnnData objects and extract top_features rankings for comparison against SPARK-X.
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: SPARK-X
  role: Fastest spatial transcriptomics method; supplies the second per-gene p-value/rank
    list that the singleCellHaystack results are verified against.
- name: Python
  role: Environment for loading the HDST data with scanpy (sc.read_h5ad), running
    the comparison, and writing svg_rank_comparison.csv.
- name: R
  role: Environment for the R implementation of singleCellHaystack (CRAN version >=
    1.0.0) if the verification pipeline is run in R rather than Python.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:85453c7a99bdd85ae46d8b42e445b8bc071651391162d33df8f41cd3d79f099a
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# p-value-significance-verification

## Summary

Cross-check per-gene p-values and ranks produced by two spatially-variable-gene methods (singleCellHaystack and SPARK-X) on the same count matrix and 2D spatial coordinates, quantifying where the methods agree (e.g. Gm42418 as top SVG) and where they diverge (e.g. Gm10925: rank 13 by singleCellHaystack, p=2.7e-39, vs rank 105 by SPARK-X, p=0.013).

## When to use

Use when two SVG/DEG detection methods have been run on the same spatial dataset and you must verify and report which features agree or show discrepant ranks and p-values between them.

## When NOT to use

- When only one method's output is available — the skill requires matched ranked feature lists and p-values from both singleCellHaystack and SPARK-X on the same data.
- When the analysis goal is multi-condition comparison (e.g. wild-type vs knockout) — the paper notes such comparisons 'cannot be easily conducted' with singleCellHaystack in its current version.
- When a cluster-oriented DEG analysis is required — the paper states singleCellHaystack results are 'not as straightforward to incorporate into a cluster-oriented scRNA-seq analysis'.

## Inputs

- HDST (or other spatial transcriptomics) count matrix from the Broad Single Cell Portal (SCP420) as .h5ad / AnnData
- 2D spatial coordinates (spot positions) for the same tissue section
- Per-gene ranked feature lists and p-values from singleCellHaystack (res.top_features)
- Per-gene spatial variability p-values and ranks from SPARK-X

## Outputs

- Side-by-side rank and p-value comparison table (svg_rank_comparison.csv)
- Verified list of concordant top-scoring SVGs (e.g. Gm42418 ranked first by both methods)
- Verified discrepant feature ranking (e.g. Gm10925: 13th vs 105th, p=2.7e-39 vs 0.013)

## How to apply

Run each method on the identical input — here, the mouse olfactory bulb HDST count matrix (Broad Single Cell Portal, SCP420) loaded into an AnnData object with scanpy, plus the same 2D spatial coordinates. For singleCellHaystack, call haystack on all genes without a detection threshold or cell clustering, then extract res.top_features(n) rankings. For SPARK-X, obtain per-gene spatial variability p-values and ranks from the same matrix and coordinates. Locate the agreed top feature (Gm42418) and the discrepant feature (Gm10925) in both ranked lists, record side-by-side rank and p-value pairs, and write the comparison to svg_rank_comparison.csv. Judge correctness by confirming that the shared top hit and the discrepant gene's ranks/p-values reproduce the reported values (Gm10925: 13th, p=2.7e-39 by singleCellHaystack; 105th, p=0.013 by SPARK-X); a large rank gap with both p-values nominally significant reflects the higher sensitivity (but longer runtime) of singleCellHaystack described in the paper.

## Related tools

- **singleCellHaystack** (Produces the per-gene p-values and ranked feature list on 2D spatial coordinates (Python implementation hs.haystack, basis='pca'); its higher sensitivity surfaces discrepant hits like Gm10925.) — https://github.com/alexisvdb/singleCellHaystack
- **singleCellHaystack-py** (Python implementation (PyPI version 0.0.5) used to run haystack on scanpy AnnData objects and extract top_features rankings for comparison against SPARK-X.) — https://github.com/ddiez/singleCellHaystack-py
- **SPARK-X** (Fastest spatial transcriptomics method; supplies the second per-gene p-value/rank list that the singleCellHaystack results are verified against.)
- **Python** (Environment for loading the HDST data with scanpy (sc.read_h5ad), running the comparison, and writing svg_rank_comparison.csv.)
- **R** (Environment for the R implementation of singleCellHaystack (CRAN version >= 1.0.0) if the verification pipeline is run in R rather than Python.)

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Gm42418 appears as the top-ranked SVG in both singleCellHaystack and SPARK-X ranked lists for the HDST dataset.
- Gm10925 is recovered at rank 13 with p=2.7e-39 in singleCellHaystack and rank 105 with p=0.013 in SPARK-X, matching the reported discrepancy.
- svg_rank_comparison.csv contains rank and p-value columns for both methods covering the genes of interest.
- Both methods were run on the same AnnData count matrix and identical 2D spatial coordinates (SCP420), with no detection threshold or clustering applied before singleCellHaystack.

## Limitations

- singleCellHaystack has 'relatively longer runtimes' than SPARK-X for spatial transcriptomics, which the authors accept in exchange for better sensitivity — expect the two methods to disagree on weak-to-moderate hits.
- Comparisons between multiple conditions (e.g. wild-type and knockout) cannot be easily conducted with the current version of singleCellHaystack.
- The Python implementation of singleCellHaystack is in beta; hierarchical clustering of genes (cluster_genes) and advanced sampling weights are not yet available.

## Evidence

- [discussion] For spatial transcriptomics the fastest method available is SPARK-X.: "For spatial transcriptomics the fastest method available is SPARK-X."
- [discussion] Better sensitivity of singleCellHaystack despite longer runtimes: "The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes."
- [abstract] singleCellHaystack handles 2D spatial coordinates: "it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates"
- [readme] README example invocation for the Python package: "res = hs.haystack(adata, basis="pca")
res.top_features(n=10)"
