---
name: single-cell-embedding-clustering-accuracy-assessment
description: Use when comparing batch-correction methods (e.g., scCobra, Seurat, Harmony,
  Scanorama, scVI, scDML, scDREAMER) on integrated single-cell embeddings where ground-truth
  cell-type labels exist and you need quantitative clustering/bio-conservation and
  batch-mixing metrics.
when_to_use_negative:
- No ground-truth cell-type annotations are available — ARI/NMI/CellType ASW cannot
  be computed and the aggregate score is undefined
- Input is raw counts or a single batch with no integration to evaluate (no batch
  mixing to measure)
- Comparison relies on the original high-dimensional feature space rather than an
  integrated embedding — the metrics here are defined on 2D UMAP embeddings of integrated
  outputs
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: scIB
  role: Pipeline that applies Leiden clustering across the 0.1–2.0 resolution range
    and computes the batch/bio-conservation metrics
- name: Scanpy
  role: Preprocessing and Leiden clustering utilities on the integrated embedding
- name: scCobra
  role: One of the integration methods whose embedding is assessed; three-phase training
    optimization
  repo: github.com/mcgilldinglab/scCobra
- name: Seurat v3.2.3
  role: Benchmark integration method using 2000 HVGs via the 'vst' method
- name: scvi-tools v0.11
  role: Benchmark VAE integration method run with the 'n_batch' parameter
- name: Harmony
  role: Benchmark integration method run with default parameters
- name: Scanorama
  role: Benchmark integration method run with default parameters
- name: scDML
  role: Benchmark integration method run with default parameters
- name: scDREAMER
  role: Benchmark integration method run with default parameters
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:aaff36c8f451b77f32d1b0cffcb832430ee3079ed9dac36619c020b5341f0ba1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - MAESTRO
  - PyTorch
  - Scanorama
  - Scanpy
  - Seurat
  - Seurat v3.2.3
  - scCobra
  - scDML
  - scDREAMER
  - scIB
  - scVI
  - scvi-tools v0.11
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# single-cell-embedding-clustering-accuracy-assessment

## Summary

Benchmarks single-cell batch-integration methods by Leiden-clustering each method's integrated embedding across a resolution sweep and computing scIB metrics (ARI, NMI, Batch ASW, CellType ASW, Batch mixing score) from 2D UMAP embeddings, aggregating them into a summed ARI+NMI+CellType ASW score.

## When to use

Use when comparing batch-correction methods (e.g., scCobra, Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) on integrated single-cell embeddings where ground-truth cell-type labels exist and you need quantitative clustering/bio-conservation and batch-mixing metrics.

## When NOT to use

- No ground-truth cell-type annotations are available — ARI/NMI/CellType ASW cannot be computed and the aggregate score is undefined
- Input is raw counts or a single batch with no integration to evaluate (no batch mixing to measure)
- Comparison relies on the original high-dimensional feature space rather than an integrated embedding — the metrics here are defined on 2D UMAP embeddings of integrated outputs

## Inputs

- Integrated low-dimensional embedding (e.g., .h5ad with obsm embedding) from each integration method (scCobra, Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER)
- Cell-level cell-type ground-truth labels for ARI, NMI, and CellType ASW computation
- Batch/modality labels (e.g., RNA vs. ATAC treated as two batches) for Batch mixing score and Batch ASW

## Outputs

- Per-method metric table: Batch mixing score, Batch ASW, ARI, NMI, CellType ASW (batch_integration_metrics.csv)
- Aggregated summed ARI+NMI+CellType ASW ranking comparable to Fig. 4b/4c
- Leiden cluster assignments across resolutions 0.1–2.0 (increments of 0.1)

## How to apply

Take each method's integrated embedding (e.g., scCobra's three-phase trained embedding, Seurat v3.2.3 on 2000 HVGs via 'vst', scVI scvi-tools v0.11 with 'n_batch'), embed it in 2D with UMAP following the SCALEX methodology, and apply the scIB pipeline to run Leiden clustering across resolutions 0.1 to 2.0 in increments of 0.1. For each resolution, compute the Batch mixing score, Batch ASW, ARI, NMI, and CellType ASW against known cell-type labels. Aggregate the metrics into the summed ARI+NMI+CellType ASW score so batch mixing and bio-conservation are jointly ranked, and save the full comparison table as batch_integration_metrics.csv (reproducing the Fig. 4b/4c ranking in which scCobra, scVI, and Seurat effectively integrate the 10X PBMC multi-omic data).

## Related tools

- **scIB** (Pipeline that applies Leiden clustering across the 0.1–2.0 resolution range and computes the batch/bio-conservation metrics)
- **Scanpy** (Preprocessing and Leiden clustering utilities on the integrated embedding)
- **scCobra** (One of the integration methods whose embedding is assessed; three-phase training optimization) — github.com/mcgilldinglab/scCobra
- **Seurat v3.2.3** (Benchmark integration method using 2000 HVGs via the 'vst' method)
- **scvi-tools v0.11** (Benchmark VAE integration method run with the 'n_batch' parameter)
- **Harmony** (Benchmark integration method run with default parameters)
- **Scanorama** (Benchmark integration method run with default parameters)
- **scDML** (Benchmark integration method run with default parameters)
- **scDREAMER** (Benchmark integration method run with default parameters)

## Examples

```
pip install scanpy scib  # then run Leiden 0.1–2.0 sweep via scIB on each method's UMAP embedding
```

## Evaluation signals

- Metrics are computed from two-dimensional UMAP embeddings following the SCALEX methodology — verify a 2D UMAP embedding exists per method before scoring
- Leiden clustering must sweep resolutions 0.1 to 2.0 in increments of 0.1 (20 resolutions) via the scIB pipeline
- Final output is a batch_integration_metrics.csv table whose aggregate ARI+NMI+CellType ASW ranking reproduces Fig. 4b/4c: scCobra, scVI, and Seurat at the top on 10X PBMC multi-omic data
- Each of the five metrics (Batch mixing score, Batch ASW, ARI, NMI, CellType ASW) must be reported for all seven benchmark methods
- Good integration should show low batch ASW/batch mixing penalties alongside high ARI, NMI, and CellType ASW (bio-conservation without over-correction)

## Limitations

- Benchmark methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were run with default parameters, which may not reflect their tuned performance
- Over-correction scores derived from two-dimensional UMAP embeddings, introducing dependence on UMAP
- Harmony, Scanorama, and scDREAMER struggle with significant modality differences on the 10X PBMC multi-omic dataset, so their low scores partly reflect method limitations rather than metric artifacts

## Evidence

- [discussion] Resolution sweep and metric engine: "The scIB pipeline applied Leiden clustering across a range of resolutions from 0.1 to 2.0 (in increments of 0.1)"
- [discussion] Metric definitions and UMAP dependence: "The scores were derived from two-dimensional UMAP embeddings, following the methodology described in the original SCALEX26 paper"
- [results] Expected ranking outcome: [evidence span withheld — anchor status: not_found]
- [discussion] Default-parameter caveat: "we used Seurat, Harmony, Scanorama, scVI, scDML, and scDREAMER as benchmarking methods with default parameters"
- [readme] Environment install for metric stack: "pip install scanpy scib"
