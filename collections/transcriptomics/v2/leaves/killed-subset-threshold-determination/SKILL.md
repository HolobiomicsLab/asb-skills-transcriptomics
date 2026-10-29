---
name: killed-subset-threshold-determination
description: Use when estimating which tumor sub-clusters a LINCS L1000 perturbation
  kills, requiring >90% proportion reduction plus a 75% consistency check at higher
  concentrations.
when_to_use_negative:
- When predicting per-cluster IC50/drug kill via CaDRReS-Sc or PRISM AUC — this skill
  applies only to the LINCS L1000 bulk-perturbation proportion-reduction path, not
  to the Drug Response Prediction module
- When the compound's perturbation series lacks replicates or higher-concentration
  experiments, since the 75% consistency check across concentrations cannot be performed
- When sub-cluster proportions come from something other than CIBERSORTx deconvolution
  of treated/control bulk profiles — the >90% 'killed' criterion is defined on those
  estimated percentages
edam_topics:
- http://edamontology.org/topic_0208
tools:
- name: scDrug
  role: Treatment Selection module implementing the killed-subset thresholds and greedy
    combination search
  repo: https://github.com/ailabstw/scDrug
- name: CIBERSORTx
  role: Docker image cibersortx/fractions estimates per-sample proportions of each
    tumor sub-cluster whose percentage reduction defines 'killed' subsets
- name: LINCS L1000
  role: Source of bulk perturbation GEPs (treated vs. control samples) whose sub-cluster
    proportion changes are thresholded
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:e4dac10f9da142c87f20753a999b5ff310820e0396791e495faa513cd99a655d
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

# killed-subset-threshold-determination

## Summary

Classifies tumor sub-clusters as 'killed' by a LINCS L1000 perturbation in scDrug's Treatment Selection module when CIBERSORTx-estimated sub-cluster proportions drop by more than 90% after treatment, with a 75% consistency check at higher compound concentrations. These labels drive the greedy selection of minimal combined therapies.

## When to use

Use when estimating which tumor sub-clusters a LINCS L1000 perturbation kills, requiring >90% proportion reduction plus a 75% consistency check at higher concentrations.

## When NOT to use

- When predicting per-cluster IC50/drug kill via CaDRReS-Sc or PRISM AUC — this skill applies only to the LINCS L1000 bulk-perturbation proportion-reduction path, not to the Drug Response Prediction module
- When the compound's perturbation series lacks replicates or higher-concentration experiments, since the 75% consistency check across concentrations cannot be performed
- When sub-cluster proportions come from something other than CIBERSORTx deconvolution of treated/control bulk profiles — the >90% 'killed' criterion is defined on those estimated percentages

## Inputs

- CIBERSORTx-estimated cell-subset proportions for LINCS L1000 treated and control bulk samples
- Anti-logarithmic bulk GEPs of perturbation experiments on the selected reference cell line (e.g. A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC)
- Single-cell tumor sub-cluster assignments (Scanpy Anndata object, e.g. scanpyobj.h5ad)
- Sensitivity threshold (-0.9) and consistency threshold (-0.75) parameters

## Outputs

- Per-perturbation list of 'killed' tumor sub-clusters (subsets reduced >90%)
- Consistency-checked perturbation report containing only compounds whose killed subsets also satisfy the 75% higher-concentration requirement
- Killed-subset table consumed by the greedy combination search (e.g. minimal combined therapy killing 11 of 14 HCC sub-clusters)

## How to apply

After extracting anti-logarithmic bulk GEPs of perturbation experiments on the LINCS L1000 cell line most correlated (Pearson) with the single-cell cluster GEP, run CIBERSORTx (cibersortx/fractions Docker) to estimate the proportion of each tumor sub-cluster in every treated and control bulk sample. Label a sub-cluster as 'killed' by a perturbation when its percentage is reduced by more than 90% relative to control (sensitivity threshold -0.9). Then verify the consistency requirement: the reduction of the 'killed' subset (default threshold 75%) must also be observed for perturbations using the same compound at higher concentrations (consistency threshold -0.75). Feed only perturbations passing both criteria into the greedy search that iteratively picks the perturbation killing the most subpopulations with the greatest summed reduction.

## Related tools

- **scDrug** (Treatment Selection module implementing the killed-subset thresholds and greedy combination search) — https://github.com/ailabstw/scDrug
- **CIBERSORTx** (Docker image cibersortx/fractions estimates per-sample proportions of each tumor sub-cluster whose percentage reduction defines 'killed' subsets)
- **LINCS L1000** (Source of bulk perturbation GEPs (treated vs. control samples) whose sub-cluster proportion changes are thresholded)

## Examples

```
/etc/init.d/docker start; docker pull cibersortx/fractions
```

## Evaluation signals

- Every perturbation in the output report has killed subsets whose percentage reduction exceeds 90% versus control
- For each killed subset, the same reduction direction is confirmed at ≥75% for perturbations of the same compound at higher concentrations
- Greedy search is run with sensitivity threshold -0.9 and consistency threshold -0.75, matching the default killed/consistency parameters
- The resulting combination covers the maximum achievable number of the 14 hepatocellular carcinoma sub-clusters (11 in the published run)
- No perturbation is included whose killed subsets were determined without the CIBERSORTx treated-vs-control proportion comparison

## Limitations

- Prediction is limited to experimentally screened compounds in LINCS L1000
- Lack of scRNA-seq data with drug treatments makes the killed-subset calls hard to evaluate experimentally
- Thresholds are defaults (>90% reduction, 75% consistency) that users may need to tune; mis-set thresholds propagate directly into the greedy combination result

## Evidence

- [methods] Killed subset determination: [evidence span withheld — anchor status: not_found]
- [methods] Consistency threshold: [evidence span withheld — anchor status: not_found]
- [readme] Treatment Selection module: "**Treatment Selection** listed treatment combinations of given cell clusters."
- [readme] CIBERSORTx fractions Docker used in treatment selection: "pull the Docker image `cibersortx/fractions` used in treatment selection."
