---
name: spatial-coordinate-based-feature-screening
description: Use when you have an expression matrix (e.g. a Visium AnnData/h5ad) with
  2D spatial coordinates and want to rank all genes by non-random spatial activity,
  or compare SVG calls across methods like singleCellHaystack vs SPARK-X.
when_to_use_negative:
- The goal is differential expression between defined sample conditions (e.g. wild-type
  vs knockout) — the paper notes multi-condition comparisons cannot easily be conducted
  with singleCellHaystack.
- Runtime-critical large spatial screens where speed dominates sensitivity — the discussion
  notes singleCellHaystack has 'relatively longer runtimes compared to SPARK-X for
  spatial transcriptomics', though it is more sensitive.
- Analysis pipelines that are cluster-oriented, since singleCellHaystack results are
  'not as straightforward to incorporate' into cluster-based scRNA-seq analyses.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: singleCellHaystack
  role: 'Primary method: uses Kullback-Leibler divergence on continuous feature activity
    across 2D/3D spatial coordinates to rank spatially variable genes without clustering'
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: singleCellHaystack (Python)
  role: Python implementation (hs.haystack, res.top_features) applied to scanpy AnnData
    objects; version 0.0.5 on PyPI
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: SPARK-X
  role: Fastest spatial transcriptomics method used as comparator to assess rank discrepancies
    and sensitivity of singleCellHaystack SVG calls
- name: scanpy
  role: Loads the spatial dataset from h5ad into an AnnData object for the Python
    singleCellHaystack workflow
- name: R / singleCellHaystack R package
  role: Alternative R implementation (CRAN, version >= 1.0.0) for the same spatial
    coordinate-based screening
  repo: https://github.com/alexisvdb/singleCellHaystack
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:ae733e2c2da9dd2acb666e54229c7c91b6f7d3bcc0076a5a91209ab78cf945f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - Python
  - R / singleCellHaystack R package
  - SPARK-X
  - scanpy
  - singleCellHaystack
  - singleCellHaystack (Python)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# spatial-coordinate-based-feature-screening

## Summary

Screen all genes/features for spatially variable activity directly from 2D (or 3D) spatial coordinates using singleCellHaystack's Kullback-Leibler divergence approach, which detects non-random positioning of feature activity without clustering, and benchmark sensitivity against faster methods such as SPARK-X.

## When to use

Use when you have an expression matrix (e.g. a Visium AnnData/h5ad) with 2D spatial coordinates and want to rank all genes by non-random spatial activity, or compare SVG calls across methods like singleCellHaystack vs SPARK-X.

## When NOT to use

- The goal is differential expression between defined sample conditions (e.g. wild-type vs knockout) — the paper notes multi-condition comparisons cannot easily be conducted with singleCellHaystack.
- Runtime-critical large spatial screens where speed dominates sensitivity — the discussion notes singleCellHaystack has 'relatively longer runtimes compared to SPARK-X for spatial transcriptomics', though it is more sensitive.
- Analysis pipelines that are cluster-oriented, since singleCellHaystack results are 'not as straightforward to incorporate' into cluster-based scRNA-seq analyses.

## Inputs

- AnnData object (.h5ad) with a gene count matrix and 2D spatial coordinates (e.g. 10x Visium posterior1 mouse brain)
- Spatial coordinates used as the input space for singleCellHaystack (basis other than pca, e.g. 2D coordinates)

## Outputs

- Ranked table of features (genes) with singleCellHaystack scores and p-values for all genes
- Top features list (res.top_features(n=10))
- Merged per-gene rank/p-value comparison table across methods (e.g. haystack_sparkx_rank_comparison.csv)

## How to apply

Load the spatial dataset (e.g. 10x Visium posterior1 mouse brain) with scanpy via sc.read_h5ad so the AnnData object contains the count matrix and 2D spatial coordinates. Run the Python singleCellHaystack (hs.haystack) on the AnnData using the spatial coordinates as the input space — singleCellHaystack uses Kullback-Leibler Divergence to find features whose activity is non-randomly positioned in 1D, 2D, or higher-dimensional input spaces, without clustering cells. Extract rankings with res.top_features(n=10) and the full ranked per-gene table with p-values. Optionally run SPARK-X on the same expression matrix and coordinates and merge per-gene ranks/p-values (e.g. into a haystack_sparkx_rank_comparison.csv) to identify genes detected by one method but not the other, such as Ttr ranked 58th of 16,596 genes (p=1.1e-105) by singleCellHaystack but 11,881st (p=0.0087) by SPARK-X. Validate by checking that rank and p-value columns exist for both methods and that known discordant genes (Ttr, Arpp19, Ccdc36) match expected ranks within each method's ordering.

## Related tools

- **singleCellHaystack** (Primary method: uses Kullback-Leibler divergence on continuous feature activity across 2D/3D spatial coordinates to rank spatially variable genes without clustering) — https://github.com/alexisvdb/singleCellHaystack
- **singleCellHaystack (Python)** (Python implementation (hs.haystack, res.top_features) applied to scanpy AnnData objects; version 0.0.5 on PyPI) — https://github.com/ddiez/singleCellHaystack-py
- **SPARK-X** (Fastest spatial transcriptomics method used as comparator to assess rank discrepancies and sensitivity of singleCellHaystack SVG calls)
- **scanpy** (Loads the spatial dataset from h5ad into an AnnData object for the Python singleCellHaystack workflow)
- **R / singleCellHaystack R package** (Alternative R implementation (CRAN, version >= 1.0.0) for the same spatial coordinate-based screening) — https://github.com/alexisvdb/singleCellHaystack

## Examples

```
import scanpy as sc; import singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- The output comparison CSV contains rank and p-value columns for both methods (singleCellHaystack and SPARK-X) covering all genes (e.g. 16,596 genes in the Visium posterior1 dataset).
- Known discordant genes reproduce expected ranks: Ttr 58th (p=1.1e-105) by singleCellHaystack vs 11,881st (p=0.0087) by SPARK-X; Arpp19 18th (SPARK-X) vs 983rd (singleCellHaystack); Ccdc36 73rd vs 1585th.
- Top hits include known spatial markers of the posterior mouse brain such as Mbp, Calb2, Plp1, and Ttr.
- res.top_features(n=10) returns a valid ranked feature list consistent with the full ranked table.
- The Python implementation reports no missing required features: hs.haystack runs with the spatial-coordinate basis and completes on all genes.

## Limitations

- Comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted with singleCellHaystack.
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses.
- singleCellHaystack has relatively longer runtimes than SPARK-X for spatial transcriptomics, though its better sensitivity makes it the best alternative.
- The Python package is in beta: weights.advanced.Q, the seeding grid-point method, and hierarchical clustering for cluster_genes are not yet available.
- The original binary-threshold version could overestimate significance of modest expression differences; the 1.0+ continuous-activity version addresses this.

## Evidence

- [readme] KL divergence on spatial coordinates: [evidence span withheld — anchor status: not_found]
- [methods] Python workflow on AnnData: "adata = sc.read_h5ad("data.h5ad")"
- [methods] haystack call and top features: "res = hs.haystack(adata, basis="pca")
res.top_features(n=10)"
- [discussion] SPARK-X as fastest spatial comparator: "For spatial transcriptomics the fastest method available is SPARK-X."
- [discussion] Sensitivity vs runtime trade-off: "The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes."
- [abstract] Handles spatial coordinates as input space: "it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates"
- [discussion] Multi-condition limitation: "One is that comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted."
- [readme] Beta missing features: "This package is currently in beta. The most important functionality in the R package works, but some features are not yet available."
