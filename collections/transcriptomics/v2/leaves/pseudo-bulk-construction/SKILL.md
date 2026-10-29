---
name: pseudo-bulk-construction
description: Use when integrating atlas-scale multi-sample multi-condition single-cell
  collections (millions of cells, thousands of samples) where per-cell processing
  of scRNA-seq or CyTOF matrices is computationally prohibitive.
when_to_use_negative:
- When the goal is joint integration across modalities (e.g. combining scRNA-seq and
  CyTOF features); the paper's integration was performed within each modality only,
  with multi-modal incorporation deferred to future work
- When the study is small enough that per-cell processing is tractable and pseudo-bulk
  compression offers no scalability benefit
- When no reliable cell-type annotations exist, since the pseudo-bulk grouping depends
  on annotated cell types per sample
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMerge2
  role: Builds pseudo-bulk profiles per annotated cell type within each sample and
    assigns pseudo-replicates within phenotype/condition groups to enable scalable
    batch-effect estimation on the compressed matrix
  repo: https://github.com/SydneyBioX/scMerge
- name: Seurat
  role: Computes joint UMAP embeddings on the scMerge2-adjusted matrix downstream
    of pseudo-bulk-based integration (CyTOF application)
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:b050623af4d7cbd87333f654db35751ad3d26c8df77cae07bf510b34b8eb86b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - BBKNN
  - Harmony
  - Scanorama
  - Seurat
  - scANVI
  - scMerge2
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# pseudo-bulk-construction

## Summary

Aggregate single cells into pseudo-bulk profiles (per cell type within each sample, or per sample-by-condition combination) as implemented in scMerge2, to reduce atlas-scale collections (~5 million cells, 1298 samples) to a tractable pseudo-replicate structure before batch-effect estimation.

## When to use

Use when integrating atlas-scale multi-sample multi-condition single-cell collections (millions of cells, thousands of samples) where per-cell processing of scRNA-seq or CyTOF matrices is computationally prohibitive.

## When NOT to use

- When the goal is joint integration across modalities (e.g. combining scRNA-seq and CyTOF features); the paper's integration was performed within each modality only, with multi-modal incorporation deferred to future work
- When the study is small enough that per-cell processing is tractable and pseudo-bulk compression offers no scalability benefit
- When no reliable cell-type annotations exist, since the pseudo-bulk grouping depends on annotated cell types per sample

## Inputs

- raw single-cell count matrices (SingleCellExperiment objects, CITE-seq: 17,446 genes + 87 proteins; CyTOF marker-expression matrices)
- sample/cell metadata with batch, condition and refined cell-type annotations (used as pseudo-replicate information)
- study/batch grouping labels (dataset, protocol, technology)

## Outputs

- pseudo-bulk expression matrix per cell-type-by-sample (scRNA-seq) or sample-by-condition (CyTOF) combination
- pseudo-replicate assignments within phenotype/condition groups
- scMerge2-adjusted single-cell expression matrix after propagating batch-correction factors back to cells

## How to apply

Aggregate raw count matrices into pseudo-bulk profiles by summing counts per annotated cell type within each sample (scRNA-seq) or per sample-by-condition combination (CyTOF), which minimises the processing demand of the atlas-scale data while retaining sample-level variation. Use refined cell-type annotations as the grouping variable, and in the COVID-19 collection setting this reduced ~5 million cells from 963 donors and 1298 samples across 20 studies to a manageable pseudo-bulk matrix. Next, assign pseudo-replicates within phenotype/condition groups so that pseudo-replication accounts for circumstances with phenotypes. Feed the pseudo-bulk matrix into scMerge2 (optionally hierarchically, integrating batches within one study or studies of similar size first) to estimate the unwanted-variation factors, then propagate the adjustment back to the single-cell level. Rationale: estimating dataset/protocol/technology effects on the compressed pseudo-replicate structure makes batch-effect removal scalable (scMerge2 integrated the 5M-cell collection within a day) while preserving biological cell-state discrimination.

## Related tools

- **scMerge2** (Builds pseudo-bulk profiles per annotated cell type within each sample and assigns pseudo-replicates within phenotype/condition groups to enable scalable batch-effect estimation on the compressed matrix) — https://github.com/SydneyBioX/scMerge
- **Seurat** (Computes joint UMAP embeddings on the scMerge2-adjusted matrix downstream of pseudo-bulk-based integration (CyTOF application))

## Examples

```
BiocManager::install("scMerge")
```

## Evaluation signals

- Technical-label PCA separation (dataset, protocol, technology) is diminished in scMerge2-adjusted data versus raw data while COVID-19 severity-level cell-state discrimination is preserved
- Post-integration UMAP shows datasets mixed within shared cell types (e.g. COMBAT + Geanon CyTOF) while the dataset-unique Granulocyte (Neutrophil/Eosinophil) cluster is retained
- Per-marker density distributions across datasets overlap after integration, indicating dataset-effect removal
- Benchmark metrics (batch-ARI, cell-type-ARI, batch-ASW, cell-type-ASW) on the integrated representation show batch removal without loss of cell-type structure
- The pseudo-bulk grouping exactly follows the refined cell-type annotations per sample (or sample-by-condition for CyTOF), with no cells aggregated across samples

## Limitations

- Integration order is data-guided only (batches within one study or studies of similar size first); a knowledge-guided order is an alternative not used
- The concatenating integration approach is more computationally intensive than the balanced tree approach
- Pseudo-replication requires existing phenotype/condition groupings and cell-type annotations; multi-modal information is not incorporated into pseudo-replicate identification

## Evidence

- [discussion] This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [intro] we present scMerge2, a scalable, high-capacity algorithm that allows data integration of atlas-scale multi-sample multi-condition single-cell studies.: "we present scMerge2, a scalable, high-capacity algorithm that allows data integration of atlas-scale multi-sample multi-condition single-cell studies."
- [readme] `scMerge` is a R package for merging and normalising single-cell RNA-Seq datasets.: "`scMerge` is a R package for merging and normalising single-cell RNA-Seq datasets."
