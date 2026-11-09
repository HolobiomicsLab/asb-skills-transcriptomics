---
name: differential-regulatory-factor-overexpression-analysis
description: Use when analyzing scRNA-Seq data of a bifurcating multipotent population
  (e.g. MPPs vs. two downstream lineages) to find which regulatory factors are overexpressed
  in one lineage's progenitors and may encode primed states prior to computing distance
  covariance entropy.
when_to_use_negative:
- The input data already includes matched scATAC-Seq in the same cells (e.g. SHARE-Seq)
  and the goal is direct epigenomic validation of primed states rather than scRNA-Seq-only
  candidate RF discovery
- The population is unipotent or terminally differentiated with no competing downstream
  lineages, since the differential comparison requires MPPs versus progenitors of
  each of two lineages
- The aim is tipping-point/bifurcation-timing detection over pseudotime (use DICE's
  CreateDPTbin workflow or alternatives like BioTIP/scGET) rather than candidate RF
  identification in a snapshot population
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DICE
  role: R package providing the FindCandidateFactors function that performs the differential
    overexpression analysis and applies the type1fth='Mean' frequency filter to yield
    type-1 candidate RFs
  repo: https://github.com/aet21/DICE
- name: R
  role: Runtime environment in which the DICE package is installed and executed (devtools::install_github("aet21/DICE"))
  repo: https://github.com/aet21/DICE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:deff86776f49292fe414a7c77c994d5e5f7f4794ef6dd72b5d55319763a8bfa3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aet21/DICE
  repo_url: https://github.com/aet21/DICE
  tools:
  - DICE
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# differential-regulatory-factor-overexpression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies candidate regulatory factors (RFs) implicated in lineage priming by performing differential overexpression analysis of transcription factors and chromatin-state modifiers, comparing one lineage's progenitor population against multipotent progenitors (MPPs) and the competing lineage, then filtering to type-1 (low-frequency, primed-subset) factors.

## When to use

Use when analyzing scRNA-Seq data of a bifurcating multipotent population (e.g. MPPs vs. two downstream lineages) to find which regulatory factors are overexpressed in one lineage's progenitors and may encode primed states prior to computing distance covariance entropy.

## When NOT to use

- The input data already includes matched scATAC-Seq in the same cells (e.g. SHARE-Seq) and the goal is direct epigenomic validation of primed states rather than scRNA-Seq-only candidate RF discovery
- The population is unipotent or terminally differentiated with no competing downstream lineages, since the differential comparison requires MPPs versus progenitors of each of two lineages
- The aim is tipping-point/bifurcation-timing detection over pseudotime (use DICE's CreateDPTbin workflow or alternatives like BioTIP/scGET) rather than candidate RF identification in a snapshot population

## Inputs

- Log-normalized scRNA-Seq gene-by-cell expression matrix restricted to regulatory factors (e.g. ranzoni: 1159 regulatory factors x 1659 cells)
- Regulatory-factor annotation vector (data(humanTF) from the DICE R package)
- Cell-type labels distinguishing MPPs from each downstream lineage's progenitor population (e.g. Granulocyte Prog, Comm-Prog, MEMP)

## Outputs

- Type-1 candidate regulatory factors per lineage (RFs with MPP expression frequency below the mean; e.g. 26 type-1 GP-RFs and 45 type-1 MEMP-RFs)
- Candidate RF sets per lineage ready as input to CompDICE for distance covariance entropy computation

## How to apply

Load a log-normalized scRNA-Seq matrix of regulatory factors x cells (e.g. the ranzoni dataset: 1159 regulatory factors x 1659 cells) together with a regulatory-factor vector (humanTF) into the DICE R package. Run FindCandidateFactors comparing each lineage progenitor population (e.g. Granulocyte Progenitors, Common Progenitors) against MPPs and the other lineage, with type1fth='Mean' so that candidate RFs whose frequency of expression in the MPP population is below the mean over all candidate overexpressed RFs are retained as type-1 factors. This type-1 filter is the key decision point: it selects low-frequency, primed-subset factors rather than broadly expressed ones, since type-1 RFs are expected to be expressed in only a small primed subset of MPPs. The output — type-1 candidate RF lists per lineage (e.g. 26 type-1 GP-RFs and 45 type-1 MEMP-RFs in the ranzoni fetal liver data) — then feeds directly into CompDICE for distance covariance entropy estimation and subsequent priming P-value assessment.

## Related tools

- **DICE** (R package providing the FindCandidateFactors function that performs the differential overexpression analysis and applies the type1fth='Mean' frequency filter to yield type-1 candidate RFs) — https://github.com/aet21/DICE
- **R** (Runtime environment in which the DICE package is installed and executed (devtools::install_github("aet21/DICE"))) — https://github.com/aet21/DICE

## Examples

```
library(DICE); data(ranzoni); data(humanTF); cand.gp <- FindCandidateFactors(ranzoni, cellType.v, progenitors='GP', compareAgainst=c('MPP','MEMP'), type1fth='Mean')
```

## Evaluation signals

- The type-1 filter retains only RFs whose frequency of expression in the MPP population is below the mean over all candidate overexpressed RFs — verify the retained set satisfies this criterion
- Downstream validity: the resulting type-1 RF sets should yield empirical priming P-values < 0.05 from CompPrimingPval with nMC=1000 Monte-Carlo runs when fed into CompDICE
- Biological sanity check against the published ranzoni result: 26 type-1 GP-RFs and 45 type-1 MEMP-RFs, with top priming pairs such as UHRF1-IRF8 (GP) and GATA1-KLF1 (MEMP) recovered
- FindPrimingFactors LOO ranking over the candidate RFs should show that removing top-ranked RFs substantially perturbs the DICE statistic, confirming the candidates actually drive expression covariation

## Limitations

- scRNA-Seq has low sensitivity to detect dynamic changes in TF expression, so candidate RF identification depends on the frequency-threshold filter rather than strong expression differences
- Candidate RFs identified from scRNA-Seq alone require separate validation with matched scATAC-Seq data (e.g. aggregating peaks 3kb upstream of marker genes plus gene body, log-normalized with Seurat) or experimental work; no experimental validation of RF importance was performed in the source study
- The type1fth='Mean' threshold is a heuristic; the method relies on the mean frequency of the candidate overexpressed RF set, which can shift with the compared populations

## Evidence
