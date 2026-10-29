---
name: multimodal-tcr-transcriptome-model-training
description: Use when you have paired scRNA-seq and TCR sequencing data (with cell
  type, clonotype, or antigen specificity annotations) and need a joint cell-level
  embedding for clustering, antigen specificity prediction, or atlas-query label transfer.
when_to_use_negative:
- Data are unimodal (TCR-only or expression-only) — mvTCR requires both modalities,
  and the paper notes it was mainly applied to fully paired data with alpha- and beta-CDR3
  plus gene expression
- Only a small exploratory dataset without GPU access — a machine with GPU- and CUDA-support
  is heavily encouraged and training is impractical on CPU-only hardware
- The goal is TCR sequence-only specificity prediction with pretrained TCR embeddings
  or VDJ-gene encoding — the paper states these are not yet incorporated; only the
  CDR3 sequence is currently used
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: mvTCR
  role: Multi-view VAE trained to fuse transcriptome and TCR into a shared cell-level
    embedding; installed via pip, pinned to v0.1.3 with PyTorch v1.8.0 for paper reproduction
  repo: https://github.com/SchubertLab/mvTCR
- name: scArches
  role: Architectural surgery to freeze reference weights and fine-tune query datasets
    into the reference latent space for kNN label transfer
- name: Scanpy
  role: Preprocessing, clustering of the joint embedding, and rank_genes_groups differential
    expression on Scanpy-format data
- name: Scirpy
  role: TCR preprocessing and annotation of the paired TCR sequences during the preprocessing
    tutorial stage
- name: PyTorch
  role: Deep-learning backend required for GPU training (v1.8.0 for reproducibility,
    v2.1.2 for current install)
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_006
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:90b7ab8e5a1e621bd01ee86269e6c39db5bcde045bb5034aa523b9d90b55a58b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - CoNGA
  - PyTorch
  - Scanpy
  - Scirpy
  - mvTCR
  - scArches
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# multimodal-tcr-transcriptome-model-training

## Summary

Train mvTCR, a multi-view Variational Autoencoder, to fuse modality-specific transcriptome and TCR (CDR3 sequence) information into a shared cell-level embedding for antigen specificity prediction, clustering, and reference mapping of single-cell immune profiling data.

## When to use

Use when you have paired scRNA-seq and TCR sequencing data (with cell type, clonotype, or antigen specificity annotations) and need a joint cell-level embedding for clustering, antigen specificity prediction, or atlas-query label transfer.

## When NOT to use

- Data are unimodal (TCR-only or expression-only) — mvTCR requires both modalities, and the paper notes it was mainly applied to fully paired data with alpha- and beta-CDR3 plus gene expression
- Only a small exploratory dataset without GPU access — a machine with GPU- and CUDA-support is heavily encouraged and training is impractical on CPU-only hardware
- The goal is TCR sequence-only specificity prediction with pretrained TCR embeddings or VDJ-gene encoding — the paper states these are not yet incorporated; only the CDR3 sequence is currently used

## Inputs

- Paired single-cell gene expression and TCR (alpha/beta CDR3) data in Scanpy/AnnData or MuData format
- Cell type, clonotype, and/or antigen specificity annotations (e.g., from 10x >60k-cell, Minervina, TIL, or SARS-CoV-2 PBMC datasets)
- Preprocessed reference atlas plus query datasets for scArches-based mapping

## Outputs

- Trained mvTCR model checkpoint (e.g., trained reference model for scArches surgery)
- Shared cell-level multimodal embedding for clustering, antigen specificity prediction (F1-Score), and clustering (NMI) evaluation
- Per-condition metric tables such as benchmark_comparison.csv or knn_label_transfer_metrics.csv

## How to apply

