---
name: scanvi-semisupervised-cross-dataset-integration
description: Use when several independently annotated single-cell datasets (e.g. per-study
  JOINTLY labels from Tabula Sapiens, Jaitin GSE128518, Vijay GSE129363, Hildreth
  GSE155960, Emont SCP1376, Barboza) must be harmonised into a consensus atlas of
  cell types and subtypes.
when_to_use_negative:
- When datasets are unannotated — JOINTLY joint clustering (per-dataset labelling)
  must precede scANVI integration in this workflow
- When the goal is within-study batch correction/clustering rather than harmonising
  labels across studies; JOINTLY alone (or scVI/Harmony) addresses that task
- For complex multi-condition, multi-sample datasets where all samples do not share
  the same axes of variation, the article advises regular PCA in JOINTLY rather than
  its consensus PCA step
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: JOINTLY
  role: Per-dataset joint clustering and interpretable labelling of cells prior to
    cross-study integration
  repo: https://github.com/madsen-lab/rJOINTLY
- name: scVI
  role: Deep-learning batch integration framework underlying scANVI's semisupervised
    label harmonisation across the six datasets
- name: aricode
  role: Computes ARI/NMI to validate integrated annotations against the deposited
    WATLAS reference
- name: Seurat
  role: Framework holding the expression objects processed by JOINTLY before integration
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:4a973b7da97f77cde4c7aabc77ba2aeeeeec01930af6fcdbb5ea2faaefa5d32d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - JOINTLY
  - Seurat
  - aricode
  - scVI
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# scANVI semisupervised cross-dataset integration

## Summary

Harmonise cell type/state annotations across independent scRNA-seq studies by integrating JOINTLY-labelled datasets with scANVI, as done to build the WATLAS white adipose tissue reference atlas (~300,000 cells from VAT and SAT) from six public studies.

## When to use

Use when several independently annotated single-cell datasets (e.g. per-study JOINTLY labels from Tabula Sapiens, Jaitin GSE128518, Vijay GSE129363, Hildreth GSE155960, Emont SCP1376, Barboza) must be harmonised into a consensus atlas of cell types and subtypes.

## When NOT to use

- When datasets are unannotated — JOINTLY joint clustering (per-dataset labelling) must precede scANVI integration in this workflow
- When the goal is within-study batch correction/clustering rather than harmonising labels across studies; JOINTLY alone (or scVI/Harmony) addresses that task
- For complex multi-condition, multi-sample datasets where all samples do not share the same axes of variation, the article advises regular PCA in JOINTLY rather than its consensus PCA step

## Inputs

- Six independently preprocessed and JOINTLY-labelled white adipose tissue scRNA-seq datasets (Tabula Sapiens, Jaitin GSE128518, Vijay GSE129363, Hildreth GSE155960, Emont SCP1376, Barboza)
- Per-dataset cell type/state labels produced by JOINTLY
- Batch variable per dataset (batch_label)

## Outputs

- Harmonised cross-study cell type/state annotations for ~300,000 cells covering VAT and SAT
- watlas_cell_annotations.csv: cell-by-cell-type/state table with 17 major cell types and 43 subtypes
- Integrated atlas comparable to the WATLAS reference at Broad SCP2289

## How to apply

First annotate each dataset separately with JOINTLY (jointly(data, batch.var = "batch_label")), re-running up to 5 times with different seeds and keeping the best solution, since results vary between runs. Then integrate the labelled datasets across studies with scANVI, using the parameters stated in the article, to harmonise cell type/state annotations across donors, conditions and depots. Compile the consensus annotations into a single cell-by-cell-type/state table — for WATLAS this yielded ~300,000 cells at 17 major-type and 43 subtype resolution — saved as watlas_cell_annotations.csv. Validation compares the compiled annotations against the deposited WATLAS reference at Broad SCP2289 using ARI/NMI from the aricode package.

## Related tools

- **JOINTLY** (Per-dataset joint clustering and interpretable labelling of cells prior to cross-study integration) — https://github.com/madsen-lab/rJOINTLY
- **scVI** (Deep-learning batch integration framework underlying scANVI's semisupervised label harmonisation across the six datasets)
- **aricode** (Computes ARI/NMI to validate integrated annotations against the deposited WATLAS reference)
- **Seurat** (Framework holding the expression objects processed by JOINTLY before integration) — https://github.com/satijalab/seurat

## Examples

```
liver <- jointly(liver, batch.var = "batch_label"); aricode::ARI(liver$object$celltype, liver$object$seurat_clusters)
```

## Evaluation signals

- watlas_cell_annotations.csv exists and its row count is ~300,000
- The annotation table contains 43 distinct subtype labels
- Cell type/state assignments agree with the deposited WATLAS reference at Broad SCP2289, quantified by ARI/NMI comparison using aricode
- JOINTLY runs repeated with different seeds (up to 5 times) and the best-scoring solution is retained before integration
- The integrated atlas recovers 17 major WAT cell types and 4 adipocyte subpopulations

## Limitations

- JOINTLY is stochastic: setting a seed and re-running produces a different result, so up to 5 runs may be needed to select the best solution
- No integration method is best across all tasks; performance must be evaluated on each new integration task
- Consensus PCA within JOINTLY is not advised for complex multi-condition, multi-sample datasets; regular PCA is recommended instead
- All manuscript analyses were performed with the R version of JOINTLY; the R and Python versions do not produce identical results

## Evidence

- [discussion] by processing and labelling six independent datasets with JOINTLY and subsequently, integrating across datasets using scANVI: "by processing and labelling six independent datasets with JOINTLY and subsequently, integrating across datasets using scANVI"
- [results] After annotating cells from six studies with JOINTLY and integrating them with scANVI, the resulting WATLAS (~300,000 cells from VAT and SAT) recovered 17 major cell types in WAT and 43 total subtypes.: [evidence span withheld — anchor status: not_found]
- [readme] Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution.: "Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution."
- [discussion] For more complex datasets, where all samples do not necessarily share the same axes of variation, such as multi-condition, multi-sample datasets, we advise users to use regular PCA in JOINTLY: "For more complex datasets, where all samples do not necessarily share the same axes of variation, such as multi-condition, multi-sample datasets, we advise users to use regular PCA in JOINTLY"
- [discussion] no method is always the best across all tasks, and therefore, that it is critical to evaluate the performance of a chosen method on any new integration task.: "no method is always the best across all tasks, and therefore, that it is critical to evaluate the performance of a chosen method on any new integration task."
