---
name: primed-cell-state-calling
description: Use when a multipotent cell population (e.g. MPPs or TACs) has been shown
  by CompDICE/CompPrimingPval to be significantly primed (empirical P < 0.05 from
  nMC >= 1000 randomizations) and top RF pairs are known, and you need per-cell primed-state
  assignments from scRNA-Seq alone.
when_to_use_negative:
- The multipotent population has not first been shown to be significantly primed via
  CompPrimingPval against randomized null matrices — calling cells without this significance
  step is unsupported
- No candidate regulatory-factor pairs have been ranked by FindPrimingFactors (LOO
  perturbation and Spearman correlation), since FindPrimedCells requires the top RF
  pairs as input
- The goal is to detect bifurcation timing over pseudotime rather than call primed
  cells in a static snapshot population — DICE's binning (CreateDPTbin, minBins=7)
  workflow is a different skill
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DICE
  role: R package providing FindPrimedCells, FindPrimingFactors, CompDICE and CompPrimingPval
    to quantify priming and call primed cells from scRNA-Seq covariation entropy
  repo: https://github.com/aet21/DICE
- name: Palantir
  role: Estimates cell-fate probabilities for all multipotent cells to validate the
    primed-cell assignments
  repo: https://github.com/dpeerlab/Palantir
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:db4fef684fa4d49829b46134dcbf86c249aa1ff5902203ace499b51fc7b06582
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# primed-cell-state-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calls individual primed cells within a multipotent progenitor population using the DICE R package's FindPrimedCells function, based on co-expression of the top-ranked regulatory-factor pairs identified by leave-one-out DICE perturbation and Spearman correlation.

## When to use

Use when a multipotent cell population (e.g. MPPs or TACs) has been shown by CompDICE/CompPrimingPval to be significantly primed (empirical P < 0.05 from nMC >= 1000 randomizations) and top RF pairs are known, and you need per-cell primed-state assignments from scRNA-Seq alone.

## When NOT to use

- The multipotent population has not first been shown to be significantly primed via CompPrimingPval against randomized null matrices — calling cells without this significance step is unsupported
- No candidate regulatory-factor pairs have been ranked by FindPrimingFactors (LOO perturbation and Spearman correlation), since FindPrimedCells requires the top RF pairs as input
- The goal is to detect bifurcation timing over pseudotime rather than call primed cells in a static snapshot population — DICE's binning (CreateDPTbin, minBins=7) workflow is a different skill

## Inputs

- Log-normalized scRNA-Seq expression matrix restricted to regulatory factors (e.g. ranzoni: 1159 RFs x 1659 cells, or SHARE-Seq mouse skin GSE140203)
- Vector of regulatory factors (e.g. humanTF from the DICE package)
- Top-ranked lineage-specific RF pairs from FindPrimingFactors (rankedLOO and rankedPairs outputs)
- Cell-type labels distinguishing the multipotent population (MPPs/TACs) from downstream lineage populations

## Outputs

- Primed-cell identifiers per lineage (e.g. 88 GP-primed and 24 MEMP-primed MPPs; 196 IRS-primed and 131 HS-primed TACs)
- Primed-cell fractions relative to the multipotent population
- Optionally a table of primed-cell IDs, DICE values, P-values and ranked RF pairs (e.g. primed_tacs.csv)

## How to apply

After FindCandidateFactors (with type1fth='Mean' retaining low-frequency, primed-subset RFs), CompDICE (useSpear=FALSE, std=TRUE) and CompPrimingPval (nMC=1000) establish significant priming, run FindPrimingFactors to rank candidate RFs by leave-one-out DICE perturbation and RF pairs by Spearman correlation (e.g. UHRF1-IRF8 for GP, GATA1-KLF1 for MEMP in Ranzoni fetal liver; Foxo3-Lmo7 for IRS, Runx2-Zfhx3 for HS in SHARE-Seq skin). Then run FindPrimedCells on the multipotent population using the top-ranked RF pairs to call lineage-primed cells based on co-expression of the pair members. Report primed fractions and, when overlapping assignments occur between lineages, exclude the overlapping cells (intersect of the two primed-cell vectors). Validate calls with Palantir-derived cell-fate probabilities or matched scATAC-Seq accessibility, and expect sparse priming fractions (e.g. ~7% GP-primed / 88 cells and ~2% MEMP-primed / 24 cells).

## Related tools

- **DICE** (R package providing FindPrimedCells, FindPrimingFactors, CompDICE and CompPrimingPval to quantify priming and call primed cells from scRNA-Seq covariation entropy) — https://github.com/aet21/DICE
- **Palantir** (Estimates cell-fate probabilities for all multipotent cells to validate the primed-cell assignments) — https://github.com/dpeerlab/Palantir

## Examples

```
library(DICE); data(ranzoni); data(humanTF); prime.GP.o <- FindPrimedCells(mpp.expr.m = ranzoni.m[, mpp.cell.v], pair.v = c('UHRF1','IRF8'), lin.expr.m = gp.expr.m)
```

## Evaluation signals

- Empirical P-value from CompPrimingPval (nMC >= 1000 Monte-Carlo runs) is below 0.05 (article: P < 0.01 in the SHARE-Seq application) before calling cells
- Primed-cell fractions are a low proportion of the multipotent population, consistent with sparse priming (e.g. ~7% GP-primed / 88 cells, ~2% MEMP-primed / 24 cells; 196 IRS-primed and 131 HS-primed TACs)
- Primed cells show higher Palantir-derived lineage-fate probabilities than non-primed multipotent cells (e.g. HS-primed TACs confirmed by higher HS-fate probabilities)
- Overlapping primed-cell assignments between competing lineages are identified and excluded via intersect of the two primed-cell vectors
- Wilcoxon rank-sum tests find marker genes significantly higher in primed vs non-primed multipotent cells, and matched scATAC-Seq accessibility at those markers supports the calls

## Limitations

- The DICE package version number and dependency versions (destiny, chromVAR, Seurat, Palantir) are not recorded, complicating exact reproducibility
- Validation of primed states in scATAC-Seq data relied on an invented, approximate method for assigning corresponding primed states, and no experimental work demonstrated the importance of the identified RFs in priming
- Explicit computed DICE statistics and P-values were printed but not shown in the text, so exact reference values are unavailable
- Palantir's release notes note that since switching to numpy.random.Generator (PCG64), numerical outputs may differ from previous versions — pin prior versions for exact reproducibility of the validation step

## Evidence
