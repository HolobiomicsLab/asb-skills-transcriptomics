---
name: single-cell-rna-seq-quality-control-filtering
description: Use when raw single-cell RNA-seq count data must be filtered (mitochondrial
  content, gene/cell detection thresholds) and normalized (TPM, log1p, HVG selection)
  prior to trajectory-preserving feature selection or trajectory inference.
when_to_use_negative:
- The data are already available as a preprocessed .h5ad file (e.g., from the DELVE
  Zenodo record at zenodo.org/records/10534873) — re-filtering would double-process
  and alter the published analysis
- The data are not scRNA-seq (e.g., iterative immunofluorescence imaging), where TPM
  normalization and gene-detection cutoffs do not apply
- A single-cell dataset where removing low-variance or low-detection features would
  discard lineage-specific features relevant to the trajectory of interest
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scanpy
  role: Python library (>=1.8.1) used for reading .h5ad inputs and performing cell/gene
    QC filtering, normalization, and highly variable gene selection
  repo: https://github.com/jranek/delve
- name: DELVE (delve-fs)
  role: Downstream consumer of the QC'd .h5ad matrix; the filtered and HVG-selected
    data feed DELVE feature-module identification
  repo: https://github.com/jranek/delve
- name: SketchKH
  role: Companion sketching tool that operates on preprocessed .h5ad objects; QC-filtered
    samples are the input for distribution-aware kernel-herding subsampling (num_subsamples=1000
    in the DELVE workflow)
  repo: https://github.com/CompCy-lab/SketchKH
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:05cfa63d57ebf09e6163088c75a7a19e361d97b489821e80b2701f541050e850
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE (delve-fs)
  - Python
  - SketchKH
  - UMAP (uniform manifold approximation and projection)
  - scanpy
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# single-cell-rna-seq-quality-control-filtering

## Summary

Quality-control and normalization pipeline for single-cell RNA-seq count matrices (e.g., mouse splenic CD8+ T cells from GEO GSE131847) that removes low-quality cells/genes, TPM-normalizes, log1p-transforms, and selects highly variable genes to produce a clean cells x genes matrix suitable for DELVE feature-module identification.

## When to use

Use when raw single-cell RNA-seq count data must be filtered (mitochondrial content, gene/cell detection thresholds) and normalized (TPM, log1p, HVG selection) prior to trajectory-preserving feature selection or trajectory inference.

## When NOT to use

- The data are already available as a preprocessed .h5ad file (e.g., from the DELVE Zenodo record at zenodo.org/records/10534873) — re-filtering would double-process and alter the published analysis
- The data are not scRNA-seq (e.g., iterative immunofluorescence imaging), where TPM normalization and gene-detection cutoffs do not apply
- A single-cell dataset where removing low-variance or low-detection features would discard lineage-specific features relevant to the trajectory of interest

## Inputs

- Raw single-cell RNA-seq count matrix (e.g., GEO accession GSE131847 mouse splenic CD8+ T cell LCMV counts)
- Per-cell mitochondrial read fractions
- Gene and cell detection counts (genes per cell, cells per gene, counts per gene)

## Outputs

- Preprocessed .h5ad annotated data object with filtered cells/genes, TPM-normalized and log1p-transformed expression (reference dimensions: 29893 cells x 2000 highly variable genes)

## How to apply

Download raw counts (e.g., mouse splenic CD8+ T cell LCMV data from GEO accession GSE131847) and remove cells with >20% mitochondrial reads and genes observed in <3 cells or with <400 counts. TPM-normalize the filtered matrix, apply a log+1 transform, and then apply highly variable gene selection to retain the target feature space (in the reference run this yielded 29893 cells x 2000 genes). Store the result as an .h5ad annotated data object, since DELVE and its dependencies (scanpy, anndata) read preprocessed .h5ad files. The rationale is that filtering noisy, information-poor, or irrelevant features is a crucial preprocessing step before cell-state modeling, so that downstream DELVE clustering (k=10 neighbors, 1000 sketched neighborhoods, n_clusters=5) operates on informative dynamic modules rather than confounding signals.

## Related tools

- **scanpy** (Python library (>=1.8.1) used for reading .h5ad inputs and performing cell/gene QC filtering, normalization, and highly variable gene selection) — https://github.com/jranek/delve
- **DELVE (delve-fs)** (Downstream consumer of the QC'd .h5ad matrix; the filtered and HVG-selected data feed DELVE feature-module identification) — https://github.com/jranek/delve
- **SketchKH** (Companion sketching tool that operates on preprocessed .h5ad objects; QC-filtered samples are the input for distribution-aware kernel-herding subsampling (num_subsamples=1000 in the DELVE workflow)) — https://github.com/CompCy-lab/SketchKH

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('cd8_tcell_lcmv.h5ad'); sc.pp.filter_cells(adata, min_genes=3); sc.pp.filter_genes(adata, min_cells=3); sc.pp.normalize_total(adata, target_sum=1e6); sc.pp.log1p(adata); sc.pp.highly_variable_genes(adata, n_top_genes=2000)
```

## Evaluation signals

- No cells remain with >20% mitochondrial reads and no genes remain that were observed in <3 cells or had <400 counts
- Output matrix dimensions match the reference preprocessed dataset (29893 cells x 2000 genes after HVG selection)
- The output loads cleanly as an .h5ad object compatible with DELVE dependencies (anndata >= 0.7.6, scanpy >= 1.8.1)
- Downstream DELVE run on the QC'd matrix recovers three dynamic modules (early activation/proliferation, effector, memory) plus one static module
- Expression values are TPM-normalized and log+1 transformed (non-negative, log-scale) rather than raw integer counts

## Limitations

- Feature filtering and selection can greatly bias interpretation of the underlying cellular trajectory, so careful consideration is needed before applying cutoffs
- Removing low-variance or low-detection features may be insensitive to lineage-specific features (e.g., transcription factors) that have small variance and gradual expression progression

## Evidence

- [other] mitochondrial and gene-count thresholds from the source card: [evidence span withheld — anchor status: not_found]
- [intro] article rationale for filtering before trajectory analysis: "appropriate filtering of noisy, information-poor, or irrelevant features can serve as a crucial and necessary step for cell type identification, inference of dynamic phenotypes"
- [readme] README on preprocessed .h5ad data access: "You can download all of the preprocessed single-cell datasets (`.h5ad` files) from the [Zenodo](https://zenodo.org/records/10534873) repository."
- [discussion] article caution on feature selection bias: "feature selection can greatly bias the interpretation of the underlying cellular trajectory45, thus careful consideration should be made when performing feature selection for trajectory analysis"
