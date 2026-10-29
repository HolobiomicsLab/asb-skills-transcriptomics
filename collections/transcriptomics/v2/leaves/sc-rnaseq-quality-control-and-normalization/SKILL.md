---
name: sc-rnaseq-quality-control-and-normalization
description: Use when raw single-cell RNA-seq count matrices (e.g., GSE149614 HCC01T/HCC03N
  .h5ad or similar) require QC, normalization, and HVG selection before batch correction
  or DEG analysis.
when_to_use_negative:
- Input is already normalized/log1p-transformed — re-normalizing a scaled matrix would
  double-transform the data
- The goal is multi-omic scATAC-seq processing, which requires a different route (e.g.,
  conversion to a gene activity matrix via MAESTRO in this paper's pipeline)
- Analysis requires retaining low-mito or low-gene cells (e.g., plate-based datasets
  where the 5% mito and 200-gene cutoffs would over-filter)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Scanpy
  role: Performs cell/gene filtering, normalization to 10,000 counts per cell, log1p
    transformation, and 2000-HVG (vst) selection
- name: Seurat v3.2.3
  role: Alternative normalization and highly variable gene selection reference ('vst',
    2000 HVGs) cited by the paper
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:7b92d20e2f73fab8a62b625c4527925df9f8465904669613bd3f2719b598bd7c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - Scanorama
  - Scanpy
  - Seurat
  - Seurat v3.2.3
  - scCobra
  - scVI
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# sc-rnaseq-quality-control-and-normalization

## Summary

Preprocessing skill for raw scRNA-seq count matrices: cell/gene QC filtering, count-depth normalization to 10,000 per cell with log1p stabilization, and highly variable gene selection, preparing data for downstream batch correction such as scCobra.

## When to use

Use when raw single-cell RNA-seq count matrices (e.g., GSE149614 HCC01T/HCC03N .h5ad or similar) require QC, normalization, and HVG selection before batch correction or DEG analysis.

## When NOT to use

- Input is already normalized/log1p-transformed — re-normalizing a scaled matrix would double-transform the data
- The goal is multi-omic scATAC-seq processing, which requires a different route (e.g., conversion to a gene activity matrix via MAESTRO in this paper's pipeline)
- Analysis requires retaining low-mito or low-gene cells (e.g., plate-based datasets where the 5% mito and 200-gene cutoffs would over-filter)

## Inputs

- Raw scRNA-seq count matrix (e.g., .h5ad or 10x-style) per sample/batch, e.g., GSE149614 HCC01T and HCC03N
- Cell-level mitochondrial gene annotation (list of mitochondrial gene IDs)

## Outputs

- QC-filtered AnnData object (cells ≥200 genes, genes in ≥3 cells, mito fraction ≤5%)
- Log1p-normalized expression matrix with counts-per-cell scaled to 10,000
- Set of 2000 highly variable genes (vst) used for downstream analysis

## How to apply

Starting from raw count matrices, filter out cells expressing fewer than 200 genes, genes detected in fewer than 3 cells, and cells whose mitochondrial genes account for more than 5% of total gene expression. Normalize total counts per cell to 10,000 and apply a log1p transformation to stabilize variance, using Scanpy (the article also references Seurat v3.2.3-style normalization). Identify 2000 highly variable genes using the 'vst' method for downstream analysis. These QC/normalization settings are the preprocessing stage that precedes embedding-based batch correction (scCobra, Seurat integration, Harmony, Scanorama, scVI) and tumor-vs-normal DEG calling with Omicverse; decide on the 5% mitochondrial threshold per tissue, since it was calibrated on liver cancer tumor/normal samples.

## Related tools

- **Scanpy** (Performs cell/gene filtering, normalization to 10,000 counts per cell, log1p transformation, and 2000-HVG (vst) selection)
- **Seurat v3.2.3** (Alternative normalization and highly variable gene selection reference ('vst', 2000 HVGs) cited by the paper)

## Examples

```
sc.pp.filter_cells(adata, min_genes=200); sc.pp.filter_genes(adata, min_cells=3); sc.pp.normalize_total(adata, target_sum=1e4); sc.pp.log1p(adata); sc.pp.highly_variable_genes(adata, n_top_genes=2000, flavor='vst')
```

## Evaluation signals

- After filtering, every remaining cell expresses ≥200 genes and has mitochondrial fraction ≤5%; every remaining gene is detected in ≥3 cells
- Column sums of the pre-log1p matrix equal 10,000 counts per cell
- HVG step returns exactly 2000 genes when flavor='vst' and n_top_genes=2000
- Downstream batch-correction methods (scCobra, Seurat, Harmony, Scanorama, scVI) run without errors on the preprocessed object
- Reproducing the survival pipeline yields per-method log-rank P values in the reported range (e.g., scCobra 4.1e-21) only when preprocessing matches these thresholds

## Limitations

- The 200-gene, 3-cell, and 5% mitochondrial thresholds were tuned on the GSE149614 liver cancer (HCC01T/HCC03N) samples; applying them unchanged to other tissues or chemistries may over-filter or under-filter cells
- The downstream paper reports over-correction risk for integration methods, so overly aggressive QC upstream can compound loss of biologically meaningful (e.g., disease-state) gene signal
- Benchmarking in the source study used default parameters, so QC/normalization settings may not represent tuned performance for other pipelines

## Evidence

- [discussion] cell and gene filters: "we first filtered out cells that expressed fewer than 200 genes"
- [discussion] gene detection filter: "and genes that were detected in fewer than 3 cells"
- [discussion] mitochondrial QC cutoff: "we removed cells where mitochondrial genes accounted for more than 5% of the total gene expression"
- [discussion] normalization and HVG selection: "We normalized the total counts per cell to 10,000 and applied a log1p transformation"
- [discussion] HVG selection: "we identified 2000 highly variable genes for downstream analysis"
- [readme] tooling for preprocessing: "# Install scanpy scib
# You can install addtional packages: https://scib.readthedocs.io/en/latest/index.html"
