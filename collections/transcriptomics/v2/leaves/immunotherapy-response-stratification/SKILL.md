---
name: immunotherapy-response-stratification
description: Use when you have pre-treatment bulk RNA-seq from immunotherapy (anti-PD-1)
  patients with survival and clinical response metadata, and need to test whether
  a published gene-signature risk score separates responders from non-responders.
when_to_use_negative:
- The dataset has no paired clinical survival or treatment-response metadata, so neither
  the Kaplan-Meier split nor the responder/non-responder Wilcoxon comparison can be
  evaluated
- The goal is deconvolving cell-type abundances from bulk RNA-seq (that is DCNet's
  own task), not scoring a gene signature for patient stratification
- Expression values are already log-transformed by the provider — a second log2(x+1)
  transformation would distort the risk-score terms
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3517
tools:
- name: DCNet
  role: Upstream interpretable deep-learning framework that embeds relationships between
    434 cell types and their marker genes; the 33-gene signature derives from differentially
    expressed marker genes of DCNet-defined cell types
  repo: https://github.com/xindd/DCNet
- name: Python
  role: Implementation language for GEOparse download, log2(x+1) preprocessing, risk-score
    computation, and KM/log-rank/Wilcoxon testing
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:418e040f5e12eb0799240611b0e94e3f9fa150af1bf2e5ac29db2f251103265d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# immunotherapy-response-stratification

## Summary

Stratifies pre-treatment melanoma patients into anti-PD-1 responder-relevant high- and low-risk groups by scoring a 33-gene prognostic signature (log10 hazard ratio × log2(x+1) expression) on bulk RNA-seq and testing Kaplan-Meier survival and response-group differences.

## When to use

Use when you have pre-treatment bulk RNA-seq from immunotherapy (anti-PD-1) patients with survival and clinical response metadata, and need to test whether a published gene-signature risk score separates responders from non-responders.

## When NOT to use

- The dataset has no paired clinical survival or treatment-response metadata, so neither the Kaplan-Meier split nor the responder/non-responder Wilcoxon comparison can be evaluated
- The goal is deconvolving cell-type abundances from bulk RNA-seq (that is DCNet's own task), not scoring a gene signature for patient stratification
- Expression values are already log-transformed by the provider — a second log2(x+1) transformation would distort the risk-score terms

## Inputs

- GSE78220 bulk transcriptome matrix for 38 pre-treatment melanoma biopsy specimens (rows = human gene symbols, columns = samples)
- Clinical metadata: overall survival times and anti-PD-1 responder/non-responder labels for the 38 samples
- The 33-gene prognostic signature with per-gene hazard ratios (log10 HR coefficients)

## Outputs

- risk_scores.csv: per-sample 33-gene risk scores
- High- and low-risk patient groups (median risk-score split)
- Kaplan-Meier survival curves with log-rank p-value (Figure 5c analogue)
- Wilcoxon rank-sum comparison of risk scores between responders and non-responders (Figure 5d analogue)

## How to apply

Download the GSE78220 transcriptome matrix and clinical/response metadata for the 38 pre-treatment melanoma biopsy specimens via GEOparse, orient rows as human gene symbols and columns as samples, and apply the recommended log2(x+1) transformation. Compute each sample's risk score as the sum over the 33 published prognostic genes of log10(hazard ratio) multiplied by the transformed expression, and write per-sample scores to risk_scores.csv. Split samples at the median risk score into high/low-risk groups and run Kaplan-Meier curves with a log-rank test, then compare risk scores between anti-PD-1 responders and non-responders with a Wilcoxon rank-sum test. The signature derives from differentially expressed marker genes of DCNet-defined cell types, so the DCNet cell-landscape framework (434 cell types, 9078 marker genes) is the biological context linking genes to tumor microenvironment composition.

## Related tools

- **DCNet** (Upstream interpretable deep-learning framework that embeds relationships between 434 cell types and their marker genes; the 33-gene signature derives from differentially expressed marker genes of DCNet-defined cell types) — https://github.com/xindd/DCNet
- **Python** (Implementation language for GEOparse download, log2(x+1) preprocessing, risk-score computation, and KM/log-rank/Wilcoxon testing)

## Examples

```
import pandas as pd, numpy as np; expr = np.log2(pd.read_csv('gse78220_expr.csv', index_col=0) + 1); risk = expr.loc[genes33].mul(np.log10(hr[genes33], axis=0)).sum(); risk.to_csv('risk_scores.csv')
```

## Evaluation signals

- Log-rank test between median-split high/low-risk groups must yield p < 0.05, matching reported Figure 5c
- Wilcoxon rank-sum test of responder vs non-responder risk scores must give p ≈ 0.00051, with non-responders elevated, matching reported Figure 5d
- risk_scores.csv must contain one score per each of the 38 GSE78220 pre-treatment samples
- Expression matrix must be oriented rows = human gene symbols, columns = samples, and transformed as log2(x+1) before scoring
- Risk score values must equal the sum over exactly 33 genes of log10(HR) × transformed expression per sample

## Limitations

- Only 7 drugs and 740 drug-treatment patients were retained after the article's 48-patient-per-drug threshold, limiting immunotherapy drug-response analyses
- The published validation relies on a single small cohort (38 GSE78220 melanoma specimens); no independent anti-PD-1 replication cohort is provided in the workflow
- No random seed or reproducibility controls are reported for the upstream DCNet training, and trained model weights are only indirectly available via the DCNet-Use .params file

## Evidence

- [methods] Compute each sample's risk score as the sum over the 33 published prognostic genes of log10(hazard ratio) multiplied by the transformed gene expression.: [evidence span withheld — anchor status: not_found]
- [results] High-risk group survival finding: [evidence span withheld — anchor status: not_found]
- [readme] README input orientation: [evidence span withheld — anchor status: not_found]
