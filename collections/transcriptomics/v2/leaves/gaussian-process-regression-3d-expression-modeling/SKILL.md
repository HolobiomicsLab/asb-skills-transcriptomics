---
name: gaussian-process-regression-3d-expression-modeling
description: Use when consecutive ST slices have been 3D-aligned by Scube (e.g., the
  STARmap mouse brain dataset) and you need to model continuous gene-expression surfaces
  in 3D and identify spatially varying genes via Bayes Factor scoring.
when_to_use_negative:
- The slices have not yet been aligned into a stacked 3D architecture by Scube (GPR
  requires the aligned 3D coordinates)
- The goal is identifying spatial domains across slices — that is Splane's task, not
  3D expression modeling
- The dataset is seq-based ST (e.g., 10X Visium) requiring cell-type deconvolution
  as the primary goal — use Spoint first
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: SPACEL
  role: Python toolkit whose Scube module provides the GPR-based 3D expression modeling;
    PyPI package SPACEL
  repo: https://github.com/QuKunLab/SPACEL
- name: Scube
  role: Module that constructs the stacked 3D architecture and provides Scube.gpr.GPRmodel
    for GPR training, prediction, Bayes Factor scoring, and plot_gpr_expr rendering
  repo: https://github.com/QuKunLab/SPACEL
- name: Python
  role: User-friendly Python code wrapping the original MATLAB GPR implementation
    and third-party GUI software
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:f1eeda1dcb179b3f49b3fa1d5c79cac2e22d2259b384d33b1f91382c1fe89009
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QuKunLab/SPACEL
  tools:
  - Python
  - SPACEL
  - Scube
  - Scube.gpr.GPRmodel
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gaussian-process-regression-3d-expression-modeling

## Summary

Fit a Gaussian process regression (GPR) model in SPACEL's Scube module to per-gene expression over stacked 3D tissue coordinates, producing continuous 3D expression surfaces and a Bayes Factor that flags genes varying significantly along any spatial direction.

## When to use

Use when consecutive ST slices have been 3D-aligned by Scube (e.g., the STARmap mouse brain dataset) and you need to model continuous gene-expression surfaces in 3D and identify spatially varying genes via Bayes Factor scoring.

## When NOT to use

- The slices have not yet been aligned into a stacked 3D architecture by Scube (GPR requires the aligned 3D coordinates)
- The goal is identifying spatial domains across slices — that is Splane's task, not 3D expression modeling
- The dataset is seq-based ST (e.g., 10X Visium) requiring cell-type deconvolution as the primary goal — use Spoint first

## Inputs

- Scube-aligned stacked 3D coordinates of spots/cells from consecutive ST slices (e.g., STARmap mouse brain dataset)
- Per-gene expression matrix for the spatially resolved spots/cells
- 3D meshed manifold of the tissue (built with the Open3D library)

## Outputs

- Fitted per-gene GPR models with optimized kernel length scale
- Bayes Factor per gene comparing fitted model to the infinite-length-scale null model
- Predicted continuous 3D expression surfaces (gpr_3d_expression.png via GPRmodel.plot_gpr_expr)

## How to apply

After Scube has aligned and stacked consecutive slices into 3D coordinates of spots/cells (e.g., the STARmap mouse brain dataset from the STARmap_mouse_brain_GPR tutorial), instantiate Scube.gpr.GPRmodel and train it per gene over the 3D coordinates using GPRmodel.train, grid-searching the kernel length scale and optimizing hyperparameters with Adam. Predict continuous expression across a 3D meshed manifold of the tissue — meshing is handled via the Open3D library, which is required for the Scube GPR module. For each gene, compute the Bayes Factor comparing the optimized-length-scale model against the infinite-length-scale (no spatial structure) null model; a large BF indicates expression varying significantly in some direction within the 3D space. Render the predicted continuous expression surfaces with GPRmodel.plot_gpr_expr and save them as gpr_3d_expression.png. The GPR model was originally MATLAB-based; SPACEL provides a user-friendly Python wrapper integrating the original MATLAB code and third-party GUI software to enhance accessibility.

## Related tools

- **SPACEL** (Python toolkit whose Scube module provides the GPR-based 3D expression modeling; PyPI package SPACEL) — https://github.com/QuKunLab/SPACEL
- **Scube** (Module that constructs the stacked 3D architecture and provides Scube.gpr.GPRmodel for GPR training, prediction, Bayes Factor scoring, and plot_gpr_expr rendering) — https://github.com/QuKunLab/SPACEL
- **Python** (User-friendly Python code wrapping the original MATLAB GPR implementation and third-party GUI software)

## Examples

```
from SPACEL import Scube; gpr = Scube.gpr.GPRmodel(); gpr.train(adata_3d, grid_length_scale=True); gpr.plot_gpr_expr(save='gpr_3d_expression.png')
```

## Evaluation signals

- gpr_3d_expression.png exists after running GPRmodel.plot_gpr_expr
- Predicted expression surfaces are visually coherent with spatially structured expression expected in the STARmap_mouse_brain_GPR tutorial demonstration
- For each gene, the Bayes Factor of the optimized-length-scale model versus the infinite-length-scale (no spatial structure) null model is computable and finite
- GPRmodel.train successfully grid-searches the kernel length scale and converges under Adam optimization
- Predictions are produced over the full Open3D-meshed 3D manifold of the tissue, not just the original spot coordinates

## Limitations

- No quantitative threshold is reported for the GPR module — validation relies on visual coherence of the rendered surfaces
- The GPR model originally relied on MATLAB code and third-party GUI software, motivating the Python wrapper; users depend on the Open3D library for meshing
- The current version of Scube does not support nonlinear alignment (unlike STAligner and PASTE), which bounds the accuracy of the 3D coordinates feeding the GPR model

## Evidence

- [methods] STARmap dataset used for 3D expression modeling: "3D expression modeling with gaussian process regression <tutorials/STARmap_mouse_brain_GPR>"
- [discussion] GPR model wrapped in user-friendly Python code: "To enhance the accessibility of the GPR model for 3D spatial transcriptomics, we have developed a user-friendly Python code by integrating the original MATLAB code and third-party GUI software"
- [discussion] Scube limitation: no nonlinear alignment: "the current version of Scube does not support nonlinear alignment as well as STAligner and PASTE"
- [readme] SPACEL Python package description: "SPACEL (**SP**atial **A**rchitecture **C**haracterization by d**E**ep **L**earning) is a Python package of deep-learning-based methods for ST data analysis."
- [abstract] Scube constructs 3D tissue architecture: "Scube automatically transforms the spatial coordinate systems of consecutive slices and stacks them together to construct a 3D architecture of the tissue"
