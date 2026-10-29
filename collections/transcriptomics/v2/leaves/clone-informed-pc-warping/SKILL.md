---
name: clone-informed-pc-warping
description: Use when you have a PCA (or similar reduced-dimension) matrix plus per-cell
  clone barcode assignments and want a UMAP-style 2D visualization in which cells
  sharing a barcode coalesce as a single tunable parameter s increases.
when_to_use_negative:
- You need hybrid clustering assignments rather than a 2D visualization — use ClonoCluster's
  clonocluster() with tunable alpha instead of PC warping
- Your input is a raw gene-by-cell count matrix and no PCA has been computed — run
  irlba_wrap first; the warp operates on PC values, not raw counts
- Your data have no clone barcode assignments (no clonal origin information to warp
  toward)
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ClonoCluster
  role: Provides engage_warp, which implements the Warp Factor transformation (wrapping
    uwot/umap_matrix) to run UMAP on the warped PC matrix
  repo: https://github.com/leeprichman/ClonoCluster
- name: Warp Factor
  role: The complementary tool whose warp transformation Ms = M − (s/10)(M − Mb) incorporates
    clone information into the UMAP input
  repo: https://github.com/leeprichman/ClonoCluster
- name: data.table
  role: Used for fread to load the PCA and barcode tables and rbindlist to combine
    per-warp embedding tables with a 'warp' column
- name: UMAP
  role: The 2D visualization technique run on each warped PC matrix (s = 0, 5, 10)
    to produce the final embeddings
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xgen.2022.100247
    title: 'ClonoCluster: A method for using clonal origin to inform transcriptome
      clustering'
  source_content_hash: sha256:6301a61a235bd0831f8cf82c9f1f9cfd4fe44657f32ecc5b191011fae1b3c603
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leeprichman/ClonoCluster
  tools:
  - ClonoCluster
  - R (version >=)
  - UMAP
  - Warp Factor
  - data.table
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# clone-informed-pc-warping

## Summary

Warp Factor modifies the PCA matrix input to UMAP so that each cell's principal component values approach the mean PC value of its clone (barcode) cluster, controlled by a tunable parameter s from 0 to 10, enabling clonal origin to shape 2D transcriptome visualizations.

## When to use

Use when you have a PCA (or similar reduced-dimension) matrix plus per-cell clone barcode assignments and want a UMAP-style 2D visualization in which cells sharing a barcode coalesce as a single tunable parameter s increases.

## When NOT to use

- You need hybrid clustering assignments rather than a 2D visualization — use ClonoCluster's clonocluster() with tunable alpha instead of PC warping
- Your input is a raw gene-by-cell count matrix and no PCA has been computed — run irlba_wrap first; the warp operates on PC values, not raw counts
- Your data have no clone barcode assignments (no clonal origin information to warp toward)

## Inputs

- PCA matrix of reduced dimensions (e.g. 25 PCs from irlba_wrap) with one row per cell
- Two-column barcode table with columns 'rn' (cell ID) and 'Barcode' (clone assignment)

## Outputs

- Warped 2D UMAP embedding tables (one per warp factor s = 0, 5, 10) with a 'warp' column
- umap_warped_embeddings.tsv containing the combined warp function results and embeddings

## How to apply

Load the deposited PCA matrix (25 PCs generated with irlba_wrap) and the two-column barcode table ('rn', 'Barcode') into R with data.table::fread. Group cells by Barcode and compute the mean PC value vector Mb for each clone cluster, then apply the warp transformation Ms = M − (s/10)(M − Mb), which shifts each cell's PCs toward its clone-cluster mean by fraction s/10; s = 0 leaves the PC matrix unmodified and s = 10 makes clone-cluster variation the only variation. Run UMAP on each warped matrix for s in {0, 5, 10} using ClonoCluster's engage_warp, which wraps uwot/umap_matrix. Combine the per-warp embedding tables into one data table with a 'warp' column via data.table::rbindlist and save as umap_warped_embeddings.tsv.

## Related tools

- **ClonoCluster** (Provides engage_warp, which implements the Warp Factor transformation (wrapping uwot/umap_matrix) to run UMAP on the warped PC matrix) — https://github.com/leeprichman/ClonoCluster
- **Warp Factor** (The complementary tool whose warp transformation Ms = M − (s/10)(M − Mb) incorporates clone information into the UMAP input) — https://github.com/leeprichman/ClonoCluster
- **data.table** (Used for fread to load the PCA and barcode tables and rbindlist to combine per-warp embedding tables with a 'warp' column)
- **UMAP** (The 2D visualization technique run on each warped PC matrix (s = 0, 5, 10) to produce the final embeddings)

## Examples

```
umap_wf0 <- engage_warp(pca, bt, 0)
```

## Evaluation signals

- The s = 0 embedding matches a default UMAP of the unmodified 25-PC matrix exactly, since Ms = M when s = 0
- Cells sharing the same Barcode visibly/numerically coalesce in the embedding as s increases toward 10, consistent with the warp behavior
- At s = 10, within-clone-cluster PC variation is the only variation remaining, since Ms collapses each cell's PCs to its clone-cluster mean Mb
- Combined output table contains a 'warp' column and one set of embeddings per tested s value (0, 5, 10) from rbindlist

## Limitations

- The Warp Factor does not alter clustering itself — it only modifies the dimensionality-reduction input to UMAP for visualization; clustering uses the separate tunable alpha
- The paper did not address full-lineage tracing data (CRISPR-Cas9 genome editing or imaging-based family trees), though ClonoCluster could be adapted to weight distances by phylogenetic distance
- The paper did not reveal a reliable method to identify which experimental systems have intrinsic determinants of cell fate and would therefore benefit from clonal information

## Evidence

- [intro] a tunable “Warp Factor” parameter, s, warps the values of PCs for each cell to approach the mean value of the principal component for the clone cluster it belongs to: "a tunable “Warp Factor” parameter, s, warps the values of PCs for each cell to approach the mean value of the principal component for the clone cluster it belongs to"
- [intro] Objective: develop a tunable parameter (the Warp Factor, ranging from 0 to 10) that incorporates clonality information into the dimensionality reduction step prior to UMAP visualization: "we developed a tunable parameter (the Warp Factor, ranging from 0 to 10), that incorporates clonality information into the dimensionality reduction step prior to the commonly used Uniform Manifold"
- [methods] engage_warp call wrapping the PCA matrix, barcode table, and warp factor 0: "umap_wf0 <- engage_warp(pca, bt, 0)"
- [readme] README describing ClonoCluster as a method for using clonal origin to inform transcriptome clustering: "*a method for using clonal origin to inform transcriptome clustering*"
