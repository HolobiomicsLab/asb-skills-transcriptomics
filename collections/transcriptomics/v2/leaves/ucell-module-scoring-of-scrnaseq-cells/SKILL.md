---
name: ucell-module-scoring-of-scrnaseq-cells
description: Use when a JOINTLY (or similar NMF-based) joint clustering run has produced
  interpretable gene modules that must be quantified per cell to annotate cell types,
  map factor activity, or compare module gene specificity against traditional markers.
when_to_use_negative:
- When no interpretable gene modules/factor gene sets exist (e.g., using purely embedding-based
  integration like Harmony or scVI without factor gene sets)
- When the goal is batch integration or clustering itself rather than interpretation
  — JOINTLY clustering with FindNeighbors/FindClusters (dims 1:15, res = 0.05) or
  ARI/NMI evaluation via aricode is the appropriate step
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: UCell
  role: Computes per-cell gene module signature scores via AddModuleScore_UCell using
    JOINTLY's learned modules
  repo: https://github.com/carmonalab/UCell
- name: JOINTLY
  role: Produces the interpretable gene modules (factor gene sets) that are scored
    per cell
  repo: https://github.com/madsen-lab/rJOINTLY
- name: Seurat
  role: Hosts the single-cell object, metadata module scores, and UMAP visualization
    via FeaturePlot
  repo: https://github.com/satijalab/seurat
- name: presto
  role: wilcoxauc computes per-cell-type AUC of module genes versus marker genes for
    the specificity comparison
  repo: https://github.com/immunogenomics/presto
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:48669bb3f47cd370db1ba6f5a73ba6c610f83991c3de22ad6eb88f46497f5729
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/UCell
  tools:
  - JOINTLY
  - Seurat
  - UCell
  - aricode
  - presto
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# UCell module scoring of scRNA-seq cells

## Summary

Score every JOINTLY-derived interpretable gene module per single cell using AddModuleScore_UCell from the UCell package, enabling visualization of factor activity (e.g., factor_1_UCell) on UMAP embeddings and downstream cell-type specificity analyses.

## When to use

Use when a JOINTLY (or similar NMF-based) joint clustering run has produced interpretable gene modules that must be quantified per cell to annotate cell types, map factor activity, or compare module gene specificity against traditional markers.

## When NOT to use

- When no interpretable gene modules/factor gene sets exist (e.g., using purely embedding-based integration like Harmony or scVI without factor gene sets)
- When the goal is batch integration or clustering itself rather than interpretation — JOINTLY clustering with FindNeighbors/FindClusters (dims 1:15, res = 0.05) or ARI/NMI evaluation via aricode is the appropriate step

## Inputs

- JOINTLY output list with a Seurat object (liver$object) and gene modules (liver$modules)
- Seurat or SingleCellExperiment object with raw expression assay (e.g., RNA assay of snRNA-seq data from SCP1376 nine-donor VAT donors)
- Cell-type labels imported from a labels TSV (e.g., Human_Liver_labels.tsv) added to object metadata

## Outputs

- Per-cell UCell module scores added to Seurat object metadata (e.g., factor_1_UCell columns)
- FeaturePlot panels of module scores on the JOINTLY UMAP
- Module-by-cell-type heatmap and AUC comparison table (module genes vs marker genes per cell type)

## How to apply

After running jointly() with batch.var (e.g., 'batch_label') to obtain a list holding a Seurat object and learned modules, call AddModuleScore_UCell on the Seurat object with features = modules (the factor gene sets). This yields per-cell scores named like 'factor_1_UCell' that can be visualized with FeaturePlot on the JOINTLY-derived UMAP (RunUMAP reduction='JOINTLY', dims 1:15). In the visceral adipose tissue application, module scores for factor 10 (insulin-signalling) are split by insulin-signalling status and sex to reproduce Fig. 4A-H, and module gene AUCs are compared to marker AUCs (logFC >= log2(1.1), padj <= 0.05, auc >= 0.6) via wilcoxauc from presto. The rationale is that JOINTLY's module genes are more discriminatory between cell types than traditional marker genes, so per-cell module scores guide cell type annotation and reveal active biological processes.

## Related tools

- **UCell** (Computes per-cell gene module signature scores via AddModuleScore_UCell using JOINTLY's learned modules) — https://github.com/carmonalab/UCell
- **JOINTLY** (Produces the interpretable gene modules (factor gene sets) that are scored per cell) — https://github.com/madsen-lab/rJOINTLY
- **Seurat** (Hosts the single-cell object, metadata module scores, and UMAP visualization via FeaturePlot) — https://github.com/satijalab/seurat
- **presto** (wilcoxauc computes per-cell-type AUC of module genes versus marker genes for the specificity comparison) — https://github.com/immunogenomics/presto

## Examples

```
liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules); FeaturePlot(liver$object, features = "factor_1_UCell")
```

## Evaluation signals

- The Seurat object metadata contains one '_UCell' score column per JOINTLY factor (e.g., 'factor_1_UCell')
- FeaturePlot of module scores on the JOINTLY UMAP shows spatially coherent, factor-specific cell populations consistent with cluster/cell-type assignments
- Module genes that are also markers achieve significantly higher average AUC than marker genes not in the most enriched module (marker filters: logFC >= log2(1.1), padj <= 0.05, auc >= 0.6)
- Module score distributions split by condition (e.g., factor 10 insulin-signalling scores by status and sex) reproduce the expected Fig. 4A-H patterns
- Downstream clustering on the JOINTLY reduction yields high ARI/NMI against transferred cell-type labels (liver test case: ARI 0.7415981, NMI 0.6325386)

## Limitations

- JOINTLY's result is seed-dependent: 'Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution.'
- For complex multi-condition, multi-sample datasets, consensus PCA in JOINTLY is not advised — regular PCA is recommended, which affects the factors being scored
- Batch-smoothed (imputed) expression derived from the H and F matrices should be used with caution and only for visualization purposes

## Evidence

- [methods] UCell module scoring step: "## Calculate module scores
liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules)"
- [readme] FeaturePlot of module score: "FeaturePlot(liver$object, features = "factor_1_UCell")"
- [readme] Install UCell from GitHub: "remotes::install_github("carmonalab/UCell")"
- [intro] Research question on factor gene specificity: "the genes associated with the interpretable factors are more specific for cell types than traditional marker genes and that they can be used to guide cell type annotation and discover active"
- [methods] wilcoxauc for marker AUC comparison: "res.raw <- wilcoxauc(liver, group_by = "celltype", seurat_assay = "RNA")"
