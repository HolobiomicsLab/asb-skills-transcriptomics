---
name: specificity-stratified-aggregation
description: Use when you have a cell-level embedding score for >60k paired TCR/transcriptome
  cells (10x dataset) with specificity annotations (e.g., GILGFVFTL, GLCTLVAML, IVTDFSVIK,
  ELAGIGILTV) and want per-epitope summaries plus correlation with TCRdist similarity.
when_to_use_negative:
- When no antigen-specificity annotation is available for the cells, since stratification
  and per-epitope aggregation cannot be performed
- When the question requires cell-level (not specificity-group-level) resolution —
  aggregation loses within-clonotype and within-epitope heterogeneity
- When data are unpaired or lack TCR sequences, since TCR-contribution scores and
  TCRdist similarity cannot be computed
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2830
tools:
- name: mvTCR
  role: Trains the multi-view model that fuses transcriptome and TCR into a shared
    cell-level embedding from which TCR-contribution (angular distance between joint
    and TCR-unimodal embeddings) is computed
  repo: https://github.com/SchubertLab/mvTCR
- name: Scirpy
  role: Provides the TCRdist implementation used to compute TCR sequence similarity
    across specificity groups
- name: PyTorch
  role: Deep-learning backend for training the mvTCR model (v1.8.0) used to derive
    the embeddings
- name: Scanpy
  role: Single-cell data structure and workflow integration for handling the preprocessed
    10x dataset
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:66f3ac586610a429a81f2f44dc4d93aaeea1c2fdfadab0b615ac9367937c0813
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
yes_rate: 0.75
n_claims: 8
---

# specificity-stratified-aggregation

## Summary

Stratifies single-cell T cells by antigen specificity (epitope) and aggregates per-cell scores (e.g., mvTCR TCR-contribution, the angular distance between joint and TCR-unimodal embeddings) into per-specificity averages, then relates them to TCR sequence similarity (TCRdist) via Pearson correlation.

## When to use

Use when you have a cell-level embedding score for >60k paired TCR/transcriptome cells (10x dataset) with specificity annotations (e.g., GILGFVFTL, GLCTLVAML, IVTDFSVIK, ELAGIGILTV) and want per-epitope summaries plus correlation with TCRdist similarity.

## When NOT to use

- When no antigen-specificity annotation is available for the cells, since stratification and per-epitope aggregation cannot be performed
- When the question requires cell-level (not specificity-group-level) resolution — aggregation loses within-clonotype and within-epitope heterogeneity
- When data are unpaired or lack TCR sequences, since TCR-contribution scores and TCRdist similarity cannot be computed

## Inputs

- Preprocessed 10x single-cell dataset (>60k cells) with paired TCR and transcriptome data
- Trained mvTCR joint embedding (mvTCR v0.1.3, PyTorch v1.8.0)
- TCR-unimodal embedding from the same model
- Per-cell antigen specificity annotations (epitopes: GILGFVFTL, GLCTLVAML, IVTDFSVIK, ELAGIGILTV)

## Outputs

- Per-specificity average TCR-contribution scores (percent per epitope)
- Pearson correlation coefficient between average TCR contribution and TCRdist-based similarity
- tcr_contribution_results.csv containing per-specificity averages and the correlation value

## How to apply

Load the preprocessed 10x dataset (>60k cells) from the SchubertLab/mvTCR_reproducibility repository and train the joint mvTCR model (mvTCR v0.1.3 with PyTorch v1.8.0 on a GPU machine). For each cell, compute the TCR-Contribution score as the angular distance between the joint embedding and the TCR-unimodal embedding. Group cells by their annotated antigen specificity and average the contribution scores per epitope (expected: GILGFVFTL 55.4%, GLCTLVAML 54.8%, IVTDFSVIK 53.0%, ELAGIGILTV 51.8%). Compute TCRdist-based sequence similarity across specificity groups using Scirpy's TCRdist implementation, then calculate the Pearson correlation between average per-specificity contribution and TCRdist similarity (expected r = 0.64). Save per-specificity averages and the correlation coefficient to tcr_contribution_results.csv.

## Related tools

- **mvTCR** (Trains the multi-view model that fuses transcriptome and TCR into a shared cell-level embedding from which TCR-contribution (angular distance between joint and TCR-unimodal embeddings) is computed) — https://github.com/SchubertLab/mvTCR
- **Scirpy** (Provides the TCRdist implementation used to compute TCR sequence similarity across specificity groups)
- **PyTorch** (Deep-learning backend for training the mvTCR model (v1.8.0) used to derive the embeddings)
- **Scanpy** (Single-cell data structure and workflow integration for handling the preprocessed 10x dataset)

## Examples

```
conda create --name mvTCR python=3.8.8 -y && conda activate mvTCR && pip install mvtcr==0.1.3 && conda install nb_conda_kernels -y
```

## Evaluation signals

- Per-specificity average TCR contributions reproduce the reported values: GILGFVFTL 55.4%, GLCTLVAML 54.8%, IVTDFSVIK 53.0%, ELAGIGILTV 51.8%
- Pearson correlation between average TCR contribution per specificity and TCRdist-based similarity matches r = 0.64
- Average TCR-contribution per specificity shows a positive correlation with TCRdist similarity
- All four expected epitope groups are present in the aggregated output with non-empty cell counts
- Output CSV tcr_contribution_results.csv contains both per-specificity averages and the correlation coefficient

## Limitations

- Adjusting the contribution of each modality requires repeated retraining of mvTCR, ideally with an additional hyperparameter search
- The desired modality contribution is study-dependent and may vary across or even within one dataset, so no fixed automated setting applies
- Results may diverge from the publication due to differences in hardware, CUDA versions, and runtimes during model training

## Evidence

- [results] The four highest average TCR contributions were observed for GILGFVFTL (55.4%), GLCTLVAML (54.8%), IVTDFSVIK (53.0%), and ELAGIGILTV (51.8%): [evidence span withheld — anchor status: not_found]
- [results] the average TCR-contribution per specificity correlates positively with TCRdist similarity (Pearson r = 0.64): [evidence span withheld — anchor status: not_found]
- [abstract] we develop the multimodal generative model mvTCR to fuse modality-specific information across transcriptome and TCR into a shared representation: "we develop the multimodal generative model mvTCR to fuse modality-specific information across transcriptome and TCR into a shared representation"
- [readme] Additionally, you will need to install mvTCR v0.1.3 together with PyTorch v1.8.0.: [evidence span withheld — anchor status: external_doc]
- [intro] While previous tools such as Scirpy8 and scRepertoire9 offered utilities for TCR analysis: "While previous tools such as Scirpy8 and scRepertoire9 offered utilities for TCR analysis and shared data structures"
