---
name: svg-recovery-overlap-analysis
description: Use when Splane-style spatial domain labels exist for Visium slices (e.g.,
  the 12 human DLPFC slices) and you must quantify how many annotated layer SVGs each
  domain-detection method recovers.
when_to_use_negative:
- When spatial-domain labels have not been produced (no Splane/PRECAST/STAligner output),
  since the Wilcoxon domain DEG test presupposes domain assignments per spot
- For seq-based 10X Visium data where cell type deconvolution has not been run first
  — Splane needs cell type composition input before its domains can be used
- When the goal is nonlinear 3D slice alignment rather than domain/SVG quality assessment
  — the paper notes the current version of Scube does not support nonlinear alignment,
  so this skill's overlap metric is out of scope there
edam_operation: http://edamontology.org/operation_2495
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SPACEL (Splane)
  role: Source of the spatial domain labels whose DEGs are intersected with the ground-truth
    SVG list
  repo: https://github.com/QuKunLab/SPACEL
- name: Spoint
  role: Mandatory upstream cell type composition deconvolution feeding Splane for
    seq-based 10X Visium data
  repo: https://github.com/QuKunLab/SPACEL
- name: scanpy
  role: Provides rank_genes_groups Wilcoxon rank-sum test used to derive per-domain
    DEGs on the normalized expression matrix
- name: Python
  role: Implementation language for the pandas-based SVG overlap computation and ROC
    AUC reporting
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:33a4bdf338341603f60519aa328eea466fe885b1a06108a360bf4fd6c4bdd231
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QuKunLab/SPACEL
  tools:
  - Python
  - SPACEL
  - SPACEL (Splane)
  - Splane
  - Spoint
  - scanpy
  edam_operation: http://edamontology.org/operation_2495
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# svg-recovery-overlap-analysis

## Summary

Benchmarks how faithfully spatial-domain labels recover spatially variable genes (SVGs) by intersecting Wilcoxon-derived domain DEGs against a ground-truth SVG list and scoring recovery with ROC AUC, as used to show Splane (ROC AUC 0.90) beats PRECAST (0.89), STAligner (0.88) and other methods (AUC < 0.83) on DLPFC cortical-layer SVGs.

## When to use

Use when Splane-style spatial domain labels exist for Visium slices (e.g., the 12 human DLPFC slices) and you must quantify how many annotated layer SVGs each domain-detection method recovers.

## When NOT to use

- When spatial-domain labels have not been produced (no Splane/PRECAST/STAligner output), since the Wilcoxon domain DEG test presupposes domain assignments per spot
- For seq-based 10X Visium data where cell type deconvolution has not been run first — Splane needs cell type composition input before its domains can be used
- When the goal is nonlinear 3D slice alignment rather than domain/SVG quality assessment — the paper notes the current version of Scube does not support nonlinear alignment, so this skill's overlap metric is out of scope there

## Inputs

- Splane-identified spatial domain labels for the 12 human DLPFC Visium slices
- Normalized gene expression matrix (e.g., AnnData) for the same spots
- Ground-truth layer-annotated SVG list covering cortical layers 1-6 and white matter (WM), 1917 genes
- Baseline SVG recovery results for PRECAST and STAligner for comparison

## Outputs

- Per-domain DEG membership table with logFC > 0.5 and P < 0.01 filters
- SVG recovery counts (recovered / total, e.g. 1714 / 1917) per method
- svg_recovery_metrics.csv with recovery counts and ROC AUC per method (Splane 0.90, PRECAST 0.89, STAligner 0.88)

## How to apply

Load Splane-identified spatial domain labels for the 12 human DLPFC Visium slices together with the layer-annotated ground-truth SVG list (cortical layers 1-6 and WM) into Python/pandas. For each spatial domain, run a Wilcoxon rank-sum differential expression test with scanpy's rank_genes_groups on the normalized expression matrix, then filter domain DEGs at logFC > 0.5 and P < 0.01 to build the recovered SVG candidate set. Intersect the domain DEG set with the ground-truth SVG list to count recovered SVGs (Splane is expected to recover 1714 of 1917), compute the ROC AUC of SVG recovery, and report it alongside the PRECAST (0.89) and STAligner (0.88) baselines. Because Splane requires cell-type composition as input for seq-based data like 10X Visium, the upstream Spoint deconvolution must have been run before the domain labels are valid for this test. The rationale: DEGs enriched per domain should recapitulate genes annotated to cortical layers if the domains are transcriptomically and spatially coherent, making the overlap fraction and ROC AUC a direct accuracy proxy for domain identification.

## Related tools

- **SPACEL (Splane)** (Source of the spatial domain labels whose DEGs are intersected with the ground-truth SVG list) — https://github.com/QuKunLab/SPACEL
- **Spoint** (Mandatory upstream cell type composition deconvolution feeding Splane for seq-based 10X Visium data) — https://github.com/QuKunLab/SPACEL
- **scanpy** (Provides rank_genes_groups Wilcoxon rank-sum test used to derive per-domain DEGs on the normalized expression matrix)
- **Python** (Implementation language for the pandas-based SVG overlap computation and ROC AUC reporting)

## Examples

```
import scanpy as sc; sc.tl.rank_genes_groups(adata, groupby='splane_domain', method='wilcoxon')
```

## Evaluation signals

- SVG recovery count reaches the expected magnitude: Splane recovers 1714 of the 1917 ground-truth cortical-layer SVGs
- ROC AUC for Splane equals ~0.90 and stays at or above baselines PRECAST (0.89) and STAligner (0.88), with all other examined methods below 0.83
- Every reported domain DEG satisfies both filter thresholds: logFC > 0.5 and P < 0.01 from the Wilcoxon rank-sum test
- Ground-truth SVG list is complete across cortical layers 1-6 plus WM and covers all 12 DLPFC slices before intersection
- svg_recovery_metrics.csv contains both the DEG membership table and the recovery/ROC AUC summary for each compared method

## Limitations

- The comparison is anchored to a specific annotated ground-truth SVG list (1917 genes on the DLPFC slices); recovery metrics are only meaningful against an equally curated SVG annotation
- Splane depends on cell type composition input, so any error in the Spoint deconvolution propagates into the domain labels and hence the SVG recovery score
- Comparative AUC values are specific to the DLPFC cortical-layer benchmark and the examined method set; they should not be extrapolated to other tissues or ST technologies without rerunning the overlap analysis

## Evidence

- [abstract] Splane GCN domain identification: "Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains that are transcriptomically and spatially coherent across multiple ST slices"
- [discussion] Spoint prerequisite for seq-based ST: "one has to employ a cell type deconvolution algorithm such as Spoint to obtain the cell type composition information before Splane can be applied"
- [readme] Splane benchmarking on Visium data: "Splane tutorial: Identify uniform spatial domain on human breast cancer Visium dataset"
- [discussion] SPACEL comparative performance claim: "the SPACEL modules outperform state-of-the-art methods for each of these tasks through analyses of 32 simulated and 11 real ST datasets acquired using five distinct ST technologies"
