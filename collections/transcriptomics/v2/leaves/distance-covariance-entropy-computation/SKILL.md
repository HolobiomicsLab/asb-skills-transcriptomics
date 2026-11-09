---
name: distance-covariance-entropy-computation
description: Use when a multipotent progenitor population (e.g. MPPs, TACs) plus competing
  downstream lineages are available in log-normalized scRNA-Seq data, and the goal
  is to quantify priming or locate a bifurcation point by entropy of expression covariation.
when_to_use_negative:
- When the goal is clustering or cell-type annotation rather than quantifying priming/bifurcation
  — chromVAR-style motif variability or standard trajectory tools are more appropriate
- When only a single lineage or no multipotent progenitor population is present, since
  DICE requires covariation among competing lineage RF sets over a multipotent population
- When a rigorous variance-based or DNB-based tipping-point analysis is specifically
  required — the article notes DICE outperformed BioTIP here, but BioTIP may be the
  mandated comparator
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DICE
  role: R package providing CompDICE, CompPrimingPval, FindCandidateFactors, FindPrimingFactors,
    FindPrimedCells and CreateDPTbin for entropy-based priming and bifurcation analysis
  repo: https://github.com/aet21/DICE
- name: Palantir
  role: Used downstream to estimate cell-fate probabilities for MPP cells, validating
    HS-primed assignments from DICE-called primed cells
  repo: https://github.com/dpeerlab/Palantir
- name: destiny
  role: DiffusionMap (k=30) and DPT used to compute diffusion pseudotime that is binned
    with CreateDPTbin for liver time-course DICE trajectories
- name: Seurat
  role: Log-normalization of the gene-level accessibility matrix in the scATAC-Seq
    validation branch
- name: BioTIP
  role: Competing variance/DNB-based tipping-point method that failed to show a clear
    transition where DICE succeeded
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_006
  - task_005
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:e827be02fe61ba819dc8c58fc17732fc3f40a112f653389ac463b5c62fb973cc
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
  - BioTIP
  - Biobase
  - CompDICE
  - DICE
  - Hmisc
  - Palantir
  - R
  - Seurat
  - destiny
  - gridExtra
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# distance-covariance-entropy-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute the Distance Covariance Entropy (DICE) — the multivariate Gaussian entropy of the double-centered distance-covariance (dCov) matrix of regulatory-factor expression covariation — to quantify priming in multipotent cell populations and pinpoint bifurcation timing from scRNA-Seq data using the DICE R package.

## When to use

Use when a multipotent progenitor population (e.g. MPPs, TACs) plus competing downstream lineages are available in log-normalized scRNA-Seq data, and the goal is to quantify priming or locate a bifurcation point by entropy of expression covariation.

## When NOT to use

- When the goal is clustering or cell-type annotation rather than quantifying priming/bifurcation — chromVAR-style motif variability or standard trajectory tools are more appropriate
- When only a single lineage or no multipotent progenitor population is present, since DICE requires covariation among competing lineage RF sets over a multipotent population
- When a rigorous variance-based or DNB-based tipping-point analysis is specifically required — the article notes DICE outperformed BioTIP here, but BioTIP may be the mandated comparator

## Inputs

- Log-normalized scRNA-Seq expression matrix restricted to regulatory factors (e.g. ranzoni: 1159 RFs x 1659 cells)
- Regulatory-factor vector (e.g. humanTF from the DICE package)
- Cell-type label vector identifying multipotent and lineage populations (e.g. ranzoniCT.v)
- Candidate type-1 lineage-specific RF lists (from FindCandidateFactors with type1fth='Mean')
- For bifurcation timing: diffusion pseudotime values or TFA profiles per developmental stage/timepoint (e.g. yangLiver ExpressionSet, Hu retina GSE107618)

## Outputs

- Observed DICE value per lineage over the multipotent population
- Monte-Carlo null DICE distribution and empirical P-value (CompPrimingPval)
- Leave-one-out RF rankings and Spearman-correlated RF pairs (FindPrimingFactors)
- Primed-cell identifiers per lineage (FindPrimedCells, e.g. ~7% GP-primed / 88 cells, ~2% MEMP-primed / 24 cells)
- Stage- or pseudotime-binned DICE trajectory marking the bifurcation point (e.g. week 13 retina, post-E13.5 liver)

