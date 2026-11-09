---
name: mofa-multi-group-factor-model-training
description: Use when a cell-level PAS matrix has been pseudobulked into a long-table
  dataframe with sample/donor, cell type, and technical batch columns, and you need
  latent factors capturing PAS variance among samples (≥15 samples required).
when_to_use_negative:
- The study contains fewer than 15 samples (not cells) — MOFA-based multicellular
  pathway module identification requires at least 15 samples
- Batch effects are absent/unknown and the single-group general framework (or no factor
  analysis) is appropriate
- Input is raw single-cell gene expression rather than pseudobulk PAS — run scPAFA
  step 1/2 first
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: mofapy2
  role: Python implementation of Multi-Omics Factor Analysis used to train the MOFA
    model on the pseudobulk PAS long-table dataframe (single-group or multi-group
    framework)
  repo: https://github.com/bioFAM/mofapy2
- name: scPAFA
  role: Generates the pathway activity scores and the mofapy2-compatible long-table
    input; reutilizes the MOFA framework for multicellular integration; tutorial notebook
    steps2&3 covers pseudobulk processing and MOFA model training
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: MOFA
  role: Factor-analysis framework whose latent factors capture variance in PAS among
    different samples to define disease-related multicellular pathway modules
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:3d2b07c8915c9a0854924d94a166b5a2a9af30ab026e47b95ee6ea532921d6c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioFAM/mofapy2
  tools:
  - MOFA
  - Python
  - UCell
  - mofapy2
  - scPAFA
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mofa-multi-group-factor-model-training

## Summary

Trains a MOFA/mofapy2 factor-analysis model on a pseudobulk pathway-activity-score (PAS) matrix, optionally with the multi-group MOFA+ framework to correct known batch effects, yielding latent factors that represent disease-related multicellular pathway modules.

## When to use

Use when a cell-level PAS matrix has been pseudobulked into a long-table dataframe with sample/donor, cell type, and technical batch columns, and you need latent factors capturing PAS variance among samples (≥15 samples required).

## When NOT to use

- The study contains fewer than 15 samples (not cells) — MOFA-based multicellular pathway module identification requires at least 15 samples
- Batch effects are absent/unknown and the single-group general framework (or no factor analysis) is appropriate
- Input is raw single-cell gene expression rather than pseudobulk PAS — run scPAFA step 1/2 first

## Inputs

- Pseudobulk pathway activity score (PAS) matrix reformatted as a long-table-like pandas dataframe (mofapy2 input format), one row per sample x pathway-cell-type feature
- Cell-level metadata: sample/donor, cell type, technical batch
- Sample-level clinical metadata for downstream factor testing (e.g., tumor site, SLE vs healthy status)

## Outputs

- Trained MOFA model (mofapy2) containing latent factors and variance decomposition over the PAS samples
- Latent factor values per pseudobulk sample (e.g., factor 1 elevated, factor 6 decreased in SLE; factors 4/6 metastasis-related, factor 5 malignant-related in CRC-CRLM)
- Pathway weights across cell types for each factor, defining high-weighted pathway-cell type pairs

## How to apply

After scPAFA step 1 (fast_Ucell/fast_score_genes PAS on NCATS BioPlanet pathways) and step 2 (reformat the single-cell PAS matrix into the long-table-like pandas dataframe expected by mofapy2, joined with sample/donor, cell type, and batch metadata), feed the pseudobulk matrix to mofapy2's entry_point and train the model so it captures variance in PAS among different samples (e.g., the 40 CRC-CRLM samples or 175 lupus pseudobulk samples). Decide between the single-group general framework and the multi-group framework: when batch effects are clearly known, use the multi-group MOFA+ framework so batch sources of variability are modeled as groups while shared variability is captured by latent factors. Extract latent factors (e.g., factors 4/5/6 in CRC-CRLM, factors 1/6 in lupus) with their pathway weights across cell types for downstream statistical analysis against sample-level clinical metadata (Kruskal-Wallis and Mann-Whitney U tests).

## Related tools

- **mofapy2** (Python implementation of Multi-Omics Factor Analysis used to train the MOFA model on the pseudobulk PAS long-table dataframe (single-group or multi-group framework)) — https://github.com/bioFAM/mofapy2
- **scPAFA** (Generates the pathway activity scores and the mofapy2-compatible long-table input; reutilizes the MOFA framework for multicellular integration; tutorial notebook steps2&3 covers pseudobulk processing and MOFA model training) — https://github.com/ZhuoliHuang/scPAFA
- **MOFA** (Factor-analysis framework whose latent factors capture variance in PAS among different samples to define disease-related multicellular pathway modules)

## Examples

```
pip install mofapy2
```

## Evaluation signals

- Training converges on the expected number of pseudobulk samples (e.g., 40 CRC-CRLM samples from 10 patients; 175 lupus pseudobulk samples), with latent factors capturing variance in PAS among samples
- Multi-group training with known batches finds sources of variability shared between groups, per the MOFA2 multi-group FAQ rationale
- Downstream factor values reproduce the paper's significance patterns: factor 4/6 metastasis-related and factor 5 malignant-related across CRC-CRLM sites (Fig. 4); factor 1 elevated and factor 6 decreased in SLE (Figs. 5, 6a-e)
- High-weighted pathway-cell type pairs include the expected modules, e.g., interferon alpha/beta, Type II interferon, and Granzyme A in the SLE atlas
- Long-table input dataframe schema matches mofapy2's getting-started example (sample/donor, cell type, batch columns)

## Limitations

- MOFA is a linear model; interpretability is achieved by sacrificing information content in each factor, and interpretability may decrease with multiple disease states
- scPAFA requires studies to include at least 15 samples (not cells), which is the requirement of using the MOFA model
- Pseudobulk integration of the PAS matrix leads to information loss since PAS of multiple cells is aggregated

## Evidence

- [readme] In the presence of clearly known batch effects, we recommend using multi-group MOFA+ framework for correction.: "In the presence of clearly known batch effects, we recommend using multi-group MOFA+ framework for correction."
- [methods] MOFA model captures variance in PAS among samples: "MOFA model is trained to capture variance in PAS among different samples"
- [discussion] MOFA linear model tradeoff: "Besides, MOFA is a linear model, interpretability is achieved by sacrificing information content in each factor"
- [discussion] MOFA sample requirement: "Most importantly, scPAFA requires studies to include at least 15 samples (not cells), which is the requirement of using the MOFA model for identifying multicellular pathway modules."
- [readme] Input format for mofapy2: "the single-cell PAS matrix is reformatted into a suitable input ([a long-table-like pandas dataframe]"
