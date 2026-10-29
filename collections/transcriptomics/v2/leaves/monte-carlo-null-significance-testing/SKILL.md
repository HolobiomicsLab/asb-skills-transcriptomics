---
name: monte-carlo-null-significance-testing
description: Use when a computed DICE value over candidate regulatory factors in a
  multipotent population (e.g. MPPs or TACs) must be tested against randomized data
  matrices to establish whether expression covariation indicates significant priming.
when_to_use_negative:
- The multipotent population is not identifiable or the lineage-progenitor populations
  needed for the FindCandidateFactors differential overexpression comparison are absent
  from the dataset
- The goal is testing a variance- or trajectory-based statistic rather than a distance
  covariance entropy (dCov) metric — competing variance-based or tipping-point methods
  (BioTIP, scGET) follow a different null framework
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DICE
  role: R package whose CompPrimingPval function computes empirical P-values of the
    observed DICE against randomized data matrices; CompDICE generates the observed
    statistic being tested
  repo: https://github.com/aet21/DICE
- name: R
  role: Implementation environment in which the DICE package functions (CompDICE,
    CompPrimingPval) are executed
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:15fbd0d2ac1caafaee0f34958f707d948eaa3b78167264fbbabf8300997b5226
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# monte-carlo-null-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess the significance of a DICE (distance covariance entropy) priming statistic by comparing the observed DICE value over type-1 lineage-specific regulatory factors to a null distribution built from Monte-Carlo randomizations of the scRNA-Seq data matrix, deriving an empirical P-value.

## When to use

Use when a computed DICE value over candidate regulatory factors in a multipotent population (e.g. MPPs or TACs) must be tested against randomized data matrices to establish whether expression covariation indicates significant priming.

## When NOT to use

- The multipotent population is not identifiable or the lineage-progenitor populations needed for the FindCandidateFactors differential overexpression comparison are absent from the dataset
- The goal is testing a variance- or trajectory-based statistic rather than a distance covariance entropy (dCov) metric — competing variance-based or tipping-point methods (BioTIP, scGET) follow a different null framework

## Inputs

- Observed DICE value computed over type-1 lineage-specific candidate RFs in the multipotent population (output of CompDICE with useSpear=FALSE, std=TRUE)
- Log-normalized scRNA-Seq expression matrix restricted to candidate regulatory factors and multipotent cells (e.g. ranzoni: 1159 RFs x 1659 cells; SHARE-Seq mouse skin restricted to 4378 TACs)
- Cell-type label vector (e.g. ranzoniCT.v) identifying the multipotent (MPP/TAC) population and downstream lineage populations

## Outputs

- Empirical P-value for the observed DICE relative to the Monte-Carlo null distribution
- Null distribution of DICE values from randomized data matrices (nMC=100 minimum, 1000 advised)
- Per-lineage priming significance calls (e.g. GP vs MEMP; IRS vs HS) written to results such as dice_priming_results.csv

## How to apply

First compute the observed DICE value with CompDICE on the type-1 candidate RFs (e.g. 26 type-1 GP-RFs or 45 type-1 MEMP-RFs) over the multipotent cells (e.g. 1200 MPP cells), using useSpear=FALSE and std=TRUE. Then generate Monte-Carlo randomized null data matrices — a minimum of nMC=100, but nMC=1000 is advised for stable empirical P-values, with ncores=4 for parallelization — and compute the null DICE distribution. Call CompPrimingPval to obtain the empirical P-value that the observed DICE lies below (lower entropy than) the randomized null. Decision point: an empirical P-value below 0.05 (or below 0.01 when nMC=100) indicates significant priming of the multipotent population toward that lineage. The rationale is that randomizing the expression matrix destroys genuine inter-RF covariation while preserving marginal expression structure, so only truly coordinated, primed covariation patterns fall significantly below the null.

## Related tools

- **DICE** (R package whose CompPrimingPval function computes empirical P-values of the observed DICE against randomized data matrices; CompDICE generates the observed statistic being tested) — https://github.com/aet21/DICE
- **R** (Implementation environment in which the DICE package functions (CompDICE, CompPrimingPval) are executed)

## Examples

```
library(DICE); data(ranzoni); data(humanTF);
```

## Evaluation signals

- Empirical P-value from the nMC=1000 null must be below 0.05 (and below 0.01 when nMC=100) for the priming call to be significant
- The observed DICE value must lie below the randomized null distribution — lower entropy, not higher — since priming entails non-random, coordinated covariation
- The null distribution must be reproducibly stable across reruns (e.g. ncores=4 parallelized randomizations) and the 1000-run result should be consistent with the 100-run minimum
- Significant DICE calls should be obtained only for lineages with genuine type-1 candidate RFs (26 GP-RFs, 45 MEMP-RFs in ranzoni; IRS- and HS-specific RFs in SHARE-Seq), not for arbitrary RF sets
- Downstream consistency: primed-cell calls (e.g. ~7% GP-primed/88 cells, ~2% MEMP-primed/24 cells) and FindPrimingFactors rankings should follow only from lineages whose empirical P-value passes the threshold

## Limitations

- Explicit values of the computed DICE statistics and P-values were printed but not shown in the article text, so exact reference values cannot be cross-checked from the paper alone
- DICE package version numbers and dependency versions (destiny, chromVAR, Seurat, Palantir) are not recorded, which can affect exact numerical reproducibility of the randomized null
- In the real datasets analyzed, covariation patterns between RFs were not observed to deviate strongly from monotonic linear dependencies, so the non-linear sensitivity of the dCov metric is less exercised on real data — the null test may be less discriminative than theoretically expected

## Evidence
