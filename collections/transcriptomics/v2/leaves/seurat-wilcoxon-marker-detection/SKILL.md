---
name: seurat-wilcoxon-marker-detection
description: Use when computing case-vs-control differential expression per cell cluster
  from integrated Seurat objects as input to drug repurposing, or when benchmarking
  DE methods (Wilcoxon vs Limma/DESeq2/edgeR) by downstream AUC.
when_to_use_negative:
- When the goal is the highest-accuracy ASGARD input rather than a method comparison
  — the benchmark found Limma (Bayes) significantly better (P<0.05, Student's t test)
  than Wilcoxon, so Limma was used for subsequent analysis
- When the dataset is bulk RNA-seq rather than single-cell data processed through
  Seurat integration and clustering
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Provides FindMarkers with Wilcoxon rank-sum test to compute cluster-level
    case-vs-control DE profiles from the integrated scRNA-seq object
- name: ASGARD
  role: Consumes the Wilcoxon-derived Gene.list via GetDrug/DrugScore to produce drug
    rankings and evaluate AUC
  repo: https://github.com/lanagarmire/Asgard
- name: pROC
  role: Computes ROC curves and AUCs for the Wilcoxon mode against the positive-drug
    ground truth
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:e46fbc580997cee6feb8489cdc930d3d3a28daedf60a79fd82a372dd081dd8f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/Asgard
  tools:
  - ASGARD
  - Seurat
  - pROC
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# seurat-wilcoxon-marker-detection

## Summary

Wilcoxon rank-sum marker detection via Seurat FindMarkers, used as one of the differential expression modes benchmarked against Limma, DESeq2, and edgeR when feeding DE gene lists into ASGARD drug repurposing for breast cancer, leukemia, and COVID-19 scRNA-seq datasets.

## When to use

Use when computing case-vs-control differential expression per cell cluster from integrated Seurat objects as input to drug repurposing, or when benchmarking DE methods (Wilcoxon vs Limma/DESeq2/edgeR) by downstream AUC.

## When NOT to use

- When the goal is the highest-accuracy ASGARD input rather than a method comparison — the benchmark found Limma (Bayes) significantly better (P<0.05, Student's t test) than Wilcoxon, so Limma was used for subsequent analysis
- When the dataset is bulk RNA-seq rather than single-cell data processed through Seurat integration and clustering

## Inputs

- Integrated Seurat object (SC.integrated) with cluster/cell-type metadata and case-vs-control labels
- Cluster-level cell metadata with at least min.cells=3 cells per condition per cluster
- Positive-drug ground truth list (Supplementary Data 4) for AUC evaluation

## Outputs

- Per-cluster DE gene table (Gene.list) with Wilcoxon statistics, avg_log2FC, and adjusted/raw p-values
- ASGARD drug rankings per cluster and patient-level DrugScore aggregations
- ROC/AUC values per dataset for the Wilcoxon DE mode (reported ~0.80-0.86)

## How to apply

After Seurat integration (NormalizeData, FindVariableFeatures with anchor.features=2000, cell-cycle regression via cc.genes S/G2M scores, FindIntegrationAnchors/IntegrateData, PCA to 15 dims, Louvain clustering at resolution=0.4), call FindMarkers for each cell type/cluster with more than min.cells=3 case and control cells, using test.use='wilcox' (Wilcoxon rank-sum). The DE output contributes to a Gene.list scored by the Seurat default statistic, with avg_log2FC used for the Seurat mode with log2FC > 4.0, and adjusted and raw p-values stored per gene. Genes detected in fewer than 3 cells are removed beforehand to limit false discoveries from single-cell sparsity. The resulting cluster-level DE lists are passed to ASGARD GetDrug against the breast tissue drug reference (repurposing.unit='drug', connectivity='negative', drug.type='FDA'), then aggregated to patient-level drug scores via DrugScore and evaluated with pROC against known positive drugs. Note: in the published benchmark, Seurat Wilcoxon achieved intermediate AUC (~0.80-0.86), below Limma (Bayes) at 0.90-0.92, so Wilcoxon mode is a comparison arm rather than the recommended default.

## Related tools

- **Seurat** (Provides FindMarkers with Wilcoxon rank-sum test to compute cluster-level case-vs-control DE profiles from the integrated scRNA-seq object)
- **ASGARD** (Consumes the Wilcoxon-derived Gene.list via GetDrug/DrugScore to produce drug rankings and evaluate AUC) — https://github.com/lanagarmire/Asgard
- **pROC** (Computes ROC curves and AUCs for the Wilcoxon mode against the positive-drug ground truth)

## Examples

```
markers <- FindMarkers(SC.integrated, ident.1="case", ident.2="control", group.by="type", subset.ident=cluster_id, test.use="wilcox", min.cells=3)
```

## Evaluation signals

- Every cluster submitted to FindMarkers has >=3 case and >=3 control cells (min.cells=3)
- Genes detected in fewer than 3 cells were removed before DE testing
- Wilcoxon-mode drug rankings, when aggregated via DrugScore and scored with pROC, yield AUC in the ~0.80-0.86 range reported for Seurat methods
- AUC ordering reproduces the benchmark: Limma (Bayes) > edgeR > Seurat Wilcoxon > DESeq2, with Limma significantly better (P<0.05, Student's t test)
- Gene.list entries carry both adjusted and raw p-values alongside the test statistic for downstream GetDrug calls

## Limitations

- Wilcoxon mode yields intermediate AUC (~0.80-0.86), significantly below Limma (Bayes, 0.90-0.92), so it should not be the default DE engine for ASGARD input
- Single-cell sparsity and heterogeneity can cause false discoveries of differentially expressed genes regardless of test choice
- DE method performance varies with cell population sizes, proportions, and differential expression levels, so the best method must be selected per dataset

## Evidence

- [discussion] DE methods benchmarked: "Seurat18, DESeq226, edgeR27, and Limma25 are among the top methods in discovering differentially expressed genes using single-cell data"
- [discussion] Limma best AUC: "Limma (Bayes) method showed the best average AUC performance compared to all other three methods"
- [discussion] DE method choice matters: "DE methods should be carefully selected according to the status of the dataset to achieve the best performance"
- [discussion] Sparsity caveat: "Sparsity and heterogeneity are two major challenges in analyzing single-cell data, and usually cause false discoveries of differentially expressed genes"
- [discussion] Benchmark evaluation: "ROCs and AUCs are calculated for each pipeline using the pROC package79"
- [discussion] AUC variability: "AUC performance still varies with methods for single-cell differential expression"
