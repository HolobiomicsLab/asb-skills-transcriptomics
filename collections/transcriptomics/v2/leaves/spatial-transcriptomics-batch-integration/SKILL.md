---
name: spatial-transcriptomics-batch-integration
description: Use when integrating a spatial transcriptomics dataset (e.g. MERFISH
  on mouse hypothalamus) with a matched scRNA-seq reference, needing batch-effect
  removal that preserves biological signal, or benchmarking integration methods with
  scIB metrics.
when_to_use_negative:
- When the reference and query datasets originate from different tissue types or contain
  dissimilar cell types — scCobra's label transfer requires shared tissue/cell-type
  composition for optimal performance
- When the modality pair has few shared features (e.g. CITE-seq protein data with
  only a few hundred features), making it challenging to identify shared features
  and optimize the shared-encoder model
- When the input is already a batch-corrected embedding rather than raw shared-gene
  count matrices
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3934
- http://edamontology.org/topic_3170
tools:
- name: scCobra
  role: 'Primary method: contrastive cell embedding learning with domain adaptation
    for the spatial/scRNA-seq integration'
  repo: https://github.com/mcgilldinglab/scCobra
- name: Seurat
  role: Benchmark method (v3.2.3); also used for 2000 highly variable gene selection
    with the 'vst' method
- name: Harmony
  role: Benchmark batch-correction method run with default parameters
- name: Scanorama
  role: Benchmark batch-correction method; Scanpy used for preprocessing (filtering,
    log1p normalization, HVG selection)
- name: scVI
  role: Benchmark VAE-based batch correction method (scvi-tools v0.11, 'n_batch' parameter
    adjusted)
- name: scDML
  role: Benchmark batch-correction method run with default parameters
- name: scDREAMER
  role: Benchmark deep-learning batch-correction method run with default parameters
- name: scIB
  role: 'Evaluation pipeline: Leiden clustering across resolutions 0.1–2.0 (increments
    of 0.1) to compute Batch-ASW and biological conservation'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:e51b17da26f12244ce2ab0b90b2dec6ac810b7eaa7cfc080b43e1504b4ba2872
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scCobra
  tools:
  - Harmony
  - Scanorama
  - Seurat
  - scCobra
  - scDML
  - scDREAMER
  - scIB
  - scVI
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3934
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spatial-transcriptomics-batch-integration

## Summary

Integrates spatial transcriptomics data (e.g. MERFISH) with matched scRNA-seq by restricting both matrices to shared genes and applying scCobra's contrastive cell embedding learning with domain adaptation, benchmarked against Seurat, Harmony, Scanorama, scVI, scDML, and scDREAMER using the scIB pipeline.

## When to use

Use when integrating a spatial transcriptomics dataset (e.g. MERFISH on mouse hypothalamus) with a matched scRNA-seq reference, needing batch-effect removal that preserves biological signal, or benchmarking integration methods with scIB metrics.

## When NOT to use

- When the reference and query datasets originate from different tissue types or contain dissimilar cell types — scCobra's label transfer requires shared tissue/cell-type composition for optimal performance
- When the modality pair has few shared features (e.g. CITE-seq protein data with only a few hundred features), making it challenging to identify shared features and optimize the shared-encoder model
- When the input is already a batch-corrected embedding rather than raw shared-gene count matrices

## Inputs

- Mouse hypothalamus MERFISH spatial gene expression matrix
- Matched scRNA-seq reference expression matrix
- Both matrices restricted to shared genes (2000 highly variable genes via Seurat 'vst' method)

## Outputs

- Integrated cell embeddings from scCobra and the six benchmark methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER)
- Per-method scIB metric table (batch mixing/Batch-ASW and biological conservation), e.g. hypothalamus_integration_benchmark.csv
- Fig. 4e/4f-style comparison plots of batch-effect removal vs biological conservation
- Leiden cluster embeddings verifying spatially-derived cell types (e.g. OD and OD Immature cells) are correctly adjacent

## How to apply

