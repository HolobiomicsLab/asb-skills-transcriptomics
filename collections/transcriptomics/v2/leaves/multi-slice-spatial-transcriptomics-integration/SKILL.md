---
name: multi-slice-spatial-transcriptomics-integration
description: Use when you have multiple ST slices of the same tissue (e.g., 12 human
  DLPFC 10X Visium sections) and need unified spatial domain labels across slices
  rather than independent per-slice clustering.
when_to_use_negative:
- You only have a single ST slice and no cross-slice batch effects to mitigate — joint
  training adds no benefit over per-slice analysis
- You are working with image-based ST data with single-cell resolution where Spoint
  deconvolution is unnecessary; check whether the module workflow still applies
- You need nonlinear slice alignment rather than domain identification — the current
  version of Scube does not support nonlinear alignment (unlike STAligner and PASTE)
edam_operation: http://edamontology.org/operation_3625
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: Splane
  role: GCN + adversarial learning module that jointly identifies spatial domains
    across multiple ST slices using cell type composition and spatial coordinates
  repo: github.com/QuKunLab/SPACEL
- name: Spoint
  role: Deconvolves per-spot cell type composition on each single slice, a required
    preprocessing input to Splane for seq-based ST data like 10X Visium
  repo: github.com/QuKunLab/SPACEL
- name: SPACEL
  role: Python package integrating Spoint, Splane, and Scube for the multi-slice analysis
    pipeline
  repo: github.com/QuKunLab/SPACEL
- name: Scube
  role: 'Downstream consumer: uses Splane''s spatial domains as the foundation for
    aligning consecutive slices and constructing 3D architecture'
  repo: github.com/QuKunLab/SPACEL
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:13883562499a2c8f2435ddcc9978d88a601988201b6a722cb31698eadf4085fa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SPACEL
  - Scube
  - Splane
  - Spoint
  edam_operation: http://edamontology.org/operation_3625
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multi-slice-spatial-transcriptomics-integration

## Summary

Jointly analyze multiple spatial transcriptomics (ST) slices with Splane, a graph convolutional network with adversarial learning, to identify spatial domains that are coherent across slices while minimizing batch effects. Joint modeling outperforms per-slice analysis on 12 human DLPFC Visium slices.

## When to use

Use when you have multiple ST slices of the same tissue (e.g., 12 human DLPFC 10X Visium sections) and need unified spatial domain labels across slices rather than independent per-slice clustering.

## When NOT to use

- You only have a single ST slice and no cross-slice batch effects to mitigate — joint training adds no benefit over per-slice analysis
- You are working with image-based ST data with single-cell resolution where Spoint deconvolution is unnecessary; check whether the module workflow still applies
- You need nonlinear slice alignment rather than domain identification — the current version of Scube does not support nonlinear alignment (unlike STAligner and PASTE)

## Inputs

- Multiple spatial transcriptomics slices (e.g., 12 human DLPFC 10X Visium sections) in AnnData/h5ad format with spot spatial coordinates
- Per-spot cell type composition matrices produced by Spoint deconvolution (required for seq-based ST data such as 10X Visium)

## Outputs

- Per-spot spatial domain labels that are transcriptomically and spatially coherent across all jointly analyzed slices
- Benchmark metrics comparing joint vs single-slice runs (Jaccard index, median ARI, average shifting distance), e.g., written to dlpfc_splane_benchmark_metrics.csv
- Domain labels that serve as the foundation for downstream Scube slice alignment and 3D architecture construction

## How to apply

First run cell type deconvolution on each slice with Spoint (Spoint.base_model.SpointModel.deconv_spatial), because Splane requires cell type composition as input for seq-based ST data such as 10X Visium. Attach the composition values to each spot via Splane.utils.add_cell_type_composition. Initialize SplaneModel with Splane.model.init_model and train the graph convolutional network with the adversarial learning algorithm jointly across all slices, so batch effects between slices are minimized. Then call Splane.base_model.SplaneModel.identify_spatial_domain to obtain per-spot spatial domain labels for every slice simultaneously. As a benchmark decision point, compare against a Splane-single baseline (one GCN per slice, no joint cell type composition input) using Jaccard index against annotated cortical layers, median ARI across slices, and average shifting distance; joint analysis reached JI/ARI 0.61/0.61 vs 0.53/0.57 and shifting distance 50 vs 262 um on 12 DLPFC slices.

## Related tools

- **Splane** (GCN + adversarial learning module that jointly identifies spatial domains across multiple ST slices using cell type composition and spatial coordinates) — github.com/QuKunLab/SPACEL
- **Spoint** (Deconvolves per-spot cell type composition on each single slice, a required preprocessing input to Splane for seq-based ST data like 10X Visium) — github.com/QuKunLab/SPACEL
- **SPACEL** (Python package integrating Spoint, Splane, and Scube for the multi-slice analysis pipeline) — github.com/QuKunLab/SPACEL
- **Scube** (Downstream consumer: uses Splane's spatial domains as the foundation for aligning consecutive slices and constructing 3D architecture) — github.com/QuKunLab/SPACEL

## Examples

```
Splane.utils.add_cell_type_composition(adata_list, prop_name); model = Splane.model.init_model(adata_list, ...); Splane.base_model.SplaneModel.identify_spatial_domain(model)
```

## Evaluation signals

- Jaccard index (JI) between predicted domains and annotated cortical layers should approach the joint-analysis benchmark of ~0.61 (vs 0.53 for Splane-single) on 12 DLPFC slices
- Median adjusted Rand index (ARI) across slices should approach ~0.61 joint vs ~0.57 single
- Average shifting distance of domain boundaries should be markedly lower with joint training (~50 um vs ~262 um per-slice)
- Every slice must receive per-spot domain labels via Splane.base_model.SplaneModel.identify_spatial_domain, and labels must be consistent (transcriptomically and spatially coherent) across slices
- For seq-based ST input, verify Spoint deconvolution was run first and attached via Splane.utils.add_cell_type_composition before model training

## Limitations

- When a new slice is added to the dataset, the entire deep learning model needs to be retrained from scratch, limiting computational efficiency for large-scale datasets
- Splane uses cell type composition as input, so seq-based ST datasets such as 10X Visium require a prior deconvolution step (e.g., Spoint)
- The downstream Scube alignment built on Splane domains does not support nonlinear alignment; this is deferred to a future version

## Evidence

- [abstract] Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains that are transcriptomically and spatially coherent across multiple ST slices: "Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains that are transcriptomically and spatially coherent across multiple ST slices"
- [discussion] using cell type composition as input and introducing adversarial training to the GCN model significantly minimizes batch effects: "using cell type composition as input and introducing adversarial training to the GCN model significantly minimizes batch effects, leading to a more robust and efficient method for spatial domain"
- [methods] Splane.base_model.SplaneModel.identify_spatial_domain: "Splane.base_model.SplaneModel.identify_spatial_domain"
- [discussion] one has to employ a cell type deconvolution algorithm such as Spoint to obtain the cell type composition information before Splane can be applied: "one has to employ a cell type deconvolution algorithm such as Spoint to obtain the cell type composition information before Splane can be applied"
- [readme] Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify uniform spatial domains that are transcriptomically and spatially coherent across multiple ST slices.: "Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify uniform spatial domains that are transcriptomically and spatially coherent across multiple ST"
