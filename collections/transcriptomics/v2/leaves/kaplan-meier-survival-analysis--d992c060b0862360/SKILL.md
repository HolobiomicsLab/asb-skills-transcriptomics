---
name: kaplan-meier-survival-analysis
description: Use when a per-sample grouping (subtype label or risk score) must be
  validated against patient overall survival, requiring KM curves plus a log-rank
  p-value as the success criterion.
when_to_use_negative:
- Input samples lack matched clinical survival data (e.g., the single-cell validation
  sets GSE81861/GSE75688/GSE86146/GSE72056), so no KM curve can be drawn
- The analysis goal is cell-abundance estimation itself rather than survival validation
  — run DCNet/deconvolution first; KM is only the downstream validation step
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: ConsensusClusterPlus
  role: R consensus clustering (manhattan distance, PAM, 1000 iterations) that produces
    the IPS/NIPS subtype labels tested by KM/log-rank
- name: DCNet
  role: Infers the per-sample cell-type abundance landscape whose derived groups (subtypes
    or gene-signature risk score) are survival-tested
  repo: https://github.com/xindd/DCNet
- name: Python
  role: Computes per-sample 33-gene risk scores (sum of log10(hazard ratio) × transformed
    expression) and saves risk_scores.csv prior to stratification
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:571f40255945d828122ccc994a5101336e8132504778bcd69494bd09303237e0
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
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# kaplan-meier-survival-analysis

## Summary

Stratify samples by a grouping variable (e.g., DCNet-derived IPS/NIPS consensus clusters or a median-split 33-gene risk score), draw Kaplan-Meier overall-survival curves, and test separation with a log-rank test — the survival-validation backbone of the DCNet TME paper.

## When to use

Use when a per-sample grouping (subtype label or risk score) must be validated against patient overall survival, requiring KM curves plus a log-rank p-value as the success criterion.

## When NOT to use

- Input samples lack matched clinical survival data (e.g., the single-cell validation sets GSE81861/GSE75688/GSE86146/GSE72056), so no KM curve can be drawn
- The analysis goal is cell-abundance estimation itself rather than survival validation — run DCNet/deconvolution first; KM is only the downstream validation step

## Inputs

- Per-sample grouping: IPS/NIPS consensus-cluster assignments or per-sample 33-gene risk scores (risk_scores.csv)
- Clinical overall-survival metadata matched to samples (e.g., GDC TCGA clinical data; GSE78220 response/clinical metadata for 38 melanoma biopsy specimens)
- Optionally the upstream DCNet cell-abundance matrix (rows = 434 cell types, columns = samples) to feed consensus clustering

## Outputs

- Kaplan-Meier overall-survival curves per group (Figure 4b-style for IPS/NIPS; Figure 5c-style for risk groups)
- Log-rank test p-value (target p < 0.05 for risk groups; p < .0001 for IPS/NIPS)
- Wilcoxon rank-sum p-value comparing risk scores of responders vs non-responders (target ≈ 0.00051)

## How to apply

First obtain a per-sample grouping: either (a) consensus-cluster a cell-abundance landscape (e.g., 10,176 TCGA samples from DCNet) with the R package ConsensusClusterPlus using manhattan distance, PAM clustering, and 1000 iterations to recover IPS/NIPS subtypes, or (b) compute a risk score as the sum over the 33 published prognostic genes of log10(hazard ratio) × log2(x+1)-transformed expression and split samples at the median risk score. Next merge the group labels with clinical survival data from the matched source (TCGA/GDC or the 38 pre-treatment melanoma specimens of GSE78220). Plot Kaplan-Meier overall-survival curves per group and compute the log-rank test; the paper targets p < 0.05 for the 33-gene risk groups and reported p < .0001 for IPS vs NIPS with IPS patients surviving better. As a parallel check on the risk model, compare risk scores between anti-PD-1 responders and non-responders with a Wilcoxon rank-sum test, expecting significant elevation in non-responders (p ≈ 0.00051).

## Related tools

- **ConsensusClusterPlus** (R consensus clustering (manhattan distance, PAM, 1000 iterations) that produces the IPS/NIPS subtype labels tested by KM/log-rank)
- **DCNet** (Infers the per-sample cell-type abundance landscape whose derived groups (subtypes or gene-signature risk score) are survival-tested) — https://github.com/xindd/DCNet
- **Python** (Computes per-sample 33-gene risk scores (sum of log10(hazard ratio) × transformed expression) and saves risk_scores.csv prior to stratification)

## Examples

```
risk = pd.read_csv('risk_scores.csv', index_col=0); cut = risk.median(axis=1); high = risk[risk >= cut]; low = risk[risk < cut]; stats.logrank_test(high_t, low_t)
```

## Evaluation signals

- Log-rank test between median-split high/low 33-gene risk groups yields p < 0.05, matching reported Figure 5c
- Wilcoxon rank-sum test of risk scores between anti-PD-1 responders and non-responders yields p ≈ 0.00051, matching Figure 5d
- IPS vs NIPS KM curves show significant separation with log-rank p < .0001, with IPS patients surviving better
- Median split yields roughly balanced group sizes across the 38 GSE78220 samples
- Group labels merge 1:1 with clinical survival records (no sample-name mismatches after GEO/TCGA ID alignment)

## Limitations

- Only 38 pre-treatment melanoma biopsy specimens (GSE78220) support the PD-1 responder risk-score analysis, so results are sample-size limited
- No random seed, software version numbers, or reproducibility controls are reported for training or clustering, so survival-separated group assignments may vary between runs
- After the 48-patient drug-treatment threshold only 7 drugs and 740 patients were kept, limiting drug-response survival analyses

## Evidence

- [other] Stratify samples into high- and low-risk groups at the median risk score and plot Kaplan-Meier survival curves with a log-rank test, targeting a significant survival difference (p<0.05) as in Figure 5c.: [evidence span withheld — anchor status: not_found]
