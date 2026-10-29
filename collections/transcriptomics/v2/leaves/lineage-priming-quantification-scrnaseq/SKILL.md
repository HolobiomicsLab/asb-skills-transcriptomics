---
name: lineage-priming-quantification-scrnaseq
description: Use when a scRNA-Seq dataset contains a multipotent progenitor (MPP)
  population plus competing downstream lineages (e.g. GP vs MEMP bifurcation) and
  you need to detect primed subpopulations and their regulatory factors without matched
  scATAC-Seq data.
when_to_use_negative:
- The question is identifying a cell-fate transition (tipping) point along a pseudotime
  trajectory rather than detecting primed subpopulations in a snapshot multipotent
  population
- The dataset lacks a clearly defined multipotent progenitor population with two competing
  downstream lineages, so the differential overexpression comparison of progenitors
  vs MPPs cannot be performed
- You need differential expression markers of known lineage-committed cells rather
  than de novo detection of primed states and their regulatory factors
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DICE
  role: Core R package providing FindCandidateFactors, CompDICE, CompPrimingPval,
    FindPrimingFactors and FindPrimedCells to quantify priming and identify RF pairs
    and primed cells from scRNA-Seq
  repo: github.com/aet21/DICE
- name: Palantir
  role: Used downstream to estimate cell-fate probabilities for all MPP cells, validating
    the primed cells identified by DICE
  repo: dpeerlab/Palantir
- name: Seurat
  role: Used to log-normalize the gene-level accessibility matrix built from scATAC-Seq
    peaks when validating primed states with matched epigenomic data
  repo: GreenleafLab/chromVAR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:d06f3b8a2a0c7229e411c9e6989896d08dd341406a46648c99c791a9c6eef4ef
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
  - Seurat
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# lineage-priming-quantification-scrnaseq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify lineage priming in multipotent progenitor populations from scRNA-Seq data alone using the DICE R-package, which computes Distance Covariance Entropy (DICE) over candidate regulatory factors, tests significance against randomized null matrices, and identifies the primed RF pairs and primed cells.

## When to use

Use when a scRNA-Seq dataset contains a multipotent progenitor (MPP) population plus competing downstream lineages (e.g. GP vs MEMP bifurcation) and you need to detect primed subpopulations and their regulatory factors without matched scATAC-Seq data.

## When NOT to use

- The question is identifying a cell-fate transition (tipping) point along a pseudotime trajectory rather than detecting primed subpopulations in a snapshot multipotent population
- The dataset lacks a clearly defined multipotent progenitor population with two competing downstream lineages, so the differential overexpression comparison of progenitors vs MPPs cannot be performed
- You need differential expression markers of known lineage-committed cells rather than de novo detection of primed states and their regulatory factors

## Inputs

- Log-normalized scRNA-Seq matrix restricted to regulatory factors (TFs and chromatin-state modifiers), e.g. the ranzoni matrix (1159 RFs x 1659 cells) loaded via data(ranzoni)
- Regulatory-factor annotation vector, e.g. data(humanTF)
- Cell-type labels distinguishing the MPP population from each downstream lineage progenitor population (e.g. Granulocyte Prog, Comm-Prog)

## Outputs

- Type-1 candidate RF lists per lineage (low-frequency, primed-subset factors)
- Empirical DICE priming P-values computed from 1000 Monte-Carlo randomized null matrices
- Ranked priming RF pairs by LOO DICE perturbation and Spearman correlation (e.g. UHRF1-IRF8 for GP, GATA1-KLF1 for MEMP)
- Primed-cell calls and primed fractions (e.g. ~7% GP-primed / 88 cells, ~2% MEMP-primed / 24 cells)

## How to apply

Load the log-normalized RF-by-cell matrix (e.g. ranzoni: 1159 regulatory factors x 1659 cells) and the humanTF RF vector from the DICE R-package (installed via devtools::install_github('aet21/DICE')). Run FindCandidateFactors comparing each lineage progenitor population against the MPPs and the other lineage with type1fth='Mean', so candidate RFs whose MPP expression frequency is below the mean are retained as type-1 (low-frequency, primed-subset) factors. Compute distance covariance entropy per lineage with CompDICE over the MPP cells (useSpear=FALSE, std=TRUE), then assess significance with CompPrimingPval against randomized null data matrices using nMC=1000 Monte-Carlo runs. Rank candidate RFs by leave-one-out (LOO) DICE perturbation and rank RF pairs by Spearman correlation with FindPrimingFactors (e.g. UHRF1-IRF8 for GP, GATA1-KLF1 for MEMP), then call primed cells with FindPrimedCells using the top-ranked pairs. Rationale: primed states are defined by epigenetic regulators or pioneer factors alongside lineage-specific TFs, and DICE covariation entropy is sensitive to the non-random expression co-activation that precedes bifurcation.

## Related tools

- **DICE** (Core R package providing FindCandidateFactors, CompDICE, CompPrimingPval, FindPrimingFactors and FindPrimedCells to quantify priming and identify RF pairs and primed cells from scRNA-Seq) — github.com/aet21/DICE
- **Palantir** (Used downstream to estimate cell-fate probabilities for all MPP cells, validating the primed cells identified by DICE) — dpeerlab/Palantir
- **Seurat** (Used to log-normalize the gene-level accessibility matrix built from scATAC-Seq peaks when validating primed states with matched epigenomic data) — GreenleafLab/chromVAR

## Examples

```
library(DICE); data(ranzoni); data(humanTF); cand <- FindCandidateFactors(ranzoni, humanTF, type1fth='Mean'); dice.v <- CompDICE(cand, useSpear=FALSE, std=TRUE); pvals <- CompPrimingPval(ranzoni, nMC=1000)
```

## Evaluation signals

- Empirical P-values from the 1000-run Monte-Carlo null (CompPrimingPval) must be below 0.05 for significant priming
- Primed-cell fractions should reproduce the reported scale: ~7% GP-primed (88 cells) and ~2% MEMP-primed (24 cells) in the fetal liver MPP population
- Top RF pairs should match known biology of the bifurcation, e.g. UHRF1-IRF8 for granulocyte and GATA1-KLF1 for MEMP priming
- Type-1 candidate RFs must all have MPP expression frequency below the mean frequency over all candidate overexpressed RFs (type1fth='Mean' invariant)
- Overlapping primed-cell assignments between the two lineages should be excluded before reporting primed fractions

## Limitations

- Matched scATAC-Seq validation relied on an invented, approximate method for assigning corresponding primed states because SHARE-Seq (joint scRNA-Seq & scATAC-Seq) profiles were not available for the liver hematopoiesis study
- No experimental work was performed to demonstrate the importance of the identified RFs in priming; only scATAC-Seq validation was done
- In the real datasets analyzed, covariation patterns between RFs did not deviate strongly from monotonic linear dependencies, leaving DICE's non-linear capability less exercised
- Explicit computed DICE statistics and P-values are printed but their values are not shown in the text, and software/package version numbers are not specified

## Evidence
