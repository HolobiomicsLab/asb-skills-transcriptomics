---
name: conditional-reference-atlas-training
description: Use when a new paired TCR + transcriptome query dataset must be mapped
  into an existing T cell reference atlas and labels (e.g., cancer source, tissue
  type) transferred without retraining the full reference model.
when_to_use_negative:
- Data lacks paired TCR sequences and gene expression (mvTCR was applied mainly to
  paired data; partially missing data handling is insufficient)
- The goal is de novo clustering or specificity prediction rather than mapping queries
  into an existing reference atlas
- No GPU/CUDA machine is available — training and scArches fine-tuning require GPU
  support
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3168
tools:
- name: mvTCR
  role: Multi-view VAE that fuses transcriptome and TCR into a shared cell-level embedding;
    trained conditionally on study and cell-type annotations
  repo: https://github.com/SchubertLab/mvTCR
- name: scArches
  role: Architectural surgery to freeze reference weights and fine-tune query studies
    into the reference latent space
- name: Scanpy
  role: Scanpy-format data structures host the mvTCR embedding for downstream clustering
    and label transfer workflows
- name: PyTorch
  role: Deep learning backend (v1.8.0 pinned for reproducibility with mvTCR v0.1.3)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:ac6199b2452b2663805e5fe684424d0fbdbf2a43eda6ae28e8a3a22f48d6f2bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - PyTorch
  - Scanpy
  - mvTCR
  - scArches
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# conditional-reference-atlas-training

## Summary

Train a conditional mvTCR multi-view VAE on a large TIL reference atlas (conditioned on study and cell-type annotations), then use scArches architectural surgery to map held-out query studies into the reference latent space for kNN label transfer of cancer source, tissue type, and cell type.

## When to use

Use when a new paired TCR + transcriptome query dataset must be mapped into an existing T cell reference atlas and labels (e.g., cancer source, tissue type) transferred without retraining the full reference model.

## When NOT to use

- Data lacks paired TCR sequences and gene expression (mvTCR was applied mainly to paired data; partially missing data handling is insufficient)
- The goal is de novo clustering or specificity prediction rather than mapping queries into an existing reference atlas
- No GPU/CUDA machine is available — training and scArches fine-tuning require GPU support

## Inputs

- Preprocessed paired TCR + gene expression reference atlas (12-study TIL reference with cell type, cancer source, tissue type annotations)
- Held-out query studies (e.g., two lung cancer query datasets) with paired TCR-seq and scRNA-seq
- Cell type, cancer source, and study annotation columns

## Outputs

- Trained conditional mvTCR reference model checkpoint
- Query cell embeddings in the reference latent space
- kNN-transferred labels for cancer source, tissue type, and cell type per query cell
- knn_label_transfer_metrics.csv with conditional vs. non-conditional accuracy per condition

## How to apply

Preprocess the reference (e.g., the 12-study TIL atlas from SchubertLab/mvTCR_reproducibility) with tutorials/01_preprocessing.ipynb to add cell type, cancer source, and tissue type annotations. Train a conditional mvTCR model (mvTCR v0.1.3 with PyTorch v1.8.0) using tutorials/02_training_analysis.ipynb on a GPU machine, conditioning on study and cell-type covariates. Freeze the reference weights and fine-tune on the held-out query studies via scArches architectural surgery, mapping query cells into the reference latent space. Transfer labels by k-nearest-neighbor voting in the reference embedding, once with the conditional model and once with a non-conditional projection baseline, and compute per-condition accuracy improvements.

## Related tools

- **mvTCR** (Multi-view VAE that fuses transcriptome and TCR into a shared cell-level embedding; trained conditionally on study and cell-type annotations) — https://github.com/SchubertLab/mvTCR
- **scArches** (Architectural surgery to freeze reference weights and fine-tune query studies into the reference latent space)
- **Scanpy** (Scanpy-format data structures host the mvTCR embedding for downstream clustering and label transfer workflows)
- **PyTorch** (Deep learning backend (v1.8.0 pinned for reproducibility with mvTCR v0.1.3))

## Examples

```
conda create --name mvTCR python=3.8.8 -y && conda activate mvTCR && pip install mvtcr==0.1.3 && conda install nb_conda_kernels -y
```

## Evaluation signals

- Conditional model's kNN label-transfer accuracy on both held-out lung cancer queries exceeds the non-conditional projection baseline
- Observed improvement approximates the reported gains of 4.1% for cancer source and 15.2% for tissue type within a small tolerance
- Cell type annotation accuracy is comparable between conditional and non-conditional models (the paper reports similar performance)
- Per-condition metrics are written to knn_label_transfer_metrics.csv alongside the trained reference checkpoint
- Reproduced embedding recovers reference structure; note UMAP may not be fully recreated due to reproducibility issues in the underlying library

## Limitations

- mvTCR was applied mainly to fully paired data (gene expression + α- and β-CDR3); handling of partially missing modalities is insufficiently addressed
- Only the CDR3 sequence is used for the TCR representation; VDJ-gene encoding and pre-trained TCR embeddings are not yet incorporated
- Adjusted modality contribution requires repeated retraining, ideally with an additional hyperparameter search; HPO took 48 GPU-hours per dataset
- Trained models may diverge from published ones across hardware/CUDA versions; pre-trained models are provided via Zenodo (doi:10.5281/zenodo.7215447)

## Evidence

- [discussion] When combined with tools like scArches, mvTCR adeptly mapped new multimodal studies into reference atlases: "When combined with tools like scArches, mvTCR adeptly mapped new multimodal studies into reference atlases"
- [readme] Experiments pinned to mvTCR v0.1.3 and PyTorch v1.8.0: "The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
- [methods] Training tutorial for models with cell type and clonotype annotations: "tutorials/02_training_analysis.ipynb : training the model, when you have some cell type and clonotype annotation."
- [readme] GPU requirement for training and HPO: "To train mvTCR a machine with GPU support is required."
- [discussion] Missing modality limitation: "Yet, we applied it mainly to paired data, where gene expression is available in combination with α- and β-CDR3 of the TCR."
