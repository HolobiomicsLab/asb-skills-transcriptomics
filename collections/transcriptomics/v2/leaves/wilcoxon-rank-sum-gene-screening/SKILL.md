---
name: wilcoxon-rank-sum-gene-screening
description: Use when you have per-spot/per-cell normalized spatial transcriptomics
  expression with assigned spatial-domain labels and need domain-enriched genes or
  SVG candidates, e.g. recovering DLPFC cortical-layer SVGs.
when_to_use_negative:
- Spatial domain labels have not yet been assigned — Splane domain identification
  (after Spoint deconvolution for seq-based data like 10X Visium) must precede the
  per-domain differential test
- The goal is nonlinear slice alignment or 3D stacking rather than domain-enriched
  gene detection
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Splane
  role: Provides the spatial-domain labels whose per-domain marker genes are tested
    by the Wilcoxon rank-sum screen
  repo: https://github.com/QuKunLab/SPACEL
- name: SPACEL
  role: Toolkit whose Splane module outputs the spatial domains used for downstream
    SVG recovery; tutorial covers the human brain Visium DLPFC dataset
  repo: https://github.com/QuKunLab/SPACEL
- name: Python
  role: Implementation language for the pandas/scanpy DEG screening and ROC AUC computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:ccd8366521a56b32945c0057afca367105f25a201f975286dbde60b0c579279a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QuKunLab/SPACEL
  tools:
  - Python
  - SPACEL
  - Splane
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Wilcoxon rank-sum gene screening

## Summary

Screens genes enriched in identified spatial domains by running a Wilcoxon rank-sum differential expression test on the normalized expression matrix (scanpy rank_genes_groups), filtering DEGs at logFC > 0.5 and P < 0.01 to recover spatially variable genes (SVGs) matching known cortical-layer markers.

## When to use

Use when you have per-spot/per-cell normalized spatial transcriptomics expression with assigned spatial-domain labels and need domain-enriched genes or SVG candidates, e.g. recovering DLPFC cortical-layer SVGs.

## When NOT to use

- Spatial domain labels have not yet been assigned — Splane domain identification (after Spoint deconvolution for seq-based data like 10X Visium) must precede the per-domain differential test
- The goal is nonlinear slice alignment or 3D stacking rather than domain-enriched gene detection

## Inputs

- Normalized spatial transcriptomics expression matrix (e.g. 12 human DLPFC Visium slices)
- Per-spot spatial domain labels (e.g. Splane-identified spatial domains)
- Optional ground-truth SVG/marker list (cortical layers 1–6 and WM; 1917 genes)

## Outputs

- Per-domain DEG membership table (logFC > 0.5, P < 0.01)
- SVG recovery counts and ROC AUC summary (svg_recovery_metrics.csv)

## How to apply

Load the spatial-domain labels (e.g. Splane-identified domains for the 12 human DLPFC Visium slices) together with the normalized expression matrix using Python/pandas. For each spatial domain, run a Wilcoxon rank-sum differential expression test with scanpy's rank_genes_groups on the normalized expression. Filter the domain DEGs at logFC > 0.5 and P < 0.01 to obtain the gene candidate set for that domain. If a ground-truth marker/SVG list is available (e.g. cortical layers 1–6 and WM SVGs, 1917 genes), intersect it with the filtered DEG set to count recovered markers (expected 1714 of 1917 for Splane domains on DLPFC) and compute ROC AUC of recovery to benchmark the domain-identification method (Splane 0.90 vs PRECAST 0.89, STAligner 0.88, others < 0.83). Write the DEG membership table and recovery/ROC AUC summary to a CSV such as svg_recovery_metrics.csv.

## Related tools

- **Splane** (Provides the spatial-domain labels whose per-domain marker genes are tested by the Wilcoxon rank-sum screen) — https://github.com/QuKunLab/SPACEL
- **SPACEL** (Toolkit whose Splane module outputs the spatial domains used for downstream SVG recovery; tutorial covers the human brain Visium DLPFC dataset) — https://github.com/QuKunLab/SPACEL
- **Python** (Implementation language for the pandas/scanpy DEG screening and ROC AUC computation)

## Examples

```
import scanpy as sc; sc.tl.rank_genes_groups(adata, groupby='spatial_domain', method='wilcoxon'); sc.tl.filter_rank_genes_groups(adata, logfoldreg_thr=0.5, pval_cutoff=0.01)
```

## Evaluation signals

- Filtered DEGs at logFC > 0.5 and P < 0.01 recover 1714 of 1917 ground-truth DLPFC cortical-layer SVGs when intersected
- ROC AUC of SVG recovery reaches ~0.90 for Splane domains (vs PRECAST 0.89, STAligner 0.88, others < 0.83)
- Output CSV svg_recovery_metrics.csv contains both DEG membership and recovery/ROC AUC summary rows
- Per-domain Wilcoxon tests are run on the normalized expression matrix (scanpy rank_genes_groups), not raw counts

## Limitations

- For seq-based ST datasets such as 10X Visium, cell type composition deconvolution with Spoint must be run before Splane, so the whole chain is needed before this screen
- Benchmark sensitivity depends on the choice of ground-truth SVG list and on the logFC/P thresholds; SPACEL's model must be retrained from scratch when a new slice is added, affecting scalability of the upstream domains

## Evidence

- [other] Filter domain DEGs at logFC > 0.5 and P < 0.01 to obtain the recovered SVG candidate set.: [evidence span withheld — anchor status: not_found]
- [results] Splane achieved the highest SVG-identification accuracy (ROC AUC = 0.90), outperforming PRECAST (0.89), STAligner (0.88): [evidence span withheld — anchor status: not_found]
- [readme] After spatial domains are identified, genes that are enriched in each spatial domain can be detected by differential expression analysis between domains.: [evidence span withheld — anchor status: external_doc]
- [abstract] Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains: "Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains"
- [methods] Deconvolution of cell types compostion on human brain Visium dataset <tutorials/Visium_human_DLPFC_Spoint>: "Deconvolution of cell types compostion on human brain Visium dataset <tutorials/Visium_human_DLPFC_Spoint>"
