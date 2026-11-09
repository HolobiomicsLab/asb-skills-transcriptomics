---
name: spatial-domain-identification-gcn
description: Use when you have several spatial transcriptomics slices (e.g., 12 human
  DLPFC 10X Visium sections) with Spoint-derived per-spot cell type proportions and
  need uniform spatial domain labels across slices, plus a quantitative joint-vs-single
  comparison.
when_to_use_negative:
- Input is a single ST slice with no plan for cross-slice comparison — the joint adversarial
  training has no batch-effect target to act on; use Splane-single instead
- For seq-based ST datasets like 10X Visium where cell type deconvolution has not
  been run — Splane depends on Spoint composition input and cannot be applied directly
  to raw expression
- A new slice is added to an already-trained dataset — SPACEL requires retraining
  the entire model from scratch, which may be computationally prohibitive
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: SPACEL
  role: Toolkit containing Splane; installed from PyPI, docs at spacel.readthedocs.io
  repo: https://github.com/QuKunLab/SPACEL
- name: Splane
  role: GCN with adversarial learning that identifies uniform spatial domains across
    multiple ST slices from cell type composition
  repo: https://github.com/QuKunLab/SPACEL
- name: Spoint
  role: MLP + probabilistic model that deconvolutes per-spot cell type composition,
    the required input for Splane
  repo: https://github.com/QuKunLab/SPACEL
- name: Python
  role: Implementation language for running the SPACEL pipeline and computing benchmark
    metrics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:815649f363b7943693b78acfcfc7546f53422dbb7bbe1ea33c65c587d2f2c795
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QuKunLab/SPACEL
  tools:
  - Python
  - SPACEL
  - Splane
  - Spoint
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spatial-domain-identification-gcn

## Summary

Identify spatial domains that are transcriptomically and spatially coherent across multiple ST slices by training Splane's graph convolutional network with an adversarial learning algorithm on cell type composition input, and benchmark the joint model against per-slice (Splane-single) runs using Jaccard index, median ARI, and shifting distance.

## When to use

Use when you have several spatial transcriptomics slices (e.g., 12 human DLPFC 10X Visium sections) with Spoint-derived per-spot cell type proportions and need uniform spatial domain labels across slices, plus a quantitative joint-vs-single comparison.

## When NOT to use

- Input is a single ST slice with no plan for cross-slice comparison — the joint adversarial training has no batch-effect target to act on; use Splane-single instead
- For seq-based ST datasets like 10X Visium where cell type deconvolution has not been run — Splane depends on Spoint composition input and cannot be applied directly to raw expression
- A new slice is added to an already-trained dataset — SPACEL requires retraining the entire model from scratch, which may be computationally prohibitive

## Inputs

- Multiple spatial transcriptomics slices (e.g., 12 human DLPFC 10X Visium sections) with spot-level expression and spatial coordinates
- Per-spot cell type composition matrices produced by Spoint deconvolution
- Annotated cortical layer labels per spot for computing Jaccard index and ARI

## Outputs

- Per-spot spatial domain labels for each slice (joint Splane model)
- Per-slice baseline domain labels from Splane-single runs
- Benchmark table (e.g., dlpfc_splane_benchmark_metrics.csv) with JI, median ARI, and average shifting distance for joint vs single-slice analysis

## How to apply

First run Spoint deconvolution (required for seq-based ST like 10X Visium, since Splane consumes cell type composition, not raw expression) and attach per-spot compositions via Splane.utils.add_cell_type_composition. Initialize the SplaneModel with Splane.model.init_model and train the GCN with adversarial learning jointly across all slices — the adversarial term minimizes inter-slice batch effects. Call Splane.base_model.SplaneModel.identify_spatial_domain to get per-spot domain labels for the joint model. Then retrain one GCN per slice with no joint composition input (Splane-single) as the baseline, and compute per-slice Jaccard index against annotated cortical layers, median ARI across slices, and average shifting distance; on 12 DLPFC slices joint analysis gave JI/ARI 0.61/0.61 vs 0.53/0.57 and shifting distance 50 μm vs 262 μm. Write labels and the comparison to a metrics CSV (e.g., dlpfc_splane_benchmark_metrics.csv).

## Related tools

- **SPACEL** (Toolkit containing Splane; installed from PyPI, docs at spacel.readthedocs.io) — https://github.com/QuKunLab/SPACEL
- **Splane** (GCN with adversarial learning that identifies uniform spatial domains across multiple ST slices from cell type composition) — https://github.com/QuKunLab/SPACEL
- **Spoint** (MLP + probabilistic model that deconvolutes per-spot cell type composition, the required input for Splane) — https://github.com/QuKunLab/SPACEL
- **Python** (Implementation language for running the SPACEL pipeline and computing benchmark metrics)

## Examples

```
Splane.utils.add_cell_type_composition(ad_st, ad_st_list); Splane.model.init_model(GPU='auto', random_seed=10); Splane.base_model.SplaneModel.identify_spatial_domain(slice_ids_list=[0,1,2], method='leiden', start_res=0.4)
```

## Evaluation signals

- Jaccard index between predicted domains and annotated cortical layers should be higher for the joint run than Splane-single (paper: 0.61 vs 0.53 on 12 DLPFC slices)
- Median ARI across slices should be higher for joint analysis (paper: 0.61 vs 0.57)
- Average shifting distance between paired spatial domains across slices should drop sharply (paper: ~50 μm joint vs ~262 μm single)
- Output CSV must contain the JI/ARI/shifting-distance comparison for both joint and single-slice runs
- Domain labels must be produced for every spot of all 12 slices after identify_spatial_domain

## Limitations

- Adding a new slice requires retraining the entire deep learning model from scratch, limiting computational efficiency for large-scale datasets
- Splane requires cell type composition input, so a deconvolution step (Spoint) must precede it for seq-based ST datasets such as 10X Visium

## Evidence

- [abstract] Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains: "Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains"
- [discussion] joint analysis improves accuracy: "using cell type composition as input and introducing adversarial training to the GCN model significantly minimizes batch effects, leading to a more robust and efficient method for spatial domain"
- [methods] identify_spatial_domain API: "Splane.base_model.SplaneModel.identify_spatial_domain"
- [methods] add_cell_type_composition API: "Splane.utils.add_cell_type_composition"
- [discussion] Spoint prerequisite for Splane: "one has to employ a cell type deconvolution algorithm such as Spoint to obtain the cell type composition information before Splane can be applied"
- [methods] DLPFC tutorial dataset: "Deconvolution of cell types compostion on human brain Visium dataset <tutorials/Visium_human_DLPFC_Spoint>"
