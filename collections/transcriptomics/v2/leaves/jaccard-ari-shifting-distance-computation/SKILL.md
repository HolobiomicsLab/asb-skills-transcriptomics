---
name: jaccard-ari-shifting-distance-computation
description: Use when comparing spatial domain labels against layer annotations on
  multi-slice ST data (e.g., 12 DLPFC Visium slices) to quantify accuracy via JI,
  median ARI, and average shifting distance.
when_to_use_negative:
- The tissue has no ground-truth layer/domain annotations (e.g., a breast cancer Visium
  dataset), so JI against annotated cortical layers and shifting distance cannot be
  computed.
- The data are single slices with no joint vs single comparison to make — ARI/JI benchmarking
  of multi-slice integration is not the goal.
edam_operation: http://edamontology.org/operation_3643
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: Splane
  role: Generates the spatial domain labels (joint and single-slice) that are scored
    by JI, median ARI, and shifting distance
  repo: github.com/QuKunLab/SPACEL
- name: Spoint
  role: Produces the cell type composition input that must be attached to spots before
    Splane domain identification on the seq-based 10X Visium DLPFC data
  repo: github.com/QuKunLab/SPACEL
- name: SPACEL
  role: Toolkit providing the Splane/Spoint modules and the DLPFC tutorial pipeline
    against which the metrics are computed
  repo: github.com/QuKunLab/SPACEL
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:c978c498a801a4cd6bdd27e5199a6a4644ca98218ad20e63561e4a7825c112c5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SPACEL
  - Splane
  - Spoint
  edam_operation: http://edamontology.org/operation_3643
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# jaccard-ari-shifting-distance-computation

## Summary

Benchmarking metric suite for spatial domain identification on human DLPFC Visium slices: compute the Jaccard index (JI) between predicted domains and annotated cortical layers, the median adjusted Rand index (ARI) across slices, and the average shifting distance, to compare joint vs single-slice analyses (e.g., Splane vs Splane-single).

## When to use

Use when comparing spatial domain labels against layer annotations on multi-slice ST data (e.g., 12 DLPFC Visium slices) to quantify accuracy via JI, median ARI, and average shifting distance.

## When NOT to use

- The tissue has no ground-truth layer/domain annotations (e.g., a breast cancer Visium dataset), so JI against annotated cortical layers and shifting distance cannot be computed.
- The data are single slices with no joint vs single comparison to make — ARI/JI benchmarking of multi-slice integration is not the goal.

## Inputs

- Per-spot spatial domain labels for the joint model (e.g., from Splane.base_model.SplaneModel.identify_spatial_domain)
- Per-spot spatial domain labels for the single-slice baseline (Splane-single)
- Annotated cortical layer labels for the 12 human DLPFC Visium slices
- Spoint-derived cell type proportions attached to spots via Splane.utils.add_cell_type_composition

## Outputs

- Per-slice and average Jaccard index (JI) between predicted domains and cortical layer annotations
- Median adjusted Rand index (ARI) across slices
- Average shifting distance (μm) between predicted and annotated layers
- dlpfc_splane_benchmark_metrics.csv containing domain labels and the joint-vs-single metric comparison

## How to apply

Run the candidate spatial domain identification method (e.g., Splane joint model trained jointly across all 12 DLPFC slices with adversarial learning) and the baseline (Splane-single, one GCN per slice without joint cell type composition input) on the same Spoint-deconvolved DLPFC Visium data. For each run, compute the Jaccard index between predicted spatial domains and annotated cortical layers per slice, the median adjusted Rand index across slices, and the average shifting distance between predicted and annotated domain boundaries. In the SPACEL benchmark this separated joint analysis (JI 0.61, median ARI 0.61, shifting distance 50 μm) from single-slice analysis (0.53/0.57/262 μm). Write the domain labels and the joint-vs-single JI/ARI/shifting-distance comparison to a metrics CSV (e.g., dlpfc_splane_benchmark_metrics.csv).

## Related tools

- **Splane** (Generates the spatial domain labels (joint and single-slice) that are scored by JI, median ARI, and shifting distance) — github.com/QuKunLab/SPACEL
- **Spoint** (Produces the cell type composition input that must be attached to spots before Splane domain identification on the seq-based 10X Visium DLPFC data) — github.com/QuKunLab/SPACEL
- **SPACEL** (Toolkit providing the Splane/Spoint modules and the DLPFC tutorial pipeline against which the metrics are computed) — github.com/QuKunLab/SPACEL

## Examples

```
Splane.utils.add_cell_type_composition(adata); model = Splane.model.init_model(...); Splane.base_model.SplaneModel.identify_spatial_domain(model)
```

## Evaluation signals

- Joint analysis should exceed the single-slice baseline: JI 0.61 vs 0.53 and median ARI 0.61 vs 0.57 on the 12 DLPFC slices
- Average shifting distance should be far lower for joint analysis (50 μm) than single-slice analysis (262 μm)
- Domain label output must contain per-spot labels for all 12 slices in both runs before metrics can be computed
- Metrics CSV (dlpfc_splane_benchmark_metrics.csv) must contain JI, median ARI, and shifting distance for both joint and single conditions

## Limitations

- Metrics assume trusted cortical layer annotations; on tissues lacking layered annotations the JI/shifting-distance measures are undefined.
- Adding a new slice requires retraining the entire deep learning model from scratch, which affects computational efficiency and may alter the compared domain labels.
- Splane depends on cell type composition input, so on seq-based ST data like 10X Visium the metrics also inherit error from the upstream Spoint deconvolution.

## Evidence

- [results] JI/ARI thresholds joint vs single: [evidence span withheld — anchor status: not_found]
- [methods] Splane domain identification step: "Splane.base_model.SplaneModel.identify_spatial_domain"
- [abstract] Splane GCN adversarial learning for joint slices: "Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains that are transcriptomically and spatially coherent across multiple ST slices"
- [discussion] Deconvolution prerequisite for seq-based ST: "one has to employ a cell type deconvolution algorithm such as Spoint to obtain the cell type composition information before Splane can be applied"
- [methods] SPACEL DLPFC tutorial data: "Deconvolution of cell types compostion on human brain Visium dataset <tutorials/Visium_human_DLPFC_Spoint>"
