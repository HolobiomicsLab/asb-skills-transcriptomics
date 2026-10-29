---
name: scrnaseq-reference-mapping
description: Use when a query scRNA-seq (or spatial MERFISH) dataset must be annotated
  by transferring labels from a matched reference scRNA-seq atlas sharing the same
  tissue, especially when batch effects separate the two datasets.
when_to_use_negative:
- Reference and query datasets originate from different tissues or contain dissimilar
  cell types — the article states label transfer requires same-tissue or similar cell-type
  composition for optimal performance
- The modality pair has few shared features (e.g. CITE-seq protein data with only
  a few hundred features), making shared-feature selection and model optimization
  impractical
- The goal is de novo clustering of a single batch without any reference atlas — over-correction-free
  integration machinery is unnecessary
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCobra
  role: Contrastive cell embedding learning with domain adaptation; shared encoder
    maps query cells into the reference embedding space for online label transfer
  repo: https://github.com/mcgilldinglab/scCobra
- name: Scanpy
  role: 'Preprocessing: filtering (<200 genes/cell, <3 cells/gene), mitochondrial-fraction
    cutoff, log1p normalization, 2000 HVG selection (''vst'')'
- name: MAESTRO
  role: Converts scATAC-seq open chromatin profiles into a gene activity matrix for
    multimodal reference mapping
- name: scIB
  role: Post-hoc evaluation of the mapped embedding via Leiden clustering (resolutions
    0.1–2.0) and batch/bio-conservation metrics
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:db23fce7c7dbc83c519e505e43fae995c2798f92800a83684e713052046dd9b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scCobra
  tools:
  - Harmony
  - MAESTRO
  - Scanorama
  - Scanpy
  - Seurat
  - scCobra
  - scDML
  - scDREAMER
  - scIB
  - scVI
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8695652173913043
n_claims: 23
---

# scrnaseq-reference-mapping

## Summary

Map query single-cell data onto a reference atlas using scCobra's contrastive cell embeddings with domain adaptation, enabling online label transfer across batch-affected datasets without retraining.

## When to use

Use when a query scRNA-seq (or spatial MERFISH) dataset must be annotated by transferring labels from a matched reference scRNA-seq atlas sharing the same tissue, especially when batch effects separate the two datasets.

## When NOT to use

- Reference and query datasets originate from different tissues or contain dissimilar cell types — the article states label transfer requires same-tissue or similar cell-type composition for optimal performance
- The modality pair has few shared features (e.g. CITE-seq protein data with only a few hundred features), making shared-feature selection and model optimization impractical
- The goal is de novo clustering of a single batch without any reference atlas — over-correction-free integration machinery is unnecessary

## Inputs

- Reference scRNA-seq count matrix with cell-type labels (e.g. mouse hypothalamus scRNA-seq, or MERFISH spatial counts as query)
- Query count matrix restricted to shared genes with the reference
- Optional scATAC-seq peak matrix converted to a gene activity matrix with MAESTRO for multimodal mapping

## Outputs

- Contrastive cell embeddings with domain adaptation aligning query and reference batches
- Per-query-cell transferred cell-type labels from the pre-trained KNN classifier
- Integrated embedding (e.g. for UMAP/Leiden visualization and downstream scIB evaluation)

## How to apply

Restrict reference and query matrices to their shared genes as input features, then preprocess with Scanpy: filter cells expressing fewer than 200 genes and genes detected in fewer than 3 cells, remove cells with mitochondrial fraction above 5%, normalize with log1p, and select 2000 highly variable genes via the Seurat 'vst' method. Build the reference by training scCobra's deep generative model (contrastive learning with domain adaptation, three-phase optimization) and attach a pre-trained KNN classifier to the reference embeddings. Encode the query through the shared encoder so its cells land in the same embedding space, then use the pre-trained KNN classifier to annotate query cells — no model retraining is needed, so new query data can be integrated continuously. Because embeddings are learned without assuming a gene expression distribution (unlike scVI's negative-binomial VAE), the original feature space is preserved, aiding interpretability of cell-state differences at the gene level.

## Related tools

- **scCobra** (Contrastive cell embedding learning with domain adaptation; shared encoder maps query cells into the reference embedding space for online label transfer) — https://github.com/mcgilldinglab/scCobra
- **Scanpy** (Preprocessing: filtering (<200 genes/cell, <3 cells/gene), mitochondrial-fraction cutoff, log1p normalization, 2000 HVG selection ('vst'))
- **MAESTRO** (Converts scATAC-seq open chromatin profiles into a gene activity matrix for multimodal reference mapping)
- **scIB** (Post-hoc evaluation of the mapped embedding via Leiden clustering (resolutions 0.1–2.0) and batch/bio-conservation metrics)

## Examples

```
git clone https://github.com/mcgilldinglab/scCobra.git  # then run the label-transfer workflow in run_integration.ipynb
```

## Evaluation signals

- Query cells cluster adjacent to reference cells of the annotated type in the joint UMAP (e.g. newly formed OD and OD Immature cells correctly adjacent in the integrated embedding)
- Pre-trained KNN classifier predictions on query cells agree with known/query labels without any retraining step
- scIB metrics: scCobra embedding ranks first on batch mixing/Batch-ASW and on summed biological conservation versus Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER
- Preprocessing invariants hold: cells with <200 genes, genes in <3 cells, and >5% mitochondrial fraction are removed before embedding
- Batch-correction risk check: biological differences between batches (e.g. healthy vs disease states) are preserved rather than over-corrected

## Limitations

- Label transfer requires reference and query to originate from the same tissue type or contain similar cell types to ensure optimal performance
- Multimodal (scATAC-seq) mapping relies on converting peak matrices to gene activity matrices, contingent on the conversion algorithm and may lead to information loss
- Shared-encoder design requires selecting shared features between modalities; CITE-seq protein data with only a few hundred features is challenging to optimize

## Evidence

- [abstract] It enables online label transfer across datasets with batch effects, allowing continuous integration of new data without retraining.: "It enables online label transfer across datasets with batch effects, allowing continuous integration of new data without retraining."
- [discussion] Label transfer tissue requirement: "the label transfer performed by scCobra requires that the reference and query datasets originate from the same tissue type or contain similar cell types"
- [discussion] KNN classifier annotation of query data: [evidence span withheld — anchor status: not_found]
- [readme] README run instructions: "git clone https://github.com/mcgilldinglab/scCobra.git"
