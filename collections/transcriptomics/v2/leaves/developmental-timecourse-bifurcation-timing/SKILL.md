---
name: developmental-timecourse-bifurcation-timing
description: Use when a multi-stage (e.g. 5W-24W fetal retina) scRNA-Seq time-course
  shows a bifurcation into two competing lineages that trajectory methods like diffusion
  maps cannot resolve in time, and the exact transition stage must be identified.
when_to_use_negative:
- When there is only a static single-population snapshot with no competing lineages
  or stages to compare — DICE quantifies priming/bifurcation across populations or
  time
- When the goal is to identify primed cells or rank priming regulatory factors (use
  FindPrimedCells / FindPrimingFactors / CompPrimingPval instead), rather than pinpoint
  bifurcation timing
- When lineage-trajectory inference or cell-fate probability estimation is the goal
  (use Palantir), not tipping-point identification
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: DICE
  role: R package providing CompDICE to compute distance covariance entropy of TF
    expression covariation per stage and humanTF to restrict to regulatory factors
  repo: https://github.com/aet21/DICE
- name: BioTIP
  role: Competing DNB-based tipping-point method used as a negative/baseline comparison
    (its criticality index failed to show a clear transition in the retina case)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:8e1748e346892891a1dfbdae2827144f7dba85edbe1d8b56c1be2eb62b8ea793
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
  - CompDICE
  - DICE
  - R
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# developmental-timecourse-bifurcation-timing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pinpoints the timing of a cell-fate bifurcation in a developmental scRNA-Seq time-course by computing Distance Covariance Entropy (DICE) over selected lineage-antagonistic transcription factors with the DICE R package's CompDICE function, locating the stage where DICE reaches its minimum/rises as the critical transition point.

## When to use

Use when a multi-stage (e.g. 5W-24W fetal retina) scRNA-Seq time-course shows a bifurcation into two competing lineages that trajectory methods like diffusion maps cannot resolve in time, and the exact transition stage must be identified.

## When NOT to use

- When there is only a static single-population snapshot with no competing lineages or stages to compare — DICE quantifies priming/bifurcation across populations or time
- When the goal is to identify primed cells or rank priming regulatory factors (use FindPrimedCells / FindPrimingFactors / CompPrimingPval instead), rather than pinpoint bifurcation timing
- When lineage-trajectory inference or cell-fate probability estimation is the goal (use Palantir), not tipping-point identification

## Inputs

- Developmental time-course scRNA-Seq expression matrix (e.g. GEO GSE107618 human fetal retina, stages 5W-24W)
- Transcription factor list (DICE package data(humanTF))
- Terminal-cell labels defining the two competing downstream lineages

## Outputs

- Stage-wise DICE (distance covariance entropy) values per developmental stage
- Inferred bifurcation/critical transition stage (e.g. week 13)
- Results table (e.g. retina_dice_transition_results.csv) with DICE values and transition call

## How to apply

Load the time-course scRNA-Seq matrix (e.g. GSE107618 human fetal retina) in R and restrict it to regulatory factors using the humanTF list from the DICE package. Perform differential expression between terminal cells of the two lineages to identify antagonistic TFs (in the retina case, 17 candidates: 10 neuronal-specific and 7 epithelial-specific), then select an equal number of top TFs per lineage (top-7 each) so the inference is not biased toward one branch. For each developmental stage, run CompDICE on the selected TF expression covariation (useSpear=FALSE, std=TRUE) and plot DICE across stages; the bifurcation is marked where DICE reaches its minimum/rises (week 13 in the retina case). Optionally validate that variance-based or competing tipping-point methods (individual TF variances, BioTIP's criticality index) fail or are less smooth, supporting DICE's sharper pinpointing. Finally, write the stage-wise DICE table and the inferred transition stage to a results file such as retina_dice_transition_results.csv.

## Related tools

- **DICE** (R package providing CompDICE to compute distance covariance entropy of TF expression covariation per stage and humanTF to restrict to regulatory factors) — https://github.com/aet21/DICE
- **BioTIP** (Competing DNB-based tipping-point method used as a negative/baseline comparison (its criticality index failed to show a clear transition in the retina case))

## Examples

```
library(DICE); data(humanTF); dice.v <- CompDICE(tfExpr.m, useSpear=FALSE, std=TRUE); which.min(dice.v)
```

## Evaluation signals

- The stage-wise DICE curve should show a clear minimum/rise at a biologically consistent transition stage (week 13 for fetal retina, consistent with prior knowledge)
- Individual TF variances and BioTIP's criticality index should fail to show a comparably clear transition, serving as a contrast check
- Equal numbers of TFs must be selected per lineage (top-7 neuronal and top-7 epithelial) to avoid biasing the DICE inference
- CompDICE should be run with useSpear=FALSE and std=TRUE consistently across all stages
- Output CSV contains one DICE value per developmental stage plus the inferred transition week

## Limitations

- Results depend on the candidate TF set: DICE is computed over top-ranked antagonistic TFs of each lineage, so differential expression quality strongly affects the transition call
- In real datasets covariation patterns between RFs did not deviate strongly from monotonic linear dependencies, leaving DICE's non-linear capability less exercised
- DICE showed smoother patterns away from cell-fate transition points than DNB-based methods such as BioTIP, but a comprehensive comparison with other tipping-point algorithms (e.g. scGET) was not performed
- No experimental validation of the identified regulatory factors was performed; only computational/epigenomic validation was done

## Evidence

- [results] DICE computed over the top 7 TFs of each retinal lineage identifies 13W (week 13 of development) as the critical transition point: [evidence span withheld — anchor status: not_found]
- [results] CompDICE with useSpear=FALSE, std=TRUE: [evidence span withheld — anchor status: not_found]
- [abstract] DICE pinpoints bifurcation timing more precisely: "DICE can pinpoint the timing of bifurcations more precisely than lineage-trajectory inference algorithms or competing variance-based methods."
- [results] Equal TF selection rationale: [evidence span withheld — anchor status: not_found]
- [methods] CompDICE usage in DICE package: "We run the `CompDICE` function to estimate the distance covariance entropy of the candidate RFs for each lineage"
- [readme] DICE as R package with entropy of covariation: "DICE is an R-package which uses the concept of Distance Covariance Entropy to help quantify bifurcation dynamics from scRNA-Seq data."
