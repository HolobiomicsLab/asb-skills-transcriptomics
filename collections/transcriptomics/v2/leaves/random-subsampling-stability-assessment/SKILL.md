---
name: random-subsampling-stability-assessment
description: Use when you must demonstrate that a supervised single-cell drug-response
  predictor's performance is not an artifact of a single train/test split, by measuring
  metric variation across repeated random stratified samplings of labeled scRNA-seq
  cells.
when_to_use_negative:
- When the goal is a single best point estimate of prediction accuracy on one fixed
  train/test split rather than quantifying split-to-split stability
- When the scRNA-seq data lack binary sensitivity/resistance labels (sens_label),
  since stratified sampling and label-based metrics cannot be computed
- When the task is de novo drug-response prediction on new cell types rather than
  robustness assessment of the already-benchmarked six scDEAL datasets
edam_topics:
- http://edamontology.org/topic_3306
tools:
- name: scDEAL
  role: Deep transfer learning framework (DaNN-based) whose bulk-level and single-cell-level
    models are retrained from scratch per subsampling repeat
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: Scanpy
  role: Package applied to the AnnData (.h5ad) output objects from which sensitivity
    labels and predictions are read
- name: AnnData
  role: Output format (.h5ad) holding predicted and true sens_label values for each
    repeat's held-out cells
- name: PyTorch
  role: Backend on which bulkmodel.py and scmodel.py training runs execute (CPU version
    used in the test case)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:3fec692b1413a2b7d7cc5de58eca3df5a3a19d7e367ebf0ad3ede5f2ec26a771
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - PyTorch
  - Scanpy
  - scDEAL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3306
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# random-subsampling-stability-assessment

## Summary

A robustness-testing skill that quantifies the stability of scDEAL's single-cell drug-response predictions by repeating 80% stratified random subsampling 20 times on the six benchmark scRNA-seq datasets and reporting metric variation (e.g. standard deviation) across repeats.

## When to use

Use when you must demonstrate that a supervised single-cell drug-response predictor's performance is not an artifact of a single train/test split, by measuring metric variation across repeated random stratified samplings of labeled scRNA-seq cells.

## When NOT to use

- When the goal is a single best point estimate of prediction accuracy on one fixed train/test split rather than quantifying split-to-split stability
- When the scRNA-seq data lack binary sensitivity/resistance labels (sens_label), since stratified sampling and label-based metrics cannot be computed
- When the task is de novo drug-response prediction on new cell types rather than robustness assessment of the already-benchmarked six scDEAL datasets

## Inputs

- Six drug-treated scRNA-seq datasets as .h5ad AnnData objects (GSE117872 HN120/HN137, GSE112274, GSE140440, GSE149383, GSE110894) from scDEAL.zip with data/ placed in the scDEAL root
- Per-dataset recommended hyperparameters (bottleneck, encoder dimensions, predictor dimensions, dropout, learning rate, sampling method) from the methods table
- GDSC/CCLE-derived bulk RNA-seq expression data bundled with scDEAL (ALL_expression.csv, ALL_label_binary_wf.csv) for bulk-level model training

## Outputs

- Per-repeat sensitivity predictions on the held-out 20% of cells (adata.obs["sens_label"], 0 = resistant, 1 = sensitive) in AnnData .h5ad outputs
- Per-repeat metric values: F1, AUROC, average precision (AP), precision, recall, AMI, ARI computed with scikit-learn
- Aggregated per-metric variation (standard deviation) across 20 repeats per dataset, comparable to Source Data 6 values (0.031, 0.046, 0.027, 0.029, 0.031, 0.156, 0.198)

## How to apply

