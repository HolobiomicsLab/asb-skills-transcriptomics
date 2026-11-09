---
name: covid-19-severity-prediction
description: Use when comparing computational efficiency and classification accuracy
  of multi-context cell-cell communication tools on multi-sample single-cell data,
  e.g. predicting patient COVID-19 status/severity from communication features.
when_to_use_negative:
- Only a single sample/context is available — CellChat-style pairwise or single-context
  analysis suffices and the multi-context tensor decomposition has no advantage
- The goal is discovering new communication modules rather than benchmarking tool
  efficiency/accuracy against CellChat
- Cell types are not shared across samples — the COVID-19 tensor only includes cell
  types shared across all samples, so highly heterogeneous cohorts would require different
  handling
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Tensor-cell2cell
  role: Tensor factorization of the 4D communication tensor across all samples/contexts;
    its factor loadings serve as classification features
  repo: https://github.com/earmingol/cell2cell
- name: CellChat
  role: Baseline tool run per sample to produce joint-manifold UMAP embeddings used
    as competing classification features
  repo: https://github.com/sqjin/CellChat
- name: cell2cell
  role: Python suite providing preprocessing (subsampling to 2,000 cells/sample) and
    Tensor-cell2cell analysis pipelines
  repo: https://github.com/earmingol/cell2cell
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:20fb7a63c559b5bde1505d4e2f167f5c88530f22bde855c45871781306c1316e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - CellChat
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# covid-19-severity-prediction

## Summary

Benchmarks Tensor-cell2cell against CellChat for classifying COVID-19 status and severity, by training a Random-forest classifier (scikit-learn, 3-fold CV, 500 estimators) on Tensor-cell2cell factor loadings versus CellChat joint-manifold UMAP embeddings, and comparing runtime, memory, and AUC.

## When to use

Use when comparing computational efficiency and classification accuracy of multi-context cell-cell communication tools on multi-sample single-cell data, e.g. predicting patient COVID-19 status/severity from communication features.

## When NOT to use

- Only a single sample/context is available — CellChat-style pairwise or single-context analysis suffices and the multi-context tensor decomposition has no advantage
- The goal is discovering new communication modules rather than benchmarking tool efficiency/accuracy against CellChat
- Cell types are not shared across samples — the COVID-19 tensor only includes cell types shared across all samples, so highly heterogeneous cohorts would require different handling

## Inputs

- GSE158055 PBMC COVID-19 single-cell RNA-seq dataset (60 samples)
- Ligand-receptor pair list (e.g. Human-2020-Jin-LR-pairs.csv from CellChat's Jin et al. 2021 database)
- Per-sample COVID-19 status and severity labels

## Outputs

- Tensor-cell2cell factor loadings (context/rank decomposed communication tensor factors)
- CellChat joint-manifold UMAP embeddings per sample
- benchmark_metrics.csv with runtime, peak memory, and Random-forest AUC for both tools

## How to apply

Load the GSE158055 PBMC COVID-19 dataset and subsample to 2,000 cells per sample across 60 samples using cell2cell.preprocessing. Run Tensor-cell2cell tensor factorization across all contexts (fall back from init='svd' to init='random' if memory errors occur), recording wall-clock runtime and peak memory. Run CellChat per sample, compute its joint-manifold UMAP embeddings, and record the same metrics. Train an identical Random-forest classifier in scikit-learn with 3-fold cross-validation and 500 estimators, once on Tensor-cell2cell factor loadings and once on CellChat UMAP embeddings, to predict COVID-19 status and severity. Compile runtime, peak memory, and AUC into benchmark_metrics.csv; success is verified when Tensor-cell2cell is faster, uses less memory, and achieves ~10-20% higher classification accuracy than CellChat.

## Related tools

- **Tensor-cell2cell** (Tensor factorization of the 4D communication tensor across all samples/contexts; its factor loadings serve as classification features) — https://github.com/earmingol/cell2cell
- **CellChat** (Baseline tool run per sample to produce joint-manifold UMAP embeddings used as competing classification features) — https://github.com/sqjin/CellChat
- **cell2cell** (Python suite providing preprocessing (subsampling to 2,000 cells/sample) and Tensor-cell2cell analysis pipelines) — https://github.com/earmingol/cell2cell

## Examples

```
pip install cell2cell
```

## Evaluation signals

- benchmark_metrics.csv shows Tensor-cell2cell with lower wall-clock runtime and lower peak memory than CellChat
- Random-forest AUC (3-fold CV, 500 estimators) on Tensor-cell2cell loadings is ~10-20% higher than on CellChat UMAP embeddings
- Tensor factorization completes without memory errors; if init='svd' fails, init='random' run completes successfully
- Both classifiers use identical hyperparameters and cross-validation splits, so AUC differences are attributable to feature representations

## Limitations

- Only cell types shared across all samples were included in the COVID-19 tensor, potentially missing rare cell types
- Benchmarking CCC prediction tools is challenging due to the lack of a ground truth
- The implemented tensor decomposition algorithm is not optimized for missing values in the communication tensor

## Evidence

- [discussion] It is also more accurate, resulting in 10–20% higher classification accuracy of subjects with COVID-19 when compared to CellChat: "It is also more accurate, resulting in 10–20% higher classification accuracy of subjects with COVID-19 when compared to CellChat"
- [intro] Tensor-cell2cell is faster, demands less memory, and achieves better accuracy: "we show that Tensor-cell2cell is faster, demands less memory and can achieve better accuracy in separating context-specific information than simpler analyses"
- [methods] cell2cell.preprocessing module description: "cell2cell.preprocessing : Includes functions for manipulating data and variables (e.g. data preprocessing, integration, permutation, among others)."
- [readme] cell2cell installation via conda environment: "conda create -n cell2cell -y python=3.7 jupyter"
