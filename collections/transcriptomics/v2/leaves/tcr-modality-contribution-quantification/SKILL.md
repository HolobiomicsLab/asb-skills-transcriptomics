---
name: tcr-modality-contribution-quantification
description: Use when a trained mvTCR joint model is available and you need cell-level
  attribution of TCR information in the shared embedding, or to check if modality
  contribution correlates with TCR sequence similarity across antigen-specificity
  groups.
when_to_use_negative:
- You want to adjust the modality contribution of the model itself — the article notes
  this requires repeated retraining, ideally with an additional hyperparameter search,
  not a post-hoc score
- Your data are unpaired or partially missing (mvTCR was mainly applied to fully paired
  data where gene expression is combined with α- and β-CDR3)
- You only need clonotype-level TCR-transcriptome association (e.g. CoNGA/scRepertoire
  workflows) rather than cell-level embedding attribution
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2830
tools:
- name: mvTCR
  role: Multi-view variational autoencoder trained to fuse transcriptome and TCR into
    a shared cell-level embedding; its joint and TCR-unimodal embeddings provide the
    inputs for the angular-distance contribution score
  repo: https://github.com/SchubertLab/mvTCR
- name: Scirpy
  role: Provides the TCRdist implementation used to compute TCR sequence similarity
    across antigen-specificity groups
- name: PyTorch
  role: Deep-learning backend for training and evaluating the mvTCR model (v1.8.0
    for reproducibility)
- name: Scanpy
  role: Single-cell data structures and workflow integration for the annotated AnnData
    holding embeddings and specificity labels
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:aa1addfef3bce9a7d682093a7a45279564c187ff71e231048f20bfebf55b00f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - PyTorch
  - Scanpy
  - Scirpy
  - mvTCR
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# tcr-modality-contribution-quantification

## Summary

Quantify how much TCR sequence information each individual cell contributes to the mvTCR joint embedding, by measuring the angular distance between the joint (multimodal) and TCR-unimodal embeddings, and test whether per-specificity average contributions track TCRdist-based sequence similarity.

## When to use

Use when a trained mvTCR joint model is available and you need cell-level attribution of TCR information in the shared embedding, or to check if modality contribution correlates with TCR sequence similarity across antigen-specificity groups.

## When NOT to use

- You want to adjust the modality contribution of the model itself — the article notes this requires repeated retraining, ideally with an additional hyperparameter search, not a post-hoc score
- Your data are unpaired or partially missing (mvTCR was mainly applied to fully paired data where gene expression is combined with α- and β-CDR3)
- You only need clonotype-level TCR-transcriptome association (e.g. CoNGA/scRepertoire workflows) rather than cell-level embedding attribution

## Inputs

- Preprocessed 10x single-cell dataset (>60k cells) with paired TCR CDR3 sequences and gene expression
- Trained mvTCR joint model (mvTCR v0.1.3, PyTorch v0.1.3-compatible, PyTorch v1.8.0) or pretrained model from Zenodo (doi:10.5281/zenodo.7215447)
- Antigen specificity annotations per cell (epitope labels such as GILGFVFTL, GLCTLVAML, IVTDFSVIK, ELAGIGILTV)

## Outputs

- Per-cell TCR-Contribution scores (angular distance between joint and TCR-unimodal embeddings)
- Per-specificity average TCR contributions (e.g. GILGFVFTL 55.4%, GLCTLVAML 54.8%, IVTDFSVIK 53.0%, ELAGIGILTV 51.8%)
- Pearson correlation coefficient between average TCR contribution and TCRdist similarity (expected r = 0.64)
- tcr_contribution_results.csv containing the per-specificity averages and the correlation coefficient

## How to apply

Train the joint mvTCR model (mvTCR v0.1.3 with PyTorch v1.8.0, GPU required) on the preprocessed 10x dataset (>60k cells) using the SchubertLab/mvTCR_reproducibility pipeline, or download the pretrained models from Zenodo (doi:10.5281/zenodo.7215447). Compute each cell's TCR-Contribution score as the angular distance between its joint embedding and its TCR-unimodal embedding — a larger angular distance implies the TCR modality contributed more to the fused representation. Stratify cells by antigen specificity (e.g. epitopes GILGFVFTL, GLCTLVAML, IVTDFSVIK, ELAGIGILTV) and average the per-cell scores within each group. Separately compute TCR sequence similarity across specificity groups with Scirpy's TCRdist implementation, then compute the Pearson correlation between average TCR contribution per specificity and TCRdist similarity. Save per-specificity averages and the correlation coefficient (expected r = 0.64) to tcr_contribution_results.csv.

## Related tools

- **mvTCR** (Multi-view variational autoencoder trained to fuse transcriptome and TCR into a shared cell-level embedding; its joint and TCR-unimodal embeddings provide the inputs for the angular-distance contribution score) — https://github.com/SchubertLab/mvTCR
- **Scirpy** (Provides the TCRdist implementation used to compute TCR sequence similarity across antigen-specificity groups)
- **PyTorch** (Deep-learning backend for training and evaluating the mvTCR model (v1.8.0 for reproducibility))
- **Scanpy** (Single-cell data structures and workflow integration for the annotated AnnData holding embeddings and specificity labels)

## Examples

```
conda create --name mvTCR python=3.8.8 -y && conda activate mvTCR && pip install mvtcr==0.1.3
```

## Evaluation signals

- Per-specificity average TCR contributions reproduce the reported values: GILGFVFTL 55.4%, GLCTLVAML 54.8%, IVTDFSVIK 53.0%, ELAGIGILTV 51.8%
- Pearson correlation between average TCR contribution and TCRdist similarity matches r = 0.64
- Per-cell contribution scores are valid angular distances (bounded, non-negative) between joint and TCR-unimodal embeddings
- tcr_contribution_results.csv is produced with per-specificity averages and the correlation coefficient
- Joint model reproduces published mvTCR behavior (embedding surpasses unimodal representations in antigen-specificity prediction)

## Limitations

- Adjusting the contribution of each modality requires repeated retraining of mvTCR, ideally with an additional hyperparameter search (technical limitation)
- The desired modality contribution is study-dependent and may vary across or even within one dataset depending on the analysis objective
- mvTCR was mainly applied to fully paired data (gene expression with α- and β-CDR3); handling of partially missing data is insufficiently addressed
- Only the CDR3 sequence is used for the TCR representation; VDJ-gene encoding and pre-trained TCR embedding models are not yet incorporated
- Reproduced models may diverge from publication models due to hardware/CUDA differences; pretrained models are provided on Zenodo, and UMAP visualization may not fully reproduce

## Evidence

- [abstract] we develop the multimodal generative model mvTCR to fuse modality-specific information across transcriptome and TCR into a shared representation: "we develop the multimodal generative model mvTCR to fuse modality-specific information across transcriptome and TCR into a shared representation"
- [results] The four highest average TCR contributions were observed for GILGFVFTL (55.4%), GLCTLVAML (54.8%), IVTDFSVIK (53.0%), and ELAGIGILTV (51.8%), and the average TCR-contribution per specificity correlates positively with TCRdist similarity (Pearson r = 0.64).: [evidence span withheld — anchor status: not_found]
- [discussion] As a technical limitation, adjusting the contribution of each modality requires repeated training of mvTCR, ideally by an additional hyperparameter search.: "As a technical limitation, adjusting the contribution of each modality requires repeated training of mvTCR, ideally by an additional hyperparameter search."
- [readme] The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0.: "The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
- [methods] 8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells): "8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)"