Load each benchmark dataset (GSE117872 HN120/HN137, GSE112274, GSE140440, GSE149383, GSE110894) with its recommended hyperparameters from the methods table (e.g. GSE110894: bottleneck 512, encoder dims 256,128, predictor dims 128,64, dropout 0.1, lr 0.5, upsampling; GSE117872HN137: bottleneck 32, encoder 512,256, dropout 0.3, lr 0.01, upsampling). For each of 20 repeats, randomly subsample 80% of the cells as training data (held-out fraction consistent with the default test_size 0.2), then retrain both bulkmodel.py and scmodel.py from scratch with --checkpoint "False" using the dataset's recommended --sampling method (upsampling, SMOTE, or no). Predict sensitivity on the held-out 20% and read labels from adata.obs["sens_label"] (0 = resistant, 1 = sensitive) in the output AnnData objects. Compute F1, AUROC, average precision, precision, recall, AMI, and ARI per repeat with scikit-learn metrics, then aggregate the variation (standard deviation) across the 20 repeats per metric and dataset.

## Related tools

- **scDEAL** (Deep transfer learning framework (DaNN-based) whose bulk-level and single-cell-level models are retrained from scratch per subsampling repeat) — https://github.com/OSU-BMBL/scDEAL.git
- **Scanpy** (Package applied to the AnnData (.h5ad) output objects from which sensitivity labels and predictions are read)
- **AnnData** (Output format (.h5ad) holding predicted and true sens_label values for each repeat's held-out cells)
- **PyTorch** (Backend on which bulkmodel.py and scmodel.py training runs execute (CPU version used in the test case))

## Examples

```
python bulkmodel.py --drug "I.BET.762" --dimreduce "DAE" --encoder_h_dims "256,128" --predictor_h_dims "128,64" --bottleneck 512 --data_name "GSE110894" --sampling "upsampling" --dropout 0.1 --lr 0.5 --printgene "F" -mod "new" --checkpoint "False"
```

## Evaluation signals

- Per-metric standard deviations across the 20 repeats reproduce the Source Data 6 magnitudes (F1 0.031, AUROC 0.046, AP 0.027, precision 0.029, recall 0.031, AMI 0.156, ARI 0.198) within reasonable numeric tolerance
- Every repeat uses exactly 80% of cells for training and 20% held out, stratified by sens_label (0/1)
- Bulk and single-cell models are trained with --checkpoint "False" (from scratch) in all 20 repeats; encoder_h_dims/bulk_h_dims and predictor_h_dims match between bulkmodel.py and scmodel.py
- Predicted labels align with adata.obs["sens_label"] classes (0 = resistant, 1 = sensitive) so scikit-learn F1/AUROC/AP/precision/recall are well-defined
- Each dataset's run uses its recommended --sampling method (upsampling, SMOTE, or no) and hyperparameters from the methods table

## Limitations

- Prediction accuracy could vary depending on the collection of bulk gene expression of cell lines; scDEAL training data will be updated by integrating additional bulk-level databases in the future
- More experimentally validated drug response scRNA-seq data would be needed to determine better model hyperparameters
- A potential sample swap between Data 1 and 2 (HN120 and HN137) was pointed out by a recent study, which may confound GSE117872-based robustness estimates

## Evidence

- [readme] We can also train bulkmode.py and scmodel.py from scratch with user-defined parameters by setting --checkpoint "False":: "We can also train bulkmode.py and scmodel.py from scratch with user-defined parameters by setting --checkpoint "False":"
- [readme] The expected output format of scDEAL: "The expected output format of scDEAL is the [AnnData](https://anndata.readthedocs.io/en/latest/anndata.AnnData.html) object (.h5ad)"
- [methods] Samping method of training data: "Samping method of training data for the bulk model traning. Can be upsampling, downsampling, or SMOTE. default: no"
- [discussion] The accuracy of the prediction results in scDEAL could vary: "The accuracy of the prediction results in scDEAL could vary, depending on the collection of bulk gene expression of cell lines."
- [discussion] a potential sample swap between Data 1 and 2: "Noted that a recent study pointed out a potential sample swap between Data 1 and 2 (HN120 and HN137)49."
