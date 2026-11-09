---
name: lung-cancer-subtype-classification
description: Use when you have TCGA LUAD and LUSC bulk RNA-seq expression matrices
  and want to classify lung cancer subtype from differentially abundant tumor microenvironment
  cell types rather than raw gene expression.
when_to_use_negative:
- The goal is deconvolving cell types itself or comparing deconvolution methods (CIBERSORTx,
  EPIC, MCP-counter, quanTIseq, TIMER, xCell) — that is the upstream DCNet skill,
  not subtype classification
- Input is already a per-sample cell-type abundance/feature table — skip DCNet inference
  and train the SVM directly
- The task is pan-cancer patient stratification into survival subtypes (ConsensusClusterPlus
  clustering), not LUAD vs LUSC discrimination
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2640
tools:
- name: DCNet
  role: Pretrained deconvolution network that infers per-patient cell-type abundances
    from log2(x+1)-transformed bulk TCGA RNA-seq; its output supplies the classification
    features
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Python framework in which the DCNet model is implemented and run for abundance
    inference
  repo: https://github.com/xindd/DCNet
- name: Python
  role: Environment for running DCNet inference and training/evaluating the SVM subtype
    classifier
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:a095e6b331c8f2da42558b08afb7d2622f88cad36990841e01373d69632ea77d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  - mxnet
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# lung-cancer-subtype-classification

## Summary

Trains an SVM classifier on DCNet-inferred cell-type abundances of TCGA LUAD/LUSC patients to distinguish the two NSCLC subtypes, reproducing the paper's reported ROC AUC of 0.87 and precision of 0.975.

## When to use

Use when you have TCGA LUAD and LUSC bulk RNA-seq expression matrices and want to classify lung cancer subtype from differentially abundant tumor microenvironment cell types rather than raw gene expression.

## When NOT to use

- The goal is deconvolving cell types itself or comparing deconvolution methods (CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, xCell) — that is the upstream DCNet skill, not subtype classification
- Input is already a per-sample cell-type abundance/feature table — skip DCNet inference and train the SVM directly
- The task is pan-cancer patient stratification into survival subtypes (ConsensusClusterPlus clustering), not LUAD vs LUSC discrimination

## Inputs

- TCGA LUAD/LUSC bulk RNA-seq gene expression matrix (rows = human gene symbols, columns = samples, log2(x+1) transformed)
- index.pkl — set of genes corresponding to DCNet input and output neurons (from xindd/DCNet-Use)
- relation_matrix.pkl — zero-one gene-to-cell correspondence matrix
- net_M434_adam_T600_DCNetBnoise-D1.params — pretrained DCNet parameters (weights, biases)

## Outputs

- Per-patient DCNet cell-type abundance profiles (434 cell types)
- List of cell types significantly different between LUAD and LUSC
- SVM classifier trained on differential cell-type abundance features
- Held-out test-set metrics: ROC AUC (~0.87) and precision (~0.975), plus ROC curve plot (Figure 6d equivalent)

## How to apply

Load the TCGA LUAD/LUSC gene expression matrix (rows as human gene symbols, columns as samples) and apply the recommended log2(x+1) transformation. Run the pretrained DCNet model — an mxnet ANN with adam optimizer, relu activation, L2loss, learning rate 1e-4, 600 iterations, 256 samples per batch — using index.pkl (neuron-to-gene mapping) and the gene-cell relation matrix to infer cell-type abundances for each patient. Use statistical testing on the DCNet cell landscape to identify the cell types whose abundances are significantly different between LUAD and LUSC patients. Split patients into training and test sets at an 80%/20% ratio, train an SVM on the significantly different cell-type abundance features to predict subtype, and evaluate ROC AUC and precision on the held-out test set. Validation: the reproduced test-set ROC AUC should match the reported ~0.87 and SVM precision ~0.975 within reasonable tolerance; note the paper reports no random seed or version controls, so expect some variance.

## Related tools

- **DCNet** (Pretrained deconvolution network that infers per-patient cell-type abundances from log2(x+1)-transformed bulk TCGA RNA-seq; its output supplies the classification features) — https://github.com/xindd/DCNet
- **mxnet** (Python framework in which the DCNet model is implemented and run for abundance inference) — https://github.com/xindd/DCNet
- **Python** (Environment for running DCNet inference and training/evaluating the SVM subtype classifier)

## Examples

```
git clone https://github.com/xindd/DCNet.git && cd DCNet && pip install -r requirements.txt && python setup.py install
```

## Evaluation signals

- Held-out (20%) test-set ROC AUC is approximately 0.87, matching the paper's reported value within reasonable tolerance
- SVM precision on the test set is approximately 0.975
- DCNet was run with the published hyperparameters (adam optimizer, relu activation, L2loss, learning rate 1e-4, 600 iterations, 256 samples per batch) and expression inputs log2(x+1)-transformed
- The feature set fed to the SVM contains only cell types flagged as significantly different between LUAD and LUSC in the DCNet cell landscape
- 80%/20% train/test split is applied before training so evaluation is on held-out patients

## Limitations

- No software version numbers, random seed, or reproducibility controls are reported for mxnet or model training; CPU-only training with no runtime specified
- Only source code is deposited on GitHub — trained model weights/checkpoints availability beyond the .params file is not fully documented, and the fine-tuning procedure for the dropout-0.1 DCNet on user data is not specified
- Which gene-dropout augmentation setting (0.1/0.3/0.5) was used for the final model is not stated

## Evidence
