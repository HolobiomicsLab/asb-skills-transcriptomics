---
name: tcrdist-similarity-computation
description: Use when you have paired single-cell TCR data stratified by antigen specificity
  (e.g., epitope groups like GILGFVFTL, GLCTLVAML) and need a sequence-similarity
  measure per group or a similarity metric to correlate against embedding-based scores.
when_to_use_negative:
- The TCR data are unpaired or lack CDR3 sequences, since Scirpy's TCRdist requires
  CDR3 sequence information (mvTCR likewise uses only the CDR3 sequence in this workflow)
- The analysis goal is antigen-specificity prediction itself rather than similarity
  quantification — mvTCR's prediction workflow (tutorial 03) is the better fit
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_2830
tools:
- name: Scirpy
  role: Provides the TCRdist implementation used to compute sequence similarity across
    specificity groups
- name: mvTCR
  role: Trains the joint transcriptome+TCR embedding whose per-cell TCR-Contribution
    scores (angular distance to the TCR-unimodal embedding) are correlated with TCRdist
    similarity
  repo: https://github.com/SchubertLab/mvTCR
- name: Scanpy
  role: Single-cell workflow format (AnnData) in which the mvTCR embedding and specificity
    annotations are handled
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49806-9
    title: Multi-modal generative modeling for joint analysis of single-cell T cell
      receptor and gene expression data
  source_content_hash: sha256:ec819af9b89c0f9086969cf05d46fa02346ee0dac884c59069b3383d356cf23d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SchubertLab/mvTCR
  tools:
  - PyTorch
  - Scanpy
  - Scirpy
  - mvTCR
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# tcrdist-similarity-computation

## Summary

Computes TCR sequence similarity across antigen-specificity groups using Scirpy's TCRdist implementation, enabling quantitative comparison of sequence-based relatedness (e.g., against per-cell TCR-Contribution scores from mvTCR via Pearson correlation).

## When to use

Use when you have paired single-cell TCR data stratified by antigen specificity (e.g., epitope groups like GILGFVFTL, GLCTLVAML) and need a sequence-similarity measure per group or a similarity metric to correlate against embedding-based scores.

## When NOT to use

- The TCR data are unpaired or lack CDR3 sequences, since Scirpy's TCRdist requires CDR3 sequence information (mvTCR likewise uses only the CDR3 sequence in this workflow)
- The analysis goal is antigen-specificity prediction itself rather than similarity quantification — mvTCR's prediction workflow (tutorial 03) is the better fit

## Inputs

- Preprocessed paired single-cell transcriptome + TCR dataset (e.g., 10x dataset, >60k cells, .h5ad/AnnData with TCR annotation)
- Per-cell antigen specificity labels (epitope groups such as GILGFVFTL, GLCTLVAML, IVTDFSVIK, ELAGIGILTV)
- Per-cell scores to correlate against similarity (e.g., mvTCR TCR-Contribution scores from angular distance between joint and TCR-unimodal embeddings)

## Outputs

- TCRdist-based sequence similarity values per antigen-specificity group
- Pearson correlation coefficient between average per-specificity scores and TCRdist similarity (reference value r = 0.64)
- Results table (e.g., tcr_contribution_results.csv) with per-specificity averages and the correlation coefficient

## How to apply

Load the preprocessed paired TCR/single-cell dataset (e.g., the 10x dataset with >60k cells from the mvTCR_reproducibility repository) and train the mvTCR joint model to obtain cell-level embeddings if a modality-contribution comparison is needed. Stratify cells by antigen specificity into epitope groups and compute TCRdist-based sequence similarity across these specificity groups using Scirpy's TCRdist implementation. Compute the Pearson correlation between the per-specificity similarity values and the corresponding average scores of interest (e.g., per-specificity average TCR-Contribution, defined as the angular distance between the joint embedding and the TCR-unimodal embedding). In the reference analysis this correlation was positive with Pearson r = 0.64, and the highest average TCR contributions occurred for GILGFVFTL (55.4%), GLCTLVAML (54.8%), IVTDFSVIK (53.0%), and ELAGIGILTV (51.8%). Save the per-specificity values and correlation coefficient (e.g., tcr_contribution_results.csv) for validation.

## Related tools

- **Scirpy** (Provides the TCRdist implementation used to compute sequence similarity across specificity groups)
- **mvTCR** (Trains the joint transcriptome+TCR embedding whose per-cell TCR-Contribution scores (angular distance to the TCR-unimodal embedding) are correlated with TCRdist similarity) — https://github.com/SchubertLab/mvTCR
- **Scanpy** (Single-cell workflow format (AnnData) in which the mvTCR embedding and specificity annotations are handled)

## Examples

```
conda create --name mvTCR python=3.8.8 -y && conda activate mvTCR && pip install mvtcr==0.1.3  # then use Scirpy's TCRdist on specificity-grouped cells and scipy.stats.pearsonr(avg_contribution, tcrdist_similarity)
```

## Evaluation signals

- Per-specificity average TCR-Contribution values reproduce the reported values: GILGFVFTL 55.4%, GLCTLVAML 54.8%, IVTDFSVIK 53.0%, ELAGIGILTV 51.8%
- Pearson correlation between average TCR-Contribution per specificity and TCRdist similarity matches r = 0.64
- Correlation is positive, consistent with TCR contribution tracking TCR sequence similarity across specificity groups
- Output CSV (tcr_contribution_results.csv) contains one row per specificity group plus the correlation coefficient
- Trained model and data match the reference stack: mvTCR v0.1.3 with PyTorch v1.8.0 on the preprocessed 10x dataset

## Limitations

- mvTCR was mainly applied to fully paired data (gene expression with α- and β-CDR3); handling of partially missing data remains insufficiently addressed
- Only the CDR3 sequence is used for the TCR representation; VDJ-gene encoding and pre-trained TCR embedding models are not incorporated
- Reproduced models may diverge from publication models due to differing hardware and CUDA versions; trained models are provided via Zenodo (https://doi.org/10.5281/zenodo.7215447)

## Evidence

- [results] The four highest average TCR contributions were observed for GILGFVFTL (55.4%), GLCTLVAML (54.8%), IVTDFSVIK (53.0%), and ELAGIGILTV (51.8%), and the average TCR-contribution per specificity correlates positively with TCRdist similarity (Pearson r = 0.64).: [evidence span withheld — anchor status: not_found]
- [readme] 8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells): "8GBs of RAM were sufficient to train mvTCR on our preprocessed 10x dataset (>60k cells)"
- [readme] The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0.: "The experiments were run with mvTCR v0.1.3 together with PyTorch v1.8.0."
- [abstract] we develop the multimodal generative model mvTCR to fuse modality-specific information across transcriptome and TCR into a shared representation: "we develop the multimodal generative model mvTCR to fuse modality-specific information across transcriptome and TCR into a shared representation"
