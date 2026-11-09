---
name: tcr-transcriptome-integration-analysis
description: Use when paired scRNA-seq + TCR-seq data (e.g. 10x dataset with >60k
  cells) with specificity or pMHC multimer avidity annotations is available and the
  goal is antigen-specificity classification or pMHC read-count regression from a
  joint embedding.
when_to_use_negative:
- Data is unpaired or partially missing (no α/β CDR3 together with gene expression)
  — mvTCR was mainly applied to fully paired data
- Only a feature/expression matrix is needed and no TCR sequences or specificity/avidity
  labels exist — unimodal Scanpy workflows suffice
- B-cell receptor analysis — no large-scale B-cell datasets with specificity annotation
  exist to benchmark, per the discussion
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3934
tools:
- name: mvTCR
  role: Multiview VAE that fuses transcriptome and TCR CDR3 information into a shared
    cell-level embedding for downstream antigen and avidity prediction
  repo: https://github.com/SchubertLab/mvTCR
- name: mvTCR_reproducibility
  role: Repository with preprocessing, experiment settings, evaluation and figure
    notebooks to reproduce the paper's results with mvTCR v0.1.3 + PyTorch 1.8.0
  repo: https://github.com/SchubertLab/mvTCR_reproducibility
- name: PyTorch
  role: Deep learning backend used to train mvTCR (v1.8.0 with CUDA for reproduction;
    v2.1.2 in current install instructions)
- name: Scanpy
  role: Standard single-cell analysis format into which the mvTCR embedding integrates
    for clustering and workflow reuse
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:1558dbbca1852977cdd34e4d66c7d91e5fe8e9d763917175dcf7b1d73fdfa971
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - PyTorch
  - Scanpy
  - mvTCR
  - mvTCR_reproducibility
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tcr-transcriptome-integration-analysis

## Summary

Fuse single-cell T cell receptor (CDR3) sequence and gene expression data into a cell-level multimodal embedding using mvTCR, a multi-view variational autoencoder, enabling antigen specificity and binding avidity prediction that unimodal TCR-only or RNA-only embeddings cannot match.

## When to use

Use when paired scRNA-seq + TCR-seq data (e.g. 10x dataset with >60k cells) with specificity or pMHC multimer avidity annotations is available and the goal is antigen-specificity classification or pMHC read-count regression from a joint embedding.

## When NOT to use

- Data is unpaired or partially missing (no α/β CDR3 together with gene expression) — mvTCR was mainly applied to fully paired data
- Only a feature/expression matrix is needed and no TCR sequences or specificity/avidity labels exist — unimodal Scanpy workflows suffice
- B-cell receptor analysis — no large-scale B-cell datasets with specificity annotation exist to benchmark, per the discussion

## Inputs

- Preprocessed paired single-cell dataset (gene expression + α/β CDR3 TCR sequences) in Scanpy/MuData format, e.g. the 10x dataset (>60k cells)
- Cell-level specificity annotations (binder/non-binder) for antigen prediction
- pMHC multimer read counts as avidity labels for regression

## Outputs

- Cell-level multimodal embedding (joint TCR+RNA latent representation) per cell
- Predicted antigen specificity or pMHC multimer read counts per held-out cell
- Metrics table (e.g. avidity_metrics.csv) with MSLE and Pearson correlation per embedding type (mvTCR, RNA, TCR)
- Trained mvTCR model checkpoints

## How to apply

Preprocess the paired data following tutorials/01_preprocessing.ipynb (add required annotation and filter the cells), then train the mvTCR multiview VAE following tutorials/02_training_analysis.ipynb or, with specificity annotation, tutorials/03_training_prediction.ipynb. mvTCR fuses modality-specific information from the transcriptome and the TCR (currently CDR3 sequences only) into a shared cell-level representation. For downstream prediction, train a fully connected prediction head on the embedding — e.g. optimizing MSLE loss to regress pMHC multimer read counts — and evaluate on held-out cells. Reference values on the 10x dataset for avidity prediction are MSLE of 0.045 (TCR-only) and 0.063 (RNA-only), with the joint mvTCR embedding decreasing MSLE and raising Pearson correlation (TCR: 0.074; RNA: 0.028 gain over baseline). For reproduction, use mvTCR v0.1.3 with PyTorch v1.8.0 on a GPU-enabled machine (8GB RAM suffices for >60k cells), and pre-trained models are available on Zenodo (doi 10.5281/zenodo.7215447).

## Related tools

- **mvTCR** (Multiview VAE that fuses transcriptome and TCR CDR3 information into a shared cell-level embedding for downstream antigen and avidity prediction) — https://github.com/SchubertLab/mvTCR
- **mvTCR_reproducibility** (Repository with preprocessing, experiment settings, evaluation and figure notebooks to reproduce the paper's results with mvTCR v0.1.3 + PyTorch 1.8.0) — https://github.com/SchubertLab/mvTCR_reproducibility
- **PyTorch** (Deep learning backend used to train mvTCR (v1.8.0 with CUDA for reproduction; v2.1.2 in current install instructions))
- **Scanpy** (Standard single-cell analysis format into which the mvTCR embedding integrates for clustering and workflow reuse)

## Examples

```
conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr && conda install nb_conda_kernels -y
```

## Evaluation signals

- Avidity-prediction MSLE for the TCR-only embedding falls in 0.03–0.06 and RNA-only MSLE in 0.05–0.08, matching manuscript values (TCR 0.045, RNA 0.063)
- Joint mvTCR embedding yields lower MSLE and higher Pearson correlation than both unimodal embeddings (Pearson gains: TCR 0.074, RNA 0.028)
- Output metrics file contains one row per embedding type (mvTCR, RNA, TCR) with MSLE and Pearson correlation columns
- Training completes with 8GB RAM on a GPU machine for the preprocessed 10x dataset (>60k cells)
- Environment matches the reproduction setup: mvTCR v0.1.3, PyTorch v1.8.0, isolated conda environment

## Limitations

- mvTCR was mainly applied to fully paired data (gene expression with α- and β-CDR3); handling of partially missing data is insufficiently addressed
- Only CDR3 sequences are used for the TCR representation — VDJ-gene encoding and pre-trained TCR embedding models are not yet incorporated
- Adjusting modality contribution requires repeated retraining, ideally with an additional hyperparameter search, and the desired contribution is study-dependent
- Other modalities such as chromatin accessibility are not yet incorporated
- Hyperparameter optimization requires substantial compute (48 GPU-hours; single GPU with 32GB or a 4-GPU node with 512GB)

## Evidence

- [abstract] we develop the multimodal generative model mvTCR to fuse modality-specific information across transcriptome and TCR into a shared representation: "we develop the multimodal generative model mvTCR to fuse modality-specific information across transcriptome and TCR into a shared representation"
- [methods] For each of the three embeddings, train a fully connected prediction head optimized with mean squared logarithmic error (MSLE) loss to regress pMHC multimer read counts: "tutorials/03_training_prediction.ipynb : training the model, when you have specificity annotation."
- [results] The multimodal mvTCR embedding proved beneficial for avidity prediction, decreasing MSLE (TCR: 0.045; RNA: 0.063): [evidence span withheld — anchor status: not_found]
- [readme] The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0.: "The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
- [readme] 8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells): "8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)"
