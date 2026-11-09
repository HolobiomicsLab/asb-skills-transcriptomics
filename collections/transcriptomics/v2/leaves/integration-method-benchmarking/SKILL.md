---
name: integration-method-benchmarking
description: Use when two or more single-cell datasets (scRNA-seq batches or spatial/scRNA
  pairs) must be integrated and a defensible choice among integration methods is needed,
  measured by batch-removal vs biological-conservation metrics.
when_to_use_negative:
- When reference and query datasets originate from different tissue types or lack
  similar cell types — the article notes label transfer requires same-tissue or similar-cell-type
  reference/query
- For CITE-seq or other modalities with only a few hundred protein features, where
  the article notes it is challenging to identify shared features and optimize the
  shared-encoder model
- When the goal is single-dataset analysis without batch structure — no integration
  benchmark is meaningful
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: scCobra
  role: Deep generative network with contrastive learning and domain adaptation; the
    method under test, trained with three-phase optimization
  repo: https://github.com/mcgilldinglab/scCobra
- name: Seurat
  role: Benchmark integration method (v3.2.3); also source of the 'vst' HVG-selection
    convention
- name: Harmony
  role: Benchmark batch-correction method run with default parameters
- name: Scanorama
  role: Benchmark batch-correction method run with default parameters
- name: scVI
  role: Benchmark VAE-based method (scvi-tools v0.11) with n_batch parameter adjusted
- name: scDML
  role: Benchmark batch-correction method run with default parameters
- name: scDREAMER
  role: Benchmark deep-learning integration method run with default parameters
- name: scIB
  role: Evaluation pipeline; Leiden clustering at resolutions 0.1–2.0 to compute batch-removal
    and biological-conservation metrics
- name: Scanpy
  role: 'Preprocessing: cell/gene filtering, log1p normalization, HVG selection'
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:5753512d7d7737fdc6be49fc332f19cc5d595fb3384bad4c3d2665cccad1c50e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scCobra
  tools:
  - Harmony
  - PyTorch
  - Scanorama
  - Scanpy
  - Seurat
  - scCobra
  - scDML
  - scDREAMER
  - scIB
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# integration-method-benchmarking

## Summary

Benchmark single-cell data integration methods (scCobra vs Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) on batch-effect removal and biological conservation using the scIB metric pipeline, as done for scRNA-seq atlases (lung, immune, pancreas) and MERFISH–scRNA-seq hypothalamus integration.

## When to use

Use when two or more single-cell datasets (scRNA-seq batches or spatial/scRNA pairs) must be integrated and a defensible choice among integration methods is needed, measured by batch-removal vs biological-conservation metrics.

## When NOT to use

- When reference and query datasets originate from different tissue types or lack similar cell types — the article notes label transfer requires same-tissue or similar-cell-type reference/query
- For CITE-seq or other modalities with only a few hundred protein features, where the article notes it is challenging to identify shared features and optimize the shared-encoder model
- When the goal is single-dataset analysis without batch structure — no integration benchmark is meaningful

## Inputs

- scRNA-seq batch datasets as gene-by-cell count matrices (e.g. lung atlas: 32472 cells, 17 cell types, 16 batches; immune: 33506 cells, 16 cell types, 10 batches; pancreas: 16382 cells, 14 cell types, 9 batches; downloadable from the scCobra figshare links)
- MERFISH spatial dataset plus matched scRNA-seq reference restricted to shared genes (mouse hypothalamus)
- Scanpy AnnData/h5ad objects after QC filtering and log1p normalization with 2000 HVGs (vst)

## Outputs

- Per-dataset, per-method metric table (ARI, NMI, CellType ASW, Graph Connectivity, Batch ASW) e.g. benchmark_metrics.csv
- Integration metric table for spatial/scRNA integration, e.g. hypothalamus_integration_benchmark.csv
- Fig. 2c-f-style and Fig. 4e/4f-style benchmark comparison plots
- Learned cell embeddings (low-dimensional matrices) from each of the seven methods

## How to apply

