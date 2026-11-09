---
name: dropout-robustness-benchmarking
description: Use when stress-testing a cross-omics integration/label-transfer method
  against simulated dropout noise in scRNA-seq data, or when reporting stability of
  scBridge under count down-sampling regimes.
when_to_use_negative:
- The goal is to evaluate robustness to batch effects, normalization differences,
  or paired multi-omics differences rather than simulated dropout noise
- Input data are protein (CITE-seq/ASAP-seq) measurements, since this benchmark's
  down-sampling targets the scRNA-seq count matrix
- The method under test cannot accept the scuttle downsampleMatrix-generated corrupted
  source matrices in h5ad format
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: scBridge
  role: Integration and label transfer method benchmarked under dropout noise; run
    via 'python main.py' with default hyperparameters
  repo: https://github.com/XLearning-SCU/scBridge
- name: PyTorch 1.12.1
  role: Deep learning framework in which scBridge is implemented and executed
- name: Scikit-learn
  role: Provides the Gaussian Mixture Model used for reliability modeling during scBridge
    training
- name: Python
  role: Implementation environment for down-sampling (scuttle downsampleMatrix), running
    scBridge, and aggregating metrics
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:79c86bb65892b52cf2352c8292b4120268cadf752eb20770da96a0ef063d935d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - PyTorch 1.12.1
  - Python
  - Scikit-learn
  - scBridge
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# dropout-robustness-benchmarking

## Summary

Benchmark robustness of scBridge's scRNA-seq to scATAC-seq integration and label transfer by down-sampling source counts at 25%, 50%, and 75% dropout and comparing label transfer accuracy and silhouette score against the uncorrupted baseline.

## When to use

Use when stress-testing a cross-omics integration/label-transfer method against simulated dropout noise in scRNA-seq data, or when reporting stability of scBridge under count down-sampling regimes.

## When NOT to use

- The goal is to evaluate robustness to batch effects, normalization differences, or paired multi-omics differences rather than simulated dropout noise
- Input data are protein (CITE-seq/ASAP-seq) measurements, since this benchmark's down-sampling targets the scRNA-seq count matrix
- The method under test cannot accept the scuttle downsampleMatrix-generated corrupted source matrices in h5ad format

## Inputs

- Source scRNA-seq count matrix as h5ad AnnData with cell type annotation in obs["CellType"]
- Target scATAC-seq gene activity matrix as h5ad AnnData (peak matrix transformed to activity matrix)
- Target cell-type annotations for accuracy evaluation

## Outputs

- Three corrupted scRNA-seq source datasets at 25%, 50%, and 75% dropout
- Per-run label transfer accuracy and silhouette score metrics per dropout level
- Aggregated comparison table (dropout_benchmark_results.csv)

## How to apply

Load the source (scRNA-seq) and target (scATAC-seq) h5ad AnnData files and generate corrupted source datasets by count down-sampling the scRNA-seq matrix at 25%, 50%, and 75% dropout using the scuttle downsampleMatrix function. Select common genes between source count/activity matrices and preprocess with the default strategies (source='Standard', target='TFIDF'). Run scBridge via 'python main.py' with default hyperparameters (reliability_threshold=0.95, align_loss_epoch=1, prototype_momentum=0.9, early_stop_acc=0.99, max_iteration=20, batch_size=512, train_epoch=20, learning_rate=5e-4, random_seed=2023) for the baseline and each dropout level. Evaluate each run against target annotations by computing label transfer accuracy and silhouette score, then aggregate into a comparison table (dropout_benchmark_results.csv) and verify accuracy is near-unchanged up to 75% dropout.

## Related tools

- **scBridge** (Integration and label transfer method benchmarked under dropout noise; run via 'python main.py' with default hyperparameters) — https://github.com/XLearning-SCU/scBridge
- **PyTorch 1.12.1** (Deep learning framework in which scBridge is implemented and executed)
- **Scikit-learn** (Provides the Gaussian Mixture Model used for reliability modeling during scBridge training)
- **Python** (Implementation environment for down-sampling (scuttle downsampleMatrix), running scBridge, and aggregating metrics)

## Examples

```
python main.py --reliability_threshold=0.95 --align_loss_epoch=1 --max_iteration=20 --batch_size=512 --train_epoch=20 --learning_rate=5e-4 --random_seed=2023
```

## Evaluation signals

- Label transfer accuracy at 25%, 50%, and 75% dropout remains near-unchanged relative to the uncorrupted baseline
- Silhouette score of the joint embedding stays stable across dropout levels
- Each run completes with default hyperparameters intact: reliability_threshold=0.95, early_stop_acc=0.99, max_iteration=20, random_seed=2023
- dropout_benchmark_results.csv contains rows for the baseline plus all three dropout levels with accuracy and silhouette columns populated
- Common-gene selection and default preprocessing (source='Standard', target='TFIDF') were applied identically to baseline and corrupted runs

## Limitations

- Robustness is evaluated only at simulated dropout rates of 25%, 50%, and 75%; other noise modalities (batch effects, protein-level dropout) are not covered
- Early stopping at 99% annotated-data classification accuracy and fixed seed (2023) means results may vary under different seeds or stopping criteria
- The benchmark depends on down-sampled scRNA-seq counts only; target scATAC-seq data are left uncorrupted

## Evidence

- [methods] default preprocessing: "`--source_preprocess=["Standard", "TFIDF"]` (Default="Standard") the preprocessing strategy for source data"
- [methods] reliability threshold: "`--reliability_threshold=[0.0-1.0]` (Default=0.95) threshold for selecting reliable cells"
- [discussion] early stopping rule: "we early stopped the network optimization when the cell classification accuracy of the annotated data reached 99%"
- [discussion] framework and version: "scBridge is implemented in Python using the PyTorch41 framework, v.1.12.1."
- [readme] input data format: "scBridge accepts AnnData (https://anndata.readthedocs.io/en/latest/index.html) as inputs, including the source (scRNA-seq) and target (scATAC-seq) h5ad files."
