---
name: marker-gene-auc-specificity-comparison
description: Use when a single-cell integrated/clustering result has interpretable
  gene modules (e.g. JOINTLY factors) and you need to test if module genes are more
  cell-type-specific than conventional marker genes, or to guide cell type annotation.
when_to_use_negative:
- The object has no interpretable gene modules or factor gene sets — the comparison
  is specifically between JOINTLY module genes and traditional markers, so without
  modules only generic marker ranking is possible
- Cell type labels are unreliable or missing — wilcoxauc group_by = 'celltype' and
  the AUC comparison depend on trustworthy per-cell-type groups (the vignette subsets
  to named cell types with more than 10 cells)
- Complex multi-condition, multi-sample datasets where the article advises regular
  PCA in JOINTLY over consensus PCA, since module interpretation quality degrades
  otherwise
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: JOINTLY
  role: Produces the interpretable gene modules whose cell-type specificity is being
    benchmarked against traditional markers
  repo: https://github.com/madsen-lab/rJOINTLY
- name: UCell
  role: Scores every JOINTLY-derived module per cell via AddModuleScore_UCell
  repo: https://github.com/carmonalab/UCell
- name: presto
  role: wilcoxauc computes per-cell-type gene rankings and AUCs used in the marker
    specificity comparison
  repo: https://github.com/immunogenomics/presto
- name: Seurat
  role: Hosts the data, cell-type labels, and the JOINTLY reduction/cluster annotations
    on which scoring and ranking are performed
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:f8e1195a49220b3e140b3f1812acc447fe82076e3006f80773b37d1003fc82f5
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# marker-gene-auc-specificity-comparison

## Summary

Quantifies whether genes belonging to JOINTLY's interpretable factor modules are more cell-type-discriminative than traditional marker genes by scoring modules per cell (UCell), ranking genes per cell type with a Wilcoxon rank-sum AUC (presto::wilcoxauc), and comparing mean AUC of module genes versus non-module marker genes.

## When to use

Use when a single-cell integrated/clustering result has interpretable gene modules (e.g. JOINTLY factors) and you need to test if module genes are more cell-type-specific than conventional marker genes, or to guide cell type annotation.

## When NOT to use

- The object has no interpretable gene modules or factor gene sets — the comparison is specifically between JOINTLY module genes and traditional markers, so without modules only generic marker ranking is possible
- Cell type labels are unreliable or missing — wilcoxauc group_by = 'celltype' and the AUC comparison depend on trustworthy per-cell-type groups (the vignette subsets to named cell types with more than 10 cells)
- Complex multi-condition, multi-sample datasets where the article advises regular PCA in JOINTLY over consensus PCA, since module interpretation quality degrades otherwise

## Inputs

- Batch-integrated Seurat or SingleCellExperiment object with cell-type labels (output of jointly() with batch.var set, e.g. nine-donor visceral adipose snRNA-seq from SCP1376)
- JOINTLY module gene lists (liver$modules / factor gene sets)
- Cell type label vector (e.g. 13 VAT cell types transferred to clusters)

## Outputs

- Per-cell-type AUC comparison table of module genes vs non-module marker genes (from wilcoxauc with marker filters logFC >= log2(1.1), padj <= 0.05, auc >= 0.6)
- Per-cell UCell module score matrix (features = JOINTLY modules)
- Module-by-cell-type score heatmap and factor-level score boxplots (e.g. Fig. 4A-H style factor 10 insulin-signalling panels)

## How to apply

Run JOINTLY on the batch-labelled Seurat object (jointly(obj, batch.var = 'batch_label'), re-running with different seeds up to ~5 times if needed) and take the returned module gene lists. Score every JOINTLY module per cell with AddModuleScore_UCell from UCell. Rank genes per cell type using presto::wilcoxauc on the RNA assay, then filter markers with logFC >= log2(1.1), padj <= 0.05, and auc >= 0.6 (the vignette's marker criteria). For each cell type, compare the average AUC of marker genes that are also JOINTLY module genes versus markers not in the most enriched module — in the adipose analysis module-overlapping markers had significantly higher AUC, demonstrating greater specificity. Optionally build a module-by-cell-type heatmap of UCell scores and plot factor-level scores split by covariates (e.g. factor 10 insulin-signalling module by status and sex) to link modules to biological processes.

## Related tools

- **JOINTLY** (Produces the interpretable gene modules whose cell-type specificity is being benchmarked against traditional markers) — https://github.com/madsen-lab/rJOINTLY
- **UCell** (Scores every JOINTLY-derived module per cell via AddModuleScore_UCell) — https://github.com/carmonalab/UCell
- **presto** (wilcoxauc computes per-cell-type gene rankings and AUCs used in the marker specificity comparison) — https://github.com/immunogenomics/presto
- **Seurat** (Hosts the data, cell-type labels, and the JOINTLY reduction/cluster annotations on which scoring and ranking are performed) — https://github.com/satijalab/seurat

## Examples

```
liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules); res.raw <- wilcoxauc(liver, group_by = "celltype", seurat_assay = "RNA")
```

## Evaluation signals

- Marker genes that are also JOINTLY module genes should show significantly higher average AUC than markers not in the most enriched module (as reported for the adipose tissue analysis)
- Module-vs-marker AUC table should only contain markers passing logFC >= log2(1.1), padj <= 0.05, and auc >= 0.6
- UCell scores should be non-negative and defined for every cell and every module in liver$modules (no NA score columns)
- Module-by-cell-type heatmap should show visually enriched (high-scoring) modules concentrated in specific cell types, consistent with the clustering ARI/NMI agreement between celltype and seurat_clusters
-  wilcoxauc output should cover all labelled cell types (those with >= 10 cells after subsetting), not just clusters

## Limitations

- JOINTLY results are seed-dependent: 'Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution', so module gene sets (and hence the AUC comparison) can vary between runs
- Integration and interpretation metrics do not by themselves reveal which genes drive clustering; the AUC comparison is needed to interpret factors
- Benchmarked performance is task-dependent — no integration method is always best, so module gene specificity should be re-evaluated on each new dataset

## Evidence

- [intro] Research question: whether genes associated with JOINTLY's interpretable factors are more specific for cell types than traditional marker genes: "the genes associated with the interpretable factors are more specific for cell types than traditional marker genes and that they can be used to guide cell type annotation and discover active"
- [methods] wilcoxauc gene ranking call used for marker AUCs: "res.raw <- wilcoxauc(liver, group_by = "celltype", seurat_assay = "RNA")"
- [methods] UCell module scoring of learned modules: "liver$object <- AddModuleScore_UCell(liver$object, features = liver$modules)"
- [methods] Cell types need enough cells for reliable per-type marker statistics: "## Subset the object to only named cell types with more than 10 cells"
- [readme] Seed sensitivity of JOINTLY solutions affects module gene sets: "Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution."
