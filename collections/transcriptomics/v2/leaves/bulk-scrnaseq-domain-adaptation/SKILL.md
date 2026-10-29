---
name: bulk-scrnaseq-domain-adaptation
description: Use when predicting per-cell drug response labels for a drug-treated
  scRNA-seq dataset by leveraging pre-trained bulk RNA-seq models built from GDSC/CCLE
  drug sensitivity databases.
when_to_use_negative:
- The input is bulk RNA-seq data only with no single-cell component — use bulkmodel.py
  training alone rather than domain transfer
- No bulk drug-sensitivity training data is available for the target drug, since transfer
  depends on GDSC/CCLE bulk response annotations
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scDEAL
  role: DaNN-based transfer learning framework that harmonizes bulk and single-cell
    embeddings and predicts per-cell drug response
  repo: https://github.com/OSU-BMBL/scDEAL.git
- name: PyTorch
  role: Backend for training the DAE/DaNN encoders at bulk and single-cell levels
- name: Scanpy
  role: Applied to the output AnnData object for downstream analysis of predicted
    labels
- name: AnnData
  role: Container format carrying the predicted sens_label field
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34277-7
    title: Deep transfer learning of cancer drug responses by integrating bulk and
      single-cell RNA-seq data
  source_content_hash: sha256:12bd63cd7a7df2825e9a184cb29267c7a66409b4eca9ba7621f368e77d8c0df5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OSU-BMBL/scDEAL.git
  tools:
  - AnnData
  - PyTorch
  - Scanpy
  - conda
  - scDEAL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# bulk-scrnaseq-domain-adaptation

## Summary

Train scDEAL (a DaNN-based deep transfer learning framework) on bulk cancer cell-line drug response data from GDSC/CCLE and transfer the encoder/predictor weights to predict single-cell drug sensitivity (sens_label 0=resistance, 1=sensitivity) in scRNA-seq datasets such as GSE110894.

## When to use

Use when predicting per-cell drug response labels for a drug-treated scRNA-seq dataset by leveraging pre-trained bulk RNA-seq models built from GDSC/CCLE drug sensitivity databases.

## When NOT to use

- The input is bulk RNA-seq data only with no single-cell component — use bulkmodel.py training alone rather than domain transfer
- No bulk drug-sensitivity training data is available for the target drug, since transfer depends on GDSC/CCLE bulk response annotations

## Inputs

- Bulk drug-response RNA-seq matrices and binary sensitivity labels from GDSC and CCLE (ALL_expression.csv, ALL_label_binary_wf.csv)
- Preprocessed drug-treated scRNA-seq dataset (e.g. GSE110894 .h5ad) placed in the scDEAL data directory
- Bulk-pretrained checkpoint .pkl from the save/bulk_pre directory

## Outputs

- AnnData (.h5ad) with per-cell predicted drug response in adata.obs["sens_label"] (0=resistance, 1=sensitivity)
- Trained single-cell model checkpoint saved as save/sc_pre/..._DaNN.pkl

## How to apply

First train the bulk-level model with bulkmodel.py (e.g. --drug "I.BET.762" --dimreduce "DAE" --encoder_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64" --dropout 0.1 --lr 0.5 --sampling "upsampling"), then run scmodel.py on the scRNA-seq dataset loading the bulk checkpoint via the --checkpoint option. Keep encoder and predictor dimensions identical between the bulk and single-cell steps. Use --mod "new" to add Leiden cell-cluster label regularization to the loss each epoch, preventing loss of scRNA-seq heterogeneity during domain transfer; Ablations showed transfer learning, integrated GDSC+CCLE training data, and DAE+cell-type regularization each raise average F1 (19%, 130%/69%, and 36%/9% relative gains). Extract predictions from adata.obs["sens_label"] in the output AnnData (.h5ad) and compute F1 against benchmark labels.

## Related tools

- **scDEAL** (DaNN-based transfer learning framework that harmonizes bulk and single-cell embeddings and predicts per-cell drug response) — https://github.com/OSU-BMBL/scDEAL.git
- **PyTorch** (Backend for training the DAE/DaNN encoders at bulk and single-cell levels)
- **Scanpy** (Applied to the output AnnData object for downstream analysis of predicted labels)
- **AnnData** (Container format carrying the predicted sens_label field)

## Examples

```
python scmodel.py --sc_data "GSE110894" --dimreduce "DAE" --drug "I.BET.762" --bulk_h_dims "256,128" --bottleneck 512 --predictor_h_dims "128,64" --dropout 0.1 --lr 0.5 --sampling "upsampling" --printgene "F" -mod "new" --checkpoint "save/sc_pre/integrate_data_GSE110894_drug_I.BET.762_bottle_512_edim_256,128_pdim_128,64_model_DAE_dropout_0.1_gene_F_lr_0.5_mod_new_sam_upsampling_DaNN.pkl"
```

## Evaluation signals

- Per-cell predictions must appear in adata.obs["sens_label"] coded 0 (resistance) / 1 (sensitivity)
- Ablation benchmark: relative F1 increases should approximate 19% (transfer learning), 130%/69% (integrated GDSC+CCLE over single-database), and 36%/9% (DAE+cell-type regularization over AE/DAE alone)
- The single-cell checkpoint filename must match the naming convention (dataset, drug, bottleneck, encoder dims, dropout, lr, mod, sampling) and dimensions must match between bulkmodel.py and scmodel.py runs
- The demo test case (GSE110894, I.BET.762) should complete bulk + single-cell training in about 20 minutes on the pytorch CPU version
- Replicate the run 50 times per arm and confirm F1 values fall in the 0-1 range before averaging

## Limitations

- Prediction accuracy depends on the collection of bulk cell-line gene expression; scDEAL training data will be updated with additional bulk databases
- Loss of heterogeneity can occur if cell-cluster label regularization (--mod "new") is omitted during transfer
- Encoder and predictor dimension arguments must be kept identical across bulkmodel.py and scmodel.py, otherwise checkpoints cannot be loaded
- No software version numbers are stated for tools or the environment, which may affect reproducibility

## Evidence

- [intro] scDEAL adapts DaNN for bulk-to-single-cell transfer: "we develop scDEAL (single-cell Drug rEsponse AnaLysis) by adapting a Domain-adaptive Neural Network (DaNN)12 to predict drug responses from bulk and scRNA-seq data."
- [intro] Cell-type regularization prevents heterogeneity loss: "in order to avoid losing heterogeneity in scRNA-seq data, scDEAL includes cell cluster labels for loss function regularization in each training epoch"
- [readme] Checkpoint option for resuming training: "For resuming training, you can use the --checkpoint option of scmodel.py and bulkmodel.py."
- [methods] Expected output format: "The expected output format of scDEAL is the [AnnData](https://anndata.readthedocs.io/en/latest/anndata.AnnData.html) object (.h5ad)"
