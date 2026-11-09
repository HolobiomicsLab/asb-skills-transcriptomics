---
name: bulk-to-single-cell-transfer-learning-model-training
description: Use when training scDEAL from scratch for a drug-treated scRNA-seq dataset
  (e.g. GSE110894 with I.BET.762) using the methods-table hyperparameters, instead
  of resuming a shipped checkpoint, to predict per-cell drug sensitivity.
when_to_use_negative:
- A pre-trained checkpoint already exists under save/ (e.g. ..._sam_upsampling_DaNN.pkl)
  and only predictions are needed — load it via --checkpoint instead of retraining
  from scratch.
- The goal is robustness benchmarking over 20 repeated 80% stratified samplings with
  per-repeat metric aggregation — this skill covers one from-scratch training run,
  not the resampling loop.
- No bulk-level response annotations for the target drug exist in the GDSC/CCLE training
  tables (ALL_label_binary_wf.csv), so the bulk predictor cannot be trained and its
  labels cannot be transferred.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDEAL
  role: DaNN deep transfer learning framework; bulkmodel.py trains the bulk-level
    predictor on GDSC/CCLE data and scmodel.py transfers it to predict scRNA-seq drug
    sensitivity
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: PyTorch
  role: Deep-learning backend on which both bulk-level and single-cell-level training
    run (CPU version, ~20 min for the GSE110894 test case)
- name: Scanpy
  role: Package applied to produce the .h5ad AnnData outputs of scDEAL
- name: AnnData
  role: Output format (.h5ad) carrying per-cell predicted sensitivity in adata.obs["sens_label"]
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:5fa44d8f722b6e6e216b59d94c8558702d6279f19f66b27cf20af609b0d55798
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - PyTorch
  - Scanpy
  - scDEAL
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# bulk-to-single-cell-transfer-learning-model-training

## Summary

Trains scDEAL's bulk-level and single-cell-level DaNN models from scratch (--checkpoint "False") on GDSC/CCLE bulk drug-response data and transfers them to predict per-cell drug sensitivity for drug-treated scRNA-seq benchmarks (GSE110894, GSE112274, GSE117872, GSE140440, GSE149383).

## When to use

Use when training scDEAL from scratch for a drug-treated scRNA-seq dataset (e.g. GSE110894 with I.BET.762) using the methods-table hyperparameters, instead of resuming a shipped checkpoint, to predict per-cell drug sensitivity.

## When NOT to use

- A pre-trained checkpoint already exists under save/ (e.g. ..._sam_upsampling_DaNN.pkl) and only predictions are needed — load it via --checkpoint instead of retraining from scratch.
- The goal is robustness benchmarking over 20 repeated 80% stratified samplings with per-repeat metric aggregation — this skill covers one from-scratch training run, not the resampling loop.
- No bulk-level response annotations for the target drug exist in the GDSC/CCLE training tables (ALL_label_binary_wf.csv), so the bulk predictor cannot be trained and its labels cannot be transferred.

## Inputs

- Bulk drug-response RNA-seq training data and binary response labels from GDSC/CCLE (data/ALL_expression.csv, data/ALL_label_binary_wf.csv shipped in scDEAL.zip)
- Drug-treated scRNA-seq dataset under data/<GSE accession> (e.g. GSE110894, GSE112274, GSE117872, GSE140440, GSE149383) plus its methods-table hyperparameters (drug, bottleneck, encoder/predictor dims, dropout, learning rate, sampling method)
- Cloned OSU-BMBL/scDEAL repository and the conda-packed scdeal.tar.gz environment (scDEALenv)

## Outputs

- Trained bulk-level model checkpoint in save/bulk_pre / save/models following the documented naming scheme (e.g. integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling)
- Trained single-cell DaNN checkpoint (..._DaNN.pkl) in save/sc_pre
- AnnData (.h5ad) prediction object with per-cell drug sensitivity labels in adata.obs["sens_label"] (0 = resistant, 1 = sensitive)

## How to apply

