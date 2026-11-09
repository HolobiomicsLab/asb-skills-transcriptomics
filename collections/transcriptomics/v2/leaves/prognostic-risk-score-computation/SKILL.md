---
name: prognostic-risk-score-computation
description: Use when bulk RNA-seq samples with survival and treatment-response metadata
  must be stratified by a published prognostic gene signature into survival-differentiated
  high/low risk groups.
when_to_use_negative:
- Input is single-cell or already-deconvolved cell-abundance data rather than a bulk
  RNA-seq matrix with human gene symbol rows
- No per-gene hazard ratios are available for the candidate signature, so the log10(HR)-weighted
  risk score cannot be computed
- Samples lack survival endpoints and treatment-response labels, making Kaplan-Meier
  and Wilcoxon validation impossible
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DCNet
  role: Source of the cell-type marker genes from which the 33-gene prognostic signature
    was derived; its input conventions (gene-symbol rows, log2(x+1) transformation)
    define the preprocessing for the risk score
  repo: https://github.com/xindd/DCNet
- name: Python
  role: Implementation language for downloading GSE78220 via GEOparse, computing the
    risk score, and running Kaplan-Meier/log-rank and Wilcoxon tests
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:430b02f8098af255574effd063fd70201998763f72d396c462f7807ecec19b24
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# prognostic-risk-score-computation

## Summary

Compute a per-sample prognostic risk score from a published 33-gene signature by summing log10(hazard ratio) weighted log2(x+1)-transformed expression, then validate it by median-split Kaplan-Meier survival separation and responder/non-responder comparison, as applied to GSE78220 anti-PD-1 melanoma data in the DCNet study.

## When to use

Use when bulk RNA-seq samples with survival and treatment-response metadata must be stratified by a published prognostic gene signature into survival-differentiated high/low risk groups.

## When NOT to use

- Input is single-cell or already-deconvolved cell-abundance data rather than a bulk RNA-seq matrix with human gene symbol rows
- No per-gene hazard ratios are available for the candidate signature, so the log10(HR)-weighted risk score cannot be computed
- Samples lack survival endpoints and treatment-response labels, making Kaplan-Meier and Wilcoxon validation impossible

## Inputs

- Gene-by-sample bulk RNA-seq expression matrix (rows = human gene symbols, columns = samples), e.g. the GSE78220 transcriptome matrix for 38 pre-treatment melanoma biopsies
- Clinical metadata with overall survival times and status per sample
- Treatment-response labels (anti-PD-1 responder vs non-responder per sample)
- Published prognostic gene list with per-gene hazard ratios (the 33-gene DCNet-derived signature)

## Outputs

- Per-sample risk scores (risk_scores.csv)
- Median-split high-risk and low-risk sample group assignments
- Kaplan-Meier survival curves with log-rank p-value (Figure 5c analog)
- Wilcoxon rank-sum p-value comparing risk scores between responders and non-responders (Figure 5d analog)

## How to apply

Start from a gene-by-sample expression matrix whose rows are human gene symbols and apply the recommended log2(x+1) transformation to expression values. For each sample, compute the risk score as the sum over the 33 published prognostic genes of log10(hazard ratio) multiplied by the transformed expression of that gene, and save the per-sample scores (e.g. risk_scores.csv). Split samples at the median risk score into high- and low-risk groups and plot Kaplan-Meier survival curves with a log-rank test, targeting a significant survival difference (p<0.05) as in Figure 5c. Separately compare risk scores between anti-PD-1 responders and non-responders with a Wilcoxon rank-sum test, expecting significantly elevated scores in non-responders (reported p=0.00051, Figure 5d). The genes used must be the prognostic genes derived from differentially expressed marker genes of DCNet-defined cell types, so pair this skill with DCNet cell-landscape inference when the signature must be reproduced rather than reused from literature.

## Related tools

- **DCNet** (Source of the cell-type marker genes from which the 33-gene prognostic signature was derived; its input conventions (gene-symbol rows, log2(x+1) transformation) define the preprocessing for the risk score) — https://github.com/xindd/DCNet
- **Python** (Implementation language for downloading GSE78220 via GEOparse, computing the risk score, and running Kaplan-Meier/log-rank and Wilcoxon tests)

## Examples

```
risk = sum(hr[g]_log10 * np.log2(expr.loc[g]+1) for g in genes33); df['risk'].to_csv('risk_scores.csv')
```

## Evaluation signals

- Log-rank test between median-split high/low risk groups must yield p<0.05, matching the reported Figure 5c result
- Wilcoxon rank-sum comparison of risk scores between responders and non-responders must give p approximately 0.00051, matching Figure 5d
- Risk score is computed over exactly the 33 published prognostic genes, each weighted by log10(hazard ratio)
- Expression values are log2(x+1)-transformed and the matrix has human gene symbols as rows and samples as columns before scoring
- risk_scores.csv contains one score per pre-treatment sample (38 melanoma biopsies in the GSE78220 validation)

## Limitations

- The 33-gene signature is tied to the melanoma/PD-1 cohort (GSE78220, 38 biopsy specimens); generalization to other cancer types or treatments is not established by this validation
- Stratification at the median risk score is cohort-dependent; applying the published median cutoff to a new cohort is not validated here
- No random seed or software version numbers are reported for the surrounding pipeline, limiting exact reproducibility of the supporting analyses

## Evidence
