---
name: rmse-model-comparison-benchmarking
description: Use when multiple trained deep learning variants (e.g. autoencoders of
  different widths or dropout settings) must be compared by reconstruction RMSE on
  held-out and independent datasets to pick the deployment model.
when_to_use_negative:
- The goal is cell-type abundance inference from bulk RNA-seq rather than model selection
  — use DCNet directly with the recommended dropout-0.1 setting instead of re-benchmarking
  variants
- Input data lacks the neuron-gene index or gene-cell relation matrix, so DCNet variants
  cannot be reconstructed for RMSE comparison
- The dataset is not a gene-expression matrix with human gene symbols as rows (e.g.
  methylation or mutation data), making reconstruction RMSE over the marker-gene layer
  meaningless
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0203
tools:
- name: DCNet
  role: Deconvolution network whose variants (Full-DCNet and dropout D0–D50) are trained
    and RMSE-benchmarked for model selection
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Python framework in which the AE64–AE4096 autoencoders and Full-DCNet variants
    are built and trained with adam optimizer, relu activation, L2 loss, lr 1e-4,
    600 iterations, batch 256, CPU
- name: Python
  role: Implementation language for training, RMSE evaluation, and writing rmse_metrics.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:6ac2fc5000c4b84221f6d94622a4ef7fb848c7894d259a7cfb9985859c9beb87
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  - mxnet
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# rmse-model-comparison-benchmarking

## Summary

Benchmark trained network variants (AE64–AE4096 autoencoders, Full-DCNet, and dropout-augmented DCNet-D0–D50) by computing RMSE on a held-out TCGA test split and an independent CRC single-cell dataset (GSE81861), then tabulate results to select the best model (AE2048 for autoencoders, dropout 0.1 for DCNet).

## When to use

Use when multiple trained deep learning variants (e.g. autoencoders of different widths or dropout settings) must be compared by reconstruction RMSE on held-out and independent datasets to pick the deployment model.

## When NOT to use

- The goal is cell-type abundance inference from bulk RNA-seq rather than model selection — use DCNet directly with the recommended dropout-0.1 setting instead of re-benchmarking variants
- Input data lacks the neuron-gene index or gene-cell relation matrix, so DCNet variants cannot be reconstructed for RMSE comparison
- The dataset is not a gene-expression matrix with human gene symbols as rows (e.g. methylation or mutation data), making reconstruction RMSE over the marker-gene layer meaningless

## Inputs

- TCGA bulk RNA-seq gene expression matrix (columns = samples, rows = human gene symbols), log2(x+1) transformed
- index.pkl mapping each DCNet input/output neuron to a specific gene
- relation_matrix.pkl zero-one gene-cell correspondence matrix
- Independent CRC single-cell dataset GSE81861 (11 colorectal cancer samples, 7 cell types)
- Trained model variants: AE64–AE4096 autoencoders, Full-DCNet, DCNet dropout variants D0–D50

## Outputs

- RMSE metrics table saved as rmse_metrics.csv with RMSE per variant on the TCGA test set and GSE81861 CRC set
- Selected best variant (AE2048 among autoencoders; DCNet with dropout 0.1 among dropout variants)

## How to apply

Load the TCGA gene expression matrix (columns = samples, rows = human gene symbols), apply the recommended log2(x+1) transformation, and split into 80% training / 20% test. Train the candidate variants with mxnet in Python using the adam optimizer, relu activation, L2 loss, learning rate 1e-4, 600 iterations, 256 samples per batch on CPU; for dropout variants D0–D50, randomly delete input data with probability 0.1, 0.3, or 0.5 (zero fill) for augmentation. Match each DCNet neuron to its specific gene using index.pkl and the zero-one gene-cell relation matrix. Evaluate RMSE of every variant on the TCGA test set and on the independent CRC dataset GSE81861 (11 colorectal cancer samples, 7 cell types), then save the comparison table as rmse_metrics.csv. The expected outcome is that RMSE decreases as autoencoder width grows from 64 to 4096 neurons, reaching its lowest at 2048 neurons, and dropout 0.1 is the recommended DCNet setting.

## Related tools

- **DCNet** (Deconvolution network whose variants (Full-DCNet and dropout D0–D50) are trained and RMSE-benchmarked for model selection) — https://github.com/xindd/DCNet
- **mxnet** (Python framework in which the AE64–AE4096 autoencoders and Full-DCNet variants are built and trained with adam optimizer, relu activation, L2 loss, lr 1e-4, 600 iterations, batch 256, CPU)
- **Python** (Implementation language for training, RMSE evaluation, and writing rmse_metrics.csv)

## Examples

```
import mxnet as mx; net = build_ae(hidden=2048); net.initialize(); trainer = mx.gluon.Trainer(net.collect_params(), 'adam', {'learning_rate': 1e-4}); train(net, X_train, epochs=600, batch_size=256); print(rmse(net, X_test))
```

## Evaluation signals

- AE2048 attains the lowest RMSE among the autoencoder variants (RMSE decreases from 64 up to 2048 neurons, then does not improve further at 4096)
- Dropout 0.1 is the recommended DCNet setting, matching the RMSE distributions reported in Figure 2a,b
- rmse_metrics.csv contains an RMSE row/column for every trained variant (AE64–AE4096, Full-DCNet, D0–D50) on both the TCGA test set and GSE81861
- The 80%/20% TCGA train/test split is fixed before any training so RMSE is computed on samples unseen during training
- Each DCNet input/output neuron is aligned to its specific gene via index.pkl before RMSE evaluation

## Limitations

- No software version numbers for mxnet or random seed are reported, so exact RMSE values may not be reproducible
- Trained model weights/checkpoints availability beyond the deposited DCNet params file is not stated; only source code is on GitHub
- Training was performed on CPU only with no hardware specification or runtime reported, so benchmarking may be slow for the AE4096 variant
- Which dropout setting was used for the final deployed model beyond the recommendation of 0.1 is not stated; fine-tuning on user sample data is recommended but unspecified

## Evidence

- [results] RMSE of the autoencoder decreases as the number of neurons increases from 64 to 4096, and reaches its lowest at 2048 neurons: [evidence span withheld — anchor status: not_found]
- [readme] A log2(x+1) transformation of the expression value is recommended.: [evidence span withheld — anchor status: not_found]
