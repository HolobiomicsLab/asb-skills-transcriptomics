---
name: negative-binomial-glm-fitting
description: Use when you have scRNA-seq count matrices from multiple time points
  plus metadata with a binary Group factor and a Time variable, and need to detect
  genes whose expression trends over time differ between conditions (tDEGs).
when_to_use_negative:
- When the hypothesis involves nonlinear temporal changes in gene expression — the
  GLM can only detect linear effects (linear change by time or by the time-by-condition
  interaction)
- When the input is normalized continuous expression rather than counts — the negative
  binomial model assumes raw count data
- When samples across time points are not relatable (e.g., no shared subject/annotation
  structure), since CASi's cross-timepoint design assumes information borrowing from
  earlier time points
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: CASi
  role: R package providing the FindTemporalDEGs function that implements the negative-binomial-GLM
    tDEG scoring module; installed via devtools::install_github('yizhuo-wang/CASi')
  repo: github.com/yizhuo-wang/CASi
- name: DESeq2
  role: Baseline differential expression method used for TDR comparison against the
    NB-GLM tDEG approach on simulated PBMC benchmarks
- name: MAST
  role: Baseline differential expression method compared with CASi's tDEG detection
    across weak/medium/strong time-effect settings
- name: Seurat
  role: Source of the Wilcoxon-rank-sum FindMarkers baseline used in the tDEG benchmark
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:f1cb8a57915f744c6745f772b0855e8ac0b916833076f5e87a896a7ab1df6e17
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CASi
  - DESeq2
  - MAST
  - Seurat
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# negative-binomial-glm-fitting

## Summary

Fits per-gene negative binomial GLMs with a log link to identify temporal differentially expressed genes (tDEGs) in multi-timepoint scRNA-seq data, as implemented in CASi's FindTemporalDEGs. The interaction term between time and condition captures genes with divergent time-trending behavior across strata.

## When to use

Use when you have scRNA-seq count matrices from multiple time points plus metadata with a binary Group factor and a Time variable, and need to detect genes whose expression trends over time differ between conditions (tDEGs).

## When NOT to use

- When the hypothesis involves nonlinear temporal changes in gene expression — the GLM can only detect linear effects (linear change by time or by the time-by-condition interaction)
- When the input is normalized continuous expression rather than counts — the negative binomial model assumes raw count data
- When samples across time points are not relatable (e.g., no shared subject/annotation structure), since CASi's cross-timepoint design assumes information borrowing from earlier time points

## Inputs

- Gene expression count matrices for each time point (e.g., the CASiList_tDEG_demo object packaged with CASi)
- Metadata table with a binary Group factor and a Time variable whose row count equals the total number of cells across both time-point subsets (e.g., metadata_tDEG_demo)
- Optional parameters: pct.cutoff (default 0.3, fraction of cells expressing a gene) and p.cutoff (default 0.05)

## Outputs

- Per-gene results table written to tdeg_results.csv with columns: gene, coefficients, raw p-value, adjusted (Bonferroni) p-value, and tDEG call
- Set of tDEGs: genes with adjusted p value smaller than 0.05

## How to apply

Load the packaged demo objects with data(CASiList_tDEG_demo) and data(metadata_tDEG_demo), verifying the metadata contains a binary Group factor and a Time variable whose row count equals the total number of cells in both time-point subsets. First filter genes by detection rate, keeping only genes expressed in at least pct.cutoff = 0.3 of cells. For each retained gene, fit a negative binomial GLM with a log link, estimating coefficients and the dispersion parameter by maximum likelihood, using Time, Group, and the Time-by-Group interaction as independent variables and the gene expression count as the response. Extract the p-value for the time (and interaction) coefficient and apply a Bonferroni correction across all tested genes, calling tDEGs at p.cutoff = 0.05. The interaction term is the key quantity: a significant Time-by-Group coefficient indicates the gene's temporal trajectory diverges between the strata of interest (e.g., response status).

## Related tools

- **CASi** (R package providing the FindTemporalDEGs function that implements the negative-binomial-GLM tDEG scoring module; installed via devtools::install_github('yizhuo-wang/CASi')) — github.com/yizhuo-wang/CASi
- **DESeq2** (Baseline differential expression method used for TDR comparison against the NB-GLM tDEG approach on simulated PBMC benchmarks)
- **MAST** (Baseline differential expression method compared with CASi's tDEG detection across weak/medium/strong time-effect settings)
- **Seurat** (Source of the Wilcoxon-rank-sum FindMarkers baseline used in the tDEG benchmark)

## Examples

```
res <- CASi::FindTemporalDEGs(CASiList_tDEG_demo, metadata_tDEG_demo, pct.cutoff = 0.3, p.cutoff = 0.05)
```

## Evaluation signals

- tdeg_results.csv exists and contains one row per filtered gene (genes passing pct.cutoff = 0.3) with a p-value field
- Every gene flagged as a tDEG has an adjusted p value smaller than 0.05, per the FindTemporalDEGs criterion
- The GLM design matrix includes Time, Group, and Time-by-Group interaction terms, and the dispersion parameter is estimated by maximum likelihood
- Bonferroni adjustment is applied across all tested genes, so adjusted p >= raw p for every gene
- On the simulated PBMC benchmark (300 planted tDEGs, 200 repetitions), the reproduced TDR curves should show CASi's TDR comparable to or above DESeq2, MAST, and Seurat Wilcoxon baselines

## Limitations

- The current GLM framework only detects linear effects — the linear change of gene expression by time or the time-by-condition interaction — not nonlinear changes
- The workflow was validated for a binary Group factor; the interaction-based tDEG test assumes this strata encoding
- In the weak time-effect simulation setting, TDR for all methods is low, and CASi's advantage over baselines grows only as the time effect strengthens

## Evidence

- [methods] CASi's per-gene negative binomial GLM includes the time variable, the strata of interest (e.g., response status), and the time-by-strata interaction term as independent variables: [evidence span withheld — anchor status: not_found]
- [discussion] Limitation on linear effects: "the current GLM framework can only detect linear effect, i.e., the linear change of gene expression by time or the interaction of time and condition"
- [readme] Package scope including tDEG identification: "(4) identification of temporal differentially expressed genes (tDEGs)."
- [methods] FindTemporalDEGs function parameters: "Below are the input parameters of the FindTemporalDEGs function"
