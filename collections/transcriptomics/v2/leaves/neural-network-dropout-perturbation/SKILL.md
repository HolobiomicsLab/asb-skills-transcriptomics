---
name: neural-network-dropout-perturbation
description: Use when training DCNet (or a similar gene-embedded ANN) on bulk RNA-seq
  data and you need to test whether zero-fill input perturbation improves generalization,
  or when comparing autoencoder sizes vs. dropout variants by RMSE.
when_to_use_negative:
- Input is already a deconvolved cell-abundance/feature table rather than a gene-by-sample
  bulk RNA-seq matrix
- The marker-gene index files (index.pkl, relation_matrix.pkl) cannot be matched to
  the input gene symbols, so the neuron-gene embedding cannot be constructed
- The goal is deconvolution with other tools (CIBERSORTx, EPIC, MCP-counter, quanTIseq,
  TIMER, xCell) run under default parameters rather than training the DCNet architecture
  itself
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: mxnet
  role: Python framework used to build and train the AE64–AE4096, Full-DCNet, and
    DCNet-D0–D50 dropout variants with the specified optimizer/loss parameters
  repo: https://github.com/xindd/DCNet
- name: DCNet
  role: The gene-embedded neural network whose inputs are perturbed by random deletion
    (0.1/0.3/0.5, zero fill) to create dropout variants for RMSE-based selection
  repo: https://github.com/xindd/DCNet
- name: Python
  role: Language in which DCNet is coded and the demo pipeline (demo.ipynb) is executed
  repo: https://github.com/xindd/DCNet-Use/blob/main/demo.ipynb
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:16e71121730c9e6c4181912236179d25b5e0b92ba0bb4fc956749efb5187e9aa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  - mxnet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# neural-network-dropout-perturbation

## Summary

Augment DCNet training by randomly deleting input gene-expression values with probability 0.1, 0.3, or 0.5 (zero fill), then compare RMSE of the dropout variants (DCNet-D0–D50) against AE64–AE4096 autoencoders on the TCGA 80/20 split and the independent CRC set (GSE81861) to select the best model.

## When to use

Use when training DCNet (or a similar gene-embedded ANN) on bulk RNA-seq data and you need to test whether zero-fill input perturbation improves generalization, or when comparing autoencoder sizes vs. dropout variants by RMSE.

## When NOT to use

- Input is already a deconvolved cell-abundance/feature table rather than a gene-by-sample bulk RNA-seq matrix
- The marker-gene index files (index.pkl, relation_matrix.pkl) cannot be matched to the input gene symbols, so the neuron-gene embedding cannot be constructed
- The goal is deconvolution with other tools (CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, xCell) run under default parameters rather than training the DCNet architecture itself

## Inputs

- Bulk RNA-seq gene expression matrix (columns = samples, rows = human gene symbols), log2(x+1)-transformed
- index.pkl — gene set corresponding to DCNet input/output neurons
- relation_matrix.pkl — zero-one gene-cell correspondence matrix
- TCGA RNA-seq expression profiles with an 80%/20% train/test split
- Independent single-cell validation datasets, e.g. GSE81861 (CRC, 11 samples, 7 cell types)

## Outputs

- Trained DCNet dropout variant parameters (e.g. net_M434_adam_T600_DCNetBnoise-D1.params saved via net.save_parameters)
- rmse_metrics.csv — RMSE table for AE64–AE4096, Full-DCNet, and DCNet-D0–D50 on TCGA test and CRC sets
- Selected best model: AE2048 among autoencoders and DCNet with dropout 0.1 among DCNet variants

## How to apply

Prepare the expression matrix (columns = samples, rows = human gene symbols), apply the recommended log2(x+1) transformation, and split TCGA data 80%/20% into train/test. Load index.pkl and the zero-one gene-cell relation matrix so each DCNet neuron is matched to its specific gene, then build models in mxnet with adam optimizer, relu activation, L2 loss, learning rate 1e-4, 600 iterations, 256 samples per batch on CPU. Train the dropout variants by randomly deleting input data with probability 0.1, 0.3, and 0.5 (0 fill) as augmentation, alongside the AE64–AE4096 autoencoders and Full-DCNet baselines. Evaluate each variant by RMSE on the TCGA test set and the independent CRC dataset (GSE81861), tabulate results in rmse_metrics.csv, and select dropout 0.1 as the recommended DCNet setting, consistent with the reported Figure 2a,b distributions.

## Related tools

- **mxnet** (Python framework used to build and train the AE64–AE4096, Full-DCNet, and DCNet-D0–D50 dropout variants with the specified optimizer/loss parameters) — https://github.com/xindd/DCNet
- **DCNet** (The gene-embedded neural network whose inputs are perturbed by random deletion (0.1/0.3/0.5, zero fill) to create dropout variants for RMSE-based selection) — https://github.com/xindd/DCNet
- **Python** (Language in which DCNet is coded and the demo pipeline (demo.ipynb) is executed) — https://github.com/xindd/DCNet-Use/blob/main/demo.ipynb

## Examples

```
net.save_parameters('net_M434_adam_T600_DCNetBnoise-D1.params')
```

## Evaluation signals

- AE2048 attains the lowest RMSE among the AE64–AE4096 autoencoder variants, matching the reported trend that RMSE decreases as neuron count increases to 2048
- The dropout-0.1 DCNet variant shows the best (lowest) RMSE, consistent with the authors' recommendation of DCNet with dropout 0.1 and the Figure 2a,b distributions
- rmse_metrics.csv contains one RMSE row per variant evaluated on both the TCGA test set and the independent CRC set (GSE81861)
- Trained parameters are successfully saved via net.save_parameters as a file like net_M434_adam_T600_DCNetBnoise-D1.params
- Training config matches the reported settings: adam optimizer, relu activation, L2 loss, learning rate 1e-4, 600 iterations, 256 samples per batch, CPU

## Limitations

- No random seed or other reproducibility controls are reported for model training; training was on CPU only with no runtime or hardware specification
- Only source code is deposited — trained model weights/checkpoints are not guaranteed beyond the DCNet params file, and no software version numbers are reported for mxnet
- Which dropout probability was used for the final deployed DCNet model is not explicitly stated, and the fine-tuning procedure for applying dropout-0.1 DCNet to new user sample data is not specified

## Evidence
