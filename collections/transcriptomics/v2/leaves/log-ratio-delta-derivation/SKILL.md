---
name: log-ratio-delta-derivation
description: Use when you have eLR pair interaction scores computed under a transcriptional
  perturbation (e.g. alpha-amanitin ZGA inhibition) and matched unperturbed conditions,
  and need a log-ratio delta per LR pair to call altered communication.
when_to_use_negative:
- 'The question concerns maternal-embryo interactions: TimeTalk''s autocrine assumption
  precludes exploring maternal-embryo interactions during early embryo development'
- 'The communication mode is not ligand-receptor mediated: the current TimeTalk framework
  does not account for other communication types such as EV-mediated signaling'
- No matched perturbed/unperturbed conditions exist, so a log-ratio delta between
  ZGA-inhibited and unperturbed 2-cell states cannot be defined
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TimeTalk
  role: R framework whose scripts compute eLR pair interaction scores on pseudo-bulk
    per-condition expression, which the delta log-ratio then compares
  repo: https://github.com/ChengLiLab/TimeTalk
- name: R
  role: Analysis environment (version 3.6.3) in which the pseudo-bulk computation,
    delta derivation, and threshold classification are performed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:35044bca427cbf13c9fc2ea4b99179ddcd21f40c960df956352aa32c563fdb12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - TimeTalk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# log-ratio-delta-derivation

## Summary

Derives a per-pair delta statistic from TimeTalk eLR interaction scores, comparing ZGA-inhibited (alpha-amanitin-treated, GSE71434) against unperturbed early/late 2-cell pseudo-bulk conditions to classify which eLR pairs show significantly altered communication strength upon ZGA inhibition (the 56 pairs of Fig. 3e).

## When to use

Use when you have eLR pair interaction scores computed under a transcriptional perturbation (e.g. alpha-amanitin ZGA inhibition) and matched unperturbed conditions, and need a log-ratio delta per LR pair to call altered communication.

## When NOT to use

- The question concerns maternal-embryo interactions: TimeTalk's autocrine assumption precludes exploring maternal-embryo interactions during early embryo development
- The communication mode is not ligand-receptor mediated: the current TimeTalk framework does not account for other communication types such as EV-mediated signaling
- No matched perturbed/unperturbed conditions exist, so a log-ratio delta between ZGA-inhibited and unperturbed 2-cell states cannot be defined

## Inputs

- Normalized expression matrices for early 2-cell, late 2-cell, and alpha-amanitin-treated embryos (ZGA-inhibition RNA-seq, GEO GSE71434)
- Pseudo-bulk per-condition expression for an average single cell in each stage
- TimeTalk interaction scores for each eLR pair per condition
- Defined log-ratio delta formula and delta threshold from the manuscript (Fig. 3e, Supplementary Figs. 12-13)

## Outputs

- Delta (log-ratio) value per eLR pair
- Binary classification of significantly altered vs. unaltered eLR pairs (expected set: 56 pairs)
- Reproduction of Fig. 3e and Supplementary Figs. 12-13 summarizing the altered pairs

## How to apply

Compute interaction scores per eLR pair with TimeTalk's R analysis scripts on pseudo-bulk per-condition expression matrices for early 2-cell, late 2-cell, and alpha-amanitin-treated embryos (GSE71434, ref. 30). Derive a delta value per eLR pair using the defined log-ratio formula comparing interaction scores between the ZGA-inhibited and unperturbed 2-cell conditions. Apply the delta threshold to classify each eLR pair as significantly altered or unchanged in communication strength. Rationale: TimeTalk was built for temporal autocrine signaling during early embryo development, and ZGA inhibition via alpha-amanitin is the manuscript's causal perturbation for testing whether eLR communication strength establishment depends on ZGA.

## Related tools

- **TimeTalk** (R framework whose scripts compute eLR pair interaction scores on pseudo-bulk per-condition expression, which the delta log-ratio then compares) — https://github.com/ChengLiLab/TimeTalk
- **R** (Analysis environment (version 3.6.3) in which the pseudo-bulk computation, delta derivation, and threshold classification are performed)

## Evaluation signals

- The recovered significantly-altered eLR pair set matches the 56 pairs reported in the manuscript's ZGA-inhibition analysis (Fig. 3e)
- Reproduced figures (Fig. 3e, Supplementary Figs. 12-13) show the same altered pairs, confirming alpha-amanitin abolishes the early-to-late 2-cell communication strength change
- Deltas are computed from interaction scores derived identically (pseudo-bulk, per-condition) across early 2-cell, late 2-cell, and alpha-amanitin-treated conditions
- Delta sign per pair correctly reflects direction of communication strength change (abolished vs. retained) relative to the unperturbed 2-cell baseline

## Limitations

- The inferred relationships should be confirmed by further wet experiments; only in silico validation was performed
- Users are advised to test multiple parameter combinations for dependable results, indicating TimeTalk outputs are parameter-sensitive
- Only the 2034 CellTalkDB mouse ligand-receptor pairs are in scope, and only LR-mediated communication is covered

## Evidence

- [discussion] ZGA inhibition RNA-seq data were obtained from a study by ref. 30.: "ZGA inhibition RNA-seq data were obtained from a study by ref. 30."
- [discussion] All statistical analyses were performed using R (version 3.6.3).: "All statistical analyses were performed using R (version 3.6.3)."
- [other] Derive a delta value per eLR pair using the defined log-ratio formula comparing interaction scores between ZGA-inhibited and unperturbed 2-cell conditions.: [evidence span withheld — anchor status: not_found]
- [other] ZGA inhibition altered the cell-cell communication strength of 56 eLR pairs, demonstrating that establishment of eLR-mediated communication correlates with the ZGA process.: [evidence span withheld — anchor status: not_found]
- [readme] we advise users to conduct additional data analysis and experiment with various parameter combinations to achieve more dependable results when utilizing TimeTalk for cell communication analysis.: "we advise users to conduct additional data analysis and experiment with various parameter combinations to achieve more dependable results when utilizing TimeTalk"
