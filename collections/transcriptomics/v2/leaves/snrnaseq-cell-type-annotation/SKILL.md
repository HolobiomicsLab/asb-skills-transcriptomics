---
name: snrnaseq-cell-type-annotation
description: Use when annotating cell types in multi-batch snRNA-seq (e.g., nine VAT
  donors from SCP1376) where JOINTLY integration yields interpretable gene modules
  that must be scored per cell and compared to marker genes for cluster labeling.
when_to_use_negative:
- If the input is a single-sample dataset with no batch/donor structure, joint integration
  and batch-aware modules add no value — use standard marker-based annotation instead
- If cell type labels are already assigned and the task is only differential expression
  or pathway analysis, re-clustering and module-score-based annotation is unnecessary
- For datasets where samples do not share axes of variation (complex multi-condition,
  multi-sample data), the article advises regular PCA in JOINTLY rather than the consensus
  PCA path
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: JOINTLY
  role: Produces the joint co-embedding (H matrix) and interpretable gene modules
    used to cluster cells and guide cell type annotation
  repo: https://github.com/madsen-lab/rJOINTLY
- name: Seurat
  role: Hosts the object, runs FindNeighbors/FindClusters (dims 1:15, res = 0.05)
    and RunUMAP on the JOINTLY reduction
  repo: https://github.com/satijalab/seurat
- name: UCell
  role: Scores each JOINTLY gene module per cell via AddModuleScore_UCell for module-to-cell-type
    mapping
  repo: https://github.com/carmonalab/UCell
- name: presto
  role: wilcoxauc computes marker/module gene AUC per cell type for the marker-vs-module
    specificity comparison
  repo: https://github.com/immunogenomics/presto
- name: aricode
  role: Computes ARI and NMI between transferred cell type labels and seurat_clusters
    to validate annotation quality
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:a22b3dcaf3e5bbdb65826964244eadff7825bc373ac25e8b54b02851e332f1e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - JOINTLY
  - Seurat
  - UCell
  - aricode
  - presto
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# snrnaseq-cell-type-annotation

## Summary

Annotate cell types in single-nucleus RNA-seq data by clustering on a JOINTLY co-embedding and using JOINTLY's interpretable gene modules, whose genes are more cell-type-specific than traditional marker genes, to guide and validate cluster-to-cell-type assignment.

## When to use

Use when annotating cell types in multi-batch snRNA-seq (e.g., nine VAT donors from SCP1376) where JOINTLY integration yields interpretable gene modules that must be scored per cell and compared to marker genes for cluster labeling.

## When NOT to use

- If the input is a single-sample dataset with no batch/donor structure, joint integration and batch-aware modules add no value — use standard marker-based annotation instead
- If cell type labels are already assigned and the task is only differential expression or pathway analysis, re-clustering and module-score-based annotation is unnecessary
- For datasets where samples do not share axes of variation (complex multi-condition, multi-sample data), the article advises regular PCA in JOINTLY rather than the consensus PCA path

## Inputs

- Seurat object containing multi-donor snRNA-seq counts with batch labels (e.g., nine-donor visceral adipose data from Single Cell Portal SCP1376)
- Vector of batch/donor labels (batch.var, e.g., 'batch_label')
- Known cell-type marker gene sets for the tissue
- Optional pre-computed cell type labels (e.g., Human_Liver_labels.tsv format with X and Transferred_labels columns)

## Outputs

- Seurat object with a 'JOINTLY' DimReduc, seurat_clusters, and celltype metadata
- Per-cell JOINTLY module UCell scores (e.g., factor_1_UCell columns)
- Module-by-cell-type heatmap
- Per-cell-type AUC comparison table of JOINTLY module genes vs. marker genes from wilcoxauc
- ARI and NMI values between cell type labels and cluster labels

## How to apply

Run jointly() on the Seurat object with batch.var set to the donor label to obtain the joint co-embedding (H matrix) and gene modules; the README notes re-running with different seeds up to 5 times and keeping the best solution. Cluster cells on the 'JOINTLY' reduction with FindNeighbors and FindClusters (dims 1:15, res = 0.05) and embed with RunUMAP (dims 1:15). Score every JOINTLY-derived module per cell with AddModuleScore_UCell from UCell, then build a module-by-cell-type heatmap to map modules to candidate cell types. Quantify specificity with wilcoxauc from presto (e.g., group_by = 'celltype', seurat_assay = 'RNA') applying marker filters (logFC >= log2(1.1), padj <= 0.05, auc >= 0.6) and compare average AUC of module genes versus traditional markers per cell type. Validate the annotation by computing ARI and NMI between transferred cell-type labels and seurat_clusters with aricode.

## Related tools

- **JOINTLY** (Produces the joint co-embedding (H matrix) and interpretable gene modules used to cluster cells and guide cell type annotation) — https://github.com/madsen-lab/rJOINTLY
- **Seurat** (Hosts the object, runs FindNeighbors/FindClusters (dims 1:15, res = 0.05) and RunUMAP on the JOINTLY reduction) — https://github.com/satijalab/seurat
- **UCell** (Scores each JOINTLY gene module per cell via AddModuleScore_UCell for module-to-cell-type mapping) — https://github.com/carmonalab/UCell
- **presto** (wilcoxauc computes marker/module gene AUC per cell type for the marker-vs-module specificity comparison) — https://github.com/immunogenomics/presto
- **aricode** (Computes ARI and NMI between transferred cell type labels and seurat_clusters to validate annotation quality)

## Examples

```
liver <- jointly(liver, batch.var = "batch_label"); liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules); liver$object <- FindNeighbors(liver$object, reduction = "JOINTLY", dims = 1:15); liver$object <- FindClusters(liver$object, res = 0.05)
```

## Evaluation signals

- ARI between transferred cell type labels and seurat_clusters should reach values comparable to the README reference (~0.74 on the Human Liver test set; NMI ~0.63)
- Module genes that overlap marker genes should show significantly higher mean AUC (from wilcoxauc) than marker genes not in the most enriched module
- Module-by-cell-type heatmap should show each module score peaking in a coherent set of cell types (e.g., factor 10 insulin-signalling module in insulin-signalling cells)
- UMAP colored by celltype, seurat_clusters, and batch_label should show cell-type-coherent clusters with donors mixed within clusters
- Clustering parameters must match dims 1:15 and res = 0.05 on the 'JOINTLY' reduction, not a PCA reduction

## Limitations

- Setting a seed and re-running JOINTLY produces different results; the README advises running up to 5 times and choosing the best solution
- Integration performance is task-dependent: no method is always best, so the chosen method must be evaluated on each new integration task
- Integration metrics do not reveal which genes drive integration and clustering — interpretation requires the module gene lists and module scores
- For complex multi-condition, multi-sample datasets, consensus PCA in JOINTLY is not advised; regular PCA should be used instead

## Evidence

- [intro] The research question of module-guided annotation: "the genes associated with the interpretable factors are more specific for cell types than traditional marker genes and that they can be used to guide cell type annotation and discover active"
- [methods] Module scoring step: "liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules)"
- [methods] Marker specificity comparison: "res.raw <- wilcoxauc(liver, group_by = "celltype", seurat_assay = "RNA")"
- [readme] Annotation validation via ARI/NMI: "aricode::ARI(liver$object$celltype, liver$object$seurat_clusters) # 0.7415981"
- [readme] Clustering on the JOINTLY reduction: "liver$object <- FindNeighbors(liver$object, reduction = "JOINTLY", dims = 1:15)"
