---
name: angular-distance-embedding-analysis
description: Use when you have a trained mvTCR joint model with specificity annotations
  and need to quantify each cell's or epitope group's TCR contribution to the shared
  embedding, or to test whether that contribution tracks TCR sequence similarity.
when_to_use_negative:
- You only need cell-type or clonotype clustering from the joint embedding without
  quantifying modality-wise (TCR vs transcriptome) contributions
- Your data is unpaired or lacks antigen-specificity annotation, so per-specificity
  averaging and TCRdist group comparisons cannot be performed
- You want to adjust modality contribution in the model itself — the article notes
  this requires repeated mvTCR retraining with hyperparameter search, not post-hoc
  angular-distance analysis
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2830
tools:
- name: mvTCR
  role: Trains the multi-view VAE producing the joint cell-level embedding and the
    TCR-unimodal embedding whose angular distance yields TCR-contribution scores
  repo: https://github.com/SchubertLab/mvTCR
- name: Scirpy
  role: Provides the TCRdist implementation used to compute TCR sequence similarity
    across specificity groups
- name: Scanpy
  role: Hosts the mvTCR embedding in Scanpy-format for downstream stratification by
    antigen specificity
- name: PyTorch
  role: Deep-learning backend (v1.8.0) used to train the mvTCR models
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:fe727da33020617d4309012f945e9d259adbff6bf7d67f747e9113cbe617558d
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
yes_rate: 0.8333333333333334
n_claims: 6
---

# Angular-distance embedding analysis

## Summary

Quantifies how much TCR sequence information each cell contributes to the mvTCR joint embedding by computing the angular distance between the joint embedding and the TCR-unimodal embedding, then relates per-specificity average contributions to TCRdist sequence similarity (Pearson r = 0.64 in the mvTCR 10x dataset).

## When to use

Use when you have a trained mvTCR joint model with specificity annotations and need to quantify each cell's or epitope group's TCR contribution to the shared embedding, or to test whether that contribution tracks TCR sequence similarity.

## When NOT to use

- You only need cell-type or clonotype clustering from the joint embedding without quantifying modality-wise (TCR vs transcriptome) contributions
- Your data is unpaired or lacks antigen-specificity annotation, so per-specificity averaging and TCRdist group comparisons cannot be performed
- You want to adjust modality contribution in the model itself — the article notes this requires repeated mvTCR retraining with hyperparameter search, not post-hoc angular-distance analysis

## Inputs

- Trained mvTCR joint model and TCR-unimodal model checkpoints (mvTCR v0.1.3 / PyTorch v1.8.0) from the SchubertLab/mvTCR_reproducibility pipeline
- Preprocessed 10x dataset (>60k cells) with antigen-specificity (epitope) annotations, in Scanpy/MuData format
- TCR CDR3 sequences for TCRdist computation via Scirpy

## Outputs

- Per-cell TCR-Contribution scores (angular distance between joint and TCR-unimodal embeddings)
- Average TCR contribution per antigen specificity (e.g., GILGFVFTL 55.4%, GLCTLVAML 54.8%, IVTDFSVIK 53.0%, ELAGIGILTV 51.8%)
- Pearson correlation coefficient between average TCR contribution and TCRdist similarity (expected r = 0.64)
- tcr_contribution_results.csv containing per-specificity averages and the correlation coefficient

## How to apply

Train the joint mvTCR model (mvTCR v0.1.3 with PyTorch v1.8.0, GPU-enabled) on the preprocessed 10x dataset (>60k cells) from SchubertLab/mvTCR_reproducibility so transcriptome and TCR are fused into a shared cell-level embedding. For each cell, compute the TCR-Contribution score as the angular distance between the joint embedding vector and the TCR-unimodal embedding vector — a large angle means the joint representation deviates strongly from TCR-only information, quantifying modality mixing. Stratify cells by antigen specificity (epitopes such as GILGFVFTL, GLCTLVAML, IVTDFSVIK, ELAGIGILTV) and average the per-cell contributions within each group. Compute TCRdist-based sequence similarity across specificity groups using Scirpy's TCRdist implementation, then calculate the Pearson correlation between the per-specificity average TCR contribution and TCRdist similarity. Persist the per-specificity averages and the correlation coefficient (expected r = 0.64) to tcr_contribution_results.csv.

## Related tools

- **mvTCR** (Trains the multi-view VAE producing the joint cell-level embedding and the TCR-unimodal embedding whose angular distance yields TCR-contribution scores) — https://github.com/SchubertLab/mvTCR
- **Scirpy** (Provides the TCRdist implementation used to compute TCR sequence similarity across specificity groups)
- **Scanpy** (Hosts the mvTCR embedding in Scanpy-format for downstream stratification by antigen specificity)
- **PyTorch** (Deep-learning backend (v1.8.0) used to train the mvTCR models)

## Examples

```
import torch; z_joint = model.encode(adata); z_tcr = model.encode_tcr(adata); contrib = angular_distance(z_joint, z_tcr); adata.obs['tcr_contribution'] = contrib
```

## Evaluation signals

- Per-specificity average TCR contributions reproduce the reported values: GILGFVFTL 55.4%, GLCTLVAML 54.8%, IVTDFSVIK 53.0%, ELAGIGILTV 51.8%
- Pearson correlation between average TCR contribution and TCRdist similarity matches r = 0.64
- Per-cell angular distances are bounded (angular distance in [0, 2]) and defined for every cell in the >60k-cell dataset
- tcr_contribution_results.csv contains one row per epitope group plus the correlation coefficient
- Reproduced models may diverge from publication models across hardware/CUDA versions; deviations should be checked against the Zenodo-trained models (doi.org/10.5281/zenodo.7215447)

## Limitations

- Adjusting the contribution of each modality requires repeated mvTCR retraining, ideally with an additional hyperparameter search; the angular-distance score is a post-hoc diagnostic, not a control knob
- Reproduced models can diverge from publication models due to differing hardware, cluster runtimes, and CUDA versions, so exact metric reproduction is not guaranteed
- mvTCR was mainly applied to fully paired data (gene expression plus α- and β-CDR3); handling of partially missing TCR data remains insufficiently addressed, and only the CDR3 sequence is used for the TCR representation

## Evidence

- [results] Angular distance between joint and TCR-unimodal embeddings defines TCR contribution: [evidence span withheld — anchor status: not_found]
- [results] Expected per-specificity values and correlation: [evidence span withheld — anchor status: not_found]
- [results] Positive correlation with TCRdist similarity: [evidence span withheld — anchor status: not_found]
- [abstract] Fusing modalities into a shared representation: "we develop the multimodal generative model mvTCR to fuse modality-specific information across transcriptome and TCR into a shared representation"
- [readme] Reproducibility setup with pinned versions: [evidence span withheld — anchor status: external_doc]
- [discussion] Retraining needed to adjust modality contribution: "As a technical limitation, adjusting the contribution of each modality requires repeated training of mvTCR, ideally by an additional hyperparameter search."
