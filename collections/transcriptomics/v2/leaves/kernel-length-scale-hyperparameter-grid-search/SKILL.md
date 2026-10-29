---
name: kernel-length-scale-hyperparameter-grid-search
description: Use when 3D-aligned spot/cell coordinates and per-gene expression from
  consecutive ST slices (e.g., STARmap mouse brain) are available and genes whose
  expression varies significantly along any direction in 3D tissue must be identified.
when_to_use_negative:
- The data are a single 2D ST slice without stacked 3D coordinates — Spoint/Splane
  tasks, not Scube's 3D GPR module, apply
- The dataset is seq-based (e.g., 10X Visium) and has not yet been processed by Spoint
  deconvolution and Splane domain identification, which Scube's alignment depends
  on
- The goal is nonlinear slice alignment rather than 3D expression modeling — the current
  version of Scube does not support nonlinear alignment
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SPACEL
  role: Toolkit whose Scube module provides the GPR mechanism used to fit 3D expression
    and compute Bayes Factors
  repo: github.com/QuKunLab/SPACEL
- name: Scube
  role: Provides the stacked 3D alignment of consecutive slices and the Scube.gpr.GPRmodel
    trained via GPRmodel.train with Adam and length-scale grid search
  repo: github.com/QuKunLab/SPACEL
- name: Python
  role: User-friendly Python wrapper integrating the original MATLAB GPR code and
    third-party GUI software
  repo: github.com/QuKunLab/SPACEL
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:274e8e1ad32dd23184924dfda1ab5c2570d06b57d1cc2e6512582987dc74a23e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SPACEL
  - Scube
  - Scube.gpr.GPRmodel
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# kernel-length-scale-hyperparameter-grid-search

## Summary

Grid-searches the Gaussian process kernel length scale when fitting Scube's GPRmodel to per-gene expression over stacked 3D spatial-transcriptomics coordinates, then scores each gene by the Bayes Factor of the optimized model versus an infinite-length-scale (no spatial structure) null.

## When to use

Use when 3D-aligned spot/cell coordinates and per-gene expression from consecutive ST slices (e.g., STARmap mouse brain) are available and genes whose expression varies significantly along any direction in 3D tissue must be identified.

## When NOT to use

- The data are a single 2D ST slice without stacked 3D coordinates — Spoint/Splane tasks, not Scube's 3D GPR module, apply
- The dataset is seq-based (e.g., 10X Visium) and has not yet been processed by Spoint deconvolution and Splane domain identification, which Scube's alignment depends on
- The goal is nonlinear slice alignment rather than 3D expression modeling — the current version of Scube does not support nonlinear alignment

## Inputs

- Scube-aligned stacked 3D coordinates of spots/cells from consecutive ST slices
- Per-gene expression matrix for the STARmap mouse brain dataset (tutorials/STARmap_mouse_brain_GPR)
- 3D meshed manifold of the tissue generated with Open3D

## Outputs

- Trained Scube GPRmodel per gene with optimized kernel length scale
- Bayes Factor (BF) per gene quantifying spatially structured expression versus the infinite-length-scale null
- Continuous predicted 3D expression surfaces rendered as gpr_3d_expression.png via GPRmodel.plot_gpr_expr

## How to apply

Instantiate Scube.gpr.GPRmodel and train it (GPRmodel.train) on per-gene expression over the Scube-stacked 3D coordinates of spots/cells, grid-searching the kernel length scale while optimizing hyperparameters with Adam. Predict continuous expression across a 3D meshed manifold of the tissue, with meshing handled via the Open3D library that the Scube GPR module requires. For each gene, compute the Bayes Factor (BF) comparing the fitted model at the optimized length scale against the infinite-length-scale null model that assumes no spatial structure; genes with strong BF evidence are those varying significantly in any direction in 3D space. Render the predicted continuous expression surfaces with GPRmodel.plot_gpr_expr and save as gpr_3d_expression.png. The rationale is that the length scale controls the spatial smoothness assumption of the GP, so optimizing it per gene determines whether expression variation is spatially structured rather than noise.

## Related tools

- **SPACEL** (Toolkit whose Scube module provides the GPR mechanism used to fit 3D expression and compute Bayes Factors) — github.com/QuKunLab/SPACEL
- **Scube** (Provides the stacked 3D alignment of consecutive slices and the Scube.gpr.GPRmodel trained via GPRmodel.train with Adam and length-scale grid search) — github.com/QuKunLab/SPACEL
- **Python** (User-friendly Python wrapper integrating the original MATLAB GPR code and third-party GUI software) — github.com/QuKunLab/SPACEL

## Examples

```
from Scube.gpr import GPRmodel; model = GPRmodel(); model.train(coords, expr); model.plot_gpr_expr()  # save gpr_3d_expression.png
```

## Evaluation signals

- gpr_3d_expression.png exists after running GPRmodel.plot_gpr_expr
- The fitted/predicted expression surfaces are visually coherent with the spatially structured expression expected in the STARmap_mouse_brain_GPR tutorial demonstration
- The Bayes Factor for each gene is computed against the infinite-length-scale (no spatial structure) null model, not merely against an alternative kernel fit
- The kernel length scale was selected by grid search with Adam-based hyperparameter optimization rather than fixed ad hoc
- Open3D is installed and the 3D meshed manifold of the tissue was successfully generated before GPR prediction

## Limitations

- No quantitative threshold is reported for the GPR module; validation relies on the existence of gpr_3d_expression.png and visual coherence of the predicted surfaces
- The GPR model originally relied on MATLAB code and third-party GUI software, which is why a Python wrapper was developed; the Open3D library is required for the Scube GPR module's meshing
- This GPR module requires a stacked 3D architecture from Scube and does not address Scube's lack of nonlinear alignment support in the current version

## Evidence

- [discussion] To enhance the accessibility of the GPR model for 3D spatial transcriptomics, we have developed a user-friendly Python code by integrating the original MATLAB code and third-party GUI software: "To enhance the accessibility of the GPR model for 3D spatial transcriptomics, we have developed a user-friendly Python code by integrating the original MATLAB code and third-party GUI software"
- [methods] 3D expression modeling with gaussian process regression <tutorials/STARmap_mouse_brain_GPR>: "3D expression modeling with gaussian process regression <tutorials/STARmap_mouse_brain_GPR>"
- [readme] Scube automatically transforms the spatial coordinate systems of consecutive slices and stacks them together to construct a 3D architecture of the tissue: "Scube automatically transforms the spatial coordinate systems of consecutive slices and stacks them together to construct a three-dimensional (3D) alignment of the tissue."
