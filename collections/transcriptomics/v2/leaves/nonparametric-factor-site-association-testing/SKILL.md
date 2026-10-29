---
name: nonparametric-factor-site-association-testing
description: Use when trained MOFA latent factors (e.g., factors 4/5/6 from the 40-sample
  GSE164522 CRC-CRLM cohort) must be tested for association with sample-level clinical
  metadata such as tumor site, donor, or tumor status.
when_to_use_negative:
- The cohort has fewer than 15 samples (not cells) — scPAFA requires at least 15 samples
  to run MOFA for multicellular pathway module identification
- The analysis goal is causal inference — factor-module associations are data-driven,
  not biological causal effects
- The input is raw single-cell expression data that has not yet passed through PAS
  computation (fast_Ucell) and pseudobulk MOFA training (steps 1–3)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scPAFA
  role: Generates the PAS matrix (fast_Ucell) and trains the MOFA model whose latent
    factors are subsequently tested against sample sites
  repo: https://github.com/ZhuoliHuang/scPAFA
- name: MOFA
  role: Multi-omics factor analysis framework (multi-group MOFA+ when batch effects
    are known) whose trained latent factors and weights are the input to site-association
    testing
  repo: https://github.com/bioFAM/mofapy2
- name: Python
  role: Language in which scPAFA is implemented and in which the Kruskal-Wallis and
    Mann-Whitney U tests are run on factor scores joined with clinical metadata
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:f93cd464bf0f95db2ce21b471961ff6739df0da18d7dfbab2c5869de9be49166
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhuoliHuang/scPAFA
  tools:
  - MOFA
  - Python
  - UCell
  - scPAFA
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# nonparametric-factor-site-association-testing

## Summary

Statistical testing of scPAFA/MOFA latent factors against sample-level clinical metadata (e.g., primary tumor/normal vs. metastatic tumor/normal sites) using nonparametric Kruskal-Wallis and Mann-Whitney U tests, used in the scPAFA paper to validate disease-related multicellular pathway modules in the CRC-CRLM dataset.

## When to use

Use when trained MOFA latent factors (e.g., factors 4/5/6 from the 40-sample GSE164522 CRC-CRLM cohort) must be tested for association with sample-level clinical metadata such as tumor site, donor, or tumor status.

## When NOT to use

- The cohort has fewer than 15 samples (not cells) — scPAFA requires at least 15 samples to run MOFA for multicellular pathway module identification
- The analysis goal is causal inference — factor-module associations are data-driven, not biological causal effects
- The input is raw single-cell expression data that has not yet passed through PAS computation (fast_Ucell) and pseudobulk MOFA training (steps 1–3)

## Inputs

- Trained MOFA model (mofapy2) containing sample-level latent factor values and pathway weights across cell types
- Sample-level clinical metadata table (sample/donor, cell type, technical batch, sample site: primary tumor/normal vs. metastatic tumor/normal, tumor status)
- Pseudobulk PAS long-table pandas dataframe used as mofapy2 input (40 samples, 10 patients)

## Outputs

- Kruskal-Wallis test statistics and p-values per latent factor across tumor sites
- Pairwise Mann-Whitney U test statistics and p-values between site groups
- Sample/donor stratification plots of factor values and high-weighted pathway visualizations for each disease-related module

## How to apply

After scPAFA step 3 has trained the MOFA model on the pseudobulk pathway activity score (PAS) matrix (built via 'fast_Ucell' on the 163,347-cell GSE164522 dataset against the NCATS BioPlanet pathway dictionary), extract the latent factor values per sample (e.g., factors 4, 5, and 6) along with their pathway weights across cell types. Join factor scores with sample-level clinical metadata (sample/donor, sample site, tumor status). Group samples by site — primary tumor/normal vs. metastatic tumor/normal — and test each factor with a Kruskal-Wallis test across all site groups, followed by Mann-Whitney U tests for pairwise site comparisons. Because the cohort has only 40 samples from 10 patients, nonparametric rank-based tests are appropriate rather than parametric ANOVA. Validation is performed by confirming the factor-site significance patterns reproduce Fig. 4 of the scPAFA paper, where factors 4 and 6 are metastasis-related and factor 5 is malignant-related modules.

## Related tools

- **scPAFA** (Generates the PAS matrix (fast_Ucell) and trains the MOFA model whose latent factors are subsequently tested against sample sites) — https://github.com/ZhuoliHuang/scPAFA
- **MOFA** (Multi-omics factor analysis framework (multi-group MOFA+ when batch effects are known) whose trained latent factors and weights are the input to site-association testing) — https://github.com/bioFAM/mofapy2
- **Python** (Language in which scPAFA is implemented and in which the Kruskal-Wallis and Mann-Whitney U tests are run on factor scores joined with clinical metadata)

## Examples

```
from scipy.stats import kruskal, mannwhitneyu; stat, p = kruskal(*[factors.loc[metadata.site == s, 'Factor4'] for s in ['primary_tumor','primary_normal','metastatic_tumor','metastatic_normal']])
```

## Evaluation signals

- Kruskal-Wallis and Mann-Whitney U tests across primary tumor/normal vs. metastatic tumor/normal sites must reproduce the significance patterns reported in Fig. 4 of the scPAFA paper
- Factors 4 and 6 should stratify samples as metastasis-related modules; factor 5 should stratify as a malignant-related module
- Sample/donor stratification plots should separate samples by site and tumor status when colored by factor values
- Each tested factor must retain interpretable high-weighted pathways across cell types, consistent with the module interpretation
- The factor-score join must operate at the sample level (40 samples), not the cell level, matching the pseudobulk design

## Limitations

- MOFA is a linear model; interpretability is achieved by sacrificing information content in each factor, and interpretability may decrease with multiple disease states
- The pathways and cell types within the same module only suggest data-driven associations, rather than determining biological causal effects
- Pseudobulk integration of the PAS matrix leads to information loss since the PAS of multiple cells is aggregated

## Evidence

- [readme] disease-related multicellular pathway modules (latent factor and its corresponding weights of pathways across cell types) can be identified by statistical analysis: "In step 4, together with sample-level clinical metadata, disease-related multicellular pathway modules (latent factor and its corresponding weights of pathways across cell types) can be identified by"
- [readme] Downstream analyses include characterization and interpretation of multicellular pathway modules, sample/donor stratification, and visualization of high-weighted pathways: "Downstream analyses include characterization and interpretation of multicellular pathway modules, sample/donor stratification, and visualization of high-weighted pathways"
- [methods] MOFA model is trained to capture variance in PAS among different samples: "MOFA model is trained to capture variance in PAS among different samples"
- [discussion] the pathways and cell types within the same module can only suggest data-driven associations, rather than determining biological causal effects.: "the pathways and cell types within the same module can only suggest data-driven associations, rather than determining biological causal effects."
- [discussion] MOFA is a linear model, interpretability is achieved by sacrificing information content in each factor: "Besides, MOFA is a linear model, interpretability is achieved by sacrificing information content in each factor"
- [abstract] further identified reliable and interpretable multicellular pathway modules that capture the heterogeneity of CRC and transcriptional abnormalities in lupus patients: "further identified reliable and interpretable multicellular pathway modules that capture the heterogeneity of CRC and transcriptional abnormalities in lupus patients"
