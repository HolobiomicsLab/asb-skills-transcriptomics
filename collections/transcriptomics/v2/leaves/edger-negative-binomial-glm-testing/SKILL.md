---
name: edger-negative-binomial-glm-testing
description: Use when per-cell-type case-vs-control DE gene lists are needed as ASGARD
  GetDrug inputs and edgeR should be compared against Limma, Seurat, and DESeq2 modes
  on the same integrated Seurat object.
when_to_use_negative:
- 'The goal is the best single DE method: Limma (Bayes) attained significantly higher
  AUC (0.90-0.92, P<0.05, Student''s t test), so edgeR should serve as a comparison
  mode rather than the default'
- The dataset lacks defined case/normal contrast groups or has clusters with fewer
  than 3 cells per condition, making GLM-based testing underpowered
- Bulk RNA-seq differential expression with replicate-level design is needed — this
  skill is scoped to single-cell cluster-level DE feeding drug repurposing
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: edgeR
  role: Negative-binomial GLM differential expression (glmFit/glmLRT, quasi-likelihood)
    producing the Gene.list fed to ASGARD
- name: ASGARD
  role: Consumes the edgeR-derived gene list via GetDrug and aggregates cluster rankings
    into patient drug scores via DrugScore
  repo: https://github.com/lanagarmire/ASGARD
- name: Seurat
  role: Preprocessing, integration, clustering, and supplying per-cluster counts and
    cell metadata
- name: pROC
  role: Computes ROC/AUC per DE method against the known positive-drug ground truth
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:d872a0f0e2741f33e69867ecd73f2e5dd25003d60de5cb2a20c0ad9ab44c10b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/ASGARD
  tools:
  - ASGARD
  - Seurat
  - edgeR
  - pROC
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# edgeR negative-binomial GLM testing

## Summary

Benchmark differential expression mode that fits negative-binomial generalized linear models (glmFit/glmLRT, optionally quasi-likelihood) to case-vs-control scRNA-seq clusters so the resulting t-statistic/p-value gene lists can feed ASGARD drug repurposing; in the ASGARD benchmark edgeR achieved intermediate AUCs (~0.83-0.86), below Limma (Bayes).

## When to use

Use when per-cell-type case-vs-control DE gene lists are needed as ASGARD GetDrug inputs and edgeR should be compared against Limma, Seurat, and DESeq2 modes on the same integrated Seurat object.

## When NOT to use

- The goal is the best single DE method: Limma (Bayes) attained significantly higher AUC (0.90-0.92, P<0.05, Student's t test), so edgeR should serve as a comparison mode rather than the default
- The dataset lacks defined case/normal contrast groups or has clusters with fewer than 3 cells per condition, making GLM-based testing underpowered
- Bulk RNA-seq differential expression with replicate-level design is needed — this skill is scoped to single-cell cluster-level DE feeding drug repurposing

## Inputs

- Integrated Seurat object or raw count matrix per cell cluster with case/control labels (e.g. GSE113197+GSE123926 breast, GSE161529 TNBC, GSE145926+GSE158055 COVID-19 BALF)
- Cell-type/cluster annotations with >=3 cells per condition per cluster
- Tissue-matched ASGARD drug reference (e.g. DrugReference/breast_rankMatrix.txt with breast_gene_info.txt and breast_drug_info.txt)

## Outputs

- Per-cluster edgeR DE gene list (Gene.list with scores/t-statistics, BH-adjusted and raw p-values)
- Drug ranking per cluster from ASGARD GetDrug (FDA drugs, negative connectivity)
- Patient-level integrated drug scores from DrugScore and ROC/AUC values computed with pROC

## How to apply

After Seurat integration (NormalizeData, FindVariableFeatures with anchor.features=2000, cell-cycle regression via cc.genes S/G2M scores, IntegrateData, PCA to 15 dims, Louvain clustering at resolution=0.4), extract raw counts for each cluster/label containing more than min.cells=3 case and control cells, having first removed genes detected in fewer than 3 cells. Build a design matrix from the case/control type factor, fit a negative-binomial GLM with glmFit (or the quasi-likelihood framework), and test the case-vs-control contrast with glmLRT. Collect BH-adjusted and raw p-values into a Gene.list object (score field populated for compatibility with GetDrug), run ASGARD GetDrug against the breast tissue reference (breast_rankMatrix.txt) with repurposing.unit='drug', connectivity='negative', drug.type='FDA', then DrugScore to aggregate cluster-level rankings into patient-level scores. Rationale: edgeR is one of the top DE methods for single-cell data per the article, but its AUC must be empirically compared because DE-method choice affects downstream repurposing accuracy.

## Related tools

- **edgeR** (Negative-binomial GLM differential expression (glmFit/glmLRT, quasi-likelihood) producing the Gene.list fed to ASGARD)
- **ASGARD** (Consumes the edgeR-derived gene list via GetDrug and aggregates cluster rankings into patient drug scores via DrugScore) — https://github.com/lanagarmire/ASGARD
- **Seurat** (Preprocessing, integration, clustering, and supplying per-cluster counts and cell metadata)
- **pROC** (Computes ROC/AUC per DE method against the known positive-drug ground truth)

## Examples

```
library(edgeR); y <- DGEList(counts=cluster_counts); design <- model.matrix(~0+type); fit <- glmFit(y, design); lrt <- glmLRT(fit, contrast=makeContrasts(typeCase-typeControl, levels=design)); padj <- p.adjust(lrt$table$PValue, method='BH')
```

## Evaluation signals

- Benchmark ordering must reproduce: Limma (Bayes) best average AUC (~0.90-0.92), edgeR intermediate (~0.83-0.86), Seurat Wilcoxon (~0.80-0.86), DESeq2 lowest
- Each cluster/label passed to testing contains more than 3 case and 3 control cells after removing genes detected in fewer than 3 cells
- Returned Gene.list includes BH-adjusted and raw p-values with a score field compatible with GetDrug(gene.data=Gene.list, repurposing.unit='drug', connectivity='negative', drug.type='FDA')
- AUCs computed per dataset with the pROC package against Supplementary Data 4 positive drugs, then averaged across breast cancer, leukemia, and COVID-19 datasets

## Limitations

- AUC performance varies with cell population sizes, proportions, and differential expression levels; DE methods should be carefully selected according to the status of the dataset
- Sparsity and heterogeneity of single-cell data can cause false discoveries of differentially expressed genes, inflating or deflating GLM test results
- Drug reference is limited to 98 cell lines from LINCS L1000, constraining tissue-matched accuracy regardless of DE method

## Evidence

- [discussion] edgeR is one of the top DE methods for single-cell data: "Seurat18, DESeq226, edgeR27, and Limma25 are among the top methods in discovering differentially expressed genes using single-cell data"
- [discussion] Limma (Bayes) beat edgeR in the benchmark: "Limma (Bayes) method showed the best average AUC performance compared to all other three methods"
- [discussion] Benchmark AUCs computed with pROC: "ROCs and AUCs are calculated for each pipeline using the pROC package79"
- [discussion] Filter genes detected in fewer than three cells before DE: "genes identified in fewer than three cells are removed from the dataset"
- [other] edgeR run via glmFit/glmLRT with BH-adjusted p-values in the comparison workflow: [evidence span withheld — anchor status: not_found]