## How to apply

Load a log-normalized scRNA-Seq matrix restricted to regulatory factors (e.g. the ranzoni matrix of 1159 RFs x 1659 cells with the humanTF list) and identify candidate lineage-specific RFs with FindCandidateFactors, using type1fth='Mean' to retain type-1 factors whose MPP expression frequency is below the mean. Compute the distance covariance entropy of the candidate RFs over the multipotent cells with CompDICE (useSpear=FALSE, std=TRUE): for each RF compute the pairwise cell-cell distance matrix, double-center it, assemble the dCov matrix, and take H = 0.5 * sum(log(eigenvalues)). Assess significance by comparing the observed DICE to a Monte-Carlo randomized null (nMC >= 1000) with CompPrimingPval, expecting the observed value to fall significantly below the null (empirical P < 0.05, ideally < 0.01). For bifurcation timing, bin pseudotime with CreateDPTbin (minBins=7), compute DICE per bin over selected lineage TFs (e.g. top-7 per retina lineage or the top-4 hepatocyte LSR/ELF3/IRF6/BGN and cholangiocyte NR1I3/NR1I2/FOXA2/TRIM15 factors), bootstrap over 1000 resamples per bin, and read off the transition at the DICE minimum/rise.

## Related tools

- **DICE** (R package providing CompDICE, CompPrimingPval, FindCandidateFactors, FindPrimingFactors, FindPrimedCells and CreateDPTbin for entropy-based priming and bifurcation analysis) — https://github.com/aet21/DICE
- **Palantir** (Used downstream to estimate cell-fate probabilities for MPP cells, validating HS-primed assignments from DICE-called primed cells) — https://github.com/dpeerlab/Palantir
- **destiny** (DiffusionMap (k=30) and DPT used to compute diffusion pseudotime that is binned with CreateDPTbin for liver time-course DICE trajectories)
- **Seurat** (Log-normalization of the gene-level accessibility matrix in the scATAC-Seq validation branch)
- **BioTIP** (Competing variance/DNB-based tipping-point method that failed to show a clear transition where DICE succeeded)

## Examples

```
library(DICE); data(ranzoni); data(humanTF); dice.v <- CompDICE(rf.v=candRF.v, mpp.m=ranzoni.m[candRF.v, mpp.idx], useSpear=FALSE, std=TRUE); p.v <- CompPrimingPval(dice.v, nullDice.l, nMC=1000)
```

## Evaluation signals

- Empirical P-value from CompPrimingPval with nMC >= 1000 must fall below 0.05 (ideally < 0.01), i.e. observed DICE lies below the randomized null distribution
- Primed-cell fractions should be a low proportion of the multipotent population, consistent with reported ~7% GP-primed (88 cells) and ~2% MEMP-primed (24 cells) results
- In bifurcation mode, DICE as a function of pseudotime bin must show a transition-like minimum/rise (e.g. week 13 retina; between DPT-bin 5 and 6, after E14/E14.5 liver), unlike individual TF variances which change only asynchronously
- type-1 candidate RF filter must retain only RFs whose frequency of expression in the MPP population is below the mean over all candidate overexpressed RFs (type1fth='Mean')
- Bootstrap variability band (1000 resamples per bin, plotted with Hmisc errbar as mean +/- 1.96*SD) should not obscure the transition in the DICE trajectory

## Limitations

- In the real datasets analyzed, covariation patterns between RFs did not deviate strongly from monotonic linear dependencies, so DICE's non-linear capability was less exercised on real data
- Validation of primed states in scATAC-Seq relied on an invented, approximate method for assigning corresponding primed states, and no experimental work demonstrated the importance of the identified RFs
- DICE values and P-values were printed but explicit numeric values are not shown in the text, and package/dependency version numbers (DICE, destiny, chromVAR, Seurat, Palantir) are not recorded
- A comprehensive comparison to other tipping point algorithms such as scGET was not performed, and application to disease contexts (e.g. cancer) was not explored

## Evidence
