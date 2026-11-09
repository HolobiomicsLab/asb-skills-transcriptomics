---
name: scrna-seq-count-matrix-downsampling
description: Use when stress-testing the robustness of a single-cell integration or
  label-transfer pipeline against simulated dropout noise in scRNA-seq counts, at
  graded dropout rates.
when_to_use_negative:
- Input matrices are already normalized or log-transformed — count downsampling (downsampleMatrix)
  is only meaningful on raw integer counts
- The noise perturbation should be applied to the target scATAC-seq modality — this
  skill only corrupts the source scRNA-seq counts
- The goal is to study biological cell-type variation rather than technical robustness
  to dropout
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scBridge
  role: Integration/label-transfer method whose robustness is evaluated across dropout
    levels; run via 'python main.py' with default hyperparameters
  repo: https://github.com/XLearning-SCU/scBridge
- name: Python
  role: Implementation environment for the downsampling benchmark and scBridge runs
- name: PyTorch 1.12.1
  role: Deep-learning framework in which scBridge is implemented and executed for
    each corrupted and baseline run
- name: Scikit-learn 1.1.1
  role: Provides the Gaussian Mixture Model used in scBridge's reliability modeling
    underlying the label transfer being stress-tested
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:4f62b5359302fc5881e2f442c9a6a397f98f85b1cbacfd35ad71204fa9bab4ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - PyTorch 1.12.1
  - Python
  - Scikit-learn 1.1.1
  - scBridge
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# scrna-seq-count-matrix-downsampling

## Summary

Simulates dropout technical noise in scRNA-seq count matrices by downsampling cell counts at controlled dropout rates (25%, 50%, 75%) so that the robustness of downstream multi-omics integration and label transfer (e.g., scBridge on human hematopoiesis data) can be benchmarked against an uncorrupted baseline.

## When to use

Use when stress-testing the robustness of a single-cell integration or label-transfer pipeline against simulated dropout noise in scRNA-seq counts, at graded dropout rates.

## When NOT to use

- Input matrices are already normalized or log-transformed — count downsampling (downsampleMatrix) is only meaningful on raw integer counts
- The noise perturbation should be applied to the target scATAC-seq modality — this skill only corrupts the source scRNA-seq counts
- The goal is to study biological cell-type variation rather than technical robustness to dropout

## Inputs

- scRNA-seq count matrix in AnnData (.h5ad) format with obs.['CellType'] annotations (source)
- scATAC-seq activity matrix in AnnData (.h5ad) format (target)
- List of dropout rates to simulate (e.g., 25%, 50%, 75%)

## Outputs

- Three dropout-corrupted scRNA-seq count matrices (25%, 50%, 75%)
- Per-run label transfer accuracy and silhouette score metrics
- Comparison table saved as dropout_benchmark_results.csv

## How to apply

Load the scRNA-seq and scATAC-seq h5ad files and apply count downsampling to the raw scRNA-seq count matrix at 25%, 50%, and 75% dropout using the scuttle downsampleMatrix function, producing three corrupted source datasets while leaving the target (scATAC-seq) untouched. Intersect to common genes between source count/activity matrices and preprocess with the article's defaults (source='Standard', target='TFIDF'). Run the integration (python main.py) once on the uncorrupted baseline and once per dropout level using the scBridge default hyperparameters: reliability_threshold=0.95, align_loss_epoch=1, prototype_momentum=0.9, early_stop_acc=0.99, max_iteration=20, batch_size=512, train_epoch=20, learning_rate=5e-4, random_seed=2023. Evaluate each run against target cell-type annotations by computing label transfer accuracy and silhouette score, aggregate across dropout levels into a comparison table (dropout_benchmark_results.csv), and verify accuracy remains near-unchanged up to 75% dropout.

## Related tools

- **scBridge** (Integration/label-transfer method whose robustness is evaluated across dropout levels; run via 'python main.py' with default hyperparameters) — https://github.com/XLearning-SCU/scBridge
- **Python** (Implementation environment for the downsampling benchmark and scBridge runs)
- **PyTorch 1.12.1** (Deep-learning framework in which scBridge is implemented and executed for each corrupted and baseline run)
- **Scikit-learn 1.1.1** (Provides the Gaussian Mixture Model used in scBridge's reliability modeling underlying the label transfer being stress-tested)

## Examples

```
python main.py
```

## Evaluation signals

- Label transfer accuracy at 25%, 50%, and 75% dropout is near-unchanged relative to the uncorrupted baseline run
- Silhouette score of the joint embedding remains stable across all dropout levels
- All runs (baseline + three dropout levels) use identical preprocessing ('Standard' source, 'TFIDF' target) and identical scBridge hyperparameters (random_seed=2023), so differences are attributable only to downsampling
- dropout_benchmark_results.csv is produced with one row per dropout level containing accuracy and silhouette metrics

## Limitations

- Robustness claim is empirically bounded: stability was demonstrated for dropout rates up to 75%; behavior beyond that rate is not established
- Downsampling simulates only dropout-type technical noise; other noise modes (batch effects, doublets, ambient RNA) are not captured
- Novel-type discovery guidance and TF-IDF discussion are deferred to Supplementary Notes 2 and 3, which are not included in the provided document

## Evidence

- [results] scBridge robustness finding: [evidence span withheld — anchor status: not_found]
- [readme] h5ad input format: "scBridge accepts AnnData (https://anndata.readthedocs.io/en/latest/index.html) as inputs, including the source (scRNA-seq) and target (scATAC-seq) h5ad files."
- [discussion] scBridge implementation stack: "scBridge is implemented in Python using the PyTorch41 framework, v.1.12.1."
- [other] scBridge entry point: [evidence span withheld — anchor status: not_found]
