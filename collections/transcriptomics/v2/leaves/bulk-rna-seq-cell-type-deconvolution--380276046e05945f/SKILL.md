---
name: bulk-rna-seq-cell-type-deconvolution
description: Use when bulk or pseudobulk RNA-seq samples need cell type proportions
  estimated, and a single-cell reference (real or semi-profiled) is available, or
  when benchmarking deconvolution methods against ground-truth proportions.
when_to_use_negative:
- No single-cell reference exists — scSemiProfiler's deep generative model enables
  single-cell level deconvolution only 'provided there is a single-cell reference'
- Input is already a cell type proportion table rather than bulk/pseudobulk expression
  data
- The goal is single-cell resolution downstream analysis other than deconvolution
  (e.g., UMAP, pseudotime) for which semi-profiling rather than deconvolution benchmarking
  is appropriate
edam_operation: http://edamontology.org/operation_3634
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scSemiProfiler
  role: 'Single-cell level deconvolution: infers single-cell data from bulk profiles
    plus a single-cell reference; annotations via MLP classifier trained on reference
    data'
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: CIBERSORTx
  role: Competing bulk deconvolution method run via its official web portal
- name: Bisque
  role: Competing deconvolution method installed from its official GitHub repository
- name: TAPE
  role: Competing deep-learning deconvolution method; its PyTorch Scaden implementation
    is used
- name: MuSiC
  role: Competing deconvolution method; also provides NNLS
- name: EPIC
  role: Competing deconvolution method requiring a pre-defined signature gene matrix
- name: Scaden
  role: Deep neural network competitor trained on 5000 sampled cells
- name: NNLS
  role: Non-negative least squares deconvolution baseline from the MuSiC package
- name: SciPy
  role: Pearson correlation and one-sided Wilcoxon significance testing
- name: Scikit-learn
  role: MLP classifier for annotating generated cells trained on representatives'
    annotated data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:6a23de7a41856317efb72a645088d13809cc31b645c724fd52e2b5e19a5a831d
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
  edam_operation: http://edamontology.org/operation_3634
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# bulk-rna-seq-cell-type-deconvolution

## Summary

Benchmark and apply bulk RNA-seq cell type deconvolution on pseudobulk cohorts (e.g., the 124-sample COVID-19 cohort), comparing scSemiProfiler's single-cell-level deconvolution against CIBERSORTx, Bisque, Scaden, TAPE, EPIC, NNLS, and MuSiC using Lin's CCC, Pearson correlation, and RMSE.

## When to use

Use when bulk or pseudobulk RNA-seq samples need cell type proportions estimated, and a single-cell reference (real or semi-profiled) is available, or when benchmarking deconvolution methods against ground-truth proportions.

## When NOT to use

- No single-cell reference exists — scSemiProfiler's deep generative model enables single-cell level deconvolution only 'provided there is a single-cell reference'
- Input is already a cell type proportion table rather than bulk/pseudobulk expression data
- The goal is single-cell resolution downstream analysis other than deconvolution (e.g., UMAP, pseudotime) for which semi-profiling rather than deconvolution benchmarking is appropriate

## Inputs

- Bulk or pseudobulk RNA-seq profiles for a cohort (e.g., 124-sample COVID-19 pseudobulk)
- Annotated single-cell reference data from representative samples (28 or 4 representatives)
- Ground-truth cell type proportions per sample for benchmarking

## Outputs

- Estimated cell type proportions per sample per method
- deconvolution_benchmark.csv with 9 method rows (scSemiProfiler-28, scSemiProfiler-4, 7 competing methods) and CCC, Pearson, RMSE, Wilcoxon p-value columns
- Inferred single-cell data for target samples usable for downstream annotation

## How to apply

Compute ground-truth cell type proportions from annotated single-cell data, then run each deconvolution method on the cohort's pseudobulk profiles: scSemiProfiler infers single-cell data from bulk plus a single-cell reference (e.g., 28-representative reference, or a 4-representative reference for fair comparison), CIBERSORTx via its web portal, Bisque/TAPE/MuSiC/EPIC from their official GitHub repositories, NNLS from the MuSiC package, and Scaden via the PyTorch version in TAPE's repository (which trains on only 5000 cells). Annotate generated/semi-profiled cells with a Scikit-learn MLP classifier trained on the reference's annotated data before computing proportions. Score each method against ground truth with Lin's CCC, Pearson correlation (SciPy), and RMSE, and test pairwise significance with a one-sided Wilcoxon test. In the COVID-19 benchmark, scSemiProfiler outperformed all benchmarked methods with both the 28- and 4-representative references.

## Related tools

- **scSemiProfiler** (Single-cell level deconvolution: infers single-cell data from bulk profiles plus a single-cell reference; annotations via MLP classifier trained on reference data) — https://github.com/mcgilldinglab/scSemiProfiler
- **CIBERSORTx** (Competing bulk deconvolution method run via its official web portal)
- **Bisque** (Competing deconvolution method installed from its official GitHub repository)
- **TAPE** (Competing deep-learning deconvolution method; its PyTorch Scaden implementation is used)
- **MuSiC** (Competing deconvolution method; also provides NNLS)
- **EPIC** (Competing deconvolution method requiring a pre-defined signature gene matrix)
- **Scaden** (Deep neural network competitor trained on 5000 sampled cells)
- **NNLS** (Non-negative least squares deconvolution baseline from the MuSiC package)
- **SciPy** (Pearson correlation and one-sided Wilcoxon significance testing)
- **Scikit-learn** (MLP classifier for annotating generated cells trained on representatives' annotated data)

## Examples

```
pip install --upgrade https://github.com/mcgilldinglab/scSemiProfiler/zipball/main
```

## Evaluation signals

- deconvolution_benchmark.csv exists with exactly 9 method-condition rows (scSemiProfiler-28, scSemiProfiler-4, and 7 competing methods)
- CCC, Pearson, and RMSE values match the article's Source Data for Fig. 2j-l (doi 10.1038/s41467-024-50150-1)
- scSemiProfiler with the 4-representative reference is evaluated to match the capacity of competing methods (fair comparison)
- Pairwise one-sided Wilcoxon p-values between methods are reported for all N=124 samples
- Semi-profiled cell type proportions closely match ground truth (stacked bar plots nearly identical across disease conditions)

## Limitations

- scSemiProfiler requires a single-cell reference to perform single-cell level deconvolution
- Scaden implementation samples only 5000 cells for training, potentially limiting its performance
- CIBERSORTx relies on an external web portal; EPIC requires a pre-defined signature gene matrix
- Conventional deconvolution methods are constrained to cell-type-level resolution and cannot achieve true single-cell resolution

## Evidence

- [discussion] The tool's deep generative model enables "single-cell level deconvolution" across various studies, provided there is a single-cell reference.: [evidence span withheld — anchor status: not_found]
- [intro] conventional methods are constrained to cell type level resolution and are incapable of achieving the true single-cell resolution required: "conventional methods are constrained to cell type level resolution and are incapable of achieving the true single-cell resolution required to capture the significant variability within cell types"
- [readme] This process allows users to deconvolute bulk RNA-seq data from a target sample into single-cell data, using a single-cell reference sample as a guide.: [evidence span withheld — anchor status: external_doc]
- [intro] Another method, EPIC17, operates similarly but necessitates a pre-defined signature gene matrix.: "Another method, EPIC17, operates similarly but necessitates a pre-defined signature gene matrix."
- [methods] The COVID-19 dataset arrives already preprocessed as detailed by: "The COVID-19 dataset arrives already preprocessed as detailed by45"