Preprocess paired scRNA-seq/TCR data in Scanpy/Scirpy following tutorials/01_preprocessing.ipynb, adding the required cell type, clonotype, and specificity annotations and applying the notebook's filtering. Install the pinned reproducibility stack (mvTCR v0.1.3 with PyTorch v1.8.0 in an isolated conda environment) on a GPU- and CUDA-enabled machine, then train with tutorials/02_training_analysis.ipynb (cell type + clonotype annotations) or tutorials/03_training_prediction.ipynb (specificity annotation); hyperparameter optimization in the paper ran 48 GPU-hours. For fair comparison against clonotype-level methods like tessa, restrict the TCR input to the CDR3β sequence only. For reference mapping, condition the model on study and cell-type annotations, freeze reference weights, and fine-tune on query studies via scArches architectural surgery, then transfer labels by k-nearest-neighbor voting in the reference embedding. Memory scales with dataset size — 8 GB RAM sufficed for the preprocessed 10x dataset of >60k cells — and trained models for the paper's figures are deposited on Zenodo (doi 10.5281/zenodo.7215447).

## Related tools

- **mvTCR** (Multi-view VAE trained to fuse transcriptome and TCR into a shared cell-level embedding; installed via pip, pinned to v0.1.3 with PyTorch v1.8.0 for paper reproduction) — https://github.com/SchubertLab/mvTCR
- **scArches** (Architectural surgery to freeze reference weights and fine-tune query datasets into the reference latent space for kNN label transfer)
- **Scanpy** (Preprocessing, clustering of the joint embedding, and rank_genes_groups differential expression on Scanpy-format data)
- **Scirpy** (TCR preprocessing and annotation of the paired TCR sequences during the preprocessing tutorial stage)
- **PyTorch** (Deep-learning backend required for GPU training (v1.8.0 for reproducibility, v2.1.2 for current install))

## Examples

```
conda create --name mvTCR python=3.10 -y && conda activate mvTCR && pip install mvtcr && conda install nb_conda_kernels -y
```

## Evaluation signals

- Antigen specificity prediction and clustering: mvTCR with CDR3β-only input should outperform tessa with average gains of ~23.6% F1-Score (p=4.7e-6) and ~27.7% NMI (p=4.6e-6) across datasets
- Reference mapping: conditional mvTCR + scArches kNN label transfer should exceed non-conditional projection by ~4.1% (cancer source) and ~15.2% (tissue type) on held-out lung cancer queries
- Training completes on GPU hardware with feasible resources (8 GB RAM for >60k cells) and tutorial notebooks finish within ~15 minutes on subsampled data
- Embedding yields biologically coherent clusters, e.g., five SARS-CoV-2-specific and three bystander CD8+ effector clusters with KLRD1/GZMB up in bystander and IL7R up in specific clusters
- Resulting figures match the publication when evaluation notebooks are run on preprocessed data with Zenodo-deposited models (UMAP may differ due to library reproducibility issues)

## Limitations

- mvTCR was mainly applied to fully paired data (gene expression plus alpha- and beta-CDR3); handling of partially missing data remains insufficiently addressed
- Only the CDR3 sequence is used for the TCR representation; VDJ-gene encoding and pre-trained TCR embedding models are not yet incorporated
- Adjusting the contribution of each modality requires repeated retraining, ideally with an additional hyperparameter search, and the desired modality contribution is study-dependent
- Other modalities such as chromatin accessibility are not yet incorporated
- Results may diverge across different compute clusters, hardware, and CUDA versions, so trained reference models should be fetched from Zenodo; UMAP visualizations may not fully reproduce due to underlying library reproducibility issues

## Evidence

- [abstract] we develop the multimodal generative model mvTCR to fuse modality-specific information across transcriptome and TCR into a shared representation: "we develop the multimodal generative model mvTCR to fuse modality-specific information across transcriptome and TCR into a shared representation"
- [methods] tutorials/01_preprocessing.ipynb : adding requiered annotation and filtering on the data.: "tutorials/01_preprocessing.ipynb : adding requiered annotation and filtering on the data."
- [readme] The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0.: "The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
- [discussion] When combined with tools like scArches, mvTCR adeptly mapped new multimodal studies into reference atlases: "When combined with tools like scArches, mvTCR adeptly mapped new multimodal studies into reference atlases"
- [readme] 8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells): "8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)"
