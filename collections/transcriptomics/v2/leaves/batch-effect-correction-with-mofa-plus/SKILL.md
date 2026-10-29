---
name: batch-effect-correction-with-mofa-plus
description: Use when single-cell-derived pseudobulk feature matrices (e.g., scPAFA
  PAS aggregated per sample) contain clearly known technical batch effects and you
  need unsupervised factor analysis that separates batch from biological/disease variance.
when_to_use_negative:
- The study has fewer than 15 samples (not cells), since scPAFA requires at least
  15 samples to use the MOFA model for identifying multicellular pathway modules
- Batch effects are unknown or unannotated — the multi-group framework requires clearly
  known batch effects to declare groups; without labels the single-group framework
  is the appropriate choice
- You need causal attribution of pathway-cell type associations — MOFA is linear and
  the modules are data-driven associations, not biological causal effects
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scPAFA
  role: Python library whose pseudobulk processing (Step 2) reformats the cell-level
    PAS matrix into the long-table-like dataframe input for multi-group MOFA+ training
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: mofapy2
  role: Python implementation of the MOFA2 factor analysis model used to train the
    multi-group model on the pseudobulk PAS matrix
  repo: https://github.com/bioFAM/mofapy2
- name: MOFA
  role: Framework reused in scPAFA for multicellular integration; its multi-group
    mode finds variability shared between groups and corrects clearly known batch
    effects
  repo: https://biofam.github.io/MOFA2/index.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:e49de05563c9d1a6e5a146a02a45689633aa23fc8bbc793e13cd800b68dbf23a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuoliHuang/scPAFA
  tools:
  - MOFA
  - Python
  - UCell
  - mofapy2
  - scPAFA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# batch-effect-correction-with-mofa-plus

## Summary

Train a multi-group MOFA+ model on a pseudobulk pathway activity score (PAS) matrix so that known technical batch effects are modeled as groups, allowing latent factors to capture disease-related variance in PAS among samples rather than batch-driven artifacts.

## When to use

Use when single-cell-derived pseudobulk feature matrices (e.g., scPAFA PAS aggregated per sample) contain clearly known technical batch effects and you need unsupervised factor analysis that separates batch from biological/disease variance.

## When NOT to use

- The study has fewer than 15 samples (not cells), since scPAFA requires at least 15 samples to use the MOFA model for identifying multicellular pathway modules
- Batch effects are unknown or unannotated — the multi-group framework requires clearly known batch effects to declare groups; without labels the single-group framework is the appropriate choice
- You need causal attribution of pathway-cell type associations — MOFA is linear and the modules are data-driven associations, not biological causal effects

## Inputs

- Cell-level PAS matrix from scPAFA (e.g., fast_Ucell scores for 1383 NCATS BioPlanet pathways) as a long-table-like pandas dataframe
- Cell-level metadata: sample/donor, cell type, and technical batch assignments
- Sample-level clinical metadata (e.g., SLE vs healthy control status)
- Batch/group labels designating known technical batch effects for the multi-group framework

## Outputs

- Trained multi-group MOFA+ model with latent factors capturing sample-level PAS variance (batch-adjusted)
- Factor values per pseudobulk sample for downstream statistical testing against clinical metadata
- Factor weights for pathway-cell type pairs (multicellular pathway modules)

## How to apply

First compute cell-level pathway activity scores (e.g., 1383 NCATS BioPlanet pathways via scPAFA's fast_Ucell) and reformat the cell-level PAS matrix into a long-table-like pandas dataframe carrying sample/donor, cell type, and technical batch metadata, aggregating to training pseudobulk samples (e.g., 175 samples in the lupus atlas). Then, when batch effects are clearly known, use MOFA's multi-group framework — where the aim is to find which sources of variability are shared between groups — declaring batches as groups so they are modeled explicitly rather than confounding the latent factors. Train the mofapy2 model on the pseudobulk PAS matrix so latent factors capture variance in PAS among samples, then verify downstream that factors align with clinical metadata (e.g., factor 1 significantly elevated and factor 6 significantly decreased in SLE versus healthy controls) rather than with batch labels. Requires at least 15 samples (not cells) for the MOFA model.

## Related tools

- **scPAFA** (Python library whose pseudobulk processing (Step 2) reformats the cell-level PAS matrix into the long-table-like dataframe input for multi-group MOFA+ training) — https://github.com/ZhuoliHuang/scPAFA
- **mofapy2** (Python implementation of the MOFA2 factor analysis model used to train the multi-group model on the pseudobulk PAS matrix) — https://github.com/bioFAM/mofapy2
- **MOFA** (Framework reused in scPAFA for multicellular integration; its multi-group mode finds variability shared between groups and corrects clearly known batch effects) — https://biofam.github.io/MOFA2/index.html

## Examples

```
conda activate scPAFA_env; pip install mofapy2; # Step2&3: reformat long-table dataframe with sample/celltype/batch columns and train multi-group MOFA+ per scPAFA tutorial: steps2%263_Pseudobulk_processing_and_MOFA_model_training.ipynb
```

## Evaluation signals

- The trained model's latent factors correlate with disease status (SLE vs healthy) in sample-level clinical metadata, not with technical batch labels
- In the lupus atlas application, factor 1 is significantly increased and factor 6 significantly decreased in SLE patients versus healthy controls
- Top-weighted features of the trained model include the reported pathway-cell type pairs: interferon alpha/beta, Type II interferon, and Granzyme A associations (Figures 5 and 6a-e)
- The PAS input to mofapy2 is a valid long-table-like pandas dataframe with sample/donor, cell type, and batch columns aggregated to pseudobulk training samples
- The study meets the minimum requirement of at least 15 samples for MOFA-based multicellular pathway module identification

## Limitations

- scPAFA requires studies to include at least 15 samples (not cells), which is the requirement for using the MOFA model
- MOFA is a linear model; interpretability is achieved by sacrificing information content in each factor, and interpretability may decrease with multiple disease states
- Pseudobulk integration of the PAS matrix leads to information loss since the PAS of multiple cells is aggregated
- Pathways and cell types within the same module only suggest data-driven associations, not biological causal effects

## Evidence

- [readme] multi-group framework recommendation: "In the presence of clearly known batch effects, we recommend using multi-group MOFA+ framework for correction."
- [readme] multi-group aim: "the aim of multi-group framework is to find out which sources of variability are shared between the different groups"
- [methods] model purpose: "MOFA model is trained to capture variance in PAS among different samples"
- [readme] long-table input format: "the single-cell PAS matrix is reformatted into a suitable input ([a long-table-like pandas dataframe]"
- [discussion] sample minimum: "scPAFA requires studies to include at least 15 samples (not cells), which is the requirement of using the MOFA model"
- [discussion] linear model caveat: "MOFA is a linear model, interpretability is achieved by sacrificing information content in each factor"
