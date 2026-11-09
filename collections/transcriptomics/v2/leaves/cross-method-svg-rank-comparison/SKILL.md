---
name: cross-method-svg-rank-comparison
description: Use when you need to compare the agreement of two or more spatially variable
  gene detection methods on identical spatial transcriptomics data, or to verify whether
  a method under-ranks genes with strong spatial expression patterns.
when_to_use_negative:
- The comparison involves multiple biological conditions (e.g., wild-type vs knockout),
  since singleCellHaystack cannot easily conduct comparisons between multiple conditions
- The spatial dataset is so large that runtime is the binding constraint and speed
  matters more than sensitivity — the paper notes SPARK-X is the fastest spatial method
  while singleCellHaystack has relatively longer runtimes
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: singleCellHaystack
  role: 'Method 1: predicts differentially active features using Kullback-Leibler
    Divergence on 2D spatial coordinates; provides per-gene ranks for comparison'
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: SPARK-X
  role: 'Method 2: the fastest spatial transcriptomics method, run on the same all-gene
    matrix and identical 2D coordinates to produce SVG ranks for cross-method comparison'
- name: scanpy
  role: Loads the gene-by-bead expression matrix into an AnnData object and attaches
    2D puck coordinates as spatial input
- name: singleCellHaystack-py
  role: Python implementation of singleCellHaystack (version >= 1.0.0) available from
    PyPI; alternative to the R package for running haystack on AnnData objects
  repo: https://github.com/ddiez/singleCellHaystack-py
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:472608bf52a1b8c731ebd54d4eeb0232c2f38abcbe1a8b282c1e479576a87c83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - Python
  - R
  - SPARK-X
  - scanpy
  - singleCellHaystack
  - singleCellHaystack-py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# cross-method-svg-rank-comparison

## Summary

A benchmarking skill that runs two spatially variable gene (SVG) methods — singleCellHaystack and SPARK-X — on the same Slide-seqV2 spatial dataset with identical 2D coordinate input, extracts per-gene ranks, and quantifies rank discrepancies for target genes (e.g., Plp1, Mal) to expose sensitivity differences between methods.

## When to use

Use when you need to compare the agreement of two or more spatially variable gene detection methods on identical spatial transcriptomics data, or to verify whether a method under-ranks genes with strong spatial expression patterns.

## When NOT to use

- The comparison involves multiple biological conditions (e.g., wild-type vs knockout), since singleCellHaystack cannot easily conduct comparisons between multiple conditions
- The spatial dataset is so large that runtime is the binding constraint and speed matters more than sensitivity — the paper notes SPARK-X is the fastest spatial method while singleCellHaystack has relatively longer runtimes

## Inputs

- Slide-seqV2 puck gene-by-bead expression matrix (Stickels_hippo, Broad Single Cell Portal accession SCP815)
- 2D puck coordinates for each bead
- List of target genes to rank-check (e.g., Plp1, Mal, Zfhx3, Cox4i1)

## Outputs

- svg_rank_comparison.csv with per-gene ranks under singleCellHaystack and SPARK-X, rank differences, and agreement flags
- Per-gene differential activity ranks from singleCellHaystack
- Per-gene SVG ranks from SPARK-X

## How to apply

Download the Slide-seqV2 puck gene-by-bead expression matrix and 2D puck coordinates (Broad Single Cell Portal accession SCP815, Stickels_hippo mouse hippocampus). Load the expression matrix into a scanpy AnnData object and attach the 2D puck coordinates as spatial input coordinates. Run singleCellHaystack (R package, version 1.0.0) on all genes using the 2D spatial coordinates to obtain per-gene differential activity ranks, then run SPARK-X on the same all-gene matrix with the identical 2D coordinate input. Extract each gene's rank under both methods, compute rank differences for target genes (Plp1, Mal, Zfhx3, Cox4i1), and write a comparison table such as svg_rank_comparison.csv with per-gene ranks and agreement flags. The rationale, per the singleCellHaystack paper, is that singleCellHaystack has better sensitivity for spatial patterns (at the cost of longer runtimes than SPARK-X), so agreement checks reveal method-specific ranking biases rather than pipeline errors.

## Related tools

- **singleCellHaystack** (Method 1: predicts differentially active features using Kullback-Leibler Divergence on 2D spatial coordinates; provides per-gene ranks for comparison) — https://github.com/alexisvdb/singleCellHaystack
- **SPARK-X** (Method 2: the fastest spatial transcriptomics method, run on the same all-gene matrix and identical 2D coordinates to produce SVG ranks for cross-method comparison)
- **scanpy** (Loads the gene-by-bead expression matrix into an AnnData object and attaches 2D puck coordinates as spatial input)
- **singleCellHaystack-py** (Python implementation of singleCellHaystack (version >= 1.0.0) available from PyPI; alternative to the R package for running haystack on AnnData objects) — https://github.com/ddiez/singleCellHaystack-py

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Computed singleCellHaystack ranks match reported values: Plp1 20th, Mal 55th, Zfhx3 378th, Cox4i1 1123rd
- Computed SPARK-X ranks match reported values: Plp1 1139th, Mal 1543rd, Zfhx3 64th, Cox4i1 183rd
- svg_rank_comparison.csv contains per-gene ranks, rank differences, and agreement flags for all four target genes; mismatched rows are flagged if values diverge
- Both methods were run on the identical all-gene expression matrix with identical 2D coordinate input, confirming the comparison is a fair cross-method test
- Output CSV is well-formed with one row per target gene and no missing rank entries

## Limitations

- singleCellHaystack cannot easily conduct comparisons between multiple conditions (e.g., wild-type and knockout)
- singleCellHaystack has relatively longer runtimes than SPARK-X for spatial transcriptomics, though its better sensitivity makes it the best alternative
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses
- The Python implementation of singleCellHaystack is in beta; features like weights.advanced.Q, the seeding grid method, and hierarchical clustering for cluster_genes are not yet available

## Evidence

- [discussion] For spatial transcriptomics the fastest method available is SPARK-X.: "For spatial transcriptomics the fastest method available is SPARK-X."
- [discussion] Runtimes vs sensitivity: "The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes."
- [abstract] Coordinate input support: "it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates"
- [readme] KL divergence basis: [evidence span withheld — anchor status: external_doc]
- [methods] Python invocation: "res = hs.haystack(adata, basis="pca")"
- [readme] Slide-seqV2 tutorial availability: [evidence span withheld — anchor status: external_doc]
