---
name: 3d-expression-surface-visualization
description: Use when consecutive ST slices have already been 3D-aligned (e.g., by
  Scube) and the goal is to model, test, and render per-gene expression as continuous
  surfaces over the stacked 3D spot/cell coordinates (e.g., STARmap mouse brain).
when_to_use_negative:
- Input data has not been 3D-aligned — Scube's alignment (Scube.alignment.align) must
  produce stacked 3D coordinates before GPR modeling
- The goal is 2D spatially variable gene detection on a single slice rather than expression
  varying along any direction in 3D tissue space
- Seq-based ST data without a prior Spoint cell-type deconvolution step when downstream
  modules (e.g., Splane) require cell type composition
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SPACEL
  role: Toolkit whose Scube module provides the GPR machinery (Scube.gpr.GPRmodel)
    for 3D expression modeling and Bayes Factor scoring
  repo: https://github.com/QuKunLab/SPACEL
- name: Scube
  role: Aligns consecutive slices into stacked 3D coordinates and hosts the GPRmodel
    used for continuous 3D expression surface reconstruction
  repo: https://github.com/QuKunLab/SPACEL
- name: Python
  role: User-friendly wrapper language integrating the original MATLAB GPR code and
    third-party GUI software
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:8739d72f9e1b3032af1ab5755669529d10bb78717363417987016004fa80c889
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QuKunLab/SPACEL
  tools:
  - Python
  - SPACEL
  - Scube
  - Scube.gpr.GPRmodel
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# 3d-expression-surface-visualization

## Summary

Reconstructs continuous 3D gene-expression surfaces from Scube-stacked spatial transcriptomics coordinates using a Gaussian process regression (GPR) model, and scores each gene with a Bayes Factor against a no-spatial-structure null so that SPACEL can flag genes varying in any direction within the 3D tissue architecture.

## When to use

Use when consecutive ST slices have already been 3D-aligned (e.g., by Scube) and the goal is to model, test, and render per-gene expression as continuous surfaces over the stacked 3D spot/cell coordinates (e.g., STARmap mouse brain).

## When NOT to use

- Input data has not been 3D-aligned — Scube's alignment (Scube.alignment.align) must produce stacked 3D coordinates before GPR modeling
- The goal is 2D spatially variable gene detection on a single slice rather than expression varying along any direction in 3D tissue space
- Seq-based ST data without a prior Spoint cell-type deconvolution step when downstream modules (e.g., Splane) require cell type composition

## Inputs

- STARmap mouse brain dataset (per-gene expression matrix)
- Scube-aligned stacked 3D coordinates of spots/cells
- 3D meshed manifold of the tissue (generated via Open3D)

## Outputs

- Trained per-gene GPR models with optimized kernel length scale
- Per-gene Bayes Factor (BF) scores against the infinite-length-scale null model
- Predicted continuous 3D expression surfaces (gpr_3d_expression.png via GPRmodel.plot_gpr_expr)

## How to apply

Load the STARmap mouse brain dataset together with its Scube-aligned stacked 3D coordinates of spots/cells. For each gene, instantiate Scube.gpr.GPRmodel and train it (GPRmodel.train) on expression over the 3D coordinates, grid-searching the kernel length scale and optimizing hyperparameters with Adam; the GPR module requires the Open3D library for meshing the tissue manifold. Predict continuous expression across the 3D mesh, then compute the Bayes Factor comparing the optimized-length-scale model against an infinite-length-scale (no spatial structure) null model per gene — significant BF marks spatially varying genes. Render predicted continuous expression surfaces with GPRmodel.plot_gpr_expr and save as gpr_3d_expression.png; the Python wrapper exists because the GPR model originally relied on MATLAB code and third-party GUI software.

## Related tools

- **SPACEL** (Toolkit whose Scube module provides the GPR machinery (Scube.gpr.GPRmodel) for 3D expression modeling and Bayes Factor scoring) — https://github.com/QuKunLab/SPACEL
- **Scube** (Aligns consecutive slices into stacked 3D coordinates and hosts the GPRmodel used for continuous 3D expression surface reconstruction) — https://github.com/QuKunLab/SPACEL
- **Python** (User-friendly wrapper language integrating the original MATLAB GPR code and third-party GUI software)

## Examples

```
model = Scube.gpr.GPRmodel(); model.train(expr, coords_3d); bf = model.bayes_factor(); model.plot_gpr_expr(save='gpr_3d_expression.png')
```

## Evaluation signals

- gpr_3d_expression.png is generated and exists after running GPRmodel.plot_gpr_expr
- Bayes Factor is computed for each gene comparing the optimized-length-scale model against the infinite-length-scale null model
- GPR hyperparameter optimization via Adam converges over the kernel length-scale grid search without errors
- Predicted/pitted expression surfaces from plot_gpr_expr are visually coherent with the spatially structured expression expected in the STARmap_mouse_brain_GPR tutorial
- Open3D-based meshing of the tissue manifold completes before prediction (required dependency of the Scube GPR module)

## Limitations

- No quantitative threshold is reported for the GPR module, so validation relies on visual coherence of the predicted expression surfaces
- The GPR model originally relied on MATLAB code and third-party GUI software; the Python wrapper was built to enhance accessibility
- Requires 3D-aligned coordinates from Scube, whose current version does not support nonlinear alignment (deferred to a future version)

## Evidence

- [methods] GPR tutorial dataset: "3D expression modeling with gaussian process regression <tutorials/STARmap_mouse_brain_GPR>"
- [discussion] Python wrapper motivation: "To enhance the accessibility of the GPR model for 3D spatial transcriptomics, we have developed a user-friendly Python code by integrating the original MATLAB code and third-party GUI software"
- [readme] Scube 3D stacking: "Scube automatically transforms the spatial coordinate systems of consecutive slices and stacks them together to construct a three-dimensional (3D) alignment of the tissue."
