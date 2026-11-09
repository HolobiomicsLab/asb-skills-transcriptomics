---
name: pearson-correlation-validation
description: Use when you have per-cell TCR-contribution scores from a trained mvTCR
  model plus TCRdist similarities per specificity group, and you must validate that
  TCR contribution quantitatively tracks TCR sequence similarity across epitope groups.
when_to_use_negative:
- Your embedding does not separate joint and TCR-unimodal views (e.g. you only have
  a clonotype-level integration), since angular distance between modalities is undefined
- Data lack antigen specificity annotations and TCR sequences usable for Scirpy's
  TCRdist computation
- Only CDR3-based TCR representation with partially missing paired data is available,
  which the authors note was insufficiently addressed (mvTCR was applied mainly to
  fully paired data)
tools:
- name: mvTCR
  role: Multiview VAE producing the joint and TCR-unimodal embeddings whose angular
    distance yields per-cell TCR-contribution scores
  repo: https://github.com/SchubertLab/mvTCR
- name: Scirpy
  role: Provides the TCRdist implementation used to compute TCR sequence similarity
    across specificity groups
- name: Scanpy
  role: Single-cell data format and workflow in which the mvTCR embedding and per-cell
    contributions are computed and analyzed
- name: PyTorch
  role: Deep-learning backend (v1.8.0) used to train the mvTCR models whose embeddings
    feed the correlation analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:f583e84b6763fdba3fd18d1702af79d198f62615f709355ceabfc7cd66a47728
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - PyTorch
  - Scanpy
  - Scirpy
  - mvTCR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Pearson correlation validation of per-specificity TCR-contribution against TCRdist similarity

## Summary

Validation procedure for the TCR-Contribution analysis in mvTCR: average the per-cell TCR-contribution scores (angular distance between joint and TCR-unimodal embeddings) per antigen specificity, then test whether these averages track TCRdist-based sequence similarity via a Pearson correlation (reported r = 0.64 on the 10x dataset).

## When to use

Use when you have per-cell TCR-contribution scores from a trained mvTCR model plus TCRdist similarities per specificity group, and you must validate that TCR contribution quantitatively tracks TCR sequence similarity across epitope groups.

## When NOT to use

- Your embedding does not separate joint and TCR-unimodal views (e.g. you only have a clonotype-level integration), since angular distance between modalities is undefined
- Data lack antigen specificity annotations and TCR sequences usable for Scirpy's TCRdist computation
- Only CDR3-based TCR representation with partially missing paired data is available, which the authors note was insufficiently addressed (mvTCR was applied mainly to fully paired data)

## Inputs

- Preprocessed 10x dataset (>60k cells) from the SchubertLab/mvTCR_reproducibility preprocessing notebooks
- Trained mvTCR joint model and TCR-unimodal model checkpoints (available from Zenodo doi 10.5281/zenodo.7215447)
- Per-cell TCR-contribution scores (angular distance joint vs TCR-unimodal embedding)
- Antigen specificity annotations per cell (GILGFVFTL, GLCTLVAML, IVTDFSVIK, ELAGIGILTV)
- Scirpy TCRdist similarity values across specificity groups

## Outputs

- tcr_contribution_results.csv containing per-specificity average TCR contributions and the Pearson correlation coefficient
- Per-epitope average contribution values (GILGFVFTL 55.4%, GLCTLVAML 54.8%, IVTDFSVIK 53.0%, ELAGIGILTV 51.8%)
- Pearson correlation coefficient between average TCR contribution and TCRdist similarity (r = 0.64)

## How to apply

Train the joint mvTCR model (mvTCR v0.1.3, PyTorch v1.8.0 in an isolated conda environment) on the preprocessed 10x dataset (>60k cells), then compute per-cell TCR-Contribution as the angular distance between the joint embedding and the TCR-unimodal embedding. Stratify cells by antigen specificity (GILGFVFTL, GLCTLVAML, IVTDFSVIK, ELAGIGILTV) and average the contribution per epitope. Compute TCRdist-based sequence similarity across the specificity groups using Scirpy's TCRdist implementation. Finally compute the Pearson correlation between the per-specificity average contribution and the TCRdist-based similarity, and save both per-specificity averages and the coefficient (expected r = 0.64) to tcr_contribution_results.csv.

## Related tools

- **mvTCR** (Multiview VAE producing the joint and TCR-unimodal embeddings whose angular distance yields per-cell TCR-contribution scores) — https://github.com/SchubertLab/mvTCR
- **Scirpy** (Provides the TCRdist implementation used to compute TCR sequence similarity across specificity groups)
- **Scanpy** (Single-cell data format and workflow in which the mvTCR embedding and per-cell contributions are computed and analyzed)
- **PyTorch** (Deep-learning backend (v1.8.0) used to train the mvTCR models whose embeddings feed the correlation analysis)

## Examples

```
pip install mvtcr==0.1.3; python -c "from scipy.stats import pearsonr; r,p = pearsonr(avg_tcr_contribution_per_specificity, tcrdist_similarity_per_specificity); print(r)"
```

## Evaluation signals

- Per-specificity average TCR contributions reproduce the reported values: GILGFVFTL 55.4%, GLCTLVAML 54.8%, IVTDFSVIK 53.0%, ELAGIGILTV 51.8%
- Pearson correlation between average TCR contribution per specificity and TCRdist similarity matches r = 0.64 (positive correlation)
- tcr_contribution_results.csv exists with exactly four specificity rows plus the correlation coefficient
- Per-cell contribution scores are valid angular distances (non-negative, bounded) between joint and TCR-unimodal embeddings
- Evaluation was run with mvTCR v0.1.3 together with PyTorch v1.8.0 as specified for reproducibility

## Limitations

- Adjusting the contribution of each modality requires repeated retraining of mvTCR, ideally with an additional hyperparameter search
- The desired modality contribution is study-dependent and might vary across or within one dataset depending on the analysis objective
- Resulting models may diverge from the publication due to different hardware components and CUDA versions; trained models are therefore provided via Zenodo (doi 10.5281/zenodo.7215447)

## Evidence

- [abstract] fuse modality-specific information across transcriptome and TCR into a shared representation: "we develop the multimodal generative model mvTCR to fuse modality-specific information across transcriptome and TCR into a shared representation"
- [methods] 8GBs of RAM sufficient for the preprocessed 10x dataset: "8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)"
- [readme] mvTCR v0.1.3 with PyTorch v1.8.0 for reproducibility: "The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
- [intro] Scirpy offered utilities for TCR analysis: "While previous tools such as Scirpy8 and scRepertoire9 offered utilities for TCR analysis and shared data structures"
- [discussion] Adjusting modality contribution requires repeated retraining: "As a technical limitation, adjusting the contribution of each modality requires repeated training of mvTCR, ideally by an additional hyperparameter search."
