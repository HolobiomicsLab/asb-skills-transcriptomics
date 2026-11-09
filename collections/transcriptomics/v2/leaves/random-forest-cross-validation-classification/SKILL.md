---
name: random-forest-cross-validation-classification
description: Use when benchmarking how well cell-cell communication features (tensor
  factor loadings or UMAP embeddings from LR communication scores) separate biological
  phenotypes such as COVID-19 status and severity.
when_to_use_negative:
- The goal is unsupervised discovery of communication modules rather than quantitative
  benchmarking of classification accuracy against another tool
- No phenotype labels (e.g., COVID-19 status/severity) are available for the samples,
  so a supervised classifier cannot be trained
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Tensor-cell2cell
  role: Source of factor loadings used as classifier features for predicting COVID-19
    status and severity
  repo: https://github.com/earmingol/cell2cell
- name: CellChat
  role: Baseline tool whose joint-manifold UMAP embeddings provide the comparison
    feature set for the same classifier
  repo: https://github.com/sqjin/CellChat
- name: cell2cell
  role: Python suite providing preprocessing (subsample to 2,000 cells/sample) and
    tensor factorization that generate the loadings
  repo: https://github.com/earmingol/cell2cell
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:f127a3c0d24a2bc065a38a6fc73d1912c77c807cc0a7bcaae1175729a4bc6246
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - CellChat
  - Tensor-cell2cell
  - cell2cell
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# random-forest-cross-validation-classification

## Summary

Trains a scikit-learn Random-forest classifier (500 estimators, 3-fold cross-validation) on Tensor-cell2cell factor loadings and on CellChat joint-manifold UMAP embeddings to predict COVID-19 status and severity, quantifying each tool's classification AUC as a benchmark of accuracy.

## When to use

Use when benchmarking how well cell-cell communication features (tensor factor loadings or UMAP embeddings from LR communication scores) separate biological phenotypes such as COVID-19 status and severity.

## When NOT to use

- The goal is unsupervised discovery of communication modules rather than quantitative benchmarking of classification accuracy against another tool
- No phenotype labels (e.g., COVID-19 status/severity) are available for the samples, so a supervised classifier cannot be trained

## Inputs

- Tensor-cell2cell factor loadings from the 4D-communication tensor built on 60 GSE158055 PBMC samples (2,000 cells subsampled per sample)
- CellChat joint-manifold UMAP embeddings per sample
- Per-sample COVID-19 status and severity labels

## Outputs

- Cross-validated AUC values for each tool's feature representation
- benchmark_metrics.csv compiling runtime, peak memory, and AUC for Tensor-cell2cell and CellChat

## How to apply

Extract feature matrices from each competing representation: Tensor-cell2cell factor loadings from the 4D-communication tensor built on the GSE158055 PBMC COVID-19 samples, and CellChat's joint-manifold UMAP embeddings computed per sample. For each feature set, fit a sklearn Random-forest classifier with 500 estimators under 3-fold cross-validation to predict COVID-19 status and severity, recording the AUC. Compile runtime, peak memory, and AUC into benchmark_metrics.csv so both tools are compared on identical prediction targets. The rationale is that a supervised classifier quantifies how well context-specific information is separated, letting the faster, lower-memory Tensor-cell2cell be judged against CellChat's pairwise workflow on a common metric; the paper reports 10–20% higher COVID-19 classification accuracy for Tensor-cell2cell.

## Related tools

- **Tensor-cell2cell** (Source of factor loadings used as classifier features for predicting COVID-19 status and severity) — https://github.com/earmingol/cell2cell
- **CellChat** (Baseline tool whose joint-manifold UMAP embeddings provide the comparison feature set for the same classifier) — https://github.com/sqjin/CellChat
- **cell2cell** (Python suite providing preprocessing (subsample to 2,000 cells/sample) and tensor factorization that generate the loadings) — https://github.com/earmingol/cell2cell

## Examples

```
from sklearn.ensemble import RandomForestClassifier; clf = RandomForestClassifier(n_estimators=500); scores = cross_val_score(clf, loadings, labels, cv=3, scoring='roc_auc')
```

## Evaluation signals

- Tensor-cell2cell loadings achieve ~10–20% higher classification AUC of COVID-19 subjects than CellChat UMAP embeddings
- Both classifiers use identical settings: 500 estimators, 3-fold cross-validation, same label set (status and severity)
- benchmark_metrics.csv contains runtime, peak memory, and AUC for both tools, with Tensor-cell2cell faster and lower-memory
- AUC is above chance (0.5) for both feature sets, confirming labels are separable from communication features

## Limitations

- Benchmarking CCC prediction tools is challenging due to the lack of a ground truth, so classifier AUC serves as a proxy for accuracy
- The COVID-19 tensor included only cell types shared across all samples, potentially missing rare cell types
- GPU use further increases Tensor-cell2cell speed, but results depend on init: use init='random' instead of init='svd' if memory errors occur

## Evidence

- [discussion] Tensor-cell2cell achieved 10–20% higher classification accuracy of subjects with COVID-19 compared to CellChat: "It is also more accurate, resulting in 10–20% higher classification accuracy of subjects with COVID-19 when compared to CellChat"
- [intro] Tensor-cell2cell is faster and more accurate in separating context-specific information: "we show that Tensor-cell2cell is faster, demands less memory and can achieve better accuracy in separating context-specific information than simpler analyses accessible to other tools."
- [readme] CellChat provides the baseline communication scores and embeddings for comparison: [evidence span withheld — anchor status: external_doc]
- [methods] cell2cell package installation provides the feature-generating pipeline: [evidence span withheld — anchor status: not_found]
