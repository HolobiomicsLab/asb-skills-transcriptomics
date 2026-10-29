---
name: multi-group-factor-analysis-training
description: Use when single-cell PAS matrices from multiple samples/donors and cell
  types must be integrated into low-dimensional factors that stratify clinical groups
  (e.g., MMRd vs MMRp) while correcting for known technical batch effects.
when_to_use_negative:
- The study contains fewer than 15 samples (not cells) — scPAFA notes MOFA requires
  at least 15 samples for multicellular pathway module identification.
- No known technical batch structure exists and a single-group design suffices — the
  multi-group framework is specifically recommended for correcting clearly known batch
  effects.
- The goal is pairwise cross-condition differential pathway activity per cell type
  rather than data-driven low-dimensional modules (MOFA factors are data-driven associations,
  not causal effects).
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPAFA
  role: Provides pseudobulk processing utilities that reformat the single-cell PAS
    matrix into the long-table-like dataframe consumed by mofapy2
  repo: github.com/ZhuoliHuang/scPAFA
- name: mofapy2 (MOFA)
  role: Trains the multi-group MOFA+ factor analysis model on pseudobulk PAS with
    technical batch as group to capture variance among samples
  repo: github.com/bioFAM/mofapy2
- name: UCell
  role: 'PAS basis: fast_Ucell computes PAS equivalent to UCell, feeding the pseudobulk
    aggregation and MOFA training'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:eeda24b0f5cdf4d68968f84e79ab9246b75b92dd8dabc80b127d25d989843e5d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - UCell
  - mofapy2 (MOFA)
  - scPAFA
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# multi-group-factor-analysis-training

## Summary

Train a multi-group MOFA+ model (mofapy2) on pseudobulk-averaged pathway activity scores (PAS) computed by scPAFA's fast_Ucell, using technical batch as group, to extract disease-related multicellular pathway modules (latent factors and pathway-by-cell-type weights).

## When to use

Use when single-cell PAS matrices from multiple samples/donors and cell types must be integrated into low-dimensional factors that stratify clinical groups (e.g., MMRd vs MMRp) while correcting for known technical batch effects.

## When NOT to use

- The study contains fewer than 15 samples (not cells) — scPAFA notes MOFA requires at least 15 samples for multicellular pathway module identification.
- No known technical batch structure exists and a single-group design suffices — the multi-group framework is specifically recommended for correcting clearly known batch effects.
- The goal is pairwise cross-condition differential pathway activity per cell type rather than data-driven low-dimensional modules (MOFA factors are data-driven associations, not causal effects).

## Inputs

- cell-level PAS matrix from scPAFA fast_Ucell (or fast_score_genes)
- cell-level metadata: sample/donor ID, cell type, technical batch
- sample-level clinical metadata (e.g., MMR status: MMRd vs MMRp)
- pathway dictionary (pathway name -> gene list, e.g., from NCATS BioPlanet or MsigDB)

## Outputs

- trained MOFA+ model with latent factor values per pseudobulk sample
- pathway weights across cell types per factor (multicellular pathway modules)
- factor-vs-clinical-group statistics (e.g., MMRd vs MMRp comparisons)
- sample/donor stratification plots and tables of high-weighted pathway-cell type pairs

## How to apply

Start from the cell-level PAS matrix computed with scPAFA's fast_Ucell and aggregate it into pseudobulk values per sample/donor and cell type, attaching cell-level metadata (sample/donor, cell type, technical batch) — in the CRC tumor epithelial case this yielded 65 pseudobulk samples. Reformat the PAS matrix into the long-table-like pandas dataframe expected by mofapy2. Because the MOFA multi-group framework is designed to find variability shared across groups, use multi-group MOFA+ with technical batch as group when clearly known batch effects exist. After training, extract latent factor values and pathway weights across cell types, then statistically test factor values (e.g., factors 2, 3, and 6 in the CRC MMR study) between clinical groups using sample-level metadata. Interpret high-weighted pathway-cell type pairs to define the multicellular pathway modules.

## Related tools

- **scPAFA** (Provides pseudobulk processing utilities that reformat the single-cell PAS matrix into the long-table-like dataframe consumed by mofapy2) — github.com/ZhuoliHuang/scPAFA
- **mofapy2 (MOFA)** (Trains the multi-group MOFA+ factor analysis model on pseudobulk PAS with technical batch as group to capture variance among samples) — github.com/bioFAM/mofapy2
- **UCell** (PAS basis: fast_Ucell computes PAS equivalent to UCell, feeding the pseudobulk aggregation and MOFA training)

## Examples

```
pip install scPAFA; # then follow tutorial notebook steps2&3_Pseudobulk_processing_and_MOFA_model_training.ipynb
```

## Evaluation signals

- Milestone factors (e.g., 2, 3, 6) show statistically significant differences between MMRd and MMRp samples using sample-level clinical metadata.
- MMRd vs MMRp separation along factors 2, 3, and 6 matches the published stratification in Figs 2-3a of the article.
- High-weighted pathway-cell type pairs for the identified factors reproduce the published module composition.
- No batch-related factors are detected when technical batch is supplied as group, confirming batch correction worked.
- Pseudobulk table has the expected structure: one row per sample/donor x cell type with PAS values and metadata columns.

## Limitations

- Pseudobulk integration of the PAS matrix leads to information loss since PAS of multiple cells is aggregated.
- MOFA is a linear model; interpretability is achieved by sacrificing information content in each factor, and interpretability may decrease with multiple disease states.
- Pathway-cell type module associations are data-driven only, not biological causal effects.
- Pathway selection is supervised; redundant pathway sets increase result complexity while too few or misaligned pathways cause information loss.

## Evidence

- [readme] multi-group MOFA+ recommendation with batch as group: "In the presence of clearly known batch effects, we recommend using multi-group MOFA+ framework for correction."
- [readme] mofapy2 long-table input format: [evidence span withheld — anchor status: not_found]
- [methods] MOFA captures variance in PAS among samples: "MOFA model is trained to capture variance in PAS among different samples"
- [readme] multi-group framework purpose: "the aim of multi-group framework is to find out which sources of variability are shared between the different groups"
- [discussion] minimum sample requirement: "Most importantly, scPAFA requires studies to include at least 15 samples (not cells), which is the requirement of using the MOFA model for identifying multicellular pathway modules."
