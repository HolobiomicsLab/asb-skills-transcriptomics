---
name: spatial-transcriptomics-3d-manifold-prediction
description: Use when consecutive spatial transcriptomics slices have already been
  aligned into stacked 3D coordinates (e.g., by Scube) and the goal is to model and
  visualize continuous 3D gene-expression surfaces or identify spatially varying genes
  in 3D.
when_to_use_negative:
- The dataset is a single 2D slice with no stacked 3D coordinates — spatial domain
  identification (Splane) or SVG detection in 2D is the appropriate task instead
- Seq-based ST data (e.g., 10X Visium) still needing cell type deconvolution (Spoint)
  or slice alignment (Scube.align) — the GPR module presumes Scube-aligned 3D coordinates
  as input
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: SPACEL
  role: Python toolkit whose Scube module provides the GPR machinery (Scube.gpr.GPRmodel)
    for 3D expression modeling and Bayes Factor gene selection
  repo: https://github.com/QuKunLab/SPACEL
- name: Scube
  role: Aligns consecutive slices into stacked 3D coordinates that serve as the spatial
    input for the GPR model
  repo: https://github.com/QuKunLab/SPACEL
- name: Python
  role: Language of the user-friendly GPR wrapper that integrates the original MATLAB
    code and third-party GUI software
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:5cf42ba544bf9cc1510e369c22aa46c86537ef97b6f82b9cde1e8fe6392252ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QuKunLab/SPACEL
  tools:
  - Python
  - SPACEL
  - Scube
  - Scube.gpr.GPRmodel
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spatial-transcriptomics-3d-manifold-prediction

## Summary

Fits Gaussian process regression (GPR) models of per-gene expression over a stacked 3D tissue coordinate system built by Scube (SPACEL), predicting continuous expression surfaces on a meshed manifold and scoring each gene with a Bayes Factor to flag genes whose expression varies significantly in any 3D direction.

## When to use

Use when consecutive spatial transcriptomics slices have already been aligned into stacked 3D coordinates (e.g., by Scube) and the goal is to model and visualize continuous 3D gene-expression surfaces or identify spatially varying genes in 3D.

## When NOT to use

- The dataset is a single 2D slice with no stacked 3D coordinates — spatial domain identification (Splane) or SVG detection in 2D is the appropriate task instead
- Seq-based ST data (e.g., 10X Visium) still needing cell type deconvolution (Spoint) or slice alignment (Scube.align) — the GPR module presumes Scube-aligned 3D coordinates as input

## Inputs

- Stacked 3D coordinates of spots/cells produced by Scube alignment (e.g., STARmap mouse brain dataset)
- Per-gene expression matrix for the spatial transcriptomics dataset (e.g., STARmap)
- Open3D meshed 3D manifold of the tissue surface for prediction

## Outputs

- Trained Scube.gpr.GPRmodel per gene with optimized kernel length scale
- Bayes Factor (BF) score per gene comparing fitted GPR vs. infinite-length-scale null model
- Continuous predicted 3D gene-expression surfaces rendered by GPRmodel.plot_gpr_expr
- Saved figure gpr_3d_expression.png

## How to apply

Load the aligned 3D spot/cell coordinates (e.g., STARmap mouse brain dataset aligned by Scube) and instantiate Scube.gpr.GPRmodel. Train the model per gene over the 3D coordinates with GPRmodel.train, grid-searching the kernel length scale and optimizing hyperparameters with Adam. Predict continuous expression across a 3D meshed manifold of the tissue — meshing is handled via the Open3D library, which is required for the Scube GPR module. For each gene, compute the Bayes Factor (BF) comparing the optimized-length-scale model against the infinite-length-scale (no spatial structure) null model; SPACEL uses this BF to identify genes varying significantly along any direction in 3D. Finally render the predicted continuous expression surfaces with GPRmodel.plot_gpr_expr and save as gpr_3d_expression.png; the GPR was originally MATLAB-based and third-party-GUI-based, so the user-friendly Python wrapper is the intended entry point.

## Related tools

- **SPACEL** (Python toolkit whose Scube module provides the GPR machinery (Scube.gpr.GPRmodel) for 3D expression modeling and Bayes Factor gene selection) — https://github.com/QuKunLab/SPACEL
- **Scube** (Aligns consecutive slices into stacked 3D coordinates that serve as the spatial input for the GPR model) — https://github.com/QuKunLab/SPACEL
- **Python** (Language of the user-friendly GPR wrapper that integrates the original MATLAB code and third-party GUI software)

## Examples

```
from SPACEL import Scube; gpr = Scube.gpr.GPRmodel(); gpr.train(expr, coords_3d); gpr.plot_gpr_expr(save='gpr_3d_expression.png')
```

## Evaluation signals

- gpr_3d_expression.png is produced and exists after running plot_gpr_expr
- Fitted/predicted expression surfaces from plot_gpr_expr are visually coherent with the spatially structured expression expected in the STARmap_mouse_brain_GPR tutorial
- GPRmodel.train completes with grid search over kernel length scale and Adam hyperparameter optimization converging
- Per-gene Bayes Factors are computed comparing the optimized model against the infinite-length-scale null model, and no quantitative BF threshold is imposed for the GPR module

## Limitations

- No quantitative threshold is reported for the GPR module, so validation is largely visual (coherence of predicted surfaces)
- The GPR model originally relied on MATLAB code and third-party GUI software; accessibility depends on the Python wrapper
- Scube alignment feeding the GPR supports rigid but not nonlinear alignment — nonlinear support is deferred to a future version
- Meshing for continuous prediction requires the Open3D dependency in the Scube GPR module

## Evidence

- [methods] STARmap mouse brain dataset GPR tutorial: "3D expression modeling with gaussian process regression <tutorials/STARmap_mouse_brain_GPR>"
- [discussion] To enhance the accessibility of the GPR model: "To enhance the accessibility of the GPR model for 3D spatial transcriptomics, we have developed a user-friendly Python code by integrating the original MATLAB code and third-party GUI software"
- [readme] SPACEL three modules description: "Scube automatically transforms the spatial coordinate systems of consecutive slices and stacks them together to construct a three-dimensional (3D) alignment of the tissue."
- [other] Source card finding on GPR Bayes Factor: [evidence span withheld — anchor status: not_found]
