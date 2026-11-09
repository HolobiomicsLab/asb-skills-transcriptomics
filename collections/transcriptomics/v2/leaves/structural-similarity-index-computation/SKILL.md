---
name: structural-similarity-index-computation
description: 'Use when evaluating 2D ST slice alignment quality: you have spatial
  coordinate systems of consecutive slices (e.g., STARmap mouse brain) transformed
  by a tool like Scube and need a per-slice similarity score against a ground-truth
  3D stack.'
when_to_use_negative:
- Evaluating nonlinear alignment quality — the article states the current version
  of Scube does not support nonlinear alignment, so SSIM-based comparisons of nonlinear
  transforms are out of scope
- Assessing cell type deconvolution or spatial domain identification quality (Spoint/Splane
  tasks), where SSIM is not the relevant metric
- Input slices have no ground-truth 3D stack reference available, making per-slice
  SSIM/PCC against truth impossible
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Scube
  role: Produces the globally aligned consecutive slices whose transformed coordinate
    systems are scored by SSIM/PCC against the ground-truth 3D stack
  repo: https://github.com/QuKunLab/SPACEL
- name: SPACEL
  role: Toolkit containing Splane (spatial domain identification feeding alignment)
    and Scube (3D stacking) used to generate the slices evaluated by SSIM
  repo: https://github.com/QuKunLab/SPACEL
- name: STAligner
  role: Baseline alignment method whose per-slice SSIM/PCC (~0.76/0.77) is computed
    for comparison with Scube
- name: PASTE
  role: Baseline alignment method whose per-slice SSIM/PCC (~0.72/0.75) is computed
    for comparison with Scube
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:99d371f209d2c4f9c25f7777dcf2777f12f3556592e0defb670539f36439d23f
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# structural-similarity-index-computation

## Summary

Computes structural similarity index (SSIM, alongside Pearson correlation) per spatial-transcriptomics slice to quantify how faithfully an aligned/transformed slice reproduces the ground-truth 3D tissue stack. In the SPACEL benchmark it is the primary metric scoring Scube's 3D alignment against STAligner and PASTE.

## When to use

Use when evaluating 2D ST slice alignment quality: you have spatial coordinate systems of consecutive slices (e.g., STARmap mouse brain) transformed by a tool like Scube and need a per-slice similarity score against a ground-truth 3D stack.

## When NOT to use

- Evaluating nonlinear alignment quality — the article states the current version of Scube does not support nonlinear alignment, so SSIM-based comparisons of nonlinear transforms are out of scope
- Assessing cell type deconvolution or spatial domain identification quality (Spoint/Splane tasks), where SSIM is not the relevant metric
- Input slices have no ground-truth 3D stack reference available, making per-slice SSIM/PCC against truth impossible

## Inputs

- Aligned/transformed 2D ST slices (e.g., STARmap mouse brain slices after Scube.alignment.align global coordinate transformation)
- Ground-truth 3D tissue stack (ten consecutive 10 um slices of the STARmap mouse brain dataset)
- Aligned outputs from baseline methods (STAligner, PASTE) on the same degraded slices

## Outputs

- Per-slice SSIM and PCC values for each alignment method
- alignment_metrics.csv containing the per-slice SSIM/PCC results

## How to apply

After aligning consecutive perturbed slices (rotation, flip, translation, and edge cropping at ratios from 0 to 0.25) with Scube.alignment.align, rasterize each transformed slice and the corresponding ground-truth slice of the 3D tissue stack into comparable images. Compute SSIM and Pearson correlation (PCC) per slice for each method (Scube, STAligner, PASTE) and write the per-slice results to alignment_metrics.csv. Judge quality by comparing against the reported reference values: Scube should reach SSIM/PCC of ~0.96/0.97 at crop ratio 0.25, versus 0.76/0.77 for STAligner and 0.72/0.75 for PASTE. Higher SSIM indicates better preservation of global structure similarity, which the paper attributes to Scube's global optimization over correspondences between all spots in adjacent slices rather than pairwise slice alignment.

## Related tools

- **Scube** (Produces the globally aligned consecutive slices whose transformed coordinate systems are scored by SSIM/PCC against the ground-truth 3D stack) — https://github.com/QuKunLab/SPACEL
- **SPACEL** (Toolkit containing Splane (spatial domain identification feeding alignment) and Scube (3D stacking) used to generate the slices evaluated by SSIM) — https://github.com/QuKunLab/SPACEL
- **STAligner** (Baseline alignment method whose per-slice SSIM/PCC (~0.76/0.77) is computed for comparison with Scube)
- **PASTE** (Baseline alignment method whose per-slice SSIM/PCC (~0.72/0.75) is computed for comparison with Scube)

## Examples

```
from skimage.metrics import structural_similarity as ssim; from scipy.stats import pearsonr; s, _ = ssim(gt_slice, aligned_slice, data_range=1.0); r, _ = pearsonr(gt_slice.ravel(), aligned_slice.ravel())
```

## Evaluation signals

- At crop ratio 0.25 on the simulated STARmap mouse brain data, Scube-aligned slices should achieve SSIM/PCC ≈ 0.96/0.97
- STAligner baseline should fall near SSIM/PCC 0.76/0.77 and PASTE near 0.72/0.75 under identical degraded slices
- Every per-slice SSIM and PCC value is finite and in [0, 1], with metrics computed against the ground-truth 3D stack
- Results file alignment_metrics.csv contains one row per slice per method with SSIM and PCC columns
- Scube SSIM/PCC should remain high as crop ratio increases from 0 to 0.25, while baseline scores degrade

## Limitations

- PASTE's alignment results are sensitive to hyperparameters in datasets with significant defects (e.g., crop ratios >= 0.25), so baseline metric comparisons can shift with parameter tuning
- Scube performs only global (affine) transformation; slices requiring nonlinear alignment are not assessed in this benchmark
- SSIM/PCC require a ground-truth 3D stack, limiting the metric to simulated or manually curated slice-defect benchmarks

## Evidence

- [other] Scube achieved average SSIM/PCC of 0.96/0.97 for transformed slices: [evidence span withheld — anchor status: not_found]
- [abstract] Scube transforms spatial coordinate systems and stacks slices into a 3D architecture: "Scube automatically transforms the spatial coordinate systems of consecutive slices and stacks them together to construct a 3D architecture of the tissue"
- [discussion] Scube uses a global optimization strategy accounting for all spots in adjacent slices: "Scube adopts a global optimization strategy for 3D alignment, taking into account the correspondences between all spots in adjacent slices"
