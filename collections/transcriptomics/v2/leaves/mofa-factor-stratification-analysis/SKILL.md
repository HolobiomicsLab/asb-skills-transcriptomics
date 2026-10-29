---
name: mofa-factor-stratification-analysis
description: Use when scRNA-seq PAS matrices from ≥15 samples must be reduced to interpretable
  multicellular pathway modules whose latent factors can statistically separate clinical
  phenotypes (e.g., MMRd vs MMRp).
when_to_use_negative:
- 'The study has fewer than 15 samples (not cells): MOFA-based multicellular pathway
  module identification requires at least 15 samples'
- The goal is a causal interpretation of pathway-cell type associations — module members
  are data-driven associations only
- A single-cell-level per-cell scoring (AddModuleScore/UCell/AUCell) without cross-sample
  multicellular integration is sufficient; pseudobulk aggregation causes information
  loss
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scPAFA
  role: Computes cell-level PAS via 'fast_Ucell' and provides pseudobulk/MOFA training
    and downstream tutorials
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: mofapy2
  role: Python backend for training the multi-group MOFA+ factor analysis model on
    pseudobulk PAS
  repo: https://github.com/bioFAM/mofapy2
- name: UCell
  role: Reference PAS method that 'fast_Ucell' reproduces exactly; ranking-based within-cell
    scores
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:ecc9eed0f083fa0c6432dfa9b6e89c88ae36dafabaa1c7bced9c19d4871998a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuoliHuang/scPAFA
  tools:
  - MOFA
  - UCell
  - mofapy2
  - scPAFA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mofa-factor-stratification-analysis

## Summary

A scPAFA workflow that aggregates single-cell pathway activity scores (PAS) into per-sample pseudobulk features, trains a multi-group MOFA+ model (with technical batch as group) to derive disease-related multicellular pathway modules, and tests latent factors for sample/donor stratification between clinical groups (e.g., MMRd vs MMRp CRC).

## When to use

Use when scRNA-seq PAS matrices from ≥15 samples must be reduced to interpretable multicellular pathway modules whose latent factors can statistically separate clinical phenotypes (e.g., MMRd vs MMRp).

## When NOT to use

- The study has fewer than 15 samples (not cells): MOFA-based multicellular pathway module identification requires at least 15 samples
- The goal is a causal interpretation of pathway-cell type associations — module members are data-driven associations only
- A single-cell-level per-cell scoring (AddModuleScore/UCell/AUCell) without cross-sample multicellular integration is sufficient; pseudobulk aggregation causes information loss

## Inputs

- Cell-level PAS matrix computed with scPAFA 'fast_Ucell' (e.g., 108,497 CRC tumor epithelial cells x pathways)
- Pathway dictionary (Python dict: pathway name -> gene list) from MsigDB JSON bundle or NCATS BioPlanet pathway.csv
- Cell-level metadata: sample/donor, cell type, technical batch
- Sample-level clinical metadata (e.g., MMRd/MMRp status)

## Outputs

- Trained multi-group MOFA+ model (mofapy2) with latent factors and pathway weights across cell types
- Statistical test results of factor values (e.g., factors 2, 3, 6) between clinical groups (MMRd vs MMRp)
- Sample/donor stratification visualization along significant factors
- Table of high-weighted pathway-cell type pairs defining each multicellular pathway module

## How to apply

Compute cell-level PAS with scPAFA's 'fast_Ucell' (a computationally efficient UCell implementation whose ranking-based scores are insensitive to dataset composition). Reformat the single-cell PAS matrix into a long-table-like pandas dataframe via pseudobulk aggregation per sample/donor and cell type, attaching cell-level metadata (sample/donor, cell type, technical batch) to yield pseudobulk samples (e.g., 65 CRC tumor epithelial pseudobulk samples from 108,497 cells). Train the multi-group MOFA+ framework (mofapy2) with technical batch as group, since in the presence of clearly known batch effects the multi-group framework is recommended for correction and to find sources of variability shared between groups. Extract latent factor values and pathway weights across cell types, then statistically test factor values (e.g., factors 2, 3, 6) between clinical groups using sample-level metadata. Validate by checking that the group separation along significant factors and the high-weighted pathway-cell type pairs reproduce the reported stratification (e.g., Figs 2-3a).

## Related tools

- **scPAFA** (Computes cell-level PAS via 'fast_Ucell' and provides pseudobulk/MOFA training and downstream tutorials) — https://github.com/ZhuoliHuang/scPAFA
- **mofapy2** (Python backend for training the multi-group MOFA+ factor analysis model on pseudobulk PAS) — https://github.com/bioFAM/mofapy2
- **UCell** (Reference PAS method that 'fast_Ucell' reproduces exactly; ranking-based within-cell scores)

## Examples

```
conda activate scPAFA_env; pip install scPAFA
```

## Evaluation signals

- Factors 2, 3, and 6 are significantly higher (2, 3) / lower (6) in MMRd vs MMRp samples by statistical testing of sample-level factor values
- Sample/donor stratification visualization along factors 2, 3, 6 matches the published stratification in Figs 2-3a of the article
- No batch-related factors are detected in the multi-group model (batch captured as group, not as a disease-associated factor)
- High-weighted pathway-cell type pairs per factor match the tabulated modules from the published analysis
- Pseudobulk long-table dataframe follows the mofapy2 expected input schema (sample/donor, cell type, technical batch as view/group covariates)

## Limitations

- scPAFA requires at least 15 samples (not cells) for MOFA-based multicellular pathway module identification
- Pseudobulk integration of the PAS matrix leads to information loss since the PAS of multiple cells is aggregated
- MOFA is a linear model — interpretability is achieved by sacrificing information content in each factor, and interpretability may decrease with multiple disease states
- Pathway selection is a supervised process: redundant pathway sets increase result complexity, while too few or biologically misaligned pathways cause information loss
- Module pathways/cell types only suggest data-driven associations, not biological causal effects

## Evidence

- [readme] multi-group MOFA+ framework for batch correction: "In the presence of clearly known batch effects, we recommend using multi-group MOFA+ framework for correction."
- [discussion] fast_Ucell efficiency and dataset insensitivity: "Similar to AUCell, the PAS of “fast_ucell” is based on the ranking of gene expression within each cell, making them insensitive to the dataset in which the cells are located."
- [discussion] sample requirement: "scPAFA requires studies to include at least 15 samples (not cells), which is the requirement of using the MOFA model for identifying multicellular pathway modules."
- [discussion] pseudobulk information loss: "Moreover, pseudobulk integration of the PAS matrix would lead to information loss since the PAS of multiple cells is aggregated."
- [methods] MOFA training purpose: "MOFA model is trained to capture variance in PAS among different samples"