Clone OSU-BMBL/scDEAL, unpack the conda-pack environment (mkdir -p scDEALenv; tar -xzf scdeal.tar.gz -C scDEALenv; source scDEALenv/bin/activate), and place the extracted scDEAL.zip 'data' directory (ALL_expression.csv, ALL_label_binary_wf.csv, and the per-GSE folders) in the repo root. For a chosen dataset, run bulkmodel.py then scmodel.py with --checkpoint "False", keeping --encoder_h_dims/--bulk_h_dims, --bottleneck, and --predictor_h_dims identical across the two steps and taking the methods-table values (e.g. GSE110894: bottleneck 512, encoder 256,128, predictor 128,64, DAE, dropout 0.1, lr 0.5, --sampling "upsampling"; GSE117872HN120 uses SMOTE at dropout 0.3, lr 0.01). The bulk step trains the DaNN-based predictor on GDSC/CCLE bulk expression for the target drug (e.g. I.BET.762); the single-cell step transfers that knowledge to embed the scRNA-seq cells and predict sensitivity. Sampling options (upsampling, downsampling, SMOTE) counter class imbalance in the bulk training data, and --dimreduce "DAE" with -mod "new" matches the documented invocations. The GSE110894 test case including bulk-level and single-cell-level training takes about 20 minutes on the PyTorch CPU version.

## Related tools

- **scDEAL** (DaNN deep transfer learning framework; bulkmodel.py trains the bulk-level predictor on GDSC/CCLE data and scmodel.py transfers it to predict scRNA-seq drug sensitivity) — https://github.com/OSU-BMBL/scDEAL.git
- **PyTorch** (Deep-learning backend on which both bulk-level and single-cell-level training run (CPU version, ~20 min for the GSE110894 test case))
- **Scanpy** (Package applied to produce the .h5ad AnnData outputs of scDEAL)
- **AnnData** (Output format (.h5ad) carrying per-cell predicted sensitivity in adata.obs["sens_label"])

## Examples

```
python bulkmodel.py --drug "I.BET.762" --dimreduce "DAE" --encoder_h_dims "256,128" --predictor_h_dims "128,64" --bottleneck 512 --data_name "GSE110894" --sampling "upsampling" --dropout 0.1 --lr 0.5 --checkpoint "False"
```

## Evaluation signals

- Both steps exit with checkpoints written to save/bulk_pre and save/sc_pre (and save/models) using the documented name pattern integrate_data_<GSE>_<drug>_bottle_<n>_edim_..._sam_<method>_DaNN.pkl.
- --encoder_h_dims/--bulk_h_dims, --bottleneck, and --predictor_h_dims are identical between the bulkmodel.py and scmodel.py calls (e.g. 256,128 / 512 / 128,64 for GSE110894).
- scmodel.py output is an AnnData (.h5ad) whose adata.obs["sens_label"] contains binary 0/1 calls (0 = resistant, 1 = sensitive) for every input cell.
- The GSE110894 test case (bulk-level plus single-cell-level training) completes in roughly 20 minutes on the PyTorch CPU version.
- Held-out sensitivity metrics (F1, AUROC, AP, precision, recall, AMI, ARI) on the 20% test split fall in the benchmark range reported in Source Data 6 (per-metric variation 0.027-0.198 over 20 repeats).

## Limitations

- Prediction accuracy could vary depending on the collection of bulk gene expression of cell lines; scDEAL training data is planned to be updated by integrating additional bulk-level databases.
- A recent study pointed out a potential sample swap between Data 1 and 2 (HN120 and HN137), which undermines training/evaluation on the GSE117872HN120 and GSE117872HN137 labels.
- No software version numbers are stated for tools or the environment; for the mouse dataset (Data 6, GSE110894, Mus musculus) only homologous genes were kept and trans-species reliability was not systematically evaluated.

## Evidence

- [readme] Train-from-scratch checkpoint bypass: "We can also train bulkmode.py and scmodel.py from scratch with user-defined parameters by setting --checkpoint "False":"
- [readme] Encoder/predictor dimension consistency rule: "Remember that the dimension of the encoder and predictor should be identical (--encoder_h_dims(bulk_h_dims) "256,128", --predictor_h_dims "128,64", --bottleneck 256) in two steps."
- [methods] Bulk-level training invocation: "python bulkmodel.py --drug "I.BET.762" --dimreduce "DAE" --encoder_h_dims "256,128""
- [methods] AnnData .h5ad output format: "The expected output format of scDEAL is the [AnnData](https://anndata.readthedocs.io/en/latest/anndata.AnnData.html) object (.h5ad) applied by the scanpy package"
- [discussion] Accuracy depends on bulk data collection: "The accuracy of the prediction results in scDEAL could vary, depending on the collection of bulk gene expression of cell lines."
