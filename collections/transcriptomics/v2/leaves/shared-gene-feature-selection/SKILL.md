---
name: shared-gene-feature-selection
description: Use when integrating two single-cell modalities (e.g., MERFISH spatial
  data with matched scRNA-seq) that must be fed a common gene set as input features,
  or when a shared-encoder integration model requires identical features across datasets.
when_to_use_negative:
- CITE-seq or protein-modality integration where only a few hundred antibody features
  exist, making a meaningful shared-feature selection impractical
- Label transfer across tissues with dissimilar cell types, where sharing features
  does not fix the biological mismatch
- Datasets already preprocessed into a common feature space (e.g., shared HVG matrices
  from a prior integration run)
tools:
- name: scCobra
  role: Integration model whose shared encoder consumes the shared-gene feature matrices
    for contrastive embedding with domain adaptation
  repo: https://github.com/mcgilldinglab/scCobra
- name: Scanpy
  role: Used for cell/gene filtering, log1p normalization, and Seurat 'vst' selection
    of 2000 highly variable genes on the shared-gene matrices
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:3dab687cf82facf1df16beecfefd9e4c68f92d21b819d65d0583088944112235
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scCobra
  tools:
  - Harmony
  - Scanorama
  - Scanpy
  - Seurat
  - scCobra
  - scDML
  - scDREAMER
  - scIB
  - scVI
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# Shared-gene feature selection for single-cell data integration

## Summary

Restricts paired single-cell datasets (e.g., MERFISH spatial transcriptomics and matched scRNA-seq) to their intersecting gene symbols so both matrices share an identical feature space before integration with scCobra or benchmark batch-correction methods.

## When to use

Use when integrating two single-cell modalities (e.g., MERFISH spatial data with matched scRNA-seq) that must be fed a common gene set as input features, or when a shared-encoder integration model requires identical features across datasets.

## When NOT to use

- CITE-seq or protein-modality integration where only a few hundred antibody features exist, making a meaningful shared-feature selection impractical
- Label transfer across tissues with dissimilar cell types, where sharing features does not fix the biological mismatch
- Datasets already preprocessed into a common feature space (e.g., shared HVG matrices from a prior integration run)

## Inputs

- MERFISH spatial transcriptomics count matrix (cell x gene, e.g., mouse hypothalamus dataset)
- Matched scRNA-seq reference count matrix (cell x gene, e.g., .h5ad loaded with Scanpy)

## Outputs

- Paired count matrices restricted to the identical set of shared genes
- Scanpy AnnData objects with cells <200 genes, genes <3 cells, and >5% mitochondrial-fraction cells removed, log1p-normalized, with 2000 'vst' HVGs flagged
- Shared-gene feature list used as input for scCobra integration and benchmark comparisons

## How to apply

Load both count matrices (e.g., the mouse hypothalamus MERFISH spatial dataset and its matched scRNA-seq reference) and compute the intersection of their gene names (var_names), keeping only shared genes as input features. On the shared-gene matrices, apply the paper's Scanpy preprocessing: remove cells expressing fewer than 200 genes, drop genes detected in fewer than 3 cells, remove cells with mitochondrial fraction above 5%, normalize with a log1p transformation, and select 2000 highly variable genes using the Seurat 'vst' method. The rationale is that scCobra's shared encoder and domain adaptation operate on a common feature space, and restricting to shared genes also preserves interpretability in the original gene space, which methods that leave the original feature space lose. Order matters: intersect genes first, then filter cells/genes and select HVGs on the shared matrix so both modalities enter the integration with the same features.

## Related tools

- **scCobra** (Integration model whose shared encoder consumes the shared-gene feature matrices for contrastive embedding with domain adaptation) — https://github.com/mcgilldinglab/scCobra
- **Scanpy** (Used for cell/gene filtering, log1p normalization, and Seurat 'vst' selection of 2000 highly variable genes on the shared-gene matrices)

## Examples

```
shared = list(set(adata_merfish.var_names) & set(adata_rna.var_names)); adata_merfish = adata_merfish[:, shared]; adata_rna = adata_rna[:, shared]
```

## Evaluation signals

- Both output matrices have exactly the same var_names (gene symbol) set; no modality-specific genes remain
- After filtering, no cell expresses fewer than 200 genes and no gene is detected in fewer than 3 cells
- Exactly 2000 highly variable genes are flagged using the 'vst' method before downstream integration
- Downstream scCobra embedding places newly formed OD and OD Immature cells correctly adjacent, and scCobra ranks first on scIB Batch-ASW and summed biological conservation metrics
- Per-method metric table (hypothalamus_integration_benchmark.csv) renders Fig. 4e/4f-style comparison plots without missing methods

## Limitations

- scCobra's shared encoder requires selecting shared features between modalities; CITE-seq protein data with only a few hundred features makes this challenging to optimize effectively
- Restricting to shared genes discards modality-specific genes, which may drop informative markers not assayed by both platforms
- Label transfer with scCobra still requires that the reference and query datasets originate from the same tissue type or contain similar cell types

## Evidence

- [other] Load the mouse hypothalamus MERFISH spatial dataset and the matched scRNA-seq reference, and restrict both matrices to their shared genes as input features.: [evidence span withheld — anchor status: not_found]
- [discussion] CITE-seq shared-feature challenge: "common single-cell protein data, such as CITE-seq, typically contain only a few hundred features, making it challenging to identify shared features and optimize the model effectively"
- [intro] Original feature space interpretability rationale: "certain methods and their original implementations are challenging to apply in the original feature space8,14,17,18, leading to a loss of model interpretability"
- [discussion] Gene filtering threshold: "we first filtered out cells that expressed fewer than 200 genes and genes that were detected in fewer than 3 cells"
