---
name: multivariate-gaussian-entropy-estimation
description: Use when you need to quantify randomness of expression covariation between
  candidate regulatory factors (TFs and chromatin-state modifiers) across cells of
  a multipotent population, e.g. to detect priming toward a lineage in scRNA-Seq data.
when_to_use_negative:
- The goal is inferring cell-fate probabilities or trajectory/pseudotime rather than
  quantifying covariation entropy in a snapshot multipotent population — use a trajectory
  algorithm such as Palantir instead.
- The input is scATAC-Seq peak/accessibility data rather than scRNA-Seq expression
  covariation; the DICE metric as described is applied to log-normalized scRNA-Seq
  matrices.
- The candidate RF set has not been filtered (e.g. by frequency of expression in the
  MPP population via FindCandidateFactors / type1fth='Mean'), so the dCov matrix would
  be dominated by sparsely expressed factors.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: DICE
  role: R package providing CompDICE to build the dCov matrix and compute the entropy,
    and CompPrimingPval to derive empirical P-values against Monte-Carlo randomized
    matrices
  repo: github.com/aet21/DICE
- name: R
  role: implementation environment; DICE is an R-package installed with devtools::install_github("aet21/DICE")
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:c44f2cf84d2eaeeddfb42b7a0d584153e13459e2ad2bbd58247485405bbf3c03
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# multivariate-gaussian-entropy-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate the entropy of expression covariation among regulatory factors as a multivariate Gaussian entropy H = 0.5 * sum(log(eigenvalues)) of a double-centered distance covariance (dCov) matrix — the core DICE metric for quantifying priming in multipotent cell populations from scRNA-Seq data.

## When to use

Use when you need to quantify randomness of expression covariation between candidate regulatory factors (TFs and chromatin-state modifiers) across cells of a multipotent population, e.g. to detect priming toward a lineage in scRNA-Seq data.

## When NOT to use

- The goal is inferring cell-fate probabilities or trajectory/pseudotime rather than quantifying covariation entropy in a snapshot multipotent population — use a trajectory algorithm such as Palantir instead.
- The input is scATAC-Seq peak/accessibility data rather than scRNA-Seq expression covariation; the DICE metric as described is applied to log-normalized scRNA-Seq matrices.
- The candidate RF set has not been filtered (e.g. by frequency of expression in the MPP population via FindCandidateFactors / type1fth='Mean'), so the dCov matrix would be dominated by sparsely expressed factors.

## Inputs

- log-normalized scRNA-Seq expression matrix (genes/RFs x cells), e.g. ranzoni (1159 RFs x 1659 cells)
- cell-type label vector, e.g. ranzoniCT.v
- regulatory-factor (TF) vector, e.g. humanTF
- subset of candidate lineage-specific RFs and multipotent (MPP) cell indices
- Monte-Carlo randomized null data matrices (nMC >= 100, 1000 advised)

## Outputs

- observed DICE value (multivariate Gaussian entropy of the dCov matrix)
- null distribution of DICE values from nMC randomizations
- empirical P-value from CompPrimingPval (e.g. written to dice_priming_results.csv)

## How to apply

Load the log-normalized scRNA-Seq matrix (e.g. the ranzoni dataset, 1159 RFs x 1659 cells) with cell-type labels (ranzoniCT.v) and a regulatory-factor vector (humanTF), and subset to the candidate lineage-specific RFs and the multipotent cells (e.g. 1200 MPP cells). For each candidate RF compute the pairwise distance matrix across cells, double-center it, and assemble the distance covariance (dCov) matrix with CompDICE (useSpear=FALSE, std=TRUE). Compute DICE as the multivariate Gaussian entropy H = 0.5 * sum(log(eigenvalues of the dCov matrix)). Assess significance by comparing the observed DICE to a null distribution from Monte-Carlo randomized data matrices (nMC=100 minimum, 1000 advised, ncores=4) using CompPrimingPval, which yields an empirical P-value that the observed DICE is lower than the null.

## Related tools

- **DICE** (R package providing CompDICE to build the dCov matrix and compute the entropy, and CompPrimingPval to derive empirical P-values against Monte-Carlo randomized matrices) — github.com/aet21/DICE
- **R** (implementation environment; DICE is an R-package installed with devtools::install_github("aet21/DICE"))

## Examples

```
library(DICE); data(ranzoni); data(humanTF); dice.v <- CompDICE(ranzoni[mpp.cells, gp.rf.v], useSpear=FALSE, std=TRUE); pval <- CompPrimingPval(dice.v, nMC=1000, ncores=4)
```

## Evaluation signals

- The observed DICE value must lie below the Monte-Carlo randomized null distribution, with empirical P-value < 0.01 for nMC=100, indicating significant priming of the MPP population toward the GP lineage.
- The dCov matrix must be assembled with the CompDICE settings used in the article (useSpear=FALSE, std=TRUE) after double-centering each RF's pairwise distance matrix across MPP cells.
- The entropy must equal 0.5 * sum(log(eigenvalues of the dCov matrix)) — verify the eigenvalue computation reproduces this identity.
- Null distributions must be generated from at least nMC=100 (1000 advised) randomized data matrices; results should be stable when increasing nMC.
- All results (observed DICE, null distribution, empirical P-value) must be written to dice_priming_results.csv with matching dimensions.

## Limitations

- In the real datasets analyzed, covariation patterns between RFs were not observed to deviate strongly from monotonic linear dependencies, leaving the non-linear capability of the distance covariance metric less exercised on real data.
- The empirical P-value depends on the number of Monte-Carlo randomizations (nMC=100 minimum, 1000 advised); small nMC limits resolution of the P-value.
- No experimental work was performed to demonstrate the importance of the identified RFs in priming; validation relied on matched scATAC-Seq data, which was unavailable for some datasets (e.g. no SHARE-Seq profiles for the liver hematopoiesis study).

## Evidence

- [methods] Compute DICE as the multivariate Gaussian entropy H = 0.5 * sum(log(eigenvalues of the dCov matrix)).: [evidence span withheld — anchor status: not_found]
- [methods] Significance is assessed against a Monte-Carlo randomized null.: [evidence span withheld — anchor status: not_found]
