---
name: scatac-seq-gene-activity-matrix-conversion
description: Use when integrating paired or unpaired scRNA-seq and scATAC-seq data
  (e.g. the 10X Genomics PBMC multi-omic dataset) with tools that operate on gene-by-cell
  matrices rather than peak matrices.
when_to_use_negative:
- The integration modality is CITE-seq protein data, which contains only a few hundred
  features and makes identifying shared features between modalities difficult
- You need chromatin accessibility information at peak/region resolution rather than
  gene-level summaries — the conversion to a gene activity matrix may lead to information
  loss
- The input is a single scRNA-seq dataset with only sequencing-platform batch effects
  and no second modality
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3170
tools:
- name: MAESTRO
  role: Converts the scATAC-seq open chromatin peak matrix into a gene activity matrix
    for multi-omic integration
- name: scCobra
  role: Integrates the RNA batch with the ATAC-derived gene activity matrix using
    contrastive learning with domain adaptation
  repo: github.com/mcgilldinglab/scCobra
- name: Scanpy
  role: 'Preprocessing: filter cells/genes, normalize counts per cell to 10,000, and
    apply log1p transformation'
- name: Seurat
  role: Benchmark integration method (v3.2.3) run on the combined matrix with 2000
    HVGs via the 'vst' method
- name: scVI
  role: Benchmark integration method (scvi-tools v0.11) with the 'n_batch' parameter
    to model the two modalities as batches
- name: scIB
  role: Leiden clustering (resolutions 0.1–2.0 in 0.1 increments) and computation
    of Batch mixing score, Batch ASW, ARI, NMI, and CellType ASW
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:015ef1bd3e4c4076637d7abc05c45e5f5c8ce023923ddb094e03cf585f6b3d7b
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
  - scCobra
  - scDML
  - scDREAMER
  - scIB
  - scVI
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# scatac-seq-gene-activity-matrix-conversion

## Summary

Converts an scATAC-seq open chromatin peak matrix into a gene activity matrix using the MAESTRO package so that ATAC data can be treated as an expression-like batch and integrated with scRNA-seq data by methods such as scCobra, scVI, and Seurat.

## When to use

Use when integrating paired or unpaired scRNA-seq and scATAC-seq data (e.g. the 10X Genomics PBMC multi-omic dataset) with tools that operate on gene-by-cell matrices rather than peak matrices.

## When NOT to use

- The integration modality is CITE-seq protein data, which contains only a few hundred features and makes identifying shared features between modalities difficult
- You need chromatin accessibility information at peak/region resolution rather than gene-level summaries — the conversion to a gene activity matrix may lead to information loss
- The input is a single scRNA-seq dataset with only sequencing-platform batch effects and no second modality

## Inputs

- scATAC-seq open chromatin peak matrix (e.g. Cell Ranger example data from 10X Genomics PBMC multi-omic dataset)
- Paired scRNA-seq gene-by-cell count matrix
- MAESTRO Python/R package installation

## Outputs

- Gene activity matrix derived from the scATAC-seq peak matrix
- Combined RNA + ATAC gene activity matrix (two batches) restricted to 2000 highly variable genes
- Integrated embedding and scIB metrics (Batch mixing score, Batch ASW, ARI, NMI, CellType ASW) saved as batch_integration_metrics.csv

## How to apply

After standard preprocessing (filter cells expressing fewer than 200 genes, filter genes detected in fewer than 3 cells, remove cells with mitochondrial fraction above 5%, normalize counts per cell to 10,000 with log1p using Scanpy), take the scATAC-seq open chromatin peak matrix and convert it into a gene activity matrix with the MAESTRO package. Treat the RNA and ATAC gene activity matrices as two batches, select 2000 highly variable genes, and feed the combined matrix into the integration method (e.g. scCobra with three-phase training, Seurat v3.2.3 with 'vst' HVG selection, or scvi-tools v0.11 with the 'n_batch' parameter). The rationale is that most integration methods cannot operate directly on peak space, and MAESTRO projects accessibility onto genes so modality differences become batch effects that batch-correction methods can handle.

## Related tools

- **MAESTRO** (Converts the scATAC-seq open chromatin peak matrix into a gene activity matrix for multi-omic integration)
- **scCobra** (Integrates the RNA batch with the ATAC-derived gene activity matrix using contrastive learning with domain adaptation) — github.com/mcgilldinglab/scCobra
- **Scanpy** (Preprocessing: filter cells/genes, normalize counts per cell to 10,000, and apply log1p transformation)
- **Seurat** (Benchmark integration method (v3.2.3) run on the combined matrix with 2000 HVGs via the 'vst' method)
- **scVI** (Benchmark integration method (scvi-tools v0.11) with the 'n_batch' parameter to model the two modalities as batches)
- **scIB** (Leiden clustering (resolutions 0.1–2.0 in 0.1 increments) and computation of Batch mixing score, Batch ASW, ARI, NMI, and CellType ASW)

## Examples

```
git clone https://github.com/mcgilldinglab/scCobra.git; python -c "import scanpy as sc; adata=sc.read_h5ad('pbmc_atac_peaks.h5ad'); sc.pp.normalize_total(adata, target_sum=1e4); sc.pp.log1p(adata)"
```

## Evaluation signals

- The converted ATAC matrix has the same cell count as the input peak matrix and gene-level features matching the 2000 selected highly variable genes
- scIB metrics computed on the integrated embedding (Batch mixing score, Batch ASW, ARI, NMI, CellType ASW) reproduce the Fig. 4b/4c ranking where scCobra, scVI, and Seurat integrate multi-omic data effectively
- Aggregated summed ARI+NMI+CellType ASW score for scCobra exceeds those of Harmony, Scanorama, and scDREAMER on the PBMC multi-omic dataset
- Leiden clustering sweeps resolutions 0.1–2.0 in 0.1 increments without producing degenerate single-cluster or all-singleton solutions
- Full comparison table is saved as batch_integration_metrics.csv

## Limitations

- The gene activity matrix conversion is contingent upon the algorithm used for the conversion process and may potentially lead to information loss
- Benchmarking methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were run with default parameters, which may not reflect their tuned performance
- Over-correction and integration scores are derived from two-dimensional UMAP embeddings (SCALEX methodology), introducing dependence on UMAP

## Evidence

- [discussion] Conversion of scATAC-seq to gene activity matrix with MAESTRO for multimodal integration: "we converted the open chromatin profiles from scATAC-seq into a gene activity matrix utilizing the MAESTRO package"
- [results] Multi-omic benchmark finding: [evidence span withheld — anchor status: not_found]
- [discussion] Filtering thresholds: "we first filtered out cells that expressed fewer than 200 genes and genes that were detected in fewer than 3 cells"
- [discussion] Multi-omic preprocessing normalization: "We normalized the total counts per cell to 10,000 and applied a log1p transformation"
- [readme] README environment setup: "pip install scanpy scib
# You can install addtional packages: https://scib.readthedocs.io/en/latest/index.html"
- [intro] Integration method benchmark scope: "The most widely used methods include Seurat13, Harmony14, Scanorama15, scVI16, scDML17, and scDREAMER18."
