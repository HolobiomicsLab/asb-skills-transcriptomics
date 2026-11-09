---
name: zga-inhibition-perturbation-comparison
description: Use when testing whether cell-cell communication strength of candidate
  ligand-receptor pairs depends on a transcriptional process (e.g. ZGA) by comparing
  interaction scores between drug-perturbed and unperturbed embryo conditions.
when_to_use_negative:
- The goal is inferring which ligand-receptor pairs are development-related de novo
  from time-course scRNA-seq (use TimeTalk's eLR identification workflow instead of
  perturbation comparison)
- The perturbation does not act through transcriptional blockade (e.g. receptor-level
  or EV-mediated perturbations), since alpha-amanitin interpretation rests on abolishing
  ZGA transcription
- Studying maternal-embryo interactions, which the article's autocrine hypothesis
  and eLR framework preclude
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: TimeTalk
  role: R package providing the analysis scripts used to compute interaction scores
    for eLR pairs on pseudo-bulk per-condition expression
  repo: https://github.com/ChengLiLab/TimeTalk
- name: R
  role: Statistical analysis environment (v3.6.3 used in the study) in which the delta
    log-ratio computation and pair classification are run
  repo: https://github.com/ChengLiLab/TimeTalk
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:cbc69ff673c90da955fa69c0a67be34b42b2915539237bea4b611a56f890c371
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - TimeTalk
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# zga-inhibition-perturbation-comparison

## Summary

A perturbation-comparison skill that quantifies how blocking zygotic genome activation (ZGA) with alpha-amanitin changes eLR-mediated cell-cell communication, using TimeTalk interaction scores and a log-ratio delta statistic to recover the 56 altered eLR pairs reported in Fig. 3e.

## When to use

Use when testing whether cell-cell communication strength of candidate ligand-receptor pairs depends on a transcriptional process (e.g. ZGA) by comparing interaction scores between drug-perturbed and unperturbed embryo conditions.

## When NOT to use

- The goal is inferring which ligand-receptor pairs are development-related de novo from time-course scRNA-seq (use TimeTalk's eLR identification workflow instead of perturbation comparison)
- The perturbation does not act through transcriptional blockade (e.g. receptor-level or EV-mediated perturbations), since alpha-amanitin interpretation rests on abolishing ZGA transcription
- Studying maternal-embryo interactions, which the article's autocrine hypothesis and eLR framework preclude

## Inputs

- ZGA-inhibition bulk RNA-seq expression matrix for alpha-amanitin-treated embryos (GEO GSE71434)
- Unperturbed early 2-cell and late 2-cell scRNA-seq expression matrices
- Candidate eLR pair list (TimeTalk-derived eLRs, e.g. the 430 eLR set)
- TimeTalk R analysis scripts for interaction-score computation

## Outputs

- Per-eLR-pair interaction scores under early 2-cell, late 2-cell, and alpha-amanitin-treated conditions
- Per-eLR-pair delta values (log-ratio of interaction scores, ZGA-inhibited vs unperturbed 2-cell)
- Classified set of significantly altered eLR pairs (expected: 56 pairs)
- Reproduction of Fig. 3e and Supplementary Figs. 12-13

## How to apply

Download the ZGA-inhibition RNA-seq dataset (GEO accession GSE71434, alpha-amanitin-treated embryos from ref. 30) and normalize expression matrices alongside unperturbed early and late 2-cell data. Compute pseudo-bulk per-condition expression and run TimeTalk's R analysis scripts to derive interaction scores for each eLR pair under the early 2-cell, late 2-cell, and alpha-amanitin-treated conditions. For each eLR pair, calculate a delta value with the defined log-ratio formula comparing interaction scores between ZGA-inhibited and unperturbed 2-cell conditions. Apply the delta threshold to classify pairs as significantly altered upon ZGA inhibition, recovering the reported set of 56 pairs, then reproduce Fig. 3e and Supplementary Figs. 12-13. The rationale is that if establishment of eLR communication correlates with ZGA, blocking transcription with alpha-amanitin should abolish the interaction-strength change between early and late 2-cell stages.

## Related tools

- **TimeTalk** (R package providing the analysis scripts used to compute interaction scores for eLR pairs on pseudo-bulk per-condition expression) — https://github.com/ChengLiLab/TimeTalk
- **R** (Statistical analysis environment (v3.6.3 used in the study) in which the delta log-ratio computation and pair classification are run) — https://github.com/ChengLiLab/TimeTalk

## Examples

```
devtools::install_github("ChengLiLab/TimeTalk",ref="main")
```

## Evaluation signals

- The classified altered-pair set matches the 56 eLR pairs reported in the manuscript's ZGA-inhibition analysis (Fig. 3e)
- Interaction scores are computed under all three conditions (early 2-cell, late 2-cell, alpha-amanitin-treated) before delta derivation
- Delta values are derived with the paper's defined log-ratio formula comparing ZGA-inhibited vs unperturbed 2-cell conditions
- Classification applies the specified delta threshold without manual re-ranking of pairs
- Resulting figures reproduce Fig. 3e and Supplementary Figs. 12-13 summarizing the altered pairs

## Limitations

- The alpha-amanitin interpretation assumes the drug specifically blocks ZGA transcription; the conclusion is that blocking transcription abolishes the early-to-late 2-cell communication-strength change
- Granger/interaction conclusions in TimeTalk are affected by the choice of the winsz interpolation parameter; users are advised to test various parameter combinations
- The current TimeTalk framework covers only ligand-receptor-mediated communication; other communication types (e.g. EV-mediated) are not accounted for

## Evidence

- [results] ZGA inhibition altered the cell-cell communication strength of 56 eLR pairs: [evidence span withheld — anchor status: not_found]
- [discussion] ZGA inhibition RNA-seq data source: "ZGA inhibition RNA-seq data were obtained from a study by ref. 30."
- [intro] TimeTalk is an R package: "TimeTalk is an R package that is available at https://github.com/ChengLiLab/TimeTalk."
- [readme] parameter sensitivity of Granger conclusions: "According to our simulation results, the Granger causality conclusion is affected by choice of parameter winsz."
- [discussion] scope of communication types: "the current version of the TimeTalk framework is designed for cell-cell communication mediated by ligand-receptor interactions."
