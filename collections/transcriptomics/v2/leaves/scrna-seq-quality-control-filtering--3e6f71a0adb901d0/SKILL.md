---
name: scrna-seq-quality-control-filtering
description: Use when starting from raw single-cell count matrices (e.g. .h5ad from
  GEO deposits like GSE149614 or Figshare-hosted atlas datasets) that need cleaning
  before batch correction, clustering, or differential expression.
when_to_use_negative:
- Input is not a raw per-cell count matrix (e.g. already normalized, already batch-corrected,
  or a bulk RNA-seq sample) — the cell/gene/mitochondrial filters assume raw counts
- The analysis goal is the integration or benchmarking itself (e.g. computing ARI/NMI/ASW
  with scIB) rather than the upstream preprocessing that feeds it
- Data was converted to a derived representation such as a gene activity matrix from
  scATAC-seq via MAESTRO, where cell/gene filtering rules no longer apply to the original
  modality
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Scanpy
  role: Implements the cell/gene filtering, normalization, log1p transformation, and
    highly variable gene selection on AnnData objects
- name: Seurat v3.2.3
  role: Reference convention for selecting 2000 highly variable genes with the 'vst'
    method
- name: scCobra
  role: Downstream consumer of the QC-filtered, normalized matrix for batch correction
    and integration
  repo: https://github.com/mcgilldinglab/scCobra
- name: scIB
  role: Downstream evaluation pipeline applied to embeddings built from the filtered
    data
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:2db9527184de90b8df19ec21269fd52227c927c8c4b94342263bc4bc0432166a
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
  - Seurat v3.2.3
  - scCobra
  - scDML
  - scDREAMER
  - scIB
  - scVI
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# scrna-seq-quality-control-filtering

## Summary

Single-cell RNA-seq quality control and preprocessing as used in the scCobra benchmarks: remove low-quality cells (<200 expressed genes), lowly detected genes (<3 cells), cells with mitochondrial fraction >5%, then log1p-normalize and select 2000 highly variable genes with the 'vst' method (Seurat v3.2.3 convention) before batch correction or integration.

## When to use

Use when starting from raw single-cell count matrices (e.g. .h5ad from GEO deposits like GSE149614 or Figshare-hosted atlas datasets) that need cleaning before batch correction, clustering, or differential expression.

## When NOT to use

- Input is not a raw per-cell count matrix (e.g. already normalized, already batch-corrected, or a bulk RNA-seq sample) — the cell/gene/mitochondrial filters assume raw counts
- The analysis goal is the integration or benchmarking itself (e.g. computing ARI/NMI/ASW with scIB) rather than the upstream preprocessing that feeds it
- Data was converted to a derived representation such as a gene activity matrix from scATAC-seq via MAESTRO, where cell/gene filtering rules no longer apply to the original modality

## Inputs

- Raw single-cell gene count matrix (AnnData .h5ad, e.g. from GSE149614 or the Figshare lung/immune/pancreas datasets)
- Cell x gene count matrix with gene symbols needed to compute mitochondrial gene fraction (MT- prefixed genes)

## Outputs

- Filtered AnnData object with cells <200 genes, genes <3 cells, and cells >5% mitochondrial fraction removed
- Log1p-normalized expression matrix with 2000 highly variable genes flagged for downstream analysis
- Corrected/cleaned .h5ad files ready for batch correction (scCobra, Harmony, Scanorama, scVI) or scIB evaluation

## How to apply

Load the count matrix into an AnnData object and apply the paper's exact cutoffs with Scanpy: filter out cells expressing fewer than 200 genes, filter genes detected in fewer than 3 cells, and remove cells where mitochondrial genes account for more than 5% of total gene expression. Normalize the counts and apply a log1p transformation to stabilize variance. For downstream benchmarking or integration (scCobra, Harmony, Scanorama, scVI), identify 2000 highly variable genes using the 'vst' method as in Seurat v3.2.3 (Scanpy's flavor='seurat_v3' operates on raw counts). The same preprocessing recipe was applied identically across the lung atlas (16 batches), immune bone marrow (10 batches), pancreas (8 batches), and the two healthy liver cancer samples HCC03N/HCC04N, so parameters should not be tuned per dataset when reproducing the comparison.

## Related tools

- **Scanpy** (Implements the cell/gene filtering, normalization, log1p transformation, and highly variable gene selection on AnnData objects)
- **Seurat v3.2.3** (Reference convention for selecting 2000 highly variable genes with the 'vst' method)
- **scCobra** (Downstream consumer of the QC-filtered, normalized matrix for batch correction and integration) — https://github.com/mcgilldinglab/scCobra
- **scIB** (Downstream evaluation pipeline applied to embeddings built from the filtered data)

## Examples

```
sc.pp.filter_cells(adata, min_genes=200); sc.pp.filter_genes(adata, min_cells=3); adata = adata[adata.obs.pct_counts_mt <= 0.05]; sc.pp.normalize_total(adata); sc.pp.log1p(adata); sc.pp.highly_variable_genes(adata, n_top_genes=2000, flavor='seurat_v3')
```

## Evaluation signals

- After sc.pp.filter_cells(min_genes=200) and sc.pp.filter_genes(min_cells=3), no cell in the object expresses fewer than 200 genes and no gene is detected in fewer than 3 cells
- Every retained cell has mitochondrial gene fraction ≤ 5% of total expression
- Exactly 2000 highly variable genes are flagged (adata.var['highly_variable']) using the 'vst'/seurat_v3 flavor
- The preprocessed matrix is log1p-transformed (check max of adata.X against raw counts) and can be fed unchanged into all seven batch-correction methods for a fair benchmark
- The same filtered cell/gene counts should reproduce across the lung atlas, immune, and pancreas datasets when re-running the scCobra repository notebooks

## Limitations

- The cutoffs (200 genes, 3 cells, 5% mitochondrial fraction) are fixed defaults applied uniformly across the paper's benchmarked datasets; datasets with different depth or tissue-specific mitochondrial expression may need re-justified thresholds
- Filtering is destructive — removed cells and genes cannot be recovered, so thresholds should be chosen before batch correction since they directly change the benchmark inputs for scCobra, Seurat, Harmony, Scanorama, scVI, scDML, and scDREAMER

## Evidence

- [discussion] Cells with fewer than 200 expressed genes removed: "we first filtered out cells that expressed fewer than 200 genes"
- [discussion] Genes found in fewer than 3 cells removed: "and genes that were detected in fewer than 3 cells"
- [discussion] Mitochondrial fraction cutoff: "we removed cells where mitochondrial genes accounted for more than 5% of the total gene expression"
- [discussion] HVG selection convention: [evidence span withheld — anchor status: not_found]
- [readme] Environment setup for filtering pipeline: "pip install scanpy scib"
