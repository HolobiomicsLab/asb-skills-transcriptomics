---
name: highly-variable-gene-selection
description: Use when preparing raw scRNA-seq count matrices (e.g. h5ad from a multi-batch
  atlas) for integration or clustering and you need to restrict analysis to informative
  genes before embedding methods like scCobra, scVI, or Harmony.
when_to_use_negative:
- The input is already an embedding, feature table, or normalized matrix that has
  previously been subset to variable genes — re-running HVG selection would discard
  information
- The downstream task requires the full gene space in the original feature space (e.g.
  gene-level interpretability of cellular-state differences), since subsetting to
  2000 HVGs drops lowly-expressed genes
- Analysis is bulk RNA-seq or single-cell multimodal data where shared-feature selection
  (e.g. CITE-seq protein panels with only a few hundred features) replaces gene-variability-based
  selection
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat v3.2.3
  role: Reference implementation of the 'vst' variance-stabilizing method used to
    select the 2000 highly variable genes
- name: Scanpy
  role: Performs the upstream QC filtering (cells <200 genes, genes <3 cells, mito
    fraction >5%) and log1p normalization before HVG selection
- name: scCobra
  role: 'Downstream consumer: trains its deep generative network on the 2000-HVG subset
    for batch correction and integration'
  repo: https://github.com/mcgilldinglab/scCobra
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:ff165b72ac81996e6bbf1caa80d48cbb8a15933f1a103010e5774305eb764a87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scCobra
  tools:
  - Harmony
  - LIGER
  - PyTorch
  - Scanorama
  - Scanpy
  - Seurat
  - Seurat v3.2.3
  - scCobra
  - scDML
  - scDREAMER
  - scETM
  - scIB
  - scVAE-GM
  - scVI
  - scVI-LD
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# highly-variable-gene-selection

## Summary

Selects a reduced set of the most variable genes (2000 in this benchmark, using Seurat v3.2.3's 'vst' dispersion-based method via Scanpy) after QC filtering and log1p normalization, so downstream batch-correction/integration methods operate on an informative gene space.

## When to use

Use when preparing raw scRNA-seq count matrices (e.g. h5ad from a multi-batch atlas) for integration or clustering and you need to restrict analysis to informative genes before embedding methods like scCobra, scVI, or Harmony.

## When NOT to use

- The input is already an embedding, feature table, or normalized matrix that has previously been subset to variable genes — re-running HVG selection would discard information
- The downstream task requires the full gene space in the original feature space (e.g. gene-level interpretability of cellular-state differences), since subsetting to 2000 HVGs drops lowly-expressed genes
- Analysis is bulk RNA-seq or single-cell multimodal data where shared-feature selection (e.g. CITE-seq protein panels with only a few hundred features) replaces gene-variability-based selection

## Inputs

- Raw scRNA-seq count matrix (AnnData/h5ad) for a multi-batch dataset, e.g. the lung atlas (32472 cells, 17 cell types, 16 batches), immune (33506 cells, 16 cell types, 10 batches), or pancreas dataset from the scCobra figshare links
- QC-filtered, log1p-normalized expression matrix produced via Scanpy preprocessing

## Outputs

- A set/list of 2000 highly variable gene identifiers per dataset
- A filtered AnnData object whose expression matrix is subset to the 2000 HVGs, ready for integration/batch correction

## How to apply

First preprocess the raw counts with Scanpy: filter cells expressing fewer than 200 genes, filter genes detected in fewer than 3 cells, and remove cells with mitochondrial gene fraction above 5%; then apply log1p normalization to stabilize variance. On the normalized, filtered matrix, identify 2000 highly variable genes using the 'vst' method as implemented in Seurat v3.2.3 (available through Scanpy's flavor options or Seurat itself). Use only the resulting 2000-gene subset as input to the downstream integration/batch-correction step (e.g. scCobra training or scIB evaluation embeddings). The rationale is that batch-correction methods trained on the full gene space waste capacity on uninformative, lowly-expressed genes; the article's benchmark workflow standardizes this step across the lung atlas (16 batches), immune bone marrow (10 batches), and pancreas (8 batches) datasets before comparing scCobra against Seurat, Harmony, Scanorama, scVI v0.11, scDML, and scDREAMER.

## Related tools

- **Seurat v3.2.3** (Reference implementation of the 'vst' variance-stabilizing method used to select the 2000 highly variable genes)
- **Scanpy** (Performs the upstream QC filtering (cells <200 genes, genes <3 cells, mito fraction >5%) and log1p normalization before HVG selection)
- **scCobra** (Downstream consumer: trains its deep generative network on the 2000-HVG subset for batch correction and integration) — https://github.com/mcgilldinglab/scCobra

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('lung_atlas.h5ad'); sc.pp.filter_cells(adata, min_genes=200); sc.pp.filter_genes(adata, min_cells=3); sc.pp.normalize_total(adata); sc.pp.log1p(adata); sc.pp.highly_variable_genes(adata, n_top_genes=2000, flavor='seurat_v3')
```

## Evaluation signals

- Exactly 2000 genes are retained after selection, and every retained gene passed the upstream filters (detected in ≥3 cells)
- Cell count is unchanged by HVG subsetting (only genes are dropped), and cells with mitochondrial fraction >5% were already removed
- The 'vst' flavor is recorded as the dispersion method so results match Seurat v3.2.3 behavior
- Downstream integration metrics (ARI, NMI, CellType ASW, Graph Connectivity, Batch ASW via scIB with Leiden resolutions 0.1–2.0) are computed on embeddings derived from the 2000-HVG matrix
- The same preprocessing + HVG pipeline is applied identically across all benchmarked datasets (lung atlas, immune, pancreas) so method comparisons are not confounded by gene-space differences

## Limitations

- Selecting only 2000 HVGs discards lowly-expressed genes, which conflicts with methods aiming to interpret differences at the gene level in the original feature space
- HVG selection thresholds are dataset-dependent; the 2000-gene/vst choice mirrors Seurat v3.2.3 defaults and was fixed for this benchmark, not optimized per dataset
- For multimodal data (e.g. CITE-seq protein features), variability-based gene selection does not apply since shared-feature selection is required

## Evidence

- [discussion] Seurat (v3.2.3): employed for data normalization and the selection of 2000 highly variable genes using the "vst" method: [evidence span withheld — anchor status: not_found]
- [discussion] we first filtered out cells that expressed fewer than 200 genes and genes that were detected in fewer than 3 cells: "we first filtered out cells that expressed fewer than 200 genes"
- [discussion] This was followed by a log1p transformation to stabilize the variance: "This was followed by a log1p transformation to stabilize the variance"
- [discussion] we identified 2000 highly variable genes for downstream analysis: "we identified 2000 highly variable genes for downstream analysis"
- [readme] pip install scanpy scib: "pip install scanpy scib"
