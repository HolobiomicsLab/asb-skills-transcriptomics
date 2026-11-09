---
name: deconvolution-method-benchmarking
description: Use when pseudobulk bulk RNA-seq samples have ground-truth cell type
  proportions (e.g., from matched single-cell data) and multiple deconvolution methods
  must be compared quantitatively on CCC, Pearson, and RMSE metrics.
when_to_use_negative:
- No ground-truth cell type proportions are available, so accuracy cannot be scored
  against a reference
- The goal is single-cell-level decomposition rather than comparing cell-type proportion
  estimates across methods
- Input data lacks the matched bulk and single-cell references that scSemiProfiler
  requires for its deconvolution pipeline
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: scSemiProfiler
  role: Single-cell level deconvolution of bulk RNA-seq using a single-cell reference;
    inferred cells annotated via MLP classifier to obtain proportions
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: CIBERSORTx
  role: Competing deconvolution method run via its official web portal
- name: Bisque
  role: Competing deconvolution method installed from its official GitHub repository
- name: TAPE
  role: Competing deep neural network deconvolution method installed from its official
    GitHub repository; its PyTorch Scaden variant is also used
- name: MuSiC
  role: Competing deconvolution method; also source of the NNLS implementation
- name: EPIC
  role: Competing deconvolution method requiring a pre-defined signature gene matrix,
    installed from its official GitHub repository
- name: Scaden
  role: Competing deep learning deconvolution method run via the PyTorch version in
    TAPE's GitHub repository (trains on 5000 sampled cells)
- name: NNLS
  role: Baseline deconvolution from the MuSiC package
- name: SciPy
  role: Pearson correlation computation and one-sided Wilcoxon significance testing
- name: Scikit-learn
  role: MLP classifier for annotating generated cells before computing proportions;
    also PCA elsewhere in the pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:08c3ff7d2b10d4d6495d9c8488dc0e20894edc48d681f8fec73bda8e9f64ab25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - Bisque
  - CIBERSORTx
  - EPIC
  - MuSiC
  - NNLS
  - Scaden
  - SciPy
  - Scikit-learn
  - TAPE
  - scSemiProfiler
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# deconvolution-method-benchmarking

## Summary

Benchmark cell-type deconvolution methods (scSemiProfiler, CIBERSORTx, Bisque, Scaden, TAPE, EPIC, NNLS, MuSiC) by comparing estimated cell type proportions against ground truth using Lin's CCC, Pearson correlation, and RMSE, with one-sided Wilcoxon tests for pairwise significance.

## When to use

Use when pseudobulk bulk RNA-seq samples have ground-truth cell type proportions (e.g., from matched single-cell data) and multiple deconvolution methods must be compared quantitatively on CCC, Pearson, and RMSE metrics.

## When NOT to use

- No ground-truth cell type proportions are available, so accuracy cannot be scored against a reference
- The goal is single-cell-level decomposition rather than comparing cell-type proportion estimates across methods
- Input data lacks the matched bulk and single-cell references that scSemiProfiler requires for its deconvolution pipeline

## Inputs

- Pseudobulk bulk RNA-seq profiles for the cohort (124 COVID-19 samples)
- Ground-truth cell type proportions per sample from matched single-cell profiling
- Single-cell reference data (annotated representative samples; 28-representative and 4-representative variants)

## Outputs

- deconvolution_benchmark.csv with one row per method-condition (9 rows) and CCC, Pearson, RMSE, and Wilcoxon p-value columns
- Per-sample estimated cell type proportions for each method

## How to apply

Run each method on the same pseudobulk cohort (here the 124-sample COVID-19 cohort) against a single-cell reference. For scSemiProfiler, generate single-cell data from bulk profiles using a single-cell reference, then annotate generated cells with a Scikit-learn MLP classifier trained on the reference's annotated data to compute cell type proportions; include both a full-reference variant (28 representatives) and a fair-comparison variant confined to 4 representatives. Run competitors per their official sources: CIBERSORTx via its web portal, Bisque/TAPE/MuSiC/EPIC from their GitHub repositories, NNLS from the MuSiC package, and Scaden via the PyTorch version in TAPE's repository (which trains on only 5000 cells). For every method, compute Lin's CCC, Pearson correlation (SciPy), and RMSE of estimated versus ground-truth proportions across pseudobulk samples, then assess pairwise significance with a one-sided Wilcoxon test (SciPy). Record one row per method-condition in a benchmark CSV (9 rows: scSemiProfiler-28, scSemiProfiler-4, plus 7 competing methods).

## Related tools

- **scSemiProfiler** (Single-cell level deconvolution of bulk RNA-seq using a single-cell reference; inferred cells annotated via MLP classifier to obtain proportions) — https://github.com/mcgilldinglab/scSemiProfiler
- **CIBERSORTx** (Competing deconvolution method run via its official web portal)
- **Bisque** (Competing deconvolution method installed from its official GitHub repository)
- **TAPE** (Competing deep neural network deconvolution method installed from its official GitHub repository; its PyTorch Scaden variant is also used)
- **MuSiC** (Competing deconvolution method; also source of the NNLS implementation)
- **EPIC** (Competing deconvolution method requiring a pre-defined signature gene matrix, installed from its official GitHub repository)
- **Scaden** (Competing deep learning deconvolution method run via the PyTorch version in TAPE's GitHub repository (trains on 5000 sampled cells))
- **NNLS** (Baseline deconvolution from the MuSiC package)
- **SciPy** (Pearson correlation computation and one-sided Wilcoxon significance testing)
- **Scikit-learn** (MLP classifier for annotating generated cells before computing proportions; also PCA elsewhere in the pipeline)

## Examples

```
conda create -n semiprofiler python=3.9 && conda activate semiprofiler && pip install --upgrade https://github.com/mcgilldinglab/scSemiProfiler/zipball/main
```

## Evaluation signals

- deconvolution_benchmark.csv exists and contains exactly 9 method rows (scSemiProfiler-28, scSemiProfiler-4, and 7 competing methods)
- Metric columns (CCC, Pearson, RMSE, Wilcoxon p-value) match the article's Source Data for Fig. 2j-l (doi 10.1038/s41467-024-50150-1)
- scSemiProfiler outperforms all benchmarked methods on CCC, Pearson, and RMSE in both the 4-representative and 28-representative conditions
- Pearson correlation and Wilcoxon p-values are computed with SciPy and every pairwise comparison is one-sided
- Proportions derive from annotated single-cell data (MLP classifier) rather than being supplied directly

## Limitations

- scSemiProfiler's deep generative model requires a single-cell reference to perform single-cell level deconvolution
- Scaden as benchmarked trains on only 5000 sampled cells, which may disadvantage it in the comparison
- CIBERSORTx depends on its official web portal, limiting automation
- Conventional bulk deconvolution methods are constrained to cell type level resolution and cannot achieve true single-cell resolution

## Evidence

- [intro] Prominent among these are deconvolution techniques such as CIBERSORTx12,13, Bisque14, DWLS15, MuSiC16, and NNLS16: "Prominent among these are deconvolution techniques such as CIBERSORTx12,13, Bisque14, DWLS15, MuSiC16, and NNLS16"
- [intro] EPIC requires signature matrix: "Another method, EPIC17, operates similarly but necessitates a pre-defined signature gene matrix."
- [readme] deconvolution pipeline in README: [evidence span withheld — anchor status: external_doc]
- [discussion] deconvolution needs single-cell reference: "The tool’s deep generative model enables “single-cell level deconvolution” across various studies, provided there is a single-cell reference."
- [results] workflow step from source card: [evidence span withheld — anchor status: not_found]
