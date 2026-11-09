---
name: roc-auc-evaluation-of-domain-markers
description: Use when you have identified spatial domains on multi-slice spatial transcriptomics
  data (e.g., 12 human DLPFC Visium slices) with known cortical-layer marker genes
  and need a quantitative, comparable accuracy metric for domain-marker recovery.
when_to_use_negative:
- Data are seq-based ST slices (e.g., 10X Visium) for which no cell type deconvolution
  (e.g., Spoint) has yet been run before Splane domain identification, so domain labels
  are not yet available
- The tissue or dataset lacks a curated ground-truth list of layer/spatial-domain
  marker genes, making SVG-recovery ROC AUC undefined
- The goal is nonlinear slice alignment or 3D stacking rather than spatial domain
  marker evaluation (the current version of Scube does not support nonlinear alignment)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Splane
  role: Provides the spatial domain labels whose marker DEGs are evaluated for SVG
    recovery accuracy
  repo: github.com/QuKunLab/SPACEL
- name: SPACEL
  role: Python toolkit containing the Splane spatial-domain-identification module
    under evaluation and the Spoint deconvolution prerequisite
  repo: github.com/QuKunLab/SPACEL
- name: Python
  role: Implementation environment for loading domain labels, running scanpy Wilcoxon
    rank-sum tests, computing ROC AUC, and writing the metrics csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:5363ea3fbd8d4e38b2505bed936a0cf6ae1691cfaa99a199eda84924a70f2d71
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SPACEL
  - Splane
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# roc-auc-evaluation-of-domain-markers

## Summary

Evaluates how accurately spatial domains recovered by a spatial-domain-identification method (e.g., Splane) capture ground-truth spatially variable genes (SVGs) by intersecting Wilcoxon domain DEGs with a layer-annotated SVG list and reporting ROC AUC.

## When to use

Use when you have identified spatial domains on multi-slice spatial transcriptomics data (e.g., 12 human DLPFC Visium slices) with known cortical-layer marker genes and need a quantitative, comparable accuracy metric for domain-marker recovery.

## When NOT to use

- Data are seq-based ST slices (e.g., 10X Visium) for which no cell type deconvolution (e.g., Spoint) has yet been run before Splane domain identification, so domain labels are not yet available
- The tissue or dataset lacks a curated ground-truth list of layer/spatial-domain marker genes, making SVG-recovery ROC AUC undefined
- The goal is nonlinear slice alignment or 3D stacking rather than spatial domain marker evaluation (the current version of Scube does not support nonlinear alignment)

## Inputs

- Spatial domain label assignments (e.g., from Splane's SplaneModel.identify_spatial_domain) for 12 human DLPFC 10X Visium slices
- Normalized spot-by-gene expression matrix (AnnData/h5ad) for the DLPFC slices
- Ground-truth SVG list annotated by cortical layer (layers 1-6 and WM), e.g., 1917 layer-specific SVGs
- Baseline recovery metrics for comparison methods (PRECAST, STAligner, SpaGCN, etc.)

## Outputs

- Domain DEG membership table after logFC > 0.5 and P < 0.01 filtering
- SVG recovery counts (recovered / total ground-truth SVGs, e.g., 1714 of 1917)
- ROC AUC summary per method (svg_recovery_metrics.csv)

## How to apply

Load the Splane-identified spatial domain labels for the 12 human DLPFC Visium slices together with the ground-truth SVG list spanning cortical layers 1-6 and white matter using Python/pandas. For each spatial domain, run a Wilcoxon rank-sum differential expression test on the normalized expression matrix with scanpy's rank_genes_groups. Filter the resulting domain DEGs at logFC > 0.5 and P < 0.01 to obtain the recovered SVG candidate set, then intersect these DEGs with the ground-truth SVG list to compute the number of recovered SVGs (expected ~1714 of 1917 for Splane). Finally, compute the ROC AUC of SVG recovery for the method under test and report it alongside baselines (Splane 0.90, PRECAST 0.89, STAligner 0.88, other methods < 0.83), writing the DEG membership table and recovery/ROC AUC summary to csv. The rationale is that Splane's adversarially trained graph convolutional network should yield domains whose marker genes best match known layer-specific SVGs across slices.

## Related tools

- **Splane** (Provides the spatial domain labels whose marker DEGs are evaluated for SVG recovery accuracy) — github.com/QuKunLab/SPACEL
- **SPACEL** (Python toolkit containing the Splane spatial-domain-identification module under evaluation and the Spoint deconvolution prerequisite) — github.com/QuKunLab/SPACEL
- **Python** (Implementation environment for loading domain labels, running scanpy Wilcoxon rank-sum tests, computing ROC AUC, and writing the metrics csv)

## Examples

```
sc.tl.rank_genes_groups(adata, groupby='spatial_domain', method='wilcoxon'); degs = sc.get.rank_genes_groups_df(adata, group=None); hits = degs[(degs.logfoldchanges > 0.5) & (degs.pvals < 0.01)]
```

## Evaluation signals

- Filtered DEG count per domain is finite and reproducible at the stated thresholds (logFC > 0.5, P < 0.01)
- Intersection with the ground-truth SVG list recovers approximately 1714 of 1917 DLPFC layer SVGs for Splane domains
- Reported ROC AUC for Splane is ~0.90, exceeding PRECAST (0.89), STAligner (0.88), and all other examined methods (AUCs < 0.83)
- Output csv (svg_recovery_metrics.csv) contains both per-DEG membership rows and a recovery/ROC AUC summary row per method
- Wilcoxon rank-sum test was run on the normalized (not raw) expression matrix per spatial domain

## Limitations

- Splane requires cell type composition input, so for seq-based ST datasets such as 10X Visium a deconvolution step (e.g., Spoint) must precede domain identification before marker evaluation
- When a new slice is added, the entire SPACEL deep learning model must be retrained from scratch, which may affect computational efficiency of the upstream domains being evaluated
- The comparison is anchored to the specific DLPFC cortical-layer SVG ground truth (layers 1-6 and WM); AUC values may not transfer to other tissues or technologies

## Evidence

- [abstract] Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains that are transcriptomically and spatially coherent across multiple ST slices: "Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains that are transcriptomically and spatially coherent across multiple ST slices"
- [discussion] one has to employ a cell type deconvolution algorithm such as Spoint to obtain the cell type composition information before Splane can be applied: "one has to employ a cell type deconvolution algorithm such as Spoint to obtain the cell type composition information before Splane can be applied"
- [methods] Splane.base_model.SplaneModel.identify_spatial_domain: "Splane.base_model.SplaneModel.identify_spatial_domain"
- [readme] Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify uniform spatial domains that are transcriptomically and spatially coherent across multiple ST slices: "Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify uniform spatial domains that are transcriptomically and spatially coherent across multiple ST"
