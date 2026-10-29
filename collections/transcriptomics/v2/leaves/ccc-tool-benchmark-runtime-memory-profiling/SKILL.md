---
name: ccc-tool-benchmark-runtime-memory-profiling
description: Use when comparing computational cost and predictive power of CCC tools
  across multiple samples/contexts, e.g. benchmarking Tensor-cell2cell vs CellChat
  on runtime, peak memory, and AUC for phenotypic classification.
when_to_use_negative:
- Only one sample/context is available — CellChat's pairwise-comparison disadvantage
  and Tensor-cell2cell's multi-context advantage do not apply
- The goal is a biological CCC interpretation rather than a tool benchmark; the classifier-on-loadings
  setup here measures separation of contexts, not ground-truth interaction accuracy
- Input is already a factor-loading or embedding table and the question is purely
  classifier performance rather than end-to-end tool benchmarking
tools:
- name: Tensor-cell2cell
  role: Tool under test; runs tensor factorization across all contexts simultaneously
    (use init='random' if init='svd' hits memory errors) and supplies factor loadings
    for the Random-forest classifier
  repo: https://github.com/earmingol/cell2cell
- name: CellChat
  role: Baseline tool under test; run per sample, with joint-manifold UMAP embeddings
    serving as features for the identical Random-forest classifier
  repo: https://github.com/sqjin/CellChat
- name: cell2cell
  role: Python suite hosting Tensor-cell2cell; cell2cell.preprocessing handles subsampling
    to 2,000 cells per sample across the 60 contexts
  repo: https://github.com/earmingol/cell2cell
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:34fb1e7aee6f125c59349c64b65c6a2ecc2ceb1dc7599c7cf1fe60c492765233
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - CellChat
  - Tensor-cell2cell
  - cell2cell
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# ccc-tool-benchmark-runtime-memory-profiling

## Summary

Head-to-head benchmarking of Tensor-cell2cell against CellChat on multi-sample cell-cell communication (CCC) analysis, recording wall-clock runtime, peak memory, and classification accuracy to quantify whether tensor decomposition outperforms pairwise-comparison workflows.

## When to use

Use when comparing computational cost and predictive power of CCC tools across multiple samples/contexts, e.g. benchmarking Tensor-cell2cell vs CellChat on runtime, peak memory, and AUC for phenotypic classification.

## When NOT to use

- Only one sample/context is available — CellChat's pairwise-comparison disadvantage and Tensor-cell2cell's multi-context advantage do not apply
- The goal is a biological CCC interpretation rather than a tool benchmark; the classifier-on-loadings setup here measures separation of contexts, not ground-truth interaction accuracy
- Input is already a factor-loading or embedding table and the question is purely classifier performance rather than end-to-end tool benchmarking

## Inputs

- Multi-sample single-cell RNA-seq dataset (e.g. GSE158055 PBMC COVID-19, ~60 samples) with COVID-19 status/severity labels
- Subsampled count matrices normalized to a fixed number of cells per context (2,000 cells/sample)
- Tensor-cell2cell factor loadings from the 4D-communication tensor (contexts x LR pairs x source cells x target cells)
- CellChat joint-manifold UMAP embeddings computed per sample

## Outputs

- benchmark_metrics.csv with runtime, peak memory, and classification AUC per tool
- Random-forest classifier AUC for COVID-19 status/severity prediction (3-fold CV, 500 estimators)
- Runtime and peak-memory measurements per tool, with and without GPU use for Tensor-cell2cell

## How to apply

Subsample the single-cell dataset to a fixed cell budget per context (e.g. 2,000 cells per sample across 60 samples using cell2cell.preprocessing) so both tools process identical inputs. Run Tensor-cell2cell tensor factorization across all contexts simultaneously, switching init='svd' to init='random' if memory errors occur, and record wall-clock runtime and peak memory; optionally test GPU acceleration for further speedup. Run CellChat per sample, compute its joint-manifold UMAP embeddings, and record the same metrics. Judge accuracy by training an identical downstream classifier — a scikit-learn Random-forest with 500 estimators and 3-fold cross-validation — once on Tensor-cell2cell factor loadings and once on CellChat's joint-manifold UMAP embeddings, recording AUC for predicting COVID-19 status and severity. Compile all metrics into a single benchmark_metrics.csv so runtime, memory, and AUC differences are directly comparable; the expected result is that Tensor-cell2cell is faster, uses less memory, and achieves ~10-20% higher classification accuracy than CellChat.

## Related tools

- **Tensor-cell2cell** (Tool under test; runs tensor factorization across all contexts simultaneously (use init='random' if init='svd' hits memory errors) and supplies factor loadings for the Random-forest classifier) — https://github.com/earmingol/cell2cell
- **CellChat** (Baseline tool under test; run per sample, with joint-manifold UMAP embeddings serving as features for the identical Random-forest classifier) — https://github.com/sqjin/CellChat
- **cell2cell** (Python suite hosting Tensor-cell2cell; cell2cell.preprocessing handles subsampling to 2,000 cells per sample across the 60 contexts) — https://github.com/earmingol/cell2cell

## Examples

```
pip install cell2cell
```

## Evaluation signals

- Tensor-cell2cell wall-clock runtime is lower than CellChat's per-sample-plus-pairwise workflow on the identical 60-sample, 2,000-cells-per-sample subsample
- Tensor-cell2cell peak memory is lower than CellChat's; a 'MemoryError' under init='svd' indicates the fallback to init='random' should have been applied
- Random-forest AUC from Tensor-cell2cell factor loadings is ~10-20% higher than the AUC from CellChat joint-manifold UMAP embeddings (3-fold CV, 500 estimators)
- benchmark_metrics.csv contains runtime, peak memory, and AUC rows for both tools with matched input conditions
- GPU-enabled Tensor-cell2cell runs further reduces runtime relative to CPU-only execution

## Limitations

- The implemented tensor decomposition algorithm is not optimized for missing values, and the COVID-19/ASD tensors only include cell types shared across all samples, potentially missing rare cell types
- Benchmarking CCC prediction tools is challenging due to the lack of a ground truth, and it is hard to compare tools because of qualitative differences in their outputs
- AUC here measures separation of phenotypic contexts, not accuracy of individual inferred interactions

## Evidence

- [intro] Tensor-cell2cell is faster, demands less memory and can achieve better accuracy in separating context-specific information than simpler analyses accessible to other tools.: "we show that Tensor-cell2cell is faster, demands less memory and can achieve better accuracy in separating context-specific information than simpler analyses accessible to other tools."
- [discussion] 10-20% higher classification accuracy than CellChat: "It is also more accurate, resulting in 10–20% higher classification accuracy of subjects with COVID-19 when compared to CellChat"
- [methods] cell2cell installation via pip: [evidence span withheld — anchor status: not_found]
- [readme] conda environment setup for cell2cell: "# Create a new conda environment
conda create -n cell2cell -y python=3.7 jupyter"