Load the spatial (MERFISH) and scRNA-seq matrices and restrict both to shared genes as input features. Preprocess with Scanpy: filter cells expressing fewer than 200 genes, genes detected in fewer than 3 cells, and cells with mitochondrial fraction above 5%; normalize with log1p and select 2000 highly variable genes with the Seurat 'vst' method. Run scCobra, a deep generative neural network combining contrastive learning with domain adaptation that makes no gene-expression distribution assumptions, to learn integrated cell embeddings; run the six benchmark methods (Seurat v3.2.3, Harmony, Scanorama, scVI (scvi-tools v0.11), scDML, scDREAMER) with default parameters on the same input. Evaluate all seven embeddings with the scIB pipeline, applying Leiden clustering across resolutions 0.1–2.0 in increments of 0.1 to compute batch mixing/Batch-ASW and summed biological conservation. In the mouse hypothalamus MERFISH–scRNA-seq benchmark, scCobra ranked first on both metrics, so it is the preferred method when batch-effect removal and biological conservation are both required.

## Related tools

- **scCobra** (Primary method: contrastive cell embedding learning with domain adaptation for the spatial/scRNA-seq integration) — https://github.com/mcgilldinglab/scCobra
- **Seurat** (Benchmark method (v3.2.3); also used for 2000 highly variable gene selection with the 'vst' method)
- **Harmony** (Benchmark batch-correction method run with default parameters)
- **Scanorama** (Benchmark batch-correction method; Scanpy used for preprocessing (filtering, log1p normalization, HVG selection))
- **scVI** (Benchmark VAE-based batch correction method (scvi-tools v0.11, 'n_batch' parameter adjusted))
- **scDML** (Benchmark batch-correction method run with default parameters)
- **scDREAMER** (Benchmark deep-learning batch-correction method run with default parameters)
- **scIB** (Evaluation pipeline: Leiden clustering across resolutions 0.1–2.0 (increments of 0.1) to compute Batch-ASW and biological conservation)

## Examples

```
git clone https://github.com/mcgilldinglab/scCobra.git; conda create -n scCobra conda-forge::python=3.10; pip install scanpy scib; python -m ipykernel install --user; jupyter notebook run_integration.ipynb
```

## Evaluation signals

- scCobra ranks first on both scIB metrics (batch mixing/Batch-ASW and summed biological conservation) across the seven methods
- Leiden clustering across resolutions 0.1–2.0 (increments of 0.1) yields stable cluster structure for metric computation
- Newly formed OD and OD Immature cell types are correctly adjacent in the integrated embedding
- Preprocessing invariants hold: cells with <200 genes, genes in <3 cells, and cells with >5% mitochondrial fraction are removed before normalization
- Seurat, Harmony, Scanorama, scDML, and scVI perform similarly on batch mixing while scCobra significantly outperforms all on biological conservation (Fig. 4e/4f pattern reproduced)

## Limitations

- Benchmarking methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were run with default parameters, which may not reflect their tuned performance
- Label transfer requires reference and query datasets to originate from the same tissue type or contain similar cell types
- Shared-encoder design requires selecting shared features between modalities; CITE-seq protein data with only a few hundred features makes this challenging
- Multimodal integration relies on converting scATAC-seq peak matrices into gene activity matrices, contingent on the conversion algorithm and may lead to information loss
- Over-correction scores are derived from two-dimensional UMAP embeddings, introducing dependence on UMAP

## Evidence

- [results] Seurat, Harmony, Scanorama, scDML, and scVI perform similarly, whereas scCobra demonstrates the best performance in handling batch effects (Fig. 4e): [evidence span withheld — anchor status: not_found]
- [methods] three-phase training optimization: "The scIB pipeline applied Leiden clustering across a range of resolutions from 0.1 to 2.0 (in increments of 0.1)"
- [discussion] filtering thresholds: "we first filtered out cells that expressed fewer than 200 genes"
- [discussion] HVG selection: [evidence span withheld — anchor status: not_found]
- [discussion] benchmark default parameters: "we used Seurat, Harmony, Scanorama, scVI, scDML, and scDREAMER as benchmarking methods with default parameters"
- [abstract] scCobra design: "scCobra, a deep generative neural network designed to overcome these challenges through contrastive learning with domain adaptation"
- [discussion] label transfer limitation: "the label transfer performed by scCobra requires that the reference and query datasets originate from the same tissue type or contain similar cell types"
- [readme] installation/run: [evidence span withheld — anchor status: external_doc]
