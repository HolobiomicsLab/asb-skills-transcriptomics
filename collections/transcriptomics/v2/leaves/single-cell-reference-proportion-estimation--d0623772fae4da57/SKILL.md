---
name: single-cell-reference-proportion-estimation
description: Use when you have a bulk or pseudobulk RNA-seq cohort (e.g., 124-sample
  COVID-19 pseudobulk) plus a single-cell reference (annotated representative samples
  or public dataset) and need cell-type proportions comparable against ground truth.
when_to_use_negative:
- No single-cell reference exists — the article notes scSemiProfiler performs single-cell
  level deconvolution only 'provided there is a single-cell reference'
- The goal is true single-cell-resolution inference of within-cell-type variability,
  not cell-type-level proportion estimation, and no generative inference is feasible
- Input data are already single-cell profiles rather than bulk/pseudobulk measurements
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSemiProfiler
  role: 'Primary method: deep generative (VAE-GAN) single-cell-level deconvolution
    from bulk data using a single-cell reference; cell annotation via MLP classifier'
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: CIBERSORTx
  role: Competing deconvolution baseline run via its official web portal
- name: Bisque
  role: Competing deconvolution baseline installed from its official GitHub repository
- name: MuSiC
  role: Competing deconvolution baseline; also provides the NNLS implementation
- name: EPIC
  role: Competing baseline requiring a pre-defined signature gene matrix, installed
    from its official GitHub repository
- name: Scaden
  role: Deep-learning deconvolution baseline run via the PyTorch version in TAPE's
    repository (trains on 5000 sampled cells)
- name: TAPE
  role: Deep-learning deconvolution baseline installed from its official GitHub repository;
    hosts the Scaden PyTorch version
- name: SciPy
  role: Computes Pearson correlation coefficients and one-sided Wilcoxon significance
    tests
- name: Scikit-learn
  role: Trains the MLP classifier used to annotate generated cells and derive cell-type
    proportions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:69cfbe234ba7f53818e7591812060c6271d0c46c12d4a1eeb8a7b42d2a90b705
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# single-cell-reference-proportion-estimation

## Summary

Estimates cell-type proportions from bulk/pseudobulk RNA-seq cohorts using single-cell references — either via scSemiProfiler's deep-generative single-cell-level deconvolution or by benchmarking against established deconvolution methods (CIBERSORTx, Bisque, Scaden, TAPE, EPIC, NNLS, MuSiC).

## When to use

Use when you have a bulk or pseudobulk RNA-seq cohort (e.g., 124-sample COVID-19 pseudobulk) plus a single-cell reference (annotated representative samples or public dataset) and need cell-type proportions comparable against ground truth.

## When NOT to use

- No single-cell reference exists — the article notes scSemiProfiler performs single-cell level deconvolution only 'provided there is a single-cell reference'
- The goal is true single-cell-resolution inference of within-cell-type variability, not cell-type-level proportion estimation, and no generative inference is feasible
- Input data are already single-cell profiles rather than bulk/pseudobulk measurements

## Inputs

