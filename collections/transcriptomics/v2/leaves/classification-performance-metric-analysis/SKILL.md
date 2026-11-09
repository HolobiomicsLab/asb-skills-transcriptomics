---
name: classification-performance-metric-analysis
description: Use when benchmarking a binary cell-level drug-response predictor (e.g.
  scDEAL on scRNA-seq data with 'sens_label' in adata.obs) and you need multi-metric
  accuracy plus variation over repeated 80/20 stratified samplings.
when_to_use_negative:
- The task requires drug response prediction itself (IC50 or sensitivity inference)
  rather than evaluation of already-predicted labels — use the scDEAL training/prediction
  workflow instead.
- Labels are continuous drug response values (e.g. IC50/area under dose-response)
  rather than the binary sens_label (0/1) that classification metrics and AMI/ARI
  require.
- Only a single train/test split is available and repeated-sampling robustness estimation
  is not the goal — a single point estimate does not yield the across-run variation
  this skill targets.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: scDEAL
  role: DaNN-based deep transfer learning framework whose bulk- and single-cell-level
    models (bulkmodel.py, scmodel.py, trained with --checkpoint "False") produce the
    predicted sensitivity labels being evaluated
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: Scanpy
  role: Provides the AnnData (.h5ad) output objects from which adata.obs['sens_label']
    labels are read for metric computation
- name: AnnData
  role: Container format (.h5ad) holding true and predicted sensitivity labels for
    the held-out cells
- name: PyTorch
  role: Backend on which the scDEAL bulk and single-cell models are retrained for
    each of the 20 sampling repeats
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:93c3cbc426b5b4f2f164529d8cab20138b492f2d5eb19ac4fd1ab9fd1cf33c65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - PyTorch
  - Scanpy
  - scDEAL
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# classification-performance-metric-analysis

## Summary

Evaluate binary drug-response classification performance (resistant=0 vs sensitive=1) by computing F1-score, AUROC, average precision (AP), precision, recall, AMI, and ARI on scDEAL's predicted versus true sensitivity labels, and quantify robustness via metric variation (standard deviation) across repeated random stratified train/test splits.

## When to use

Use when benchmarking a binary cell-level drug-response predictor (e.g. scDEAL on scRNA-seq data with 'sens_label' in adata.obs) and you need multi-metric accuracy plus variation over repeated 80/20 stratified samplings.

## When NOT to use

- The task requires drug response prediction itself (IC50 or sensitivity inference) rather than evaluation of already-predicted labels — use the scDEAL training/prediction workflow instead.
- Labels are continuous drug response values (e.g. IC50/area under dose-response) rather than the binary sens_label (0/1) that classification metrics and AMI/ARI require.
- Only a single train/test split is available and repeated-sampling robustness estimation is not the goal — a single point estimate does not yield the across-run variation this skill targets.

## Inputs

- AnnData (.h5ad) objects from scmodel.py predictions with adata.obs['sens_label'] (0 = resistant, 1 = sensitive)
- Predicted sensitivity labels for the held-out 20% cells from scDEAL trained with --checkpoint "False"
- Six benchmark scRNA-seq datasets: GSE117872 (HN120/HN137), GSE112274, GSE140440, GSE149383, GSE110894 with dataset-specific hyperparameters
- 20 random 80% stratified training subsamples per dataset

## Outputs

- Per-repeat values of F1-score, AUROC, average precision, precision, recall, AMI, and ARI per dataset
- Aggregated metric variation (standard deviation) across the 20 repeats per dataset (reference values: F1 0.031, AUROC 0.046, AP 0.027, precision 0.029, recall 0.031, AMI 0.156, ARI 0.198)

## How to apply

For each repeat, randomly subsample 80% of cells as training data (held-out test fraction = the model's default test_size of 0.2), retrain both the bulk-level model (bulkmodel.py) and the single-cell model (scmodel.py) from scratch with --checkpoint "False", using the dataset's recommended --sampling method (upsampling, SMOTE, or no) and hyperparameters (e.g. GSE110894: bottleneck 512, encoder dims 256,128, predictor dims 128,64, dropout 0.1, lr 0.5; GSE117872HN137: bottleneck 32, encoder 512,256, dropout 0.3, lr 0.01). Predict sensitivity on the held-out 20% cells and read the labels from adata.obs["sens_label"] (0 = resistant, 1 = sensitive) in the output AnnData (.h5ad) objects. Compute F1, AUROC, average precision, precision, recall, AMI, and ARI per repeat with scikit-learn metrics on predicted versus true labels, then aggregate the variation (standard deviation) across the 20 repeats per dataset. Accept the result when per-metric variation values reproduce the magnitudes reported in Source Data 6 (0.031, 0.046, 0.027, 0.029, 0.031, 0.156, 0.198) within reasonable numeric tolerance.

## Related tools

- **scDEAL** (DaNN-based deep transfer learning framework whose bulk- and single-cell-level models (bulkmodel.py, scmodel.py, trained with --checkpoint "False") produce the predicted sensitivity labels being evaluated) — https://github.com/OSU-BMBL/scDEAL.git
- **Scanpy** (Provides the AnnData (.h5ad) output objects from which adata.obs['sens_label'] labels are read for metric computation)
- **AnnData** (Container format (.h5ad) holding true and predicted sensitivity labels for the held-out cells)
- **PyTorch** (Backend on which the scDEAL bulk and single-cell models are retrained for each of the 20 sampling repeats)

## Examples

```
python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64" --dropout 0.1 --lr 0.5 --sampling "upsampling" --checkpoint "False"
```

## Evaluation signals

- Per-metric standard deviations over the 20 repeats match Source Data 6 magnitudes: F1 0.031, AUROC 0.046, AP 0.027, precision 0.029, recall 0.031, AMI 0.156, ARI 0.198 within reasonable numeric tolerance
- Every repeat uses exactly 80% of cells for training and 20% held out, consistent with scDEAL's default test_size of 0.2, with stratification preserved
- Both bulkmodel.py and scmodel.py are retrained per repeat with --checkpoint "False" (no pretrained weights leakage) and identical encoder/predictor dimensions across the two steps
- True labels are read exclusively from adata.obs['sens_label'] with the 0 = resistant, 1 = sensitive convention in the output AnnData objects
- Per-dataset recommended sampling methods (upsampling, SMOTE, or no) and hyperparameters from the methods table are applied for each dataset's runs

## Limitations

- Prediction accuracy depends on the collection of bulk gene expression of cell lines used in scDEAL training, which may vary and be updated with additional bulk-level databases in the future
- The number of experimentally validated drug response scRNA-seq benchmark datasets is limited, constraining hyperparameter determination
- AMI and ARI show notably larger variation across repeats (~0.156 and 0.198) than classification metrics (~0.027–0.046), so robustness claims should be metric-specific

## Evidence

- [results] Robustness variation values: [evidence span withheld — anchor status: not_found]
- [methods] Output label format: [evidence span withheld — anchor status: not_found]
- [readme] Train-from-scratch invocation: "python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64" --dropout 0.1 --printgene "F" -mod "new" --lr 0.5"
- [methods] Sampling method option: "Samping method of training data for the bulk model traning. Can be upsampling, downsampling, or SMOTE. default: no"
