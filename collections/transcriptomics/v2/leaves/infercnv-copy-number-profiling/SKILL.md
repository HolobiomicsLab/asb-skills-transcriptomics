---
name: infercnv-copy-number-profiling
description: Use when Splane-identified spatial domains (e.g., D1–D9 across 11 breast
  cancer Visium slices) need orthogonal validation via copy-number aberration signatures
  and histology-based immune-spot enrichment.
when_to_use_negative:
- Data are image-based ST with single-cell resolution (e.g., MERFISH, STARmap), where
  Spoint deconvolution before Splane is unnecessary
- The goal is 3D alignment of consecutive slices rather than domain validation — use
  Scube's align step instead
- No matched normal/tumor expression reference is available to support inferCNV inference
  of chromosomal gains and losses
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SPACEL
  role: Toolkit providing Spoint and Splane modules that generate the cell type compositions
    and spatial domains being validated by inferCNV and immune-spot staining
  repo: github.com/QuKunLab/SPACEL
- name: Spoint
  role: Multiple-layer perceptron deconvolution (SpointModel.deconv_spatial) producing
    per-spot cell type composition from each Visium slice
  repo: github.com/QuKunLab/SPACEL
- name: Splane
  role: GCN with adversarial learning (SplaneModel.train, SplaneModel.identify_spatial_domain)
    assigning uniform spatial domains D1–D9 across the 11 slices
  repo: github.com/QuKunLab/SPACEL
- name: inferCNV
  role: Computes per-spatial-domain copy-number profiles used to flag tumor domains
    by 1q/8q gains >= 0.2 and 1p losses <= -0.2
- name: Python
  role: Language in which SPACEL's API calls (add_cell_type_composition, deconv_spatial,
    identify_spatial_domain) and the validation report are executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:1f314d3c612f836eb9eddc442501bac6caf334dad311344c82a16d60adda4abf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SPACEL
  - Splane
  - Spoint
  - inferCNV
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# infercnv-copy-number-profiling

## Summary

Validate Splane-predicted spatial domains on 10X Visium breast cancer slices by computing per-domain inferCNV copy-number profiles and quantifying immune-spot enrichment in H&E/CD3 staining, confirming tumor domains (1q/8q gains) and immune domains (D7–D9).

## When to use

Use when Splane-identified spatial domains (e.g., D1–D9 across 11 breast cancer Visium slices) need orthogonal validation via copy-number aberration signatures and histology-based immune-spot enrichment.

## When NOT to use

- Data are image-based ST with single-cell resolution (e.g., MERFISH, STARmap), where Spoint deconvolution before Splane is unnecessary
- The goal is 3D alignment of consecutive slices rather than domain validation — use Scube's align step instead
- No matched normal/tumor expression reference is available to support inferCNV inference of chromosomal gains and losses

## Inputs

- 11 breast cancer 10X Visium spatial transcriptomics slices with spot expression matrices and spatial coordinates
- H&E-stained images marking immune spots for slices S1, S2, S5, S6
- CD3 immunofluorescence-marked spot annotations for slice S10
- Single-cell or per-domain expression matrices for inferCNV copy-number inference

## Outputs

- Uniform spatial domain assignments D1–D9 per spot across all 11 slices
- Per-domain inferCNV copy-number profiles with 1q/8q gain and 1p loss statistics
- Per-domain immune-spot percentages (H&E+ and CD3+) for domains D7–D9
- validation_report.csv containing domain assignments, copy-number calls, and immune-spot percentages

## How to apply

First run Spoint's multiple-layer perceptron deconvolution (Spoint.base_model.SpointModel.deconv_spatial) on each 10X Visium slice to obtain per-spot cell type composition, since Splane requires composition input for seq-based ST. Aggregate per-slice compositions with Splane.utils.add_cell_type_composition, then train the Splane GCN with adversarial learning (SplaneModel.train) and assign uniform domains D1–D9 via SplaneModel.identify_spatial_domain. For each domain, compute inferCNV copy-number profiles and flag tumor domains when 1q/8q gains are >= 0.2 and 1p losses are <= -0.2. For immune domains D7–D9, quantify the percentage of H&E-marked immune spots (expected ~44% in D9 on slices S1/S2/S5/S6) and CD3+ IF-marked spots (~72% in D9 on slice S10), writing assignments, CNV calls, and percentages to validation_report.csv.

## Related tools

- **SPACEL** (Toolkit providing Spoint and Splane modules that generate the cell type compositions and spatial domains being validated by inferCNV and immune-spot staining) — github.com/QuKunLab/SPACEL
- **Spoint** (Multiple-layer perceptron deconvolution (SpointModel.deconv_spatial) producing per-spot cell type composition from each Visium slice) — github.com/QuKunLab/SPACEL
- **Splane** (GCN with adversarial learning (SplaneModel.train, SplaneModel.identify_spatial_domain) assigning uniform spatial domains D1–D9 across the 11 slices) — github.com/QuKunLab/SPACEL
- **inferCNV** (Computes per-spatial-domain copy-number profiles used to flag tumor domains by 1q/8q gains >= 0.2 and 1p losses <= -0.2)
- **Python** (Language in which SPACEL's API calls (add_cell_type_composition, deconv_spatial, identify_spatial_domain) and the validation report are executed)

## Examples

```
from SPACEL import Splane; Splane.utils.add_cell_type_composition(adatas); Splane.base_model.SplaneModel.train(...); Splane.base_model.SplaneModel.identify_spatial_domain(...)
```

## Evaluation signals

- Tumor domains are correctly flagged when inferCNV shows 1q/8q gains >= 0.2 and 1p losses <= -0.2
- Immune domains D9, D8, D7 on slices S1/S2/S5/S6 contain 44%, 8%, and 6% H&E-marked immune spots respectively
- Intermediate and tumor domains show <1% H&E-marked immune spots, confirming immune-domain specificity
- D9 on slice S10 contains ~72% CD3+ IF-marked spots, matching the expected immune enrichment
- All 11 slices receive a shared set of uniform domains D1–D9 after add_cell_type_composition aggregation and GCN training

## Limitations

- Adding a new slice requires retraining the entire deep learning model from scratch, limiting computational efficiency for large-scale datasets
- Splane depends on cell type composition input, so seq-based ST datasets like 10X Visium require a prior Spoint deconvolution step
- SPACEL domains are transcriptomically/spatially coherent groupings; their tumor/immune identity is only established via downstream orthogonal evidence such as inferCNV and H&E/CD3 spot counts

## Evidence

- [abstract] spatial domain assignment: "Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains"
- [discussion] deconvolution prerequisite: "one has to employ a cell type deconvolution algorithm such as Spoint to obtain the cell type composition information before Splane can be applied"
- [intro] multi-slice domain input: "Splane jointly analyzes multiple ST slices and identifies spatial domains based on the cell type composition and spatial coordinate information of spots/cells."
- [methods] per-spot composition: "Spoint embedded a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition for each spot on single ST slice."
- [readme] toolkit scope: "SPACEL consists of three modules:"
