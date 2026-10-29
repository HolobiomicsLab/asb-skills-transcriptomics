---
name: autoencoder-variant-training
description: Use when you must choose the hidden-layer width of an autoencoder or
  a dropout-augmentation strength for DCNet by comparing train/test RMSE on log2(x+1)-transformed
  TCGA expression data.
when_to_use_negative:
- Your goal is to run DCNet on new samples for cell-landscape inference using the
  pretrained parameters rather than retraining/selecting model variants.
- Your input is not a gene-symbol-by-sample expression matrix (e.g. already a feature/cell-abundance
  table) or cannot be log2(x+1)-transformed.
- You need hyperparameter tuning beyond the fixed adam/relu/L2loss, lr 1e-4, 600-iteration,
  batch-256 CPU recipe; the paper reports no seed or tuning controls.
edam_operation: http://edamontology.org/operation_3983
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: mxnet
  role: Framework in which the AE64–AE4096 and DCNet dropout variants are built and
    trained (adam, relu, L2loss, lr 1e-4, 600 iterations, batch 256, CPU).
- name: Python
  role: Implementation language for the model-variant training and RMSE evaluation
    scripts.
- name: DCNet
  role: The interpretable ANN embedding 434 cell types and marker genes whose AE and
    dropout variants are trained and compared by RMSE.
  repo: https://github.com/xindd/DCNet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:d58322113c6fd5d1b749de389e79e78fe8e08ccee3db87479e3fe250196f783e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  - mxnet
  edam_operation: http://edamontology.org/operation_3983
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# autoencoder-variant-training

## Summary

Trains a sweep of autoencoder variants (AE64–AE4096) and dropout-augmented DCNet variants (D0–D50) on TCGA bulk RNA-seq to select the best model by RMSE, anchoring the DCNet architecture in the AE with the lowest test error (AE2048).

## When to use

Use when you must choose the hidden-layer width of an autoencoder or a dropout-augmentation strength for DCNet by comparing train/test RMSE on log2(x+1)-transformed TCGA expression data.

## When NOT to use

- Your goal is to run DCNet on new samples for cell-landscape inference using the pretrained parameters rather than retraining/selecting model variants.
- Your input is not a gene-symbol-by-sample expression matrix (e.g. already a feature/cell-abundance table) or cannot be log2(x+1)-transformed.
- You need hyperparameter tuning beyond the fixed adam/relu/L2loss, lr 1e-4, 600-iteration, batch-256 CPU recipe; the paper reports no seed or tuning controls.

## Inputs

- TCGA bulk RNA-seq gene expression matrix (columns = samples, rows = human gene symbols, log2(x+1)-transformed)
- index.pkl neuron-gene index from xindd/DCNet-Use
- relation_matrix.pkl zero-one gene-cell correspondence matrix
- Independent validation expression matrix, e.g. GSE81861 CRC single-cell-derived bulk data

## Outputs

- rmse_metrics.csv tabulating RMSE for AE64–AE4096, Full-DCNet, and DCNet dropout variants D0–D50
- Trained mxnet model parameters for each variant
- Selection result: AE2048 as best autoencoder; dropout 0.1 as recommended DCNet

## How to apply

Load the gene expression matrix with each column a sample and each row a human gene symbol, apply the recommended log2(x+1) transformation, then split the TCGA data 80%/20% into training and test sets. In Python with the mxnet framework, build AE64–AE4096 autoencoders and the Full-DCNet using the adam optimizer, relu activation, L2loss, learning rate 1e-4, 600 iterations, 256 samples per batch, on CPU. Train dropout variants DCNet-D0 through D50 by randomly deleting input genes with probability 0.1, 0.3, or 0.5 (0 fill) for augmentation; the input layer is constrained by the index.pkl neuron-gene mapping and the zero-one gene-cell relation matrix so each neuron matches one marker gene. Evaluate every variant by RMSE on the TCGA test set and on the independent CRC set GSE81861, tabulate the values in rmse_metrics.csv, and select AE2048 (lowest AE RMSE) and dropout 0.1 as the recommended DCNet setting.

## Related tools

- **mxnet** (Framework in which the AE64–AE4096 and DCNet dropout variants are built and trained (adam, relu, L2loss, lr 1e-4, 600 iterations, batch 256, CPU).)
- **Python** (Implementation language for the model-variant training and RMSE evaluation scripts.)
- **DCNet** (The interpretable ANN embedding 434 cell types and marker genes whose AE and dropout variants are trained and compared by RMSE.) — https://github.com/xindd/DCNet

## Examples

```
python -c "import mxnet as mx; net=mx.gluon.nn.SymbolBlock.imports('net-symbol.json','data','net_M434_adam_T600_DCNetBnoise-D1.params')"
```

## Evaluation signals

- AE2048 attains the lowest RMSE among the autoencoder variants (RMSE decreases as neurons go from 64 up to 2048).
- Dropout 0.1 is the recommended DCNet setting, matching the reported Figure 2a,b RMSE distributions.
- rmse_metrics.csv contains one RMSE per variant for both the TCGA 20% test split and the GSE81861 CRC set.
- TCGA train/test split matches the 80%/20% ratio reported in the paper.
- Retained gene set after filtering corresponds to the 21,136 genes kept (genes with zero expression in >1/3 of samples removed).

## Limitations

- No software version numbers for mxnet, no random seed, and no other reproducibility controls are reported for model training.
- Trained model weights or checkpoints are not stated as available in the paper; only the pretrained DCNet params are deposited on GitHub.
- Training was CPU-only with no hardware specification or runtime reported; which dropout setting was used for the final model is not explicitly stated in the text.
