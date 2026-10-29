---
name: antigen-specificity-prediction-from-tcr-embeddings
description: Use when single-cell data has paired transcriptome and TCR sequences
  plus pMHC (e.g. SARS-CoV-2) specificity annotation and the goal is to predict or
  benchmark antigen specificity from cell-level embeddings.
when_to_use_negative:
- Data lack specificity annotation and the goal is clustering or reference mapping
  — use the training-analysis workflow (tutorials/02_training_analysis.ipynb) instead
- Data are unpaired or have missing α-/β-CDR3 chains — mvTCR was mainly applied to
  fully paired data and handling of partially missing data is insufficiently addressed
- Only a feature/embedding table already exists and no model training is needed —
  a simple kNN on existing embeddings suffices
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: mvTCR
  role: Multiview VAE (MoE) that fuses transcriptome and TCR into a shared cell-level
    embedding used for kNN specificity prediction
  repo: https://github.com/SchubertLab/mvTCR
- name: mvTCR_reproducibility
  role: Repository with preprocessing, experiment settings, evaluation, and figure
    notebooks to reproduce the paper's specificity-prediction results
  repo: https://github.com/SchubertLab/mvTCR_reproducibility
- name: Scanpy
  role: Standard single-cell workflow format into which mvTCR's representation integrates
    for downstream analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:b9aaabe9b72f7e1a2af4bfa1a4db05b6084efd86b1e9e24bebc5c3bc72466155
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - Scanpy
  - mvTCR
  - mvTCR_reproducibility
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# antigen-specificity-prediction-from-tcr-embeddings

## Summary

Train mvTCR (and unimodal TCR-only / RNA-only baselines) on specificity-annotated single-cell data, embed cells, and run a kNN classifier on the shared representation to predict pMHC antigen specificity, benchmarking multimodal vs unimodal F1-Score.

## When to use

Use when single-cell data has paired transcriptome and TCR sequences plus pMHC (e.g. SARS-CoV-2) specificity annotation and the goal is to predict or benchmark antigen specificity from cell-level embeddings.

## When NOT to use

- Data lack specificity annotation and the goal is clustering or reference mapping — use the training-analysis workflow (tutorials/02_training_analysis.ipynb) instead
- Data are unpaired or have missing α-/β-CDR3 chains — mvTCR was mainly applied to fully paired data and handling of partially missing data is insufficiently addressed
- Only a feature/embedding table already exists and no model training is needed — a simple kNN on existing embeddings suffices

## Inputs

- Preprocessed 10x Genomics single-cell dataset (>60k cells) with paired gene expression, TCR sequences, and pMHC specificity annotation (.h5ad / MuData)
- Environment with mvtcr (pip) and PyTorch (v2.1.2 with CUDA-matched build, per mvTCR README; the paper's experiments used mvTCR v0.1.3 + PyTorch v1.8.0)
- Random data splits for repeated training (five splits)

## Outputs

- Cell embeddings for each trained model (mvTCR MoE, TCR-only, RNA-only)
- Per-split kNN specificity-prediction F1-Scores per model and setting (pooled and per-donor)
- Aggregated mean F1-Score results table (specificity_f1_results.csv)

## How to apply

Preprocess the 10x Genomics dataset (>60k cells) following tutorials/01_preprocessing.ipynb, adding the required annotation and filtering. Train the mvTCR MoE model together with unimodal TCR-only and RNA-only baselines using the training-with-prediction setup of tutorials/03_training_prediction.ipynb, repeating each training over five random data splits in pooled and per-donor settings. Compute cell embeddings for each trained model and train a kNN classifier on the embeddings to predict pMHC specificity on the held-out split. Compute the F1-Score per model, split, and setting, aggregate the mean across five splits, and verify the expected ordering multimodal > TCR-only > RNA-only against reported values (mvTCR 0.821, TCR 0.767, RNA 0.759).

## Related tools

- **mvTCR** (Multiview VAE (MoE) that fuses transcriptome and TCR into a shared cell-level embedding used for kNN specificity prediction) — https://github.com/SchubertLab/mvTCR
- **mvTCR_reproducibility** (Repository with preprocessing, experiment settings, evaluation, and figure notebooks to reproduce the paper's specificity-prediction results) — https://github.com/SchubertLab/mvTCR_reproducibility
- **Scanpy** (Standard single-cell workflow format into which mvTCR's representation integrates for downstream analysis)

## Examples

```
conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr && conda install nb_conda_kernels -y
```

## Evaluation signals

- Aggregated F1-Scores fall within a small tolerance of the reported values: mvTCR 0.821, TCR-only 0.767, RNA-only 0.759
- Ordering of mean F1 preserves multimodal > TCR-only > RNA-only
- Difference vs unimodal baselines is significant: p-value 0.0051 vs TCR model and 0.0016 vs RNA model (five random splits)
- Each model/split/setting combination yields an F1-Score row in specificity_f1_results.csv
- Training environment matches the specified versions (mvtcr with PyTorch v2.1.2 per README; paper experiments used mvTCR v0.1.3 + PyTorch v1.8.0)

## Limitations

- mvTCR was mainly applied to fully paired data (gene expression with α- and β-CDR3); handling of partially missing data remains insufficiently addressed
- Adjusting the contribution of each modality requires repeated retraining, ideally with an additional hyperparameter search
- Only the CDR3 sequence is currently used for the TCR representation; VDJ-gene encoding and pre-trained TCR embedding models are not incorporated
- A machine with GPU- and CUDA-support is heavily encouraged; MAC with M1 chip has several unavailable dependencies; results may diverge across hardware/CUDA versions, so trained models are provided on Zenodo (doi 10.5281/zenodo.7215447)

## Evidence

- [methods] training with prediction task: "tutorials/03_training_prediction.ipynb : training the model, when you have specificity annotation."
- [methods] environment setup: "conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr"
- [methods] 10x dataset scale: "8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)"
- [abstract] multimodal advantage: "Our analysis demonstrates the added value of multimodal over unimodal approaches to capture antigen specificity"
- [readme] reproducibility setup: [evidence span withheld — anchor status: external_doc]
