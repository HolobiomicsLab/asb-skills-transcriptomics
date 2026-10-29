---
name: single-cell-bulk-gep-correlation-matching
description: Use when you have scRNA-seq-derived tumor sub-cluster GEPs plus LINCS
  L1000 bulk perturbation profiles and need to identify drug combinations killing
  the maximum number of tumor sub-clusters.
when_to_use_negative:
- When you need de novo prediction of drug sensitivity — the authors note 'Future
  studies on de novo prediction of drug sensitivity and extensive experimental validations
  of the pipeline are necessary'
- When the drug of interest was not experimentally screened — 'The prediction of drug
  response in scDrug is limited by experimentally screened compounds'
- When only monotherapy sensitivity per cluster is needed — use the CaDRReS-Sc/PRISM
  IC50-based Drug Response Prediction module instead of LINCS-based Treatment Selection
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scDrug
  role: Workflow container executing Single-Cell Data Analysis (generating the cluster
    GEP file) and Treatment Selection
  repo: https://github.com/ailabstw/scDrug
- name: CIBERSORTx
  role: Cell Fractions Docker (cibersortx/fractions) estimating the proportion of
    each tumor sub-cluster in every LINCS bulk sample
- name: LINCS L1000
  role: Source of bulk anti-logarithmic GEPs for perturbation experiments on the selected
    reference cell line
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:25bf1aa5312d8ddaeb0655358ef8c4d542fc2b473c9843df1ae0388e0ccb1b3a
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# single-cell-bulk-gep-correlation-matching

## Summary

Matches scRNA-seq tumor sub-cluster gene expression profiles (GEPs) to bulk LINCS L1000 perturbation data via Pearson correlation to a reference cell line, then uses CIBERSORTx to deconvolve sub-cluster proportions in treated vs control bulk samples and greedily select combined treatments that kill the most sub-clusters.

## When to use

Use when you have scRNA-seq-derived tumor sub-cluster GEPs plus LINCS L1000 bulk perturbation profiles and need to identify drug combinations killing the maximum number of tumor sub-clusters.

## When NOT to use

- When you need de novo prediction of drug sensitivity — the authors note 'Future studies on de novo prediction of drug sensitivity and extensive experimental validations of the pipeline are necessary'
- When the drug of interest was not experimentally screened — 'The prediction of drug response in scDrug is limited by experimentally screened compounds'
- When only monotherapy sensitivity per cluster is needed — use the CaDRReS-Sc/PRISM IC50-based Drug Response Prediction module instead of LINCS-based Treatment Selection

## Inputs

- Single-cell cluster gene expression profile (GEP) file (GEP.txt) from Single-Cell Data Analysis
- Bulk GEPs of perturbation experiments for eight reference cell lines from LINCS L1000 (anti-logarithmic)
- CIBERSORTx Cell Fractions Docker (cibersortx/fractions)

## Outputs

- Treatment combination report listing perturbations that kill tumor sub-clusters
- Per-sub-cluster proportion estimates across treated and control bulk samples (CIBERSORTx fractions)
- Killed-subset assignments per perturbation under >90% reduction and 75% consistency criteria

## How to apply

Compute the Pearson correlation between the single-cell cluster GEP and the bulk GEP of each of the eight reference cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC) and select the highest-correlation cell line. Extract anti-logarithmic bulk GEPs of all perturbation experiments on that cell line from LINCS L1000, and run CIBERSORTx (cibersortx/fractions Docker) to estimate each tumor sub-cluster's proportion in every bulk sample. Label a sub-cluster 'killed' when its percentage is reduced more than 90% after perturbation, with a consistency requirement that the reduction (default: 75%) is also observed for the same compound at higher concentrations. Run a greedy search: iteratively pick the perturbation killing the most subpopulations with the greatest summed reduction (sensitivity threshold -0.9, consistency threshold -0.75), remove killed subpopulations, and repeat until no more can be killed. The rationale is that bulk perturbation signatures, when deconvolved against cluster GEPs, reveal which tumor sub-clusters a drug eliminates in silico, enabling minimal combination therapy selection.

## Related tools

- **scDrug** (Workflow container executing Single-Cell Data Analysis (generating the cluster GEP file) and Treatment Selection) — https://github.com/ailabstw/scDrug
- **CIBERSORTx** (Cell Fractions Docker (cibersortx/fractions) estimating the proportion of each tumor sub-cluster in every LINCS bulk sample)
- **LINCS L1000** (Source of bulk anti-logarithmic GEPs for perturbation experiments on the selected reference cell line)

## Examples

```
docker pull cibersortx/fractions
```

## Evaluation signals

- Output report contains only perturbations whose killed subsets satisfy the >90% percentage-reduction criterion
- Every killed subset also shows at least 75% reduction for the same compound at higher concentrations (consistency check)
- The greedy combination covers the maximum achievable number of the 14 tumor sub-clusters (11/14 in the hepatocellular carcinoma case)
- The selected reference cell line is the one with the highest Pearson correlation between the single-cell cluster GEP and its bulk GEP among the eight candidates
- CIBERSORTx fraction outputs are produced for both treated and control LINCS bulk samples before comparison

## Limitations

- Prediction is limited by experimentally screened compounds in LINCS
- Available scRNA-seq data with drug treatments are too short for experimental evaluation
- Requires docker-in-docker to run the CIBERSORTx fractions container inside scDrug, or an external shared-mounted container if docker-in-docker is unavailable

## Evidence

- [methods] killed subset threshold: [evidence span withheld — anchor status: not_found]
- [methods] consistency threshold: [evidence span withheld — anchor status: not_found]
- [methods] LINCS-based combination strategy: "predict potential combined treatment strategies using the LINCS L1000 data [17]"
- [readme] CIBERSORTx container for treatment selection: "In the Docker container `scDrug`, pull the Docker image `cibersortx/fractions` used in treatment selection."
- [readme] Treatment Selection module output: "**Treatment Selection** listed treatment combinations of given cell clusters."
