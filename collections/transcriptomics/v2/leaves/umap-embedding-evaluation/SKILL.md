---
name: umap-embedding-evaluation
description: Use when benchmarking how well cell-cell communication features (UMAP
  embeddings or tensor factor loadings) separate phenotypic contexts such as COVID-19
  status and severity, using a supervised classifier's cross-validated AUC.
when_to_use_negative:
- Only a single sample/context is available — the benchmark's premise is separating
  information across multiple contexts, so embeddings from one context cannot be evaluated
  this way.
- Cell types are not shared across all samples — the COVID-19 and ASD tensors included
  only cell types shared across all samples, so classification features would be incomparable.
- You need cell-cell communication inference itself rather than a benchmark of representation
  quality — use the inference pipelines (CellChat or Tensor-cell2cell) directly instead.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Tensor-cell2cell
  role: Produces the factor loadings (treatments/context components of the 4D-communication
    tensor) used as classification features and the faster, lower-memory, higher-accuracy
    arm of the benchmark.
  repo: https://github.com/earmingol/cell2cell
- name: CellChat
  role: Generates the joint-manifold UMAP embeddings per sample that serve as the
    competing classification features in the head-to-head accuracy comparison.
  repo: https://github.com/sqjin/CellChat
- name: cell2cell
  role: Supplies the suite containing Tensor-cell2cell plus cell2cell.preprocessing
    functions used to load and subsample the PBMC dataset to 2,000 cells per sample.
  repo: https://github.com/earmingol/cell2cell
- name: scikit-learn
  role: Provides the Random-forest classifier (500 estimators) and 3-fold cross-validation
    used to compute AUC for each feature representation.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:453fa6eb984479d16d0086d47cfc9681634ee78a5234c2680cdeef9c79f1eef2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - CellChat
  - Tensor-cell2cell
  - cell2cell
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# umap-embedding-evaluation

## Summary

Evaluates whether low-dimensional embeddings (e.g., CellChat's joint-manifold UMAP embeddings or Tensor-cell2cell factor loadings) capture biologically meaningful, context-specific information by training a Random-forest classifier to predict phenotype labels and comparing accuracy, runtime, and memory against competing representations.

## When to use

Use when benchmarking how well cell-cell communication features (UMAP embeddings or tensor factor loadings) separate phenotypic contexts such as COVID-19 status and severity, using a supervised classifier's cross-validated AUC.

## When NOT to use

- Only a single sample/context is available — the benchmark's premise is separating information across multiple contexts, so embeddings from one context cannot be evaluated this way.
- Cell types are not shared across all samples — the COVID-19 and ASD tensors included only cell types shared across all samples, so classification features would be incomparable.
- You need cell-cell communication inference itself rather than a benchmark of representation quality — use the inference pipelines (CellChat or Tensor-cell2cell) directly instead.

## Inputs

- Per-sample cell-cell communication scores from GSE158055 PBMC COVID-19 dataset (2,000 cells subsampled per sample, 60 samples)
- Tensor-cell2cell factor loadings from the 4D-communication tensor (contexts x LR pairs x sender cells x receiver cells)
- CellChat joint-manifold UMAP embeddings computed per sample
- Phenotype labels (COVID-19 status and severity) per subject

## Outputs

- benchmark_metrics.csv with wall-clock runtime, peak memory, and Random-forest AUC (3-fold CV, 500 estimators) for each tool's representation
- Per-fold AUC values from cross-validated classification of COVID-19 status and severity
- Comparison verdict on speed, memory, and ~10-20% classification-accuracy advantage of Tensor-cell2cell over CellChat

## How to apply

On the multi-sample GSE158055 PBMC COVID-19 dataset (subsampled to 2,000 cells per sample via cell2cell.preprocessing), run CellChat per sample and compute its joint-manifold UMAP embeddings, while Tensor-cell2cell performs tensor factorization across all contexts (use init='random' instead of init='svd' if memory errors occur). Train a scikit-learn Random-forest classifier with 3-fold cross-validation and 500 estimators, once on Tensor-cell2cell factor loadings and once on CellChat's joint-manifold UMAP embeddings, to predict COVID-19 status and severity, recording the AUC for each. Simultaneously record wall-clock runtime and peak memory for each workflow. The rationale is that classification accuracy of held-out phenotype labels is a proxy for how well each representation separates context-specific communication information; the paper reports Tensor-cell2cell loadings yield 10–20% higher classification accuracy than CellChat embeddings. Compile runtime, peak memory, and AUC values into benchmark_metrics.csv for the head-to-head comparison.

## Related tools

- **Tensor-cell2cell** (Produces the factor loadings (treatments/context components of the 4D-communication tensor) used as classification features and the faster, lower-memory, higher-accuracy arm of the benchmark.) — https://github.com/earmingol/cell2cell
- **CellChat** (Generates the joint-manifold UMAP embeddings per sample that serve as the competing classification features in the head-to-head accuracy comparison.) — https://github.com/sqjin/CellChat
- **cell2cell** (Supplies the suite containing Tensor-cell2cell plus cell2cell.preprocessing functions used to load and subsample the PBMC dataset to 2,000 cells per sample.) — https://github.com/earmingol/cell2cell
- **scikit-learn** (Provides the Random-forest classifier (500 estimators) and 3-fold cross-validation used to compute AUC for each feature representation.)

## Examples

```
from sklearn.ensemble import RandomForestClassifier; from sklearn.model_selection import cross_val_score; cross_val_score(RandomForestClassifier(n_estimators=500), X_umap, labels, cv=3, scoring='roc_auc')
```

## Evaluation signals

- Random-forest AUC on Tensor-cell2cell loadings should exceed CellChat UMAP-embedding AUC by roughly 10-20% for COVID-19 status/severity classification.
- Cross-validation is 3-fold with 500 trees in the Random-forest; deviating from these settings breaks comparability with the published benchmark.
- Wall-clock runtime and peak memory of the Tensor-cell2cell arm should be lower than the CellChat per-sample + joint-manifold workflow.
- benchmark_metrics.csv must contain runtime, peak memory, and AUC for both tools' representations.
- If tensor factorization raises memory errors, re-running with init='random' should succeed and yield comparable results to init='svd'.

## Limitations

- Benchmarking CCC prediction tools is challenging due to the lack of a ground truth.
- The benchmark tensors contain only cell types shared across all samples, potentially missing rare cell types.
- Runtime and memory are hardware-dependent; GPU use further increases Tensor-cell2cell speed, so absolute timings vary across machines.

## Evidence

- [discussion] 10-20% higher classification accuracy than CellChat: "It is also more accurate, resulting in 10–20% higher classification accuracy of subjects with COVID-19 when compared to CellChat"
- [intro] faster, less memory, better accuracy: "we show that Tensor-cell2cell is faster, demands less memory and can achieve better accuracy in separating context-specific information than simpler analyses accessible to other tools."
- [discussion] CellChat pathway-level scores: "Other tools, such as CellChat, allow the generation of scores at the signaling pathway level instead of LR pairs."
- [readme] install cell2cell for reproducible environment: "pip install 'cell2cell==0.6.0'"
- [discussion] benchmarking lacks ground truth: "we note that benchmarking CCC prediction tools is challenging due to the lack of a ground truth"
