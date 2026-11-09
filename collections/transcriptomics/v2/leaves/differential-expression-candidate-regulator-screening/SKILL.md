---
name: differential-expression-candidate-regulator-screening
description: Use when you have a scRNA-Seq matrix with a multipotent cell population
  and competing downstream lineages (e.g. SHARE-Seq hair-follicle TACs vs IRS/HS,
  or Ranzoni fetal-liver MPPs vs GP/MEMP), and you need to shortlist overexpressed
  regulatory factors to feed into DICE priming analysis.
when_to_use_negative:
- When the goal is to identify tipping points or bifurcation timing over pseudotime
  rather than to shortlist candidate priming RFs — use the later DICE workflow steps
  (CompDICE, CreateDPTbin with minBins=7) instead
- When the input is already a shortlist of selected RFs (e.g. the top-4 hepatocyte
  and cholangiocyte TFs NR1I3, NR1I2, FOXA2, TRIM15, LSR, ELF3, IRF6, BGN) — proceed
  directly to CompDICE
- When regulatory factors must be validated experimentally; this screen is purely
  computational and the article notes no experimental work was performed to demonstrate
  importance of the identified RFs
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DICE
  role: R package providing FindCandidateFactors for differential overexpression screening
    of candidate RFs and the type1fth='Mean' frequency filter
  repo: github.com/aet21/DICE
- name: R
  role: Execution environment for the DICE package workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:261cd621c22e2d7c91d1088be795e7ee2d5ba1dc66b81deadaa563abd3e13e1e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aet21/DICE
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
yes_rate: 0.625
n_claims: 8
---

# differential-expression-candidate-regulator-screening

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A screening step within the DICE framework that identifies candidate regulatory factors (RFs) implicated in lineage priming by performing differential overexpression analysis of lineage-committed cells versus the multipotent progenitor (MPP) population, then filtering by expression frequency in the MPP population (type1fth='Mean').

## When to use

Use when you have a scRNA-Seq matrix with a multipotent cell population and competing downstream lineages (e.g. SHARE-Seq hair-follicle TACs vs IRS/HS, or Ranzoni fetal-liver MPPs vs GP/MEMP), and you need to shortlist overexpressed regulatory factors to feed into DICE priming analysis.

## When NOT to use

- When the goal is to identify tipping points or bifurcation timing over pseudotime rather than to shortlist candidate priming RFs — use the later DICE workflow steps (CompDICE, CreateDPTbin with minBins=7) instead
- When the input is already a shortlist of selected RFs (e.g. the top-4 hepatocyte and cholangiocyte TFs NR1I3, NR1I2, FOXA2, TRIM15, LSR, ELF3, IRF6, BGN) — proceed directly to CompDICE
- When regulatory factors must be validated experimentally; this screen is purely computational and the article notes no experimental work was performed to demonstrate importance of the identified RFs

## Inputs

- Log-normalized scRNA-Seq count matrix (genes x cells), restricted to regulatory factors (e.g. SHARE-Seq mouse skin, GEO: GSE140203)
- Cell-type annotations identifying the multipotent population (e.g. 4378 TAC cells) and downstream lineage populations (e.g. IRS and HS)
- Regulatory factor list (e.g. humanTF from the DICE package)

## Outputs

- Per-lineage lists of candidate overexpressed regulatory factors that passed the type1fth='Mean' frequency filter
- Candidate RF sets suitable as input to CompDICE for distance covariance entropy estimation

## How to apply

Load the log-normalized scRNA-Seq matrix restricted to regulatory factors (e.g. via the DICE R package with data(ranzoni) and data(humanTF), or the SHARE-Seq mouse skin matrix from GEO: GSE140203) and define the multipotent population (mpp) and downstream lineage cell-type vectors (linCT.v). Run FindCandidateFactors comparing each mature lineage's cells against the MPP cells to select candidate overexpressed RFs per lineage. Apply the type1fth='Mean' filter, which keeps only candidate RFs whose frequency of expression in the MPP population is below the mean frequency computed over all candidate overexpressed RFs — this biases selection toward factors sparsely expressed in the multipotent population, consistent with sparse priming. The resulting per-lineage candidate RF lists are then passed to CompDICE (e.g. useSpear=FALSE, std=TRUE) to compute distance covariance entropy over the MPP cells.

## Related tools

- **DICE** (R package providing FindCandidateFactors for differential overexpression screening of candidate RFs and the type1fth='Mean' frequency filter) — github.com/aet21/DICE
- **R** (Execution environment for the DICE package workflow)

## Examples

```
library(DICE); data(ranzoni); data(humanTF); cand.o <- FindCandidateFactors(mpp=mpp.cells, linCT.v=c('GP','MEMP'), type1fth='Mean')
```

## Evaluation signals

- The filtered candidate RF set should contain only factors whose frequency of expression in the MPP/TAC population is below the mean over all candidate overexpressed RFs
- Downstream DICE values on the resulting candidates must yield empirical P < 0.01 against a randomized null built with nMC >= 1000 (CompPrimingPval) for the priming call to be significant
- Leave-one-out DICE perturbation (rankedLOO) and Spearman correlation (rankedPairs) should identify a coherent RF pair (e.g. Foxo3-Lmo7 for IRS, Runx2-Zfhx3 for HS), implying the screened candidates include the true priming factors
- FindPrimedCells called on the TAC population with the identified pairs should label a low fraction of cells as primed (131 HS-primed and 196 IRS-primed of 4378 TACs), consistent with sparse priming

## Limitations

- The screen is restricted to regulatory factors; scRNA-Seq's low sensitivity to detect dynamic changes in TF expression means some true regulators may be missed
- Validation relied on matched scATAC-Seq (SHARE-Seq or separate populations) and Palantir cell-fate probabilities, not on experimental perturbation of the identified RFs
- The candidate screen depends on a defined multipotent-to-lineage contrast; it is not designed for disease contexts or non-bifurcating datasets, which the article did not explore

## Evidence