Restrict all datasets to shared genes (for spatial/MERFISH–scRNA pairs, use shared genes as input features), then preprocess each with Scanpy: filter cells expressing fewer than 200 genes, genes detected in fewer than 3 cells, cells with mitochondrial fraction above 5%, apply log1p normalization, and select 2000 highly variable genes with the Seurat 'vst' method (Seurat v3.2.3 convention). Embed the data once with scCobra (three-phase contrastive-learning + domain-adaptation training, torch backend) and once with each benchmark method run with default parameters — Seurat v3.2.3, Harmony, Scanorama, scVI (scvi-tools v0.11, adjust n_batch), scDML, scDREAMER. Evaluate all embeddings with the scIB pipeline, applying Leiden clustering across resolutions 0.1 to 2.0 in increments of 0.1 to compute batch-removal metrics (Batch ASW / batch mixing) and biological-conservation metrics (ARI, NMI, CellType ASW, Graph Connectivity). Assemble per-dataset, per-method scores into a table; in the scCobra paper, scCobra ranks first on both metric families. The rationale for benchmarking scCobra is that MNN-style methods tend to over-correct, while distribution-assuming deep methods (e.g. scVI) lack a universal gene-expression distribution.

## Related tools

- **scCobra** (Deep generative network with contrastive learning and domain adaptation; the method under test, trained with three-phase optimization) — https://github.com/mcgilldinglab/scCobra
- **Seurat** (Benchmark integration method (v3.2.3); also source of the 'vst' HVG-selection convention)
- **Harmony** (Benchmark batch-correction method run with default parameters)
- **Scanorama** (Benchmark batch-correction method run with default parameters)
- **scVI** (Benchmark VAE-based method (scvi-tools v0.11) with n_batch parameter adjusted)
- **scDML** (Benchmark batch-correction method run with default parameters)
- **scDREAMER** (Benchmark deep-learning integration method run with default parameters)
- **scIB** (Evaluation pipeline; Leiden clustering at resolutions 0.1–2.0 to compute batch-removal and biological-conservation metrics)
- **Scanpy** (Preprocessing: cell/gene filtering, log1p normalization, HVG selection)

## Examples

```
git clone https://github.com/mcgilldinglab/scCobra.git && pip install scanpy scib && pip3 install torch torchvision torchaudio torch_optimizer
```

## Evaluation signals

- scCobra attains the highest scores on biological-conservation metrics (ARI, NMI, CellType ASW) and Graph Connectivity averaged over the lung atlas, immune, and pancreas datasets (Fig. 2c-f reproduction)
- In the MERFISH–scRNA hypothalamus integration, scCobra ranks first on both batch-removal (Batch ASW / batch mixing) and biological conservation (Fig. 4e/4f reproduction)
- Leiden clustering for all metrics spans resolutions 0.1 to 2.0 in increments of 0.1 via scIB
- Newly formed OD and OD Immature cells are correctly adjacent in the integrated hypothalamus embedding
- Output tables (benchmark_metrics.csv, hypothalamus_integration_benchmark.csv) contain one row per dataset × method with all five metric columns populated

## Limitations

- Benchmark methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were run with default parameters, which may not reflect their tuned performance
- scIB-based evaluation depends on Leiden clustering resolution sweep; over-correction scores were derived from two-dimensional UMAP embeddings following the SCALEX methodology, introducing UMAP dependence
- Multimodal (scATAC-seq) integration relies on converting peak matrices to gene activity matrices (MAESTRO), which may lead to information loss; label transfer requires same-tissue/similar-cell-type reference and query

## Evidence

- [discussion] Benchmark methods run at default parameters: "we used Seurat, Harmony, Scanorama, scVI, scDML, and scDREAMER as benchmarking methods with default parameters"
- [discussion] scIB Leiden resolution sweep: "The scIB pipeline applied Leiden clustering across a range of resolutions from 0.1 to 2.0 (in increments of 0.1)"
- [discussion] QC and HVG preprocessing thresholds: "Cells expressing fewer than 200 genes and genes found in less than 3 cells were filtered out"
- [methods] Obtain the scCobra repository: "git clone https://github.com/mcgilldinglab/scCobra.git"
