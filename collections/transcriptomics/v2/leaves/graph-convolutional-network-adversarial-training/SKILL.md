---
name: graph-convolutional-network-adversarial-training
description: Use when multiple spatial transcriptomics slices (e.g., 33 MERFISH MOp
  slices) need jointly identified, uniform spatial domains that are coherent across
  slices and serve as the basis for downstream 3D alignment.
when_to_use_negative:
- Input is a single ST slice only — Splane is designed for joint analysis of multiple
  ST slices; use single-slice domain methods (e.g., SpaGCN) instead
- Seq-based ST data (e.g., 10X Visium) without a preceding cell type deconvolution
  step — Splane requires cell type composition input, so run Spoint first
edam_operation: http://edamontology.org/operation_3432
edam_topics: []
tools:
- name: SPACEL
  role: Python package providing the Splane GCN module with adversarial learning for
    cross-slice spatial domain identification
  repo: github.com/QuKunLab/SPACEL
- name: Splane
  role: Graph convolutional network with adversarial training; add_cell_type_composition
    builds inputs, train fits the model, identify_spatial_domain assigns domains
  repo: github.com/QuKunLab/SPACEL
- name: Spoint
  role: Produces the cell type composition input required by Splane for seq-based
    ST datasets such as 10X Visium
  repo: github.com/QuKunLab/SPACEL
- name: Scube
  role: Consumes the spatial domains identified by Splane to align consecutive slices
    and construct the 3D architecture
  repo: github.com/QuKunLab/SPACEL
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:e5fbebbc863e1df7e2bc0a626b803dbcee9abd138ff5d4c0252d92407b7a3d98
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# graph-convolutional-network-adversarial-training

## Summary

Identify spatial domains across multiple ST slices by training Splane's graph convolutional network with an adversarial learning algorithm on cell type composition and spatial coordinates, minimizing batch effects so domains are transcriptomically and spatially coherent across slices.

## When to use

Use when multiple spatial transcriptomics slices (e.g., 33 MERFISH MOp slices) need jointly identified, uniform spatial domains that are coherent across slices and serve as the basis for downstream 3D alignment.

## When NOT to use

- Input is a single ST slice only — Splane is designed for joint analysis of multiple ST slices; use single-slice domain methods (e.g., SpaGCN) instead
- Seq-based ST data (e.g., 10X Visium) without a preceding cell type deconvolution step — Splane requires cell type composition input, so run Spoint first

## Inputs

- Multiple ST slices (e.g., 33 MERFISH mouse primary motor cortex slices) as AnnData objects
- Per-spot/per-cell cell type composition (e.g., produced by Spoint deconvolution for seq-based ST such as 10X Visium)
- Spatial coordinates / spatial neighbor graphs for each slice

## Outputs

- Spatial domain labels per spot/cell across all slices (e.g., seven coherent domains)
- Domain-coordinate information that feeds Scube.alignment.align for 3D stacking

## How to apply

First assemble per-spot/per-cell inputs with Splane.utils.add_cell_type_composition, combining cell type composition (from Spoint deconvolution for seq-based data like 10X Visium; directly available for image-based data such as MERFISH) and spatial neighbor graphs. Train Splane.base_model.SplaneModel with its GCN and adversarial learning algorithm: the adversarial term minimizes batch effects between slices so domains are shared rather than slice-specific. Call Splane.base_model.SplaneModel.identify_spatial_domain to assign spatial domains — e.g., seven transcriptomically and spatially coherent domains across the 33 MERFISH mouse primary motor cortex slices. The rationale is that cell-type distribution input is more robust than raw gene expression for cross-slice consistency, and spatial domains identified here set the foundation for Scube's subsequent slice alignment and 3D architecture construction.

## Related tools

- **SPACEL** (Python package providing the Splane GCN module with adversarial learning for cross-slice spatial domain identification) — github.com/QuKunLab/SPACEL
- **Splane** (Graph convolutional network with adversarial training; add_cell_type_composition builds inputs, train fits the model, identify_spatial_domain assigns domains) — github.com/QuKunLab/SPACEL
- **Spoint** (Produces the cell type composition input required by Splane for seq-based ST datasets such as 10X Visium) — github.com/QuKunLab/SPACEL
- **Scube** (Consumes the spatial domains identified by Splane to align consecutive slices and construct the 3D architecture) — github.com/QuKunLab/SPACEL

## Examples

```
from SPACEL import Splane; Splane.utils.add_cell_type_composition(adatas); Splane.base_model.SplaneModel.train(adatas); Splane.base_model.SplaneModel.identify_spatial_domain(adatas)
```

## Evaluation signals

- Identified spatial domains are both transcriptomically and spatially coherent across all slices (e.g., seven domains in the 33-slice MERFISH MOp dataset)
- Downstream Scube alignment achieves average adjacent-slice SSIM/PCC of 0.71/0.76, beating STAligner (0.61/0.65) and PASTE (0.57/0.61) on the same domains
- Domains are consistent across slices rather than slice-specific, indicating adversarial training successfully minimized batch effects
- Stacked 3D architecture from Scube is not twisted, unlike STAligner- and PASTE-based stacks on the same data

## Limitations

- Adding a new slice requires retraining the entire deep learning model from scratch, limiting computational efficiency for large-scale datasets
- Splane depends on cell type composition input, requiring a deconvolution step (e.g., Spoint) for seq-based ST datasets like 10X Visium

## Evidence

- [abstract] Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains: "Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains"
- [discussion] adversarial training minimizes batch effects: "using cell type composition as input and introducing adversarial training to the GCN model significantly minimizes batch effects"
- [discussion] domains found by Splane underpin Scube alignment: "Spatial domains identified in Splane also set a foundation for Scube to achieve an accurate alignment of the consecutive slices and hence a precise 3D architecture construction"
- [methods] identify_spatial_domain is the concrete API step: "Splane.base_model.SplaneModel.identify_spatial_domain"
- [readme] README confirms Splane as GCN module of SPACEL: "Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify uniform spatial domains that are transcriptomically and spatially coherent across multiple ST"
