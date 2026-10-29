---
name: single-cell-rna-seq-priming-quantification
description: Use when a snapshot or pseudotime-ordered scRNA-Seq dataset contains
  a multipotent (MPP/TAC) population plus two or more competing downstream lineages,
  and the goal is to detect lineage-primed states, rank candidate regulatory factors/pairs,
  and call individual primed cells.
when_to_use_negative:
- The dataset lacks a clearly annotated multipotent population with two competing
  downstream lineages, so no bifurcation/priming context exists
- The goal is clustering or embedding of single-cell data rather than quantifying
  lineage priming and tipping-point dynamics (trajectory tools like Palantir alone
  are the wrong fit for the priming-quantification step)
- Only scATAC-Seq data is available without expression, since DICE analyzes entropy
  of expression covariation patterns from scRNA-Seq data
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: DICE
  role: Core R package providing CompDICE, CompPrimingPval, FindCandidateFactors,
    FindPrimingFactors, and FindPrimedCells to quantify priming, rank RFs/pairs, and
    call primed cells
  repo: https://github.com/aet21/DICE
- name: Palantir
  role: 'Downstream validation: estimates cell-fate probabilities for all multipotent
    cells to confirm primed-cell assignments (e.g. higher HS-fate probabilities for
    HS-primed TACs)'
  repo: https://github.com/dpeerlab/Palantir
- name: Seurat
  role: Used to log-normalize gene-level accessibility matrices when validating primed
    states against matched scATAC-Seq data
- name: chromVAR
  role: Dependency referenced in the DICE workflow session for epigenomic validation
    steps
  repo: https://github.com/GreenleafLab/chromVAR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:3d7481b8bd1e7517106ade77e2635e4201cf77060f5466b8420913e04f09a0b0
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
  - Palantir
  - R
  - Seurat
  - chromVAR
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# single-cell-rna-seq-priming-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify the degree of priming in a multipotent progenitor population toward competing downstream lineages and identify the regulatory-factor pairs and primed cells driving that priming, using the DICE R package's Distance Covariance Entropy framework on scRNA-Seq data alone.

## When to use

Use when a snapshot or pseudotime-ordered scRNA-Seq dataset contains a multipotent (MPP/TAC) population plus two or more competing downstream lineages, and the goal is to detect lineage-primed states, rank candidate regulatory factors/pairs, and call individual primed cells.

## When NOT to use

- The dataset lacks a clearly annotated multipotent population with two competing downstream lineages, so no bifurcation/priming context exists
- The goal is clustering or embedding of single-cell data rather than quantifying lineage priming and tipping-point dynamics (trajectory tools like Palantir alone are the wrong fit for the priming-quantification step)
- Only scATAC-Seq data is available without expression, since DICE analyzes entropy of expression covariation patterns from scRNA-Seq data

## Inputs

- Log-normalized scRNA-Seq gene-by-cell expression matrix (e.g. GSE140203 SHARE-Seq mouse skin)
- List of regulatory factors (transcription factors and chromatin-state modifiers, e.g. humanTF)
- Cell-type annotations defining the multipotent population and downstream competing lineages (e.g. TACs, IRS, HS)

## Outputs

- Per-lineage DICE (distance covariance entropy) values with empirical P-values from randomized null matrices
- Ranked list of candidate RFs by leave-one-out DICE perturbation (rankedLOO) and ranked RF pairs by Spearman correlation (rankedPairs)
- Primed-cell identifiers per lineage with DICE values and P-values (e.g. written to primed_tacs.csv)

## How to apply

Load a log-normalized gene-by-cell expression matrix (e.g. SHARE-Seq mouse skin, GEO: GSE140203) restricted to regulatory factors, defining the multipotent population (e.g. 4378 TACs) and the competing mature lineages (e.g. IRS and HS). Run FindCandidateFactors with the multipotent population and lineage vector, applying the type1fth='Mean' filter that retains overexpressed candidate RFs whose expression frequency in the multipotent population is below the mean; then estimate per-lineage distance covariance entropy with CompDICE (e.g. useSpear=FALSE, std=TRUE). Assess significance with CompPrimingPval using at least 1000 Monte-Carlo (nMC=1000) randomizations of the data matrices to build the null DICE distribution, then rank RFs by leave-one-out DICE perturbation and RF pairs by dual-perturbation Spearman correlation using FindPrimingFactors (identifying e.g. Foxo3-Lmo7 for IRS and Runx2-Zfhx3 for HS priming). Finally, run FindPrimedCells with the identified RF pairs to call lineage-primed cells and validate the assignments independently, e.g. with Palantir-derived cell-fate probabilities.

## Related tools

- **DICE** (Core R package providing CompDICE, CompPrimingPval, FindCandidateFactors, FindPrimingFactors, and FindPrimedCells to quantify priming, rank RFs/pairs, and call primed cells) — https://github.com/aet21/DICE
- **Palantir** (Downstream validation: estimates cell-fate probabilities for all multipotent cells to confirm primed-cell assignments (e.g. higher HS-fate probabilities for HS-primed TACs)) — https://github.com/dpeerlab/Palantir
- **Seurat** (Used to log-normalize gene-level accessibility matrices when validating primed states against matched scATAC-Seq data)
- **chromVAR** (Dependency referenced in the DICE workflow session for epigenomic validation steps) — https://github.com/GreenleafLab/chromVAR

## Examples

```
library(DICE); data(ranzoni); data(humanTF); cand.o <- FindCandidateFactors(mpp=tac.cells, linCT.v=c('IRS','HS'), type1fth='Mean')
```

## Evaluation signals

- The observed DICE value for each lineage must fall significantly below the randomized null distribution (empirical P < 0.01 with nMC >= 1000)
- The fraction of primed TACs per lineage should be a low proportion of the multipotent population, consistent with sparse priming
- Primed-cell assignments must be consistent with independent validation, e.g. HS-primed cells showing higher Palantir-derived HS-fate probabilities
- The type1fth='Mean' filter must exclude candidate RFs whose frequency of expression in the multipotent population is not below the mean of all candidate overexpressed RFs
- Overlap between primed-cell sets of competing lineages should be excluded/inspected (intersect primed cells across lineages)

## Limitations

- Validation in matched scATAC-Seq data relied on an invented, approximate method for assigning corresponding primed states, and no experimental work demonstrated the importance of the identified RFs
- In the real datasets analyzed, covariation patterns between RFs did not deviate strongly from monotonic linear dependencies, leaving the non-linear capability of DICE less exercised
- A comprehensive comparison to other tipping-point algorithms (e.g. scGET) was not performed; DICE improves on DNB-based methods like BioTIP but comparisons are limited
- Exact DICE statistics, P-values, and DICE/dependency version numbers are not reported in the text, complicating exact reproducibility; Palantir's newer numpy PCG64 PRNG changes numerical outputs vs. older versions

## Evidence
