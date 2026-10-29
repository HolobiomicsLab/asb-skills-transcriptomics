---
name: semi-profiled-data-fidelity-evaluation
description: Use when inferred or semi-profiled single-cell profiles (e.g., inferred_cells.h5ad
  for target samples like AP1) must be validated against real-profiled ground-truth
  cells or a representative-vs-target baseline before downstream use.
when_to_use_negative:
- The analysis goal is generating or improving inference itself (active learning or
  fine-tuning), not verifying fidelity of already-produced semi-profiled data
- No real-profiled ground-truth single-cell data exist for the target sample and no
  representative-vs-target baseline can be computed
- Data are from a new modality (e.g., proteomics) not yet implemented in scSemiProfiler,
  so no validated fidelity benchmarks exist
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSemiProfiler
  role: Provides the inferred (semi-profiled) single-cell profiles and cohort-level
    outputs that are compared against real-profiled ground truth
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: PyTorch
  role: Framework in which the VAE-GAN generator checkpoint used to produce inferred_cells.h5ad
    is trained and applied
- name: Scikit-learn
  role: Supporting package (e.g., PCA) in the pipeline whose end-to-end run is validated
    during evaluation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:dfe889d6bf85c791fcc2f94fc25dc084bd04191fa1b9009851d42d037da1cb69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - PyTorch
  - Scikit-learn
  - scSemiProfiler
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# semi-profiled-data-fidelity-evaluation

## Summary

Evaluates the fidelity of scSemiProfiler's semi-profiled (AI-inferred) single-cell data against real-profiled ground truth, using Pearson correlation of cell expression profiles, UMAP visualization overlap, and cell type proportion agreement.

## When to use

Use when inferred or semi-profiled single-cell profiles (e.g., inferred_cells.h5ad for target samples like AP1) must be validated against real-profiled ground-truth cells or a representative-vs-target baseline before downstream use.

## When NOT to use

- The analysis goal is generating or improving inference itself (active learning or fine-tuning), not verifying fidelity of already-produced semi-profiled data
- No real-profiled ground-truth single-cell data exist for the target sample and no representative-vs-target baseline can be computed
- Data are from a new modality (e.g., proteomics) not yet implemented in scSemiProfiler, so no validated fidelity benchmarks exist

## Inputs

- Inferred single-cell profiles (inferred_cells.h5ad) produced by the fine-tuned VAE-GAN generator
- Real-profiled ground-truth single-cell data for the target sample (e.g., AP1)
- Representative sample single-cell data (e.g., AP6) for the baseline comparison
- Processed cohort single-cell data with cell type annotations for proportion and UMAP comparison

## Outputs

- Pearson correlation between inferred and true target cells (expected ~0.904 vs 0.568 baseline)
- UMAP visualizations overlaying semi-profiled vs real-profiled cells
- Stacked bar plots of cell type proportions across disease conditions
- End-to-end pipeline validation confirming the script runs over the training/inference path

## How to apply

Run the training/inference path end-to-end to obtain inferred single-cell profiles (e.g., saved to inferred_cells.h5ad for target sample AP1), then quantify fidelity by computing the Pearson correlation between inferred and true target cells — scSemiProfiler reported 0.904, which must exceed the representative-vs-target baseline of 0.568. Complement the correlation check with qualitative and compositional comparisons: UMAP visualizations to confirm that inferred cells (generated from reference cells) resemble the real ground truth, and stacked bar plots to verify that semi-profiled cohorts (e.g., the 124-sample COVID-19 cohort) reproduce cell type proportions across disease conditions. This evaluates the key claim that the VAE-GAN fine-tuning with the target bulk loss (LBulkT with conversion ratio alpha = (BT+epsilon)/(BR+epsilon), epsilon = 0.1) yields cell-level fidelity beyond simple transfer of the representative's profiles.

## Related tools

- **scSemiProfiler** (Provides the inferred (semi-profiled) single-cell profiles and cohort-level outputs that are compared against real-profiled ground truth) — https://github.com/mcgilldinglab/scSemiProfiler
- **PyTorch** (Framework in which the VAE-GAN generator checkpoint used to produce inferred_cells.h5ad is trained and applied)
- **Scikit-learn** (Supporting package (e.g., PCA) in the pipeline whose end-to-end run is validated during evaluation)

## Examples

```
pip install --upgrade https://github.com/mcgilldinglab/scSemiProfiler/zipball/main; python -c "import scSemiProfiler; print(scSemiProfiler.utils.inspect_data.__doc__)"
```

## Evaluation signals

- Pearson correlation between inferred and true AP1 cells reaches the reported 0.904 and exceeds the representative-vs-target baseline of 0.568
- UMAP visualization of inferred cells (red) closely overlaps the real-profiled ground truth of the target sample
- Stacked bar plots show semi-profiled and real-profiled cohorts have nearly identical cell type proportions across disease conditions
- The validation script runs end-to-end over the training/inference path and writes inferred_cells.h5ad plus the generator checkpoint file
- Semi-profiled 124-sample COVID-19 cohort reproduces the cell distribution and cell types of the real-profiled cohort

## Limitations

- Fidelity checks rely on having real-profiled ground-truth data for the target sample, which is normally unavailable in pure semi-profiling use cases
- Evaluation was performed on specific cohorts (e.g., COVID-19, colorectal cancer, iMGL, hamster); performance on new modalities such as proteomics requires comprehensive benchmarking against established standards
- The tool's deep generative model enables single-cell level deconvolution only provided there is a single-cell reference

## Evidence

- [methods] Pearson correlation of inferred vs true cells: [evidence span withheld — anchor status: not_found]
- [discussion] Resemblance of semi-profiled and real data: "The striking resemblance between semi-profiled and real-profiled data in facets like UMAP visualization and cell type proportions ascertains the robustness of scSemiProfiler."
- [readme] README overview of fidelity claim: [evidence span withheld — anchor status: external_doc]
- [discussion] Reference requirement caveat: "The tool’s deep generative model enables “single-cell level deconvolution” across various studies, provided there is a single-cell reference."
