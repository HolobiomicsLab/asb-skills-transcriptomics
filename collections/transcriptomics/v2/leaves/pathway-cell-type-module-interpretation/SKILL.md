---
name: pathway-cell-type-module-interpretation
description: 'Use when a trained MOFA model on pseudobulk pathway activity scores
  (PAS) must be interpreted: identifying which latent factors are disease-related
  and which pathway-cell type pairs drive them.'
when_to_use_negative:
- The trained model has fewer than 15 samples (not cells) — the MOFA requirement for
  multicellular pathway module identification is not met
- Input is a raw single-cell expression matrix rather than a trained factor model
  with pathway weights — compute PAS and train MOFA first (steps 1-3)
- A causal interpretation of pathway-cell type associations is required — scPAFA modules
  are data-driven associations only
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scPAFA
  role: Python library whose step-4 downstream analysis characterizes, interprets,
    and visualizes multicellular pathway modules from the trained MOFA model
  repo: github.com/ZhuoliHuang/scPAFA
- name: mofapy2
  role: Provides the trained MOFA model object from which latent factor values and
    pathway weights across cell types are extracted
  repo: github.com/bioFAM/mofapy2
- name: MOFA
  role: Factor analysis framework whose latent factors are interpreted as disease-related
    multicellular pathway modules
provenance:
  source_task_ids:
  - task_007
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:14b4f33ccfac0d220ff691c4b3bc5fe49b5b11953c789386c591a9f0afcdb4b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MOFA
  - Python
  - UCell
  - mofapy2
  - scPAFA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# pathway-cell-type-module-interpretation

## Summary

Step-4 downstream analysis of a trained scPAFA/MOFA model that converts latent factors into disease-related multicellular pathway modules by statistically testing factor values against sample-level clinical metadata and dissecting high-weighted pathway-cell type pairs.

## When to use

Use when a trained MOFA model on pseudobulk pathway activity scores (PAS) must be interpreted: identifying which latent factors are disease-related and which pathway-cell type pairs drive them.

## When NOT to use

- The trained model has fewer than 15 samples (not cells) — the MOFA requirement for multicellular pathway module identification is not met
- Input is a raw single-cell expression matrix rather than a trained factor model with pathway weights — compute PAS and train MOFA first (steps 1-3)
- A causal interpretation of pathway-cell type associations is required — scPAFA modules are data-driven associations only

## Inputs

- Trained MOFA model (mofapy2) with latent factors and pathway weights across cell types
- Sample-level clinical metadata (disease status, MMR status, tumor site, batch)
- Pseudobulk PAS matrix per sample/donor and cell type (from fast_Ucell scoring of NCATS BioPlanet pathways)

## Outputs

- Disease-related multicellular pathway modules (factor ID + high-weighted pathway-cell type pairs)
- Statistical test results of factor values across disease groups (Kruskal-Wallis, Mann-Whitney U)
- Sample/donor stratification plots along disease-related factors (e.g., Figs 2-6a-e)

## How to apply

Load the trained MOFA model and extract latent factor values together with the corresponding pathway weights across cell types. Join factor values with sample-level clinical metadata (e.g., SLE vs healthy control, MMRd vs MMRp, primary vs metastatic site) and test associations with statistical tests such as Kruskal-Wallis and Mann-Whitney U. Tabulate and visualize the high-weighted pathway-cell type pairs of disease-related factors, and plot sample/donor stratification along those factors. Accept the interpretation only when the significance pattern (e.g., factor 1 elevated and factor 6 decreased in SLE; factors 2/3 higher and factor 6 lower in MMRd) and the top-weighted pairs (e.g., interferon alpha/beta, Type II interferon, Granzyme A) reproduce the expected modules and figures.

## Related tools

- **scPAFA** (Python library whose step-4 downstream analysis characterizes, interprets, and visualizes multicellular pathway modules from the trained MOFA model) — github.com/ZhuoliHuang/scPAFA
- **mofapy2** (Provides the trained MOFA model object from which latent factor values and pathway weights across cell types are extracted) — github.com/bioFAM/mofapy2
- **MOFA** (Factor analysis framework whose latent factors are interpreted as disease-related multicellular pathway modules)

## Examples

```
from mofapy2.run.entry_point import entry_point; factors = model.get_factors(); weights = model.get_weights(); test factor values vs clinical metadata with scipy.stats.kruskal
```

## Evaluation signals

- Disease-related factors show the expected direction: factor 1 significantly increased and factor 6 significantly decreased in SLE vs healthy controls; factors 2 and 3 higher and factor 6 lower in MMRd vs MMRp samples
- Kruskal-Wallis and Mann-Whitney U tests across tumor sites reproduce the significance patterns of Fig. 4 in the CRC-CRLM analysis
- Expected pathway-cell type pairs (interferon alpha/beta, Type II interferon, Granzyme A) appear among the high-weighted features of the identified modules
- Sample/donor stratification plots along disease-related factors separate patient groups as reported (Figs 2-3a, Fig. 4, Figs 5-6a-e)
- No batch-related factors are detected when batch is modeled as group in the multi-group MOFA+ framework

## Limitations

- Pathway-cell type associations within a module are data-driven, not biological causal effects
- MOFA is a linear model; interpretability is achieved by sacrificing information content in each factor, and interpretability may decrease with multiple disease states
- Using a redundant pathway set increases result complexity, while choosing too few or biologically misaligned pathways may cause information loss

## Evidence

- [readme] In step 4, together with sample-level clinical metadata, disease-related multicellular pathway modules (latent factor and its corresponding weights of pathways across cell types) can be identified by statistical analysis.: "In step 4, together with sample-level clinical metadata, disease-related multicellular pathway modules (latent factor and its corresponding weights of pathways across cell types) can be identified by"
- [readme] Downstream analyses include characterization and interpretation of multicellular pathway modules, sample/donor stratification, and visualization of high-weighted pathways: "Downstream analyses include characterization and interpretation of multicellular pathway modules, sample/donor stratification, and visualization of high-weighted pathways"
- [results] In the lupus atlas, the value of factor 1 was significantly increased in SLE patients while factor 6 was significantly decreased: [evidence span withheld — anchor status: not_found]
- [discussion] the pathways and cell types within the same module can only suggest data-driven associations, rather than determining biological causal effects: "the pathways and cell types within the same module can only suggest data-driven associations, rather than determining biological causal effects"
- [discussion] scPAFA requires studies to include at least 15 samples (not cells), which is the requirement of using the MOFA model for identifying multicellular pathway modules: "scPAFA requires studies to include at least 15 samples (not cells), which is the requirement of using the MOFA model for identifying multicellular pathway modules."
