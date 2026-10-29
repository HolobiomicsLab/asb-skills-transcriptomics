---
name: cell-type-composition-input-construction
description: Use when preparing seq-based ST data (e.g., 10X Visium slices such as
  the 12 human DLPFC slices) for Splane spatial domain identification, which requires
  per-spot cell type composition rather than gene expression.
when_to_use_negative:
- Input is image-based ST data with single-cell resolution (e.g., MERFISH, STARmap)
  where per-cell identity is directly observed and no deconvolution step is needed
  before Splane
- The goal is to run Splane-single per-slice baselines that explicitly do not use
  joint cell type composition input
- A new slice is being added to an already-trained SPACEL model — the entire deep
  learning model must be retrained from scratch, which may affect computational efficiency
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_6170
tools:
- name: SPACEL
  role: Python package providing the Spoint deconvolution and Splane composition-input
    modules
  repo: github.com/QuKunLab/SPACEL
- name: Spoint
  role: Deconvolutes cell type composition for each spot on a single ST slice prior
    to Splane
  repo: github.com/QuKunLab/SPACEL
- name: Splane
  role: Consumes the cell type composition via add_cell_type_composition as GCN input
    for spatial domain identification
  repo: github.com/QuKunLab/SPACEL
- name: Python
  role: Implementation language of the SPACEL workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:3a442ce397fb562863525faee41cae091f765e5d84fb58d9aac969dd0dd5897b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SPACEL
  - Splane
  - Spoint
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_6170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cell-type-composition-input-construction

## Summary

Constructs the cell type composition matrix required by Splane by running Spoint deconvolution on 10X Visium slices and attaching the per-spot composition values with Splane.utils.add_cell_type_composition, since Splane's GCN takes cell type composition (not raw gene expression) as input.

## When to use

Use when preparing seq-based ST data (e.g., 10X Visium slices such as the 12 human DLPFC slices) for Splane spatial domain identification, which requires per-spot cell type composition rather than gene expression.

## When NOT to use

- Input is image-based ST data with single-cell resolution (e.g., MERFISH, STARmap) where per-cell identity is directly observed and no deconvolution step is needed before Splane
- The goal is to run Splane-single per-slice baselines that explicitly do not use joint cell type composition input
- A new slice is being added to an already-trained SPACEL model — the entire deep learning model must be retrained from scratch, which may affect computational efficiency

## Inputs

- 10X Visium spatial transcriptomics slice (e.g., human DLPFC Visium count matrices with spot coordinates)
- Single-cell reference data for Spoint deconvolution
- Pretrained/derived Spoint cell type proportions per spot

## Outputs

- Per-spot cell type composition matrix attached to the ST AnnData via Splane.utils.add_cell_type_composition
- Composition-annotated slices ready as input to SplaneModel.init_model and identify_spatial_domain

## How to apply

For seq-based ST datasets like 10X Visium, first run Spoint's deconvolution (Spoint.base_model.SpointModel.deconv_spatial), which embeds a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition for each spot on a single ST slice. Then attach the Spoint-derived composition values to each spot using Splane.utils.add_cell_type_composition so the composition is available as model input. When training Splane jointly across multiple slices (e.g., all 12 DLPFC slices), supply this composition together with spatial coordinates so the GCN with adversarial learning can minimize batch effects between slices. Rationale: cell-type distribution is reported to be more robust than gene expression for identifying uniform domains across multiple slices.

## Related tools

- **SPACEL** (Python package providing the Spoint deconvolution and Splane composition-input modules) — github.com/QuKunLab/SPACEL
- **Spoint** (Deconvolutes cell type composition for each spot on a single ST slice prior to Splane) — github.com/QuKunLab/SPACEL
- **Splane** (Consumes the cell type composition via add_cell_type_composition as GCN input for spatial domain identification) — github.com/QuKunLab/SPACEL
- **Python** (Implementation language of the SPACEL workflow)

## Examples

```
Spoint.base_model.SpointModel.deconv_spatial(...); Splane.utils.add_cell_type_composition(ad_st, proportions); model = Splane.model.init_model(...); Splane.base_model.SplaneModel.identify_spatial_domain(model)
```

## Evaluation signals

- Every spot in the 10X Visium slice carries a cell type composition vector (summing to 1) in obs after add_cell_type_composition
- Downstream Splane identify_spatial_domain runs without errors on the composition-annotated AnnData
- Joint multi-slice runs using the composition input show lower batch effects than per-slice runs (e.g., average shifting distance 50 μm joint vs 262 μm single on the 12 DLPFC slices)
- Spatial domain accuracy improves versus non-composition baselines: joint analysis achieved average JI/median ARI of 0.61/0.61 vs 0.53/0.57 for Splane-single
- Spoint deconvolution is completed for every slice before Splane is invoked, as required for seq-based ST data

## Limitations

- Splane depends on cell type composition input, requiring a deconvolution step (e.g., Spoint) for seq-based ST datasets like 10X Visium
- Adding a new slice requires retraining the entire deep learning model from scratch, limiting computational efficiency for large-scale datasets

## Evidence

- [abstract] Spoint embeds a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition: "SPACEL comprises three modules: Spoint embeds a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition"
- [discussion] Composition required before Splane: "one has to employ a cell type deconvolution algorithm such as Spoint to obtain the cell type composition information before Splane can be applied"
- [intro] Composition is more robust than expression: "However, it has been reported that cell-type distribution is more robust for identifying uniform domains across multiple slices9."
- [discussion] Adversarial training minimizes batch effects: "using cell type composition as input and introducing adversarial training to the GCN model significantly minimizes batch effects, leading to a more robust and efficient method for spatial domain"
- [methods] add_cell_type_composition filter step: "Splane.utils.add_cell_type_composition"
- [readme] README: DLPFC Spoint tutorial: "Spoint tutorial: Deconvolution of cell types compostion on human brain Visium dataset"
