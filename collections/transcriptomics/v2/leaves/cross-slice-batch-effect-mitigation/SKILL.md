---
name: cross-slice-batch-effect-mitigation
description: Use when integrating multiple spatial transcriptomics slices (e.g., 11
  human breast cancer 10X Visium slices) whose batch effects would confound slice-by-slice
  domain identification, and uniform cross-slice domains are required.
when_to_use_negative:
- Slices with substantial defects (e.g., partial overlap, crop ratios >= 0.25) requiring
  nonlinear alignment — the current version of Scube/Splane pipeline does not support
  nonlinear alignment
- A new slice is added to an existing large-scale analysis — the entire deep learning
  model needs retraining from scratch, limiting computational efficiency
- Single-slice analysis with no cross-slice integration need — batch effect mitigation
  across slices is unnecessary
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: Spoint
  role: Deconvolutes per-spot cell type composition for each single slice using an
    MLP with a probabilistic model, providing the batch-robust input representation
  repo: github.com/QuKunLab/SPACEL
- name: Splane
  role: Aggregates cell type composition across slices (add_cell_type_composition)
    and trains a GCN with adversarial learning to identify uniform spatial domains
    across multiple slices
  repo: github.com/QuKunLab/SPACEL
- name: SPACEL
  role: Python toolkit integrating Spoint and Splane modules for cross-slice spatial
    domain identification
  repo: github.com/QuKunLab/SPACEL
- name: Python
  role: Implementation language for the SPACEL workflow
  repo: github.com/QuKunLab/SPACEL
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:ad2e079bd0daf4b09406787bac62c82b5cfbd17c97d8e213bb30b9f859f8746e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SPACEL
  - Splane
  - Spoint
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Cross-slice batch effect mitigation

## Summary

Mitigates batch effects when identifying spatial domains across multiple ST slices by using Spoint-derived cell type composition (rather than raw gene expression) as input to Splane's graph convolutional network with adversarial learning, producing domains (D1–D9) that are uniform across slices.

## When to use

Use when integrating multiple spatial transcriptomics slices (e.g., 11 human breast cancer 10X Visium slices) whose batch effects would confound slice-by-slice domain identification, and uniform cross-slice domains are required.

## When NOT to use

- Slices with substantial defects (e.g., partial overlap, crop ratios >= 0.25) requiring nonlinear alignment — the current version of Scube/Splane pipeline does not support nonlinear alignment
- A new slice is added to an existing large-scale analysis — the entire deep learning model needs retraining from scratch, limiting computational efficiency
- Single-slice analysis with no cross-slice integration need — batch effect mitigation across slices is unnecessary

## Inputs

- Multiple 10X Visium spatial transcriptomics slices (e.g., 11 human breast cancer slices)
- Per-slice single-cell reference data for Spoint deconvolution
- Per-spot spatial coordinates for each slice

## Outputs

- Aggregated cell type composition matrix shared across all slices (via add_cell_type_composition)
- Uniform spatial domain labels (D1–D9) per spot that are transcriptomically and spatially coherent across slices

## How to apply

First run Spoint's multiple-layer perceptron deconvolution (SpointModel.deconv_spatial) on each individual 10X Visium slice to obtain per-spot cell type composition, since Splane requires cell type composition rather than gene expression for seq-based ST data. Aggregate the per-slice composition matrices with Splane.utils.add_cell_type_composition so all slices (e.g., 11 breast cancer slices S1–S11) share a unified cell-type-composition representation. Train Splane's GCN with adversarial learning (SplaneModel.train) jointly on cell type composition and spatial coordinates across all slices, then call SplaneModel.identify_spatial_domain to assign uniform spatial domains (D1–D9). The rationale is that cell-type distribution is more robust than gene expression for identifying uniform domains across slices, and adversarial training to the GCN minimizes batch effects.

## Related tools

- **Spoint** (Deconvolutes per-spot cell type composition for each single slice using an MLP with a probabilistic model, providing the batch-robust input representation) — github.com/QuKunLab/SPACEL
- **Splane** (Aggregates cell type composition across slices (add_cell_type_composition) and trains a GCN with adversarial learning to identify uniform spatial domains across multiple slices) — github.com/QuKunLab/SPACEL
- **SPACEL** (Python toolkit integrating Spoint and Splane modules for cross-slice spatial domain identification) — github.com/QuKunLab/SPACEL
- **Python** (Implementation language for the SPACEL workflow) — github.com/QuKunLab/SPACEL

## Examples

```
from SPACEL import Spoint,Splane; Spoint.SpointModel.deconv_spatial(adata); Splane.utils.add_cell_type_composition(adata_list); Splane.SplaneModel.train(adata_list); Splane.SplaneModel.identify_spatial_domain(adata_list)
```

## Evaluation signals

- Assigned domain labels are uniform across all slices (same domain IDs, e.g., D1–D9, shared across all 11 breast cancer Visium slices) rather than slice-specific clusters
- Domains are transcriptomically and spatially coherent: downstream inferCNV flags tumor domains by 1q/8q gains >= 0.2 and 1p losses <= -0.2 consistent with predicted tumor domains
- Predicted immune domains D7–D9 are enriched for H&E-marked immune spots (44% in D9, 8% in D8, 6% in D7 vs <1% in intermediate/tumor domains in slices S1/S2/S5/S6) and CD3+ IF-marked spots (~72% in D9, slice S10)
- Cell type composition was aggregated consistently: every slice shares one unified composition representation before GCN training (add_cell_type_composition applied to all 11 slices)

## Limitations

- Splane depends on cell type composition input, requiring a deconvolution step (e.g., Spoint) for seq-based ST datasets like 10X Visium
- When a new slice is added, the entire deep learning model needs retraining from scratch, affecting computational efficiency for large-scale datasets
- Current Scube (downstream of Splane domains) does not support nonlinear alignment, unlike STAligner and PASTE

## Evidence

- [discussion] Adversarial training with cell type composition input reduces batch effects: "using cell type composition as input and introducing adversarial training to the GCN model significantly minimizes batch effects, leading to a more robust and efficient method for spatial domain"
- [intro] Splane joint multi-slice domain identification from composition and coordinates: "Splane jointly analyzes multiple ST slices and identifies spatial domains based on the cell type composition and spatial coordinate information of spots/cells."
- [discussion] Deconvolution prerequisite for seq-based ST data: "one has to employ a cell type deconvolution algorithm such as Spoint to obtain the cell type composition information before Splane can be applied"
- [intro] Cell type distribution more robust across slices than gene expression: "However, it has been reported that cell-type distribution is more robust for identifying uniform domains across multiple slices9."
- [readme] Splane GCN with adversarial learning identifies domains coherent across slices: "Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify uniform spatial domains that are transcriptomically and spatially coherent across multiple ST"