- Bulk RNA-seq profiles for a target cohort (e.g., COVID-19 pseudobulk, 124 samples)
- Single-cell reference data (annotated representatives' scRNA-seq, or a public reference dataset)
- Bulk data for the reference samples (required by scSemiProfiler's deconvolution pipeline)
- Ground-truth cell-type proportions for evaluation (when benchmarking)

## Outputs

- Per-sample cell-type proportion estimates for each method
- deconvolution_benchmark.csv with 9 method rows (scSemiProfiler-28, scSemiProfiler-4, 7 competing methods) and CCC, Pearson, RMSE, Wilcoxon p-value columns
- Annotated single-cell data for target samples (via classifier trained on reference annotations)

## How to apply

Annotate generated/semi-profiled cells with a Scikit-learn MLP classifier trained on the representatives' annotated single-cell data, then aggregate per-sample cell-type proportions. For scSemiProfiler deconvolution, run the pipeline with a full single-cell reference (e.g., 28 representatives) and, for fair comparison, a capacity-matched variant confined to 4 representatives. Run competing deconvolution methods on the same cohort: CIBERSORTx via its official web portal, Bisque/TAPE/MuSiC/EPIC from their official GitHub repositories, NNLS from the MuSiC package, and Scaden via the PyTorch version in TAPE's repository (which samples only 5000 cells for training). Evaluate each method's estimated proportions against ground-truth proportions using Lin's CCC, Pearson correlation (SciPy), and RMSE across pseudobulk samples, and assess pairwise significance with a one-sided Wilcoxon test in SciPy.

## Related tools

- **scSemiProfiler** (Primary method: deep generative (VAE-GAN) single-cell-level deconvolution from bulk data using a single-cell reference; cell annotation via MLP classifier) — https://github.com/mcgilldinglab/scSemiProfiler
- **CIBERSORTx** (Competing deconvolution baseline run via its official web portal)
- **Bisque** (Competing deconvolution baseline installed from its official GitHub repository)
- **MuSiC** (Competing deconvolution baseline; also provides the NNLS implementation)
- **EPIC** (Competing baseline requiring a pre-defined signature gene matrix, installed from its official GitHub repository)
- **Scaden** (Deep-learning deconvolution baseline run via the PyTorch version in TAPE's repository (trains on 5000 sampled cells))
- **TAPE** (Deep-learning deconvolution baseline installed from its official GitHub repository; hosts the Scaden PyTorch version)
- **SciPy** (Computes Pearson correlation coefficients and one-sided Wilcoxon significance tests)
- **Scikit-learn** (Trains the MLP classifier used to annotate generated cells and derive cell-type proportions)

## Examples

```
pip install --upgrade https://github.com/mcgilldinglab/scSemiProfiler/zipball/main
```

## Evaluation signals

- deconvolution_benchmark.csv exists with exactly 9 method rows (scSemiProfiler-28, scSemiProfiler-4, and the 7 competing methods) each carrying CCC, Pearson, RMSE, and Wilcoxon p-value columns
- Metric values match the article's Source Data for Fig. 2j-l (doi 10.1038/s41467-024-50150-1)
- scSemiProfiler (both 4- and 28-representative references) should outperform all benchmarked methods on Lin's CCC, Pearson, and RMSE in the COVID-19 cohort
- Semi-profiled cohort cell-type proportions should be nearly identical to real-profiled ground truth across disease conditions (stacked-bar check)
- One-sided Wilcoxon tests quantify pairwise significance between methods across the 124 pseudobulk samples

## Limitations

- The tool requires a single-cell reference to perform single-cell level deconvolution
- Deconvolution is cell-type-level unless scSemiProfiler's generative pipeline is used; conventional bulk deconvolution methods cannot achieve true single-cell resolution to capture variability within cell types
- The Scaden baseline in the benchmark samples only 5000 cells for training, and CIBERSORTx depends on an external web portal
- Benchmark comparisons depend on matching reference capacity (4 representatives) for fairness against competing methods

## Evidence

- [intro] Prominent among these are deconvolution techniques such as CIBERSORTx12,13, Bisque14, DWLS15, MuSiC16, and NNLS16: "Prominent among these are deconvolution techniques such as CIBERSORTx12,13, Bisque14, DWLS15, MuSiC16, and NNLS16"
- [discussion] single-cell reference required: "The tool’s deep generative model enables “single-cell level deconvolution” across various studies, provided there is a single-cell reference."
- [readme] README deconvolution scenario: [evidence span withheld — anchor status: external_doc]
- [readme] README annotation after pipeline: [evidence span withheld — anchor status: external_doc]
- [intro] deep-learning baselines: "Scaden18 and TAPE19 present an emerging category of methods that employ more sophisticated deep neural networks"
