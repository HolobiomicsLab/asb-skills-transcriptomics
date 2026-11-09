---
name: bayesian-model-comparison-bayes-factor
description: Use when a 3D-aligned spatial transcriptomics dataset (e.g., STARmap
  mouse brain slices stacked by Scube) is available and you need to flag spatially
  variable genes by testing directional expression variation in 3D.
when_to_use_negative:
- The data are single 2D slices not yet stacked by Scube — no 3D coordinates exist
  to regress over
- The goal is 2D spatially variable gene detection on one slice (use spatial-domain
  tools like Splane/SpaGCN instead of 3D GPR)
- Input is seq-based ST (e.g., 10X Visium) that first requires Spoint cell type deconvolution
  and slice alignment before 3D modeling is meaningful
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SPACEL
  role: Toolkit providing the GPR-based 3D expression modeling workflow
  repo: https://github.com/QuKunLab/SPACEL
- name: Scube
  role: Module whose gpr.GPRmodel fits Gaussian process regression over stacked 3D
    coordinates and computes Bayes Factors
  repo: https://github.com/QuKunLab/SPACEL
- name: Scube.gpr.GPRmodel
  role: Trains the GPR model (grid-searched kernel length scale, Adam optimizer),
    predicts on the Open3D meshed manifold, computes BF, and plots expression surfaces
- name: Python
  role: Implementation language of the user-friendly wrapper integrating the original
    MATLAB GPR code
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:786bf492b0c4e7d943dddf7a55b4058bc7f673c6587bfcb9d02fd0800612da90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QuKunLab/SPACEL
  tools:
  - Python
  - SPACEL
  - Scube
  - Scube.gpr.GPRmodel
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# bayesian-model-comparison-bayes-factor

## Summary

Fit Gaussian process regression (GPR) models to per-gene expression over Scube's stacked 3D coordinates and score them with a Bayes Factor comparing the optimized kernel length scale against an infinite-length-scale null model, identifying genes whose expression varies significantly along any direction in the 3D tissue architecture.

## When to use

Use when a 3D-aligned spatial transcriptomics dataset (e.g., STARmap mouse brain slices stacked by Scube) is available and you need to flag spatially variable genes by testing directional expression variation in 3D.

## When NOT to use

- The data are single 2D slices not yet stacked by Scube — no 3D coordinates exist to regress over
- The goal is 2D spatially variable gene detection on one slice (use spatial-domain tools like Splane/SpaGCN instead of 3D GPR)
- Input is seq-based ST (e.g., 10X Visium) that first requires Spoint cell type deconvolution and slice alignment before 3D modeling is meaningful

## Inputs

- STARmap mouse brain spatial transcriptomics data with Scube-aligned stacked 3D coordinates of spots/cells
- Per-gene expression matrix over the 3D coordinates
- 3D meshed tissue manifold (generated via the Open3D library, a dependency of the Scube GPR module)

## Outputs

- Per-gene Bayes Factor (BF) scores comparing fitted vs. infinite-length-scale GPR models
- Predicted continuous 3D gene-expression surfaces rendered by GPRmodel.plot_gpr_expr and saved as gpr_3d_expression.png

## How to apply

Load the STARmap mouse brain dataset with Scube-aligned stacked 3D coordinates of spots/cells, then instantiate Scube.gpr.GPRmodel and call GPRmodel.train to fit per-gene expression over the 3D coordinates, grid-searching the kernel length scale and optimizing hyperparameters with Adam. Open3D is required by the Scube GPR module to mesh the tissue into a 3D manifold on which the trained model predicts continuous expression. For each gene, compute the Bayes Factor comparing the fitted-length-scale GPR model against the infinite-length-scale (no spatial structure) null model; a high BF indicates expression varying significantly along some direction in 3D. Finally render the predicted continuous expression surfaces with GPRmodel.plot_gpr_expr and save as gpr_3d_expression.png. The rationale is that a GPR model with a finite kernel length scale captures spatial autocorrelation, and the BF quantifies the evidence for that spatial structure over the null.

## Related tools

- **SPACEL** (Toolkit providing the GPR-based 3D expression modeling workflow) — https://github.com/QuKunLab/SPACEL
- **Scube** (Module whose gpr.GPRmodel fits Gaussian process regression over stacked 3D coordinates and computes Bayes Factors) — https://github.com/QuKunLab/SPACEL
- **Scube.gpr.GPRmodel** (Trains the GPR model (grid-searched kernel length scale, Adam optimizer), predicts on the Open3D meshed manifold, computes BF, and plots expression surfaces)
- **Python** (Implementation language of the user-friendly wrapper integrating the original MATLAB GPR code)

## Examples

```
from SPACEL import Scube; gpr = Scube.gpr.GPRmodel(); gpr.train(expr, coords_3d); gpr.plot_gpr_expr()
```

## Evaluation signals

- gpr_3d_expression.png exists after running plot_gpr_expr
- Predicted expression surfaces are visually coherent with spatially structured expression expected in the STARmap_mouse_brain_GPR tutorial (no quantitative BF threshold is reported)
- Each gene has a BF comparing the optimized-length-scale model to the infinite-length-scale null model
- GPRmodel.train completed grid search over kernel length scales with Adam hyperparameter optimization
- The 3D prediction mesh was successfully generated with Open3D (a required dependency)

## Limitations

- No quantitative threshold is reported for the GPR module's Bayes Factor, so validation is primarily visual/qualitative
- The GPR model originally relied on MATLAB code and third-party GUI software, which the Python wrapper integrates; accessibility depends on this wrapper
- The GPR module operates on the 3D architecture built by Scube, which does not support nonlinear alignment

## Evidence

- [methods] STARmap GPR tutorial in article methods: "3D expression modeling with gaussian process regression <tutorials/STARmap_mouse_brain_GPR>"
- [abstract] Scube 3D stacking from abstract: "Scube automatically transforms the spatial coordinate systems of consecutive slices and stacks them together to construct a 3D architecture of the tissue"
- [discussion] Python GPR wrapper from discussion: "we have developed a user-friendly Python code by integrating the original MATLAB code and third-party GUI software"
- [readme] SPACEL package description from README: "is a Python package of deep-learning-based methods for ST data analysis"
