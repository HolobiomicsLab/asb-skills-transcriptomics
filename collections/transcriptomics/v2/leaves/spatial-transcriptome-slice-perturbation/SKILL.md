---
name: spatial-transcriptome-slice-perturbation
description: Use when evaluating how robustly a 3D ST alignment method reconstructs
  a tissue stack from slices with simulated defects (rotations, flips, translations,
  crop ratios up to 0.25), or when reproducing the SPACEL Scube STARmap benchmark.
when_to_use_negative:
- 'Nonlinear alignment is required: the current version of Scube does not support
  nonlinear alignment (unlike STAligner and PASTE)'
- The input is a seq-based ST dataset (e.g., 10X Visium) without prior cell type deconvolution
  — Splane requires cell type composition, so a Spoint step must be added first
- The task is single-slice spatial domain identification with no inter-slice alignment
  or 3D stacking involved
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: Scube
  role: Performs the global 3D alignment of consecutive degraded slices by transforming
    spatial coordinate systems (Scube.alignment.align)
  repo: https://github.com/QuKunLab/SPACEL
- name: Splane
  role: Identifies spatial domains across degraded slices using a graph convolutional
    network with adversarial learning on cell type composition and spatial coordinates;
    domains provide the foundation for Scube alignment
  repo: https://github.com/QuKunLab/SPACEL
- name: Spoint
  role: Deconvolutes cell type composition per spot on single ST slices, required
    as input to Splane for seq-based ST datasets
  repo: https://github.com/QuKunLab/SPACEL
- name: STAligner
  role: Baseline 3D alignment method run on the same degraded slices with optimized
    parameters
- name: PASTE
  role: Baseline 3D alignment method run on the same degraded slices with optimized
    parameters
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:c6d21224d2e806f90d80752430b6cd202625452a01a7491ee519e8c36a3711c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QuKunLab/SPACEL
  tools:
  - PASTE
  - SPACEL
  - STAligner
  - Scube
  - Splane
  - Spoint
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# spatial-transcriptome-slice-perturbation

## Summary

Benchmark protocol for 3D alignment of consecutively perturbed spatial transcriptomics slices: simulate rotation, flip, translation, and edge cropping of ST slices, then compare Scube against STAligner and PASTE using SSIM/PCC against the ground-truth 3D stack.

## When to use

Use when evaluating how robustly a 3D ST alignment method reconstructs a tissue stack from slices with simulated defects (rotations, flips, translations, crop ratios up to 0.25), or when reproducing the SPACEL Scube STARmap benchmark.

## When NOT to use

- Nonlinear alignment is required: the current version of Scube does not support nonlinear alignment (unlike STAligner and PASTE)
- The input is a seq-based ST dataset (e.g., 10X Visium) without prior cell type deconvolution — Splane requires cell type composition, so a Spoint step must be added first
- The task is single-slice spatial domain identification with no inter-slice alignment or 3D stacking involved

## Inputs

- STARmap mouse brain 3D tissue stack divided into ten consecutive 10 um slices
- Cell type composition per spot/cell (from Spoint deconvolution for seq-based ST data)
- Spatial coordinates of spots/cells per slice
- Simulated defect parameters: random rotation, flip, translation, and edge crop ratios from 0 to 0.25

## Outputs

- Aligned 3D stacked tissue architecture per method (Scube, STAligner, PASTE)
- Per-slice SSIM and PCC values for each method against the ground-truth 3D stack
- alignment_metrics.csv with comparative alignment scores

## How to apply

Load the STARmap mouse brain 3D tissue stack, divide it into ten consecutive 10 um slices, and simulate defects by applying random rotation, flip, and translation to each slice plus edge cropping at crop ratios from 0 to 0.25. Identify spatial domains across the degraded slices with Splane's GCN model (Splane.base_model.SplaneModel.identify_spatial_domain), which takes cell type composition and spatial coordinates as input; for seq-based data a Spoint deconvolution step must precede Splane. Align consecutive slices with Scube.alignment.align, which globally transforms the spatial coordinate systems using correspondences between all spots in adjacent slices, and run STAligner and PASTE with optimized parameters as baselines. Compute per-slice structural similarity (SSIM) and Pearson correlation (PCC) against the ground-truth 3D stack and write results to alignment_metrics.csv; at crop ratio 0.25, expect Scube SSIM/PCC near 0.96/0.97 versus STAligner at 0.76/0.77 and PASTE near 0.72/0.75.

## Related tools

- **Scube** (Performs the global 3D alignment of consecutive degraded slices by transforming spatial coordinate systems (Scube.alignment.align)) — https://github.com/QuKunLab/SPACEL
- **Splane** (Identifies spatial domains across degraded slices using a graph convolutional network with adversarial learning on cell type composition and spatial coordinates; domains provide the foundation for Scube alignment) — https://github.com/QuKunLab/SPACEL
- **Spoint** (Deconvolutes cell type composition per spot on single ST slices, required as input to Splane for seq-based ST datasets) — https://github.com/QuKunLab/SPACEL
- **STAligner** (Baseline 3D alignment method run on the same degraded slices with optimized parameters)
- **PASTE** (Baseline 3D alignment method run on the same degraded slices with optimized parameters)

## Examples

```
from SPACEL.Scube import align; align(ad_list, use_gpu=True)  # after Splane identify_spatial_domain on STARmap slices with rotation/flip/crop ratio 0.25
```

## Evaluation signals

- At crop ratio 0.25, Scube reaches average SSIM/PCC of about 0.96/0.97 for transformed slices
- At crop ratio 0.25, STAligner scores about 0.76/0.77 and PASTE about 0.72/0.75 SSIM/PCC — Scube must outperform both baselines
- Per-slice SSIM and PCC values are written to alignment_metrics.csv with one row per slice and method
- Alignment succeeds despite crop ratios up to 0.25 (substantial slice defects/partial overlap), where PASTE is known to be hyperparameter-sensitive
- Scube's global optimization should use correspondences between all spots in adjacent slices, not just domain centroids

## Limitations

- The current version of Scube does not support nonlinear alignment, unlike STAligner and PASTE; nonlinear alignment is deferred to a future version
- PASTE's alignment results are sensitive to hyperparameters in datasets with significant defects (e.g., slices with crop ratios >= 0.25)
- STAligner does not utilize complete information on each spot of the whole slice, limiting its ability to balance global structure similarity and alignment accuracy
- When a new slice is added to the dataset, the entire deep learning model needs to be retrained from scratch, affecting computational efficiency for large-scale datasets

## Evidence

- [methods] At crop ratio 0.25 Scube achieved SSIM/PCC: [evidence span withheld — anchor status: not_found]
- [discussion] Scube global optimization: "Scube adopts a global optimization strategy for 3D alignment, taking into account the correspondences between all spots in adjacent slices"
- [full_text] Simulated defect protocol: [evidence span withheld — anchor status: not_found]
- [discussion] Splane input and domain foundation: "Spatial domains identified in Splane also set a foundation for Scube to achieve an accurate alignment of the consecutive slices and hence a precise 3D architecture construction"
- [discussion] Scube no nonlinear alignment: "the current version of Scube does not support nonlinear alignment as well as STAligner and PASTE"
- [readme] SPACEL module description from README: "Scube automatically transforms the spatial coordinate systems of consecutive slices and stacks them together to construct a three-dimensional (3D) alignment of the tissue."
