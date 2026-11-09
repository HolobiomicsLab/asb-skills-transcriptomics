---
name: greedy-combination-treatment-search
description: Use when a set of tumor sub-cluster gene expression profiles (GEPs) must
  be matched against LINCS L1000 perturbation bulk profiles to build a minimal combination
  therapy covering as many sub-clusters as possible.
when_to_use_negative:
- When the goal is single-agent IC50/drug-kill prediction per cluster rather than
  combination therapy — use scDrug's Drug Response Prediction module (CaDRReS-Sc/GDSC
  or PRISM) instead.
- When the perturbation bulk data cannot be linked to a LINCS L1000 reference cell
  line or the scDrug prediction is limited to experimentally screened compounds and
  a de novo compound is required.
- When input is raw scRNA-seq data without sub-clusters and GEPs computed yet — run
  Single-Cell Data Analysis (sub-clustering, --GEP) first.
edam_topics:
- http://edamontology.org/topic_0208
tools:
- name: scDrug
  role: Hosts the Treatment Selection module implementing the greedy combination search
    over LINCS L1000 perturbations
  repo: https://github.com/ailabstw/scDrug
- name: Premnas
  role: Premnas-based combination treatment selection logic applied to LINCS L1000
    profiles
- name: CIBERSORTx
  role: Estimates the proportion of each tumor sub-cluster in every LINCS bulk perturbation
    sample (cibersortx/fractions Docker)
- name: LINCS L1000
  role: Source of perturbation bulk gene expression profiles used to assess treated-vs-control
    sub-cluster proportion changes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:5c23c99e6aa92062f2a7b56b2ec878104d0aaf6d57fa575210fd013ce10f2b59
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0208
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# greedy-combination-treatment-search

## Summary

A greedy search over LINCS L1000 perturbation signatures that identifies a minimal drug combination killing the maximum number of tumor sub-clusters, as implemented in scDrug's Premnas-based Treatment Selection module. Sub-clusters are deconvolved from bulk perturbation samples with CIBERSORTx and labeled 'killed' when a compound reduces their proportion by >90%.

## When to use

Use when a set of tumor sub-cluster gene expression profiles (GEPs) must be matched against LINCS L1000 perturbation bulk profiles to build a minimal combination therapy covering as many sub-clusters as possible.

## When NOT to use

- When the goal is single-agent IC50/drug-kill prediction per cluster rather than combination therapy — use scDrug's Drug Response Prediction module (CaDRReS-Sc/GDSC or PRISM) instead.
- When the perturbation bulk data cannot be linked to a LINCS L1000 reference cell line or the scDrug prediction is limited to experimentally screened compounds and a de novo compound is required.
- When input is raw scRNA-seq data without sub-clusters and GEPs computed yet — run Single-Cell Data Analysis (sub-clustering, --GEP) first.

## Inputs

- Single-cell cluster gene expression profile file (GEP.txt) from scDrug Single-Cell Data Analysis
- LINCS L1000 anti-logarithmic bulk GEPs of perturbation experiments on the selected reference cell line
- Tumor sub-cluster count/identity (e.g. the 14 hepatocellular carcinoma sub-clusters)
- CIBERSORTx Cell Fractions Docker image (cibersortx/fractions)

## Outputs

- Treatment combination report listing selected perturbations and their killed sub-cluster subsets
- Per-perturbation sub-cluster proportion changes and consistency annotations
- Final minimal combination therapy covering the maximum achievable number of sub-clusters (11 of 14 for the HCC example)

## How to apply

Compute the Pearson correlation between the single-cell cluster GEP (e.g. GEP.txt from Single-Cell Data Analysis) and the bulk GEPs of the eight reference cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC) and select the best-matching cell line. Extract anti-logarithmic bulk GEPs of all perturbation experiments on that cell line from LINCS L1000 and estimate the proportion of each tumor sub-cluster per bulk sample with CIBERSORTx (cibersortx/fractions Docker). Label a sub-cluster 'killed' when its percentage drops more than 90% after perturbation, requiring a consistency check that the killed subset's reduction (default 75%) also occurs with the same compound at higher concentrations, using sensitivity threshold -0.9 and consistency threshold -0.75. Then run the greedy search: iteratively pick the perturbation killing the most subpopulations with the greatest summed reduction, remove killed subpopulations, and repeat until no more can be killed; validate that the final report contains only perturbations satisfying both the >90% and 75% consistency criteria.

## Related tools

- **scDrug** (Hosts the Treatment Selection module implementing the greedy combination search over LINCS L1000 perturbations) — https://github.com/ailabstw/scDrug
- **Premnas** (Premnas-based combination treatment selection logic applied to LINCS L1000 profiles)
- **CIBERSORTx** (Estimates the proportion of each tumor sub-cluster in every LINCS bulk perturbation sample (cibersortx/fractions Docker))
- **LINCS L1000** (Source of perturbation bulk gene expression profiles used to assess treated-vs-control sub-cluster proportion changes)

## Examples

```
/etc/init.d/docker start; docker pull cibersortx/fractions
```

## Evaluation signals

- Every perturbation in the output report has killed subsets reduced by more than 90% in cell proportion.
- For each reported killed subset, the same reduction (default 75% threshold) is observed for the same compound at higher concentrations (consistency check).
- The greedy iteration parameters match the defaults: sensitivity threshold -0.9 and consistency threshold -0.75.
- The final combination covers the maximum achievable number of sub-clusters; for the hepatocellular carcinoma case this is 11 of the 14 sub-clusters.
- The selected reference cell line is the one with the highest Pearson correlation to the single-cell cluster GEP among A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, and YAPC.

## Limitations

- The prediction of drug response in scDrug is limited by experimentally screened compounds — no de novo sensitivity prediction.
- Available scRNA-seq data with drug treatments are short for evaluation, so extensive experimental validations of the pipeline are still necessary.
- The 'killed' labeling is purely a proportion-change heuristic (>90% reduction in CIBERSORTx-inferred fraction), not a direct measure of cell death.

## Evidence

- [methods] killed subset determination: [evidence span withheld — anchor status: not_found]
- [methods] consistency threshold: [evidence span withheld — anchor status: not_found]
- [readme] CIBERSORTx setup: "In the Docker container `scDrug`, pull the Docker image `cibersortx/fractions` used in treatment selection."
- [readme] Treatment Selection module: "**Treatment Selection** listed treatment combinations of given cell clusters."
