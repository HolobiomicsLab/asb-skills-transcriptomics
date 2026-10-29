---
name: double-centering-of-distance-matrices
description: Use when quantifying priming or bifurcation dynamics in a multipotent
  scRNA-Seq population (e.g. Ranzoni MPP cells) by measuring entropy of covariation
  among candidate regulatory factors via the DICE framework.
when_to_use_negative:
- The population is not multipotent (no competing downstream lineages to assess priming
  against) — DICE is designed for bifurcation dynamics between multipotent cells and
  mature lineages
- The input is not log-normalized scRNA-Seq expression but raw counts, methylation,
  or bulk data — distances and their covariance spectrum would not reflect expression
  covariation between RFs
- You need cell-fate probabilities or pseudotime trajectories rather than a static
  entropy statistic — Palantir is the complementary tool for that purpose
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DICE
  role: R package providing CompDICE and CompPrimingPval, which implement the distance-matrix
    computation, double-centering, dCov assembly, entropy calculation and null-distribution
    significance testing
  repo: github.com/aet21/DICE
- name: R
  role: Implementation environment in which the DICE package functions are executed
- name: Palantir
  role: Used downstream to validate that GP-primed cells identified by DICE have high
    GP cell-fate probabilities
  repo: dpeerlab/Palantir
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:ef4216f705f862cfa7c05054571239b5d4fcef3a153cbb4908b5ad35c9c756ec
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
  - Palantir
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# double-centering-of-distance-matrices

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-regulatory-factor pairwise cell-to-cell distance matrices over a multipotent cell population and double-centers them, so that the assembled distance covariance (dCov) matrix yields a multivariate Gaussian entropy (DICE) that quantifies expression-covariation randomness in primed states.

## When to use

Use when quantifying priming or bifurcation dynamics in a multipotent scRNA-Seq population (e.g. Ranzoni MPP cells) by measuring entropy of covariation among candidate regulatory factors via the DICE framework.

## When NOT to use

- The population is not multipotent (no competing downstream lineages to assess priming against) — DICE is designed for bifurcation dynamics between multipotent cells and mature lineages
- The input is not log-normalized scRNA-Seq expression but raw counts, methylation, or bulk data — distances and their covariance spectrum would not reflect expression covariation between RFs
- You need cell-fate probabilities or pseudotime trajectories rather than a static entropy statistic — Palantir is the complementary tool for that purpose

## Inputs

- Log-normalized scRNA-Seq count matrix (genes/regulatory factors x cells), e.g. ranzoni (1159 RFs x 1659 cells)
- Cell-type label vector, e.g. ranzoniCT.v identifying the 1200 MPP cells
- Regulatory-factor vector, e.g. humanTF, restricted to type-1 lineage-specific candidate RFs (from FindCandidateFactors)

## Outputs

- Double-centered pairwise distance matrices, one per candidate RF
- Distance covariance (dCov) matrix over the candidate RFs
- Observed DICE value (multivariate Gaussian entropy H = 0.5 * sum(log(eigenvalues)))
- Empirical P-value from CompPrimingPval against the Monte-Carlo randomized null (e.g. written to dice_priming_results.csv)

## How to apply

Load the log-normalized scRNA-Seq matrix (e.g. the 1159 RFs x 1659 cells ranzoni matrix), the cell-type labels (ranzoniCT.v) and the humanTF regulatory-factor vector from the DICE package, and subset to type-1 lineage-specific candidate RFs and the multipotent cells (e.g. 1200 MPP cells). For each candidate RF, compute the pairwise distance matrix across cells and double-center it, then assemble the distance covariance (dCov) matrix exactly as CompDICE does with useSpear=FALSE and std=TRUE. Compute DICE as the multivariate Gaussian entropy H = 0.5 * sum(log(eigenvalues of the dCov matrix)), which is low when covariation between lineage-specific RFs is coherent (priming) and high under randomness. Assess significance by comparing the observed DICE against a Monte-Carlo randomized null (nMC=1000 advised, ncores=4) with CompPrimingPval. The rationale is that double-centering the distance matrix is what makes the resulting eigen-spectrum interpretable as a distance covariance, analogous to classical centering in PCA but on distances rather than coordinates.

## Related tools

- **DICE** (R package providing CompDICE and CompPrimingPval, which implement the distance-matrix computation, double-centering, dCov assembly, entropy calculation and null-distribution significance testing) — github.com/aet21/DICE
- **R** (Implementation environment in which the DICE package functions are executed)
- **Palantir** (Used downstream to validate that GP-primed cells identified by DICE have high GP cell-fate probabilities) — dpeerlab/Palantir

## Examples

```
library(DICE); data(ranzoni); data(humanTF); dice.v <- CompDICE(ranzoni[candRF.v, mpp.idx], useSpear=FALSE, std=TRUE)
```

## Evaluation signals

- For nMC=100 randomizations the empirical P-value from CompPrimingPval must be below 0.01, i.e. the observed DICE lies below the randomized null distribution (significant GP priming of MPPs)
- The eigenvalues of the double-centered dCov matrix must all be positive (it must be a valid covariance matrix) for H = 0.5 * sum(log(eigenvalues)) to be defined
- The output CSV (dice_priming_results.csv) contains the observed DICE value, the full randomized null distribution, and the empirical P-value
- The subset dimensions are correct: candidate type-1 GP RFs x the 1200 MPP cells from the 1159 RFs x 1659 cells ranzoni matrix
- Running CompDICE with the specified flags (useSpear=FALSE, std=TRUE) reproduces the same DICE value as the manual distance/double-center/eigenvalue pipeline

## Limitations

- The Gaussian entropy approximation H = 0.5 * sum(log(eigenvalues)) assumes covariation structure amenable to this form; in the real datasets analyzed, covariation patterns between RFs did not deviate strongly from monotonic linear dependencies, leaving DICE's non-linear capability less exercised
- Significance is empirical and depends on the Monte-Carlo scheme: nMC=100 is a minimum while 1000 randomizations are advised, so P-values vary with nMC
- Validation of primed states relied on matched scATAC-Seq data where available (SHARE-Seq, Ranzoni matched scATAC-Seq); without such data, the priming call from the DICE statistic alone is unvalidated epigenomically

## Evidence
