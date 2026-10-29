---
name: slide-seq-spatial-data-loading-anndata
description: Use when starting from a Slide-seqV2 puck (e.g. Stickels_hippo mouse
  hippocampus, accession SCP815) consisting of a gene-by-bead count matrix plus a
  2D coordinate file, and you need an AnnData object with spatial coordinates attached
  for downstream SVG detection.
when_to_use_negative:
- Data is already a processed .h5ad with spatial coordinates in adata.obsm — reloading
  from raw puck files would risk re-introducing bead-order mismatches
- The spatial assay is not bead-based Slide-seq (e.g. Visium spot data), where coordinate
  attachment conventions differ
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3170
tools:
- name: scanpy
  role: Python library used to load the expression matrix and construct/attach coordinates
    in the AnnData object
- name: singleCellHaystack
  role: 'Downstream consumer: predicts differentially active features using the 2D
    spatial coordinates attached in AnnData'
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: SPARK-X
  role: 'Downstream consumer: runs SVG detection on the same all-gene matrix with
    identical 2D coordinate input'
- name: singleCellHaystack-py
  role: Python implementation that operates directly on the AnnData object produced
    by this loading step
  repo: https://github.com/ddiez/singleCellHaystack-py
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:341fa969201783dd2919879ccb7f1b1435e499b2f941d73cae0a1631a0ad3fa9
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Slide-seq spatial data loading into AnnData

## Summary

Load a Slide-seqV2 puck's gene-by-bead expression matrix into a scanpy AnnData object and attach the 2D puck bead coordinates as spatial inputs, so spatially-aware methods like singleCellHaystack and SPARK-X can be run on identical inputs.

## When to use

Use when starting from a Slide-seqV2 puck (e.g. Stickels_hippo mouse hippocampus, accession SCP815) consisting of a gene-by-bead count matrix plus a 2D coordinate file, and you need an AnnData object with spatial coordinates attached for downstream SVG detection.

## When NOT to use

- Data is already a processed .h5ad with spatial coordinates in adata.obsm — reloading from raw puck files would risk re-introducing bead-order mismatches
- The spatial assay is not bead-based Slide-seq (e.g. Visium spot data), where coordinate attachment conventions differ

## Inputs

- Slide-seqV2 puck gene-by-bead expression matrix (from Broad Single Cell Portal, e.g. accession SCP815)
- Slide-seqV2 2D puck bead coordinate table (x, y positions keyed by bead ID)

## Outputs

- AnnData object (adata) with the expression matrix in adata.X and puck 2D coordinates attached as cell spatial coordinates
- Coordinate-aligned AnnData ready as input to singleCellHaystack and SPARK-X

## How to apply

Download the puck gene-by-bead expression matrix and its 2D bead coordinate table from the Broad Single Cell Portal (accession SCP815 for the Stickels_hippo mouse hippocampus dataset). Read the expression matrix into Python and construct an AnnData object (genes as variables, beads as observations), using scanpy utilities such as sc.read_h5ad or equivalent loaders. Attach the 2D puck coordinates as the cells' spatial input coordinates so both singleCellHaystack (using 2D spatial coordinates) and SPARK-X consume the identical coordinate input. Verify that observation order in adata.obs matches the bead identifier order in the coordinate file before running any spatially-aware method, since rank comparisons across methods depend on consistent bead indexing.

## Related tools

- **scanpy** (Python library used to load the expression matrix and construct/attach coordinates in the AnnData object)
- **singleCellHaystack** (Downstream consumer: predicts differentially active features using the 2D spatial coordinates attached in AnnData) — https://github.com/alexisvdb/singleCellHaystack
- **SPARK-X** (Downstream consumer: runs SVG detection on the same all-gene matrix with identical 2D coordinate input)
- **singleCellHaystack-py** (Python implementation that operates directly on the AnnData object produced by this loading step) — https://github.com/ddiez/singleCellHaystack-py

## Examples

```
import scanpy as sc; import singleCellHaystack as hs; adata = sc.read_h5ad("Stickels_hippo.h5ad"); res = hs.haystack(adata, basis="spatial")
```

## Evaluation signals

- adata.n_obs equals the number of beads in the puck coordinate file and adata.n_vars equals the number of genes in the matrix
- Bead identifiers in adata.obs_names match the bead IDs in the coordinate table one-to-one and in the same order
- Coordinate ranges (x, y) in the attached spatial data fall within the puck footprint reported in the SCP815 coordinate file
- Downstream methods (singleCellHaystack and SPARK-X) run on the same object without coordinate-alignment errors
- Per-gene ranks extracted from both methods can be joined by gene symbol without ambiguity

## Limitations

- The Python singleCellHaystack package is in beta and lacks some R features (advanced sampling weights, seeding grid, hierarchical clustering of genes), so AnnData-based runs may not fully reproduce R-package behavior
- singleCellHaystack comparisons between multiple conditions (e.g. wild-type and knockout) cannot be easily conducted on the loaded object
- Loading must preserve bead ordering; a mismatch between expression matrix and coordinate file row order silently corrupts spatial assignments

## Evidence

- [abstract] singleCellHaystack now accepts continuous features ... from single-cell, spatial, and bulk genomics data: "singleCellHaystack now accepts continuous features that can be RNA or protein expression, chromatin accessibility or module scores from single-cell, spatial and even bulk genomics data"
- [methods] Loading via scanpy read_h5ad: "adata = sc.read_h5ad("data.h5ad")"
- [abstract] singleCellHaystack spatial input: "it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates"
- [readme] Slide-seqV2 spatial tutorial exists: [evidence span withheld — anchor status: external_doc]
- [readme] Python AnnData usage example: "import scanpy as sc
import singleCellHaystack as hs

adata = sc.read_h5ad("data.h5ad")"
