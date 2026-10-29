---
name: leave-one-out-entropy-perturbation-ranking
description: Use when a scRNA-Seq snapshot contains a multipotent population (e.g.
  MPPs) with competing downstream lineages, candidate priming RFs have already been
  filtered (type-1, low MPP-expression frequency), and you must prioritize which RFs
  and RF pairs define the primed state.
when_to_use_negative:
- When only a single terminal lineage exists with no competing lineage or multipotent
  population — LOO entropy covariation is defined over competing lineage priming within
  a multipotent pool
- When candidate RFs have not been pre-filtered (e.g. type-1 low-frequency-in-MPP
  factors) — ranking the full RF set by LOO perturbation will be dominated by lineage-marker
  noise rather than priming factors
- When the goal is pseudotime-based tipping-point detection along a trajectory rather
  than snapshot priming detection; the article recommends trajectory tools like Palantir
  or other tipping-point methods (BioTIP, scGET) for that setting
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DICE
  role: R package providing CompDICE (distance covariance entropy estimation) and
    FindPrimingFactors (LOO perturbation and Spearman-correlation pair ranking)
  repo: https://github.com/aet21/DICE
- name: R
  role: Implementation language of the DICE package and the LOO ranking workflow
- name: Palantir
  role: Used downstream to validate that DICE-identified primed MPP cells show elevated
    Palantir cell-fate probabilities for the corresponding lineage
  repo: https://github.com/dpeerlab/Palantir
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:e3f294c5d460407b78bc14cecad6c0ed7d6286476d8c64c9708bf2d29571d813
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# leave-one-out-entropy-perturbation-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rank candidate regulatory factors (RFs) implicated in multipotent-cell priming by measuring how much the Distance Covariance Entropy (DICE) statistic degrades when each RF is removed (leave-one-out), and rank RF pairs by Spearman correlation of their expression covariation over the multipotent progenitor cells.

## When to use

Use when a scRNA-Seq snapshot contains a multipotent population (e.g. MPPs) with competing downstream lineages, candidate priming RFs have already been filtered (type-1, low MPP-expression frequency), and you must prioritize which RFs and RF pairs define the primed state.

## When NOT to use

- When only a single terminal lineage exists with no competing lineage or multipotent population — LOO entropy covariation is defined over competing lineage priming within a multipotent pool
- When candidate RFs have not been pre-filtered (e.g. type-1 low-frequency-in-MPP factors) — ranking the full RF set by LOO perturbation will be dominated by lineage-marker noise rather than priming factors
- When the goal is pseudotime-based tipping-point detection along a trajectory rather than snapshot priming detection; the article recommends trajectory tools like Palantir or other tipping-point methods (BioTIP, scGET) for that setting

## Inputs

- Log-normalized scRNA-Seq expression matrix restricted to regulatory factors and the multipotent population (e.g. ranzoni: 1159 regulatory factors x 1659 cells, 1200 MPP cells per lineage)
- Vector of regulatory factors (transcription factors and chromatin-state modifiers) from the humanTF data object in the DICE R package
- Type-1 candidate RF list from FindCandidateFactors with type1fth='Mean'

## Outputs

- Ranked list of priming regulatory factors ordered by leave-one-out DICE perturbation
- Ranked RF pairs by Spearman correlation (e.g. UHRF1-IRF8 for GP, GATA1-KLF1 for MEMP)
- Primed-cell calls and primed fractions from downstream FindPrimedCells (88 GP-primed, 24 MEMP-primed cells)

## How to apply

After FindCandidateFactors retains type-1 RFs (MPP expression frequency below the mean over all candidate overexpressed RFs), compute the distance covariance entropy of the candidate RF set over the multipotent cells (e.g. 1200 MPP cells per lineage in the ranzoni fetal-liver dataset, useSpear=FALSE, std=TRUE via CompDICE). Then run FindPrimingFactors, which performs a leave-one-out (LOO) analysis: each RF is removed in turn while DICE is recomputed, and RFs whose removal most reduces DICE are ranked as the strongest priming factors; RF pairs are additionally ranked by Spearman correlation. In the Ranzoni fetal-liver hematopoiesis analysis this yielded UHRF1-IRF8 as the top GP-priming pair and GATA1-KLF1 as the top MEMP-priming pair. The ranked top pairs then feed FindPrimedCells to call the individual primed cells (~7% GP-primed, 88 cells; ~2% MEMP-primed, 24 cells).

## Related tools

- **DICE** (R package providing CompDICE (distance covariance entropy estimation) and FindPrimingFactors (LOO perturbation and Spearman-correlation pair ranking)) — https://github.com/aet21/DICE
- **R** (Implementation language of the DICE package and the LOO ranking workflow)
- **Palantir** (Used downstream to validate that DICE-identified primed MPP cells show elevated Palantir cell-fate probabilities for the corresponding lineage) — https://github.com/dpeerlab/Palantir

## Examples

```
library(DICE); data(ranzoni); data(humanTF);
```

## Evaluation signals

- Empirical P-values from CompPrimingPval against randomized null matrices with nMC=1000 Monte-Carlo runs must be below 0.05
- Leave-one-out perturbation should produce a clear separation: removing a top priming RF measurably decreases DICE while removing non-priming RFs does not
- Top-ranked RF pairs should reproduce the reported results on the ranzoni dataset: UHRF1-IRF8 for granulocyte progenitor priming and GATA1-KLF1 for MEMP priming
- Downstream FindPrimedCells fractions should be consistent with ~7% GP-primed (88 cells) and ~2% MEMP-primed (24 cells) with no overlapping primed cells between lineages
- Primed cells called from the ranked pairs should show elevated Palantir cell-fate probabilities toward the corresponding lineage

## Limitations

- In the real datasets analyzed, covariation patterns between RFs did not deviate strongly from monotonic linear dependencies, so DICE's non-linear sensitivity was not fully exercised on real data
- Validation relied on matched scATAC-Seq data (Ranzoni et al) and an approximate invented method to assign corresponding primed states in the ATAC data, since joint SHARE-Seq profiles were not available for fetal-liver hematopoiesis
- No experimental work was performed to demonstrate the importance of the identified RFs in priming; validation was computational only
- Explicit computed DICE statistics and P-values were printed but not shown in the text, so exact numeric thresholds for LOO perturbation magnitude are not documented

## Evidence

- [other] Task-card validation: top pairs and primed fractions: [evidence span withheld — anchor status: not_found]
