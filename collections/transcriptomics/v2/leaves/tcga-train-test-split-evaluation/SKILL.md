---
name: tcga-train-test-split-evaluation
description: Use when you need to reproduce or extend the RMSE comparison of DCNet/autoencoder
  cell-landscape variants across a TCGA train/test split and an independent single-cell
  validation set such as GSE81861.
when_to_use_negative:
- The goal is cell abundance estimation with a pretrained model rather than model
  selection — use the DCNet-Use demo workflow with net_M434_adam_T600_DCNetBnoise-D1.params
  instead of retraining variants.
- Input expression data is not bulk RNA-seq (e.g. already a cell-fraction feature
  table), so RMSE-based reconstruction comparison of network variants is not meaningful.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3360
tools:
- name: DCNet
  role: Deep learning framework embedding 434 cell types and marker-gene relationships
    in a neural network; its variants (AE, Full-DCNet, dropout variants) are the models
    compared by RMSE in this skill.
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Python framework used to code and train the AE64–AE4096 autoencoders and DCNet
    variants with adam optimizer, relu activation, L2 loss, lr 1e-4, 600 iterations,
    batch 256 on CPU.
  repo: https://github.com/xindd/DCNet
- name: Python
  role: Language in which DCNet model training, 80/20 TCGA splitting, dropout augmentation,
    and RMSE evaluation are implemented.
  repo: https://github.com/xindd/DCNet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:f2484098529bb2b80c278977e951321a5382f05a360517c5715593bdbd3bdc52
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  - mxnet
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# tcga-train-test-split-evaluation

## Summary

Reproduces the RMSE-based model-selection workflow of the DCNet paper, in which AE64–AE4096 autoencoders, Full-DCNet, and dropout-augmented DCNet variants (D0–D50) are trained on an 80/20 TCGA split with mxnet and evaluated by RMSE on the TCGA test set and the independent CRC dataset GSE81861.

## When to use

Use when you need to reproduce or extend the RMSE comparison of DCNet/autoencoder cell-landscape variants across a TCGA train/test split and an independent single-cell validation set such as GSE81861.

## When NOT to use

- The goal is cell abundance estimation with a pretrained model rather than model selection — use the DCNet-Use demo workflow with net_M434_adam_T600_DCNetBnoise-D1.params instead of retraining variants.
- Input expression data is not bulk RNA-seq (e.g. already a cell-fraction feature table), so RMSE-based reconstruction comparison of network variants is not meaningful.

## Inputs

- TCGA gene expression matrix (columns = samples, rows = human gene symbols), log2(x+1)-transformed
- index.pkl neuron-gene index from xindd/DCNet-Use
- relation_matrix.pkl zero-one gene-cell correspondence matrix
- Independent CRC single-cell dataset GSE81861 (11 colorectal cancer samples, 7 cell types)

## Outputs

- rmse_metrics.csv tabulating RMSE for every variant (AE64–AE4096, Full-DCNet, DCNet-D0–D50) on TCGA test set and GSE81861
- Trained mxnet model parameters (e.g. net_M434_adam_T600_DCNetBnoise-D1.params)
- RMSE comparison distributions matching Figure 2a,b

## How to apply

Load a gene expression matrix whose columns are samples and rows are human gene symbols, apply the recommended log2(x+1) transformation, and split TCGA expression data into 80% training and 20% test sets. Load index.pkl and the zero-one gene-cell relation matrix so each DCNet neuron maps to its specific marker gene, then build AE64–AE4096 autoencoders and Full-DCNet in mxnet using the adam optimizer, relu activation, L2 loss, learning rate 1e-4, 600 iterations, and 256 samples per batch on CPU. Train dropout variants D0–D50 by randomly deleting input data with probability 0.1, 0.3, or 0.5 (zero fill) for augmentation. Evaluate each variant by RMSE on the TCGA test set and on the independent CRC set GSE81861, then tabulate all RMSE values and save as rmse_metrics.csv. Correct application is confirmed when AE2048 attains the lowest RMSE among the autoencoder variants and dropout 0.1 emerges as the recommended DCNet setting, matching the Figure 2a,b distributions.

## Related tools

- **DCNet** (Deep learning framework embedding 434 cell types and marker-gene relationships in a neural network; its variants (AE, Full-DCNet, dropout variants) are the models compared by RMSE in this skill.) — https://github.com/xindd/DCNet
- **mxnet** (Python framework used to code and train the AE64–AE4096 autoencoders and DCNet variants with adam optimizer, relu activation, L2 loss, lr 1e-4, 600 iterations, batch 256 on CPU.) — https://github.com/xindd/DCNet
- **Python** (Language in which DCNet model training, 80/20 TCGA splitting, dropout augmentation, and RMSE evaluation are implemented.) — https://github.com/xindd/DCNet

## Examples

```
python demo.ipynb  # train AE/DCNet variants on 80/20 TCGA split, evaluate RMSE, save rmse_metrics.csv
```

## Evaluation signals

- AE2048 attains the lowest RMSE among the autoencoder variants (RMSE decreases as neurons increase from 64 to 2048, then does not improve at 4096).
- Dropout 0.1 is the recommended DCNet setting among D0–D50 variants.
- rmse_metrics.csv contains one RMSE entry per variant for both the TCGA test set and GSE81861, reproducing the Figure 2a,b distributions.
- Train/test split ratio is exactly 80%/20% of TCGA samples, and training used adam/relu/L2 loss, lr 1e-4, 600 iterations, 256-sample batches on CPU.
- Input matrix schema check: columns are samples, rows are human gene symbols, with log2(x+1) transformation applied before training.

## Limitations

- No software version numbers for mxnet, no random seed, and no reproducibility controls for model training are reported.
- Only source code is deposited; trained model weights/checkpoints for the variant comparison are not stated as available.
- Training was performed on CPU only with no hardware specification or runtime reported.
- Which dropout setting was used in the final production model is not stated, though dropout 0.1 is recommended for user fine-tuning.
- All comparison methods were run with default parameters without tuning; only 21,136 genes were retained after filtering (genes with zero expression in more than 1/3 of TCGA samples removed).

## Evidence

- [results] RMSE of the autoencoder decreases as the number of neurons increases from 64 to 4096, and reaches its lowest at 2048 neurons: [evidence span withheld — anchor status: not_found]
