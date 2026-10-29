---
name: leave-one-out-perturbation-analysis
description: Use when you have scRNA-Seq data with a multipotent (MPP/TAC) population
  and competing downstream lineages, and you need to rank which regulatory factors
  and RF pairs drive priming of one lineage over another.
when_to_use_negative:
- The dataset lacks a clearly annotated multipotent progenitor population plus competing
  downstream lineages, since DICE priming quantification is defined for that bifurcation
  setting
- The goal is general tipping-point/bifurcation timing rather than ranking priming
  factors — alternative tools like BioTIP or scGET may be more appropriate
- Only scATAC-Seq (no scRNA-Seq) expression data are available; DICE priming and RF
  ranking require gene expression matrices
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DICE
  role: R package providing FindCandidateFactors, CompDICE, CompPrimingPval, FindPrimingFactors
    (rankedLOO, rankedPairs) and FindPrimedCells used to perform the leave-one-out
    and pair-perturbation analysis
  repo: https://github.com/aet21/DICE
- name: Palantir
  role: Used post hoc to estimate cell-fate probabilities for multipotent cells, validating
    the lineage assignments of cells called as primed by the identified RF pairs
  repo: https://github.com/dpeerlab/Palantir
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:4adb6eea582a9d87d2d638a68732f10c73d9f40befb000d500069b9117eee1e3
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# leave-one-out-perturbation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A DICE-framework procedure that ranks candidate regulatory factors (RFs) implicated in lineage priming by removing each RF individually while recomputing the distance covariance entropy (leave-one-out perturbation), and identifies priming RF pairs via dual-perturbation Spearman rank correlation.

## When to use

Use when you have scRNA-Seq data with a multipotent (MPP/TAC) population and competing downstream lineages, and you need to rank which regulatory factors and RF pairs drive priming of one lineage over another.

## When NOT to use

- The dataset lacks a clearly annotated multipotent progenitor population plus competing downstream lineages, since DICE priming quantification is defined for that bifurcation setting
- The goal is general tipping-point/bifurcation timing rather than ranking priming factors — alternative tools like BioTIP or scGET may be more appropriate
- Only scATAC-Seq (no scRNA-Seq) expression data are available; DICE priming and RF ranking require gene expression matrices

## Inputs

- Log-normalized scRNA-Seq expression matrix restricted to regulatory factors (e.g. SHARE-Seq mouse skin, GEO: GSE140203)
- Cell-type annotation defining the multipotent population (e.g. 4378 TACs) and the mature downstream lineages (e.g. IRS and HS)
- List of regulatory factors / TFs (e.g. humanTF from the DICE package)

## Outputs

- Ranked list of candidate priming RFs by leave-one-out DICE perturbation
- Ranked list of RF pairs by dual-perturbation Spearman correlation (e.g. Foxo3-Lmo7 for IRS, Runx2-Zfhx3 for HS)
- Primed-cell identifiers with DICE values and empirical P-values (e.g. primed_tacs.csv)
- Empirical P-values of priming against the randomized null DICE distribution

## How to apply

First identify candidate lineage-specific RFs with FindCandidateFactors (mpp = TAC/MPP population, linCT.v = the competing lineages), applying the type1fth='Mean' filter that keeps only overexpressed RFs whose frequency of expression in the multipotent population is below the mean. Compute per-lineage DICE values with CompDICE (e.g. useSpear=FALSE, std=TRUE) and test significance with CompPrimingPval using at least 1000 Monte-Carlo randomizations (nMC=1000) of the data matrices to build the null DICE distribution. Then run FindPrimingFactors: it ranks candidate RFs by leave-one-out (LOO) DICE perturbation (rankedLOO), where each RF is removed while DICE is recomputed, and ranks RF pairs by Spearman correlation (rankedPairs) of the dual perturbations; in the SHARE-Seq hair-follicle application this identified Foxo3-Lmo7 as the IRS priming pair and Runx2-Zfhx3 as the HS priming pair. Finally call primed cells with FindPrimedCells using the identified RF pairs and validate lineage assignments against Palantir-derived fate probabilities.

## Related tools

- **DICE** (R package providing FindCandidateFactors, CompDICE, CompPrimingPval, FindPrimingFactors (rankedLOO, rankedPairs) and FindPrimedCells used to perform the leave-one-out and pair-perturbation analysis) — https://github.com/aet21/DICE
- **Palantir** (Used post hoc to estimate cell-fate probabilities for multipotent cells, validating the lineage assignments of cells called as primed by the identified RF pairs) — https://github.com/dpeerlab/Palantir

## Examples

```
library(DICE); data(humanTF); ranks.o <- FindPrimingFactors(mpp, linCT.v = c('IRS','HS'))
```

## Evaluation signals

- The observed DICE value for each lineage must fall significantly below the randomized null distribution, with empirical P < 0.01 using nMC >= 1000 Monte-Carlo randomizations
- The fraction of primed TACs per lineage should be a low proportion of the multipotent population, consistent with sparse priming (e.g. 131 HS-primed and 196 IRS-primed of 4378 TACs)
- Removing the top-ranked RF in the LOO analysis should produce the largest drop/perturbation in the recomputed DICE value relative to other candidates
- Cells called primed via FindPrimedCells should show higher Palantir-derived fate probabilities for the corresponding lineage (as reported for the HS-primed TACs)
- Primed-cell identifiers, DICE values, P-values and ranked pairs should be exported consistently (e.g. written to primed_tacs.csv)

## Limitations

- Explicit values of the computed DICE statistics and LOO P-values are printed but not shown in the article text, so exact numeric reproducibility may be hard to check
- scRNA-Seq has low sensitivity to detect dynamic changes in TF expression, which can affect candidate-RF detection
- In the real datasets analyzed, covariation patterns between RFs did not deviate strongly from monotonic linear dependencies, so DICE's non-linear capability was not strongly exercised
- No experimental work demonstrated the importance of the identified RFs in priming; validation relied on scATAC-Seq data
- Randomized results depend on the number of Monte-Carlo randomizations; at least nMC=1000 is required for stable empirical P-values

## Evidence

- [other] Monte-Carlo null for priming significance: [evidence span withheld — anchor status: not_found]
