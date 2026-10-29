---
name: cox-proportional-hazards-survival-analysis
description: Use when a bulk expression cohort (e.g. TCGA SKCM06 RNA-seq with clinical
  info) has matched event/time-to-event survival fields and you need per-feature hazard
  ratios with BH-adjusted significance for cell types or genes.
when_to_use_negative:
- The cohort lacks matched, high-quality survival information (event and time-to-event)
  — TimiCheckEvent filtering would remove the records needed to power the regression.
- The goal is immunotherapy-response prediction rather than prognosis; use the TimiGP-Response
  module instead of Cox regression.
- The input is already a curated feature table of associations rather than raw expression
  plus clinical data — regression would be redundant.
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3308
tools:
- name: TimiCOX
  role: Wrapper performing univariate Cox regression fitting each marker pair (or
    cell type / gene) as a variable, with p.adj='BH' multiple-testing correction
  repo: https://github.com/CSkylarL/TimiGP
- name: TimiCheckEvent
  role: Filters out low-quality clinical info (event and time-to-event) before survival
    fitting
  repo: https://github.com/CSkylarL/TimiGP
- name: TimiPrePropress
  role: Extracts the transcription profile of selected markers/cohorts and applies
    log transformation and gene-wise median normalization prior to regression
  repo: https://github.com/CSkylarL/TimiGP
- name: survival
  role: R package supplying the underlying Cox proportional-hazards model fitting
    invoked through TimiCOX
- name: CIBERSORTx
  role: Deconvolves bulk SKCM06 profiles against the CIBERSORT LM22 signature to produce
    per-sample cell-type abundances used as Cox covariates
- name: xCell
  role: Marker-expression-based estimation of per-sample immune cell-type scores as
    alternative Cox covariates
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xcrm.2023.101121
    title: 'TimiGP: Inferring cell-cell interactions and prognostic associations in
      the tumor immune microenvironment through gene pairs'
  source_content_hash: sha256:edec85301c2263245a8d875d4b4ef44e61b9a3476a3fa51fce9e92ad9db12cb7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/CSkylarL/TimiGP
  tools:
  - CIBERSORTx
  - R
  - TimiCOX
  - TimiCheckEvent
  - TimiGP
  - TimiPrePropress
  - survival
  - xCell
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# cox-proportional-hazards-survival-analysis

## Summary

Fits univariate Cox proportional-hazards regression per feature (immune marker pairs, cell-type scores, or marker genes) on a bulk transcriptomic cohort with matched survival data, using TimiCOX with Benjamini-Hochberg multiple-testing adjustment, to quantify prognostic associations such as the immune co-infiltration prognostic bias in TCGA SKCM06.

## When to use

Use when a bulk expression cohort (e.g. TCGA SKCM06 RNA-seq with clinical info) has matched event/time-to-event survival fields and you need per-feature hazard ratios with BH-adjusted significance for cell types or genes.

## When NOT to use

- The cohort lacks matched, high-quality survival information (event and time-to-event) — TimiCheckEvent filtering would remove the records needed to power the regression.
- The goal is immunotherapy-response prediction rather than prognosis; use the TimiGP-Response module instead of Cox regression.
- The input is already a curated feature table of associations rather than raw expression plus clinical data — regression would be redundant.

## Inputs

- TCGA SKCM06 clinical info (SKCM06info: event and time-to-event survival fields)
- TCGA SKCM06 bulk transcriptome profile (SKCM06rna, RNA-seq expression matrix)
- Per-sample immune cell-type abundance scores from CIBERSORTx LM22 (CellType_Newman2015_LM22) or xCell deconvolution
- Immune marker gene expression matrix (e.g. markers from Immune_Marker_n1293 / LM22 signature)

## Outputs

- Per-feature Cox regression table (HR, confidence interval, p-value, BH-adjusted QV) for each cell type
- Per-gene Cox regression table for each immune marker gene (e.g. PD-L1, IDO1)
- Proportion of significant (p<0.05) favorable-prognosis (HR<1) associations, e.g. 93.2% of 44 cell types and 90.5% of marker genes in SKCM06
- Combined cell-type-level and gene-level results CSV

## How to apply

First filter low-quality clinical records (event and time-to-event) with TimiCheckEvent, then preprocess the expression matrix of the selected markers with TimiPrePropress (log=TRUE, GMNorm=TRUE for log transformation plus gene-wise median normalization). Fit one univariate Cox regression per variable — each immune cell-type score (from CIBERSORT LM22 deconvolution or xCell marker-expression scores) or each immune marker gene — via the TimiCOX interface to the survival package, requesting BH-adjusted p-values (p.adj='BH'). Interpret HR < 1 as favorable prognosis; in the TimiGP marker-pair convention, reverse pairs carry HR > 1. Finally tabulate the proportion of significant associations (p < 0.05) with favorable HR among cell types and genes to quantify co-infiltration-driven prognostic bias, and write HR, CI, p-value, and BH-adjusted QV to a combined results CSV.

## Related tools

- **TimiCOX** (Wrapper performing univariate Cox regression fitting each marker pair (or cell type / gene) as a variable, with p.adj='BH' multiple-testing correction) — https://github.com/CSkylarL/TimiGP
- **TimiCheckEvent** (Filters out low-quality clinical info (event and time-to-event) before survival fitting) — https://github.com/CSkylarL/TimiGP
- **TimiPrePropress** (Extracts the transcription profile of selected markers/cohorts and applies log transformation and gene-wise median normalization prior to regression) — https://github.com/CSkylarL/TimiGP
- **survival** (R package supplying the underlying Cox proportional-hazards model fitting invoked through TimiCOX)
- **CIBERSORTx** (Deconvolves bulk SKCM06 profiles against the CIBERSORT LM22 signature to produce per-sample cell-type abundances used as Cox covariates)
- **xCell** (Marker-expression-based estimation of per-sample immune cell-type scores as alternative Cox covariates)

## Examples

```
res <- TimiCOX(mps, info, p.adj = "BH")
```

## Evaluation signals

- TimiCheckEvent should remove records with missing or low-quality event/time-to-event fields before any model fitting.
- After TimiPrePropress(log=TRUE, GMNorm=TRUE), the expression matrix should be log-transformed and gene-wise median normalized.
- Each Cox table row must carry HR, CI, raw p-value, and BH-adjusted QV; significance calls use p<0.05 (or BH-adjusted threshold).
- Direction convention must hold: HR<1 = favorable prognosis for cell types/genes, and reverse marker pairs with HR>1 under the TimiCOX convention (e.g. CXCL10-over-PD-L1 pair direction).
- Bias statistics should be reproducible: ~44 significant cell types with 93.2% favorable HR, and 90.5% of prognostic immune marker genes (including PD-L1, IDO1) associated with improved survival in TCGA SKCM06.

## Limitations

- TimiGP requires a large gene expression dataset with matched survival information to effectively identify prognostic gene pairs.
- Power of the survival analysis is determined by the sample size, the number of events, and the quality of the survival information.
- Cox associations reflect co-infiltration-driven prognostic bias (e.g. PD-L1 and IDO1 appearing favorable) and must not be interpreted as causal functional effects.

## Evidence
