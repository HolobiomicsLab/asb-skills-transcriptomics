---
name: bulk-rna-deconvolution-cell-fraction-estimation
description: Use when single-cell cluster gene expression profiles (GEPs) must be
  mapped onto bulk drug-perturbed transcriptomes (e.g. LINCS L1000 anti-log GEPs)
  to quantify per-sub-cluster proportions before and after treatment.
when_to_use_negative:
- Input is single-cell-level data for individual cells rather than bulk samples —
  proportions across a mixed bulk population cannot be estimated from pure single-cell
  matrices alone
- No suitable reference cell line exists among the eight LINCS reference GEPs (low
  Pearson correlation with all), so deconvolution signatures would be unreliable
- The goal is IC50/drug-sensitivity prediction from GDSC or PRISM models (CaDRReS-Sc,
  DepMap) rather than mapping sub-cluster proportions onto bulk perturbation profiles
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0607
tools:
- name: scDrug
  role: Hosts the Treatment Selection workflow that prepares cluster GEPs and bulk
    LINCS GEPs for deconvolution
  repo: https://github.com/ailabstw/scDrug
- name: CIBERSORTx
  role: Cell Fractions Docker (cibersortx/fractions) that estimates the proportion
    of each tumor sub-cluster in bulk perturbation samples
- name: LINCS L1000
  role: Source of bulk perturbation GEPs and the eight reference cell lines correlated
    against single-cell cluster GEPs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:b2bf0b8c771e8ec0d4090f342908b2479f28885792f02650a7e9986ec6fab80f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - CIBERSORTx
  - LINCS L1000
  - Premnas
  - Python
  - scDrug
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0607
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Bulk RNA deconvolution cell fraction estimation

## Summary

Estimates the proportion of each single-cell-derived tumor sub-cluster within bulk perturbation samples using CIBERSORTx cell-fraction estimation, enabling LINCS L1000 drug perturbations to be linked back to individual tumor sub-clusters in scDrug's Treatment Selection module.

## When to use

Use when single-cell cluster gene expression profiles (GEPs) must be mapped onto bulk drug-perturbed transcriptomes (e.g. LINCS L1000 anti-log GEPs) to quantify per-sub-cluster proportions before and after treatment.

## When NOT to use

- Input is single-cell-level data for individual cells rather than bulk samples — proportions across a mixed bulk population cannot be estimated from pure single-cell matrices alone
- No suitable reference cell line exists among the eight LINCS reference GEPs (low Pearson correlation with all), so deconvolution signatures would be unreliable
- The goal is IC50/drug-sensitivity prediction from GDSC or PRISM models (CaDRReS-Sc, DepMap) rather than mapping sub-cluster proportions onto bulk perturbation profiles

## Inputs

- Single-cell cluster gene expression profile (GEP) file, e.g. GEP.txt from scDrug Single-Cell Data Analysis
- Anti-logarithmic bulk GEPs of LINCS L1000 perturbation experiments on the highest-correlation reference cell line
- Bulk GEPs of eight LINCS reference cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC)
- Docker image cibersortx/fractions (CIBERSORTx Cell Fractions)

## Outputs

- Per-sample cell/sub-cluster fraction table from CIBERSORTx (proportion of each tumor sub-cluster in each treated and control bulk sample)
- Treated-vs-control proportion differences used to label sub-clusters as 'killed' (>90% reduction, 75% consistency at higher concentrations)

## How to apply

First compute the Pearson correlation between the single-cell cluster GEP and the bulk GEP of each of the eight LINCS reference cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC) and select the reference cell type with the highest correlation. Extract the anti-logarithmic bulk GEPs of all perturbation experiments on that cell line from LINCS L1000, then run the CIBERSORTx Cell Fractions Docker image (cibersortx/fractions) to estimate the proportion of each tumor sub-cluster in every treated and control bulk sample. The resulting per-sub-cluster fraction tables are the input to downstream kill-criterion analysis, where a sub-cluster is 'killed' when its percentage drops more than 90% after perturbation, with a 75% consistency check at higher compound concentrations.

## Related tools

- **scDrug** (Hosts the Treatment Selection workflow that prepares cluster GEPs and bulk LINCS GEPs for deconvolution) — https://github.com/ailabstw/scDrug
- **CIBERSORTx** (Cell Fractions Docker (cibersortx/fractions) that estimates the proportion of each tumor sub-cluster in bulk perturbation samples)
- **LINCS L1000** (Source of bulk perturbation GEPs and the eight reference cell lines correlated against single-cell cluster GEPs)

## Examples

```
docker pull cibersortx/fractions
```

## Evaluation signals

- The selected reference cell line has the highest Pearson correlation against the single-cell cluster GEP among all eight LINCS reference lines
- Estimated sub-cluster fractions per bulk sample are valid proportions (each in [0,1]) and each sample's fractions are interpretable against its treated/control status
- Downstream killed-subset labels satisfy the >90% reduction criterion and the 75% consistency requirement for the same compound at higher concentrations
- The CIBERSORTx fractions container ran successfully (cibersortx/fractions pulled and executed inside or alongside the scDrug Docker container with a shared mounted folder)

## Limitations

- Deconvolution quality depends on matching to one of only eight reference cell lines; poor correlation between cluster GEP and all references undermines proportion estimates
- Drug response prediction in scDrug is limited to experimentally screened compounds, so deconvolution-based treatment selection inherits that compound coverage limit
- If docker-in-docker is unavailable, the CIBERSORTx container must be run outside the scDrug container with a shared mounted folder, complicating the setup

## Evidence

- [methods] Extract the anti-logarithmic bulk GEPs of all perturbation experiments on the selected cell line from the LINCS L1000 database and estimate the proportion of each tumor sub-cluster in every bulk sample with CIBERSORTx: [evidence span withheld — anchor status: not_found]
- [methods] Compute the Pearson correlation between the single-cell cluster GEP and the bulk GEP of each of the eight reference cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC): [evidence span withheld — anchor status: not_found]
- [methods] The cell subsets whose percentage is reduced more than 90 % after perturbation were considered "killed" by that perturbation.: [evidence span withheld — anchor status: not_found]
- [readme] In the Docker container `scDrug`, pull the Docker image `cibersortx/fractions` used in treatment selection.: "In the Docker container `scDrug`, pull the Docker image `cibersortx/fractions` used in treatment selection."
- [readme] **Treatment Selection** listed treatment combinations of given cell clusters.: "**Treatment Selection** listed treatment combinations of given cell clusters."
