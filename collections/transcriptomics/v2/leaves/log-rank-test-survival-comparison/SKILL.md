---
name: log-rank-test-survival-comparison
description: Use when a workflow has partitioned patients (by consensus clustering,
  risk-score median split, or classifier label) and must show that the groups differ
  significantly in overall survival via Kaplan–Meier curves plus a log-rank test.
when_to_use_negative:
- Comparing risk scores between responders and non-responders — that contrast uses
  a Wilcoxon rank-sum test (p=0.00051), not a log-rank test
- Patient grouping is not yet defined; the log-rank test requires pre-existing binary
  labels, not the clustering step itself
- Outcome data are non-time-to-event (e.g., binary response) — survival analysis is
  inapplicable
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: ConsensusClusterPlus
  role: Produces the IPS/NIPS patient subtype labels whose survival curves are compared
    by the log-rank test
- name: Python
  role: Computes the 33-gene risk scores and performs the Kaplan–Meier/log-rank survival
    analysis (Figure 5c) on GSE78220 samples
- name: DCNet
  role: Generates the cell-type abundance landscape whose consensus-derived IPS/NIPS
    subtypes are survival-tested
  repo: https://github.com/xindd/DCNet
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:fc1184d0f83cfc41d4d018d3bc45a9a193bac34f4acc1bf419181b25dc455b4a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - ConsensusClusterPlus
  - DCNet
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# log-rank-test-survival-comparison

## Summary

Compares overall-survival curves between two stratified patient groups (e.g., DCNet-derived IPS vs NIPS TME subtypes, or 33-gene high- vs low-risk groups) using Kaplan–Meier curves and the log-rank test, the paper's standard for validating survival separation.

## When to use

Use when a workflow has partitioned patients (by consensus clustering, risk-score median split, or classifier label) and must show that the groups differ significantly in overall survival via Kaplan–Meier curves plus a log-rank test.

## When NOT to use

- Comparing risk scores between responders and non-responders — that contrast uses a Wilcoxon rank-sum test (p=0.00051), not a log-rank test
- Patient grouping is not yet defined; the log-rank test requires pre-existing binary labels, not the clustering step itself
- Outcome data are non-time-to-event (e.g., binary response) — survival analysis is inapplicable

## Inputs

- Per-sample survival metadata (overall survival time and censoring status), e.g., TCGA clinical data from GDC or GSE78220 clinical/response metadata
- Per-sample group labels: IPS/NIPS subtype assignments from ConsensusClusterPlus, or high/low-risk labels from the 33-gene risk-score median split
- risk_scores.csv with per-sample 33-gene risk scores (sum of log10(hazard ratio) x log2(x+1)-transformed expression)

## Outputs

- Kaplan–Meier overall-survival curves for the two groups (Figure 4b / Figure 5c analogs)
- Log-rank test p-value quantifying survival separation
- Survival-stratified sample table merging subtype/risk labels with clinical outcomes

## How to apply

First merge per-sample group labels (e.g., ConsensusClusterPlus IPS/NIPS assignments over the 10,176-sample TCGA abundance landscape, or high/low-risk labels from the 33-gene score split at the median) with TCGA or GEO clinical survival metadata. Fit Kaplan–Meier overall-survival curves for each group and compute the log-rank test across the curves. In the article, the IPS/NIPS contrast yielded log-rank p < .0001 (Figure 4b), while the GSE78220 PD1-treated cohort's high/low risk split had to reach p < 0.05 to match Figure 5c. Judgment of correctness is against these significance targets: a non-significant log-rank result means the upstream grouping (clustering or risk-score genes/hazard ratios) is not reproducing the reported separation.

## Related tools

- **ConsensusClusterPlus** (Produces the IPS/NIPS patient subtype labels whose survival curves are compared by the log-rank test)
- **Python** (Computes the 33-gene risk scores and performs the Kaplan–Meier/log-rank survival analysis (Figure 5c) on GSE78220 samples)
- **DCNet** (Generates the cell-type abundance landscape whose consensus-derived IPS/NIPS subtypes are survival-tested) — https://github.com/xindd/DCNet

## Examples

```
gse = GEOparse.get_GEO('GSE78220'); score = sum(log10(hr)*log2(expr[g]+1) for g in genes33); Rscript: survdiff(Surv(OS, event) ~ risk_group, data=meta) with survfit(Surv(OS, event) ~ risk_group)
```

## Evaluation signals

- IPS vs NIPS log-rank test on TCGA overall survival must reach p < .0001, reproducing Figure 4b
- High vs low 33-gene-risk groups in GSE78220 must yield log-rank p < 0.05, matching Figure 5c
- Risk-score split must be at the median so high/low groups are approximately equal in size
- Kaplan–Meier curves must be drawn per group with censoring marks and survival time on a common axis
- Merged survival table must cover the full cohort (10,176 TCGA tumor samples or 38 GSE78220 pre-treatment specimens)

## Limitations

- The log-rank test only detects differences in survival distribution shape/duration; it does not validate the biological basis of the grouping (e.g., cell-type composition differences)
- Reported significance depends on upstream grouping reproducibility; the paper reports no random seed or trained model checkpoints for DCNet, so group assignments may vary across reruns
- Only overall survival is compared; no progression-free or disease-specific endpoints are described

## Evidence

- [methods] Merge the subtype labels with TCGA clinical survival data, draw Kaplan-Meier overall-survival curves for the two groups, and compute the log-rank test.: [evidence span withheld — anchor status: not_found]
- [results] consensus clustering identified IPS and NIPS subtypes that differ significantly in survival (log-rank p < .0001): [evidence span withheld — anchor status: not_found]
- [methods] Stratify samples into high- and low-risk groups at the median risk score and plot Kaplan-Meier survival curves with a log-rank test: [evidence span withheld — anchor status: not_found]
