---
name: multi-slice-spatial-domain-identification
description: Use when analyzing multiple ST slices (e.g., 11 breast cancer 10X Visium
  slices or 33 MERFISH MOp slices) and you need a single set of spatial domain labels
  (D1–D9) shared across slices rather than per-slice clusters.
when_to_use_negative:
- Single-slice spatial clustering where no cross-slice uniform domain labels are needed
  — Splane's adversarial training is designed for multi-slice joint analysis
- Raw seq-based ST data without deconvolution — Splane needs cell type composition
  input, so Spoint (or equivalent) must be run first on 10X Visium-like data
- Constructing 3D tissue alignment itself — that is Scube's role (Scube.alignment.align),
  which consumes Splane's domains rather than producing them
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3170
tools:
- name: SPACEL
  role: Toolkit providing the Splane module for multi-slice spatial domain identification
  repo: github.com/QuKunLab/SPACEL
- name: Splane
  role: Graph convolutional network with adversarial learning that takes cell type
    composition and spatial coordinates to identify uniform spatial domains across
    slices
  repo: github.com/QuKunLab/SPACEL
- name: Spoint
  role: MLP deconvolution producing per-spot cell type composition required as Splane
    input for seq-based data (10X Visium)
  repo: github.com/QuKunLab/SPACEL
- name: Scube
  role: 'Downstream consumer: uses Splane''s spatial domains as foundation for slice
    alignment and 3D stacking'
  repo: github.com/QuKunLab/SPACEL
- name: Python
  role: Implementation language for SPACEL pipeline code
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:868141fdf3ecb16bef72c337e2a55f5bce61572fd80fbf2c6263f17d3619b679
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PASTE
  - Python
  - SPACEL
  - STAligner
  - Scube
  - Splane
  - Spoint
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# multi-slice-spatial-domain-identification

## Summary

Identify uniform spatial domains that are transcriptomically and spatially coherent across multiple spatial transcriptomics slices using Splane's graph convolutional network with adversarial learning, taking per-spot cell type composition (e.g., from Spoint deconvolution) and spatial coordinates as input.

## When to use

Use when analyzing multiple ST slices (e.g., 11 breast cancer 10X Visium slices or 33 MERFISH MOp slices) and you need a single set of spatial domain labels (D1–D9) shared across slices rather than per-slice clusters.

## When NOT to use

- Single-slice spatial clustering where no cross-slice uniform domain labels are needed — Splane's adversarial training is designed for multi-slice joint analysis
- Raw seq-based ST data without deconvolution — Splane needs cell type composition input, so Spoint (or equivalent) must be run first on 10X Visium-like data
- Constructing 3D tissue alignment itself — that is Scube's role (Scube.alignment.align), which consumes Splane's domains rather than producing them

## Inputs

- Per-slice cell type composition matrices for spots/cells (e.g., from Spoint deconv_spatial on 10X Visium data)
- Spatial coordinates of spots/cells for each slice
- Multiple aligned or consecutive ST slices from the same tissue (e.g., human breast cancer Visium, mouse brain MERFISH, STARmap)

## Outputs

- Spatial domain assignments (e.g., D1–D9) that are uniform across all slices
- Trained Splane GCN model usable for downstream Scube alignment
- Per-spot domain label table (e.g., validation_report.csv entries)

## How to apply

First run cell type deconvolution on seq-based data (e.g., 10X Visium) with Spoint's SpointModel.deconv_spatial, since Splane requires cell type composition input. Aggregate per-slice composition matrices with Splane.utils.add_cell_type_composition so all slices share a unified cell-type-composition representation. Train the Splane graph convolutional network with adversarial learning (SplaneModel.train) on joint composition and spatial coordinates across all slices, then call SplaneModel.identify_spatial_domain to assign uniform domains; the adversarial component minimizes batch effects between slices. Rationale: cell-type distribution is more robust than gene expression for identifying uniform domains across multiple slices, and spatial domains from Splane also set a foundation for downstream Scube 3D alignment.

## Related tools

- **SPACEL** (Toolkit providing the Splane module for multi-slice spatial domain identification) — github.com/QuKunLab/SPACEL
- **Splane** (Graph convolutional network with adversarial learning that takes cell type composition and spatial coordinates to identify uniform spatial domains across slices) — github.com/QuKunLab/SPACEL
- **Spoint** (MLP deconvolution producing per-spot cell type composition required as Splane input for seq-based data (10X Visium)) — github.com/QuKunLab/SPACEL
- **Scube** (Downstream consumer: uses Splane's spatial domains as foundation for slice alignment and 3D stacking) — github.com/QuKunLab/SPACEL
- **Python** (Implementation language for SPACEL pipeline code)

## Examples

```
Splane.utils.add_cell_type_composition(adjs, exp_df); Splane.base_model.SplaneModel.train(sp_adata); Splane.base_model.SplaneModel.identify_spatial_domain(sp_adata)
```

## Evaluation signals

- All slices share the same domain label space (e.g., D1–D9) with each domain transcriptomically and spatially coherent across slices
- Biological validation: predicted immune domains (D7–D9) enriched for H&E-marked immune spots — expected ~44% in D9 vs <1% in tumor domains on breast cancer slices S1/S2/S5/S6, and ~72% CD3+ IF-marked spots in D9 (slice S10)
- Tumor domains flagged by inferCNV copy-number calls (1q/8q gains >= 0.2, 1p losses <= -0.2) partition coherently with the assigned domains
- Downstream Scube alignment quality on these domains: adjacent-slice SSIM/PCC ~0.71/0.76 on 33 MERFISH MOp slices, exceeding STAligner (0.61/0.65) and PASTE (0.57/0.61)

## Limitations

- Splane depends on cell type composition input, so seq-based ST datasets like 10X Visium require a prior deconvolution step (e.g., Spoint)
- Adding a new slice requires retraining the entire deep learning model from scratch, limiting computational efficiency for large-scale datasets
- Splane outputs do not include nonlinear alignment support; the current version of Scube built on these domains does not support nonlinear alignment unlike STAligner and PASTE
