---
name: deseq2-count-based-de-testing
description: Use when benchmarking differential expression methods on scRNA-seq case-vs-control
  data prior to ASGARD drug repurposing, specifically to generate a DESeq2-derived
  Gene.list (score/p-value per gene) as an alternative to Limma, Seurat, or edgeR
  rankings.
when_to_use_negative:
- When the goal is the best-performing ASGARD input for a new dataset without benchmarking
  — the article found Limma (Bayes) had the best average AUC and DESeq2 the lowest,
  so Limma was used for subsequent analysis
- When input is normalized/transformed expression rather than raw counts — DESeq2's
  negative-binomial model assumes count data
- When clusters have fewer than min.cells=3 cells in either case or control condition
  (these clusters are excluded in the workflow)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DESeq2
  role: Negative-binomial differential expression method run in Wald and likelihood-ratio
    test modes to generate alternative Gene.list inputs for ASGARD benchmarking
- name: Seurat
  role: Preprocessing, integration, clustering, and optionally calling DESeq2 via
    FindMarkers with test.use='DESeq2'
- name: ASGARD
  role: Consumes the DESeq2 Gene.list via GetDrug/DrugScore to produce drug rankings
    whose AUC is benchmarked against other DE methods
  repo: https://github.com/lanagarmire/Asgard
- name: pROC
  role: Computes ROCs and AUCs for each DE-method pipeline against the positive-drug
    ground truth
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36637-3
    title: ASGARD is A Single-cell Guided Pipeline to Aid Repurposing of Drugs
  source_content_hash: sha256:033b3310d2d5d0e9a7266aa047ec67504bb8b7ae45446baf22b5313b9f6380ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/Asgard
  tools:
  - ASGARD
  - DESeq2
  - Seurat
  - pROC
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# deseq2-count-based-de-testing

## Summary

Apply DESeq2 negative-binomial differential expression testing (Wald and likelihood-ratio test modes) to case-vs-control scRNA-seq clusters as one of the benchmarked DE inputs to ASGARD drug repurposing. In the source study, DESeq2-based gene lists produced the lowest average ROC/AUC of the four DE methods tested.

## When to use

Use when benchmarking differential expression methods on scRNA-seq case-vs-control data prior to ASGARD drug repurposing, specifically to generate a DESeq2-derived Gene.list (score/p-value per gene) as an alternative to Limma, Seurat, or edgeR rankings.

## When NOT to use

- When the goal is the best-performing ASGARD input for a new dataset without benchmarking — the article found Limma (Bayes) had the best average AUC and DESeq2 the lowest, so Limma was used for subsequent analysis
- When input is normalized/transformed expression rather than raw counts — DESeq2's negative-binomial model assumes count data
- When clusters have fewer than min.cells=3 cells in either case or control condition (these clusters are excluded in the workflow)

## Inputs

- Seurat-integrated scRNA-seq object (SC.integrated) with case/control metadata and cluster assignments (e.g., breast cancer GSE113197+GSE123926, TNBC GSE161529, COVID-19 BALF GSE145926+GSE158055, Pre-T ALL GSE132509)
- Case-vs-control cell type/cluster labels with at least min.cells=3 cells per condition
- Tissue-specific ASGARD drug reference built from LINCS L1000 (GSE70138/GSE92742) via PrepareReference

## Outputs

- DESeq2-derived Gene.list of differentially expressed genes with scores (log2FC/t-statistic), raw p-values, and BH-adjusted p-values per cluster
- Per-cluster DE rankings passed to ASGARD GetDrug, yielding drug identification results (Drug.ident.res)
- ROC/AUC values per dataset computed with pROC against known positive-drug ground truth for comparison with Limma/Seurat/edgeR baselines

## How to apply

Preprocess and integrate the scRNA-seq datasets (e.g., Seurat NormalizeData, FindVariableFeatures with anchor.features=2000, cell-cycle regression via cc.genes S/G2M scores, IntegrateData, PCA to 15 dims, Louvain clustering at resolution=0.4). For each cell type/cluster with more than min.cells=3 case and control cells, compute case-vs-control DE either directly with DESeq2 (glm-based negative-binomial) or via Seurat FindMarkers with test.use='DESeq2', running both the Wald test and the likelihood-ratio test (LRT) modes. Store per-gene statistics (score/t-statistic or avg_log2FC, raw and BH-adjusted p-values) in a Gene.list object, then feed it to ASGARD GetDrug against the tissue-matched drug reference (e.g., GetDrugRef on DrugReference/breast_rankMatrix.txt) with repurposing.unit='drug', connectivity='negative', drug.type='FDA', followed by DrugScore to aggregate cluster-level ranks into patient-level scores. The rationale is to test how the DE method choice propagates to drug-repurposing accuracy; in this benchmark DESeq2 yielded the lowest average AUC, while Limma (Bayes) was best (AUC ~0.90-0.92, P-value < 0.05, Student's t test), so DESeq2 results should be compared against the Limma baseline rather than used as the default.

## Related tools

- **DESeq2** (Negative-binomial differential expression method run in Wald and likelihood-ratio test modes to generate alternative Gene.list inputs for ASGARD benchmarking)
- **Seurat** (Preprocessing, integration, clustering, and optionally calling DESeq2 via FindMarkers with test.use='DESeq2')
- **ASGARD** (Consumes the DESeq2 Gene.list via GetDrug/DrugScore to produce drug rankings whose AUC is benchmarked against other DE methods) — https://github.com/lanagarmire/Asgard
- **pROC** (Computes ROCs and AUCs for each DE-method pipeline against the positive-drug ground truth)

## Examples

```
Seurat.markers <- FindMarkers(SC.integrated, ident.1 = 'case', ident.2 = 'control', cluster = cluster_id, test.use = 'DESeq2', latent.vars = c('nFeature_RNA'))
```

## Evaluation signals

- Ordering check: DESeq2-derived AUCs must fall below the Limma (Bayes) baseline (~0.90-0.92) and be the lowest among Limma, Seurat Wilcoxon, and edgeR in the reported benchmark
- Every tested cluster has > min.cells=3 case and control cells before DE is computed; genes detected in fewer than 3 cells were removed beforehand
- Gene.list contains per-gene scores with both raw and BH-adjusted p-values for every cluster analyzed
- Downstream ASGARD run uses repurposing.unit='drug', connectivity='negative', drug.type='FDA' against the tissue-matched drug reference, and AUCs are computed with the pROC package

## Limitations

- Differential expression method choice materially affects downstream AUC; the article notes 'DE methods should be carefully selected according to the status of the dataset to achieve the best performance'
- Sparsity and heterogeneity of single-cell data can cause false discoveries of differentially expressed genes, affecting all DE methods including DESeq2
- AUC performance varies with cell population sizes, proportions, and differential expression levels, so a single-method ranking may not generalize across datasets

## Evidence

- [discussion] Seurat, DESeq2, edgeR, and Limma are among the top methods in discovering differentially expressed genes using single-cell data: "Seurat18, DESeq226, edgeR27, and Limma25 are among the top methods in discovering differentially expressed genes using single-cell data"
- [discussion] Limma (Bayes) showed the best average AUC among DE methods, implying DESeq2 was inferior: "Limma (Bayes) method showed the best average AUC performance compared to all other three methods"
- [discussion] DE method selection must be tailored per dataset: "DE methods should be carefully selected according to the status of the dataset to achieve the best performance"
- [discussion] ROC/AUC benchmarking procedure: "ROCs and AUCs are calculated for each pipeline using the pROC package79"
- [methods] ASGARD installation from GitHub, as used in the workflow: "devtools::install_github("lanagarmire/Asgard")"
