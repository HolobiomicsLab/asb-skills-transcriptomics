---
name: scrna-seq-stage-wise-dice-dynamics
description: Use when a time-course or staged single-cell RNA-Seq dataset spans a
  known or suspected lineage bifurcation (e.g., retinal progenitors splitting into
  neuronal vs epithelial fates) and the exact transition/tipping stage must be resolved.
when_to_use_negative:
- The goal is generic trajectory/pseudotime inference or cell clustering rather than
  localizing a bifurcation point between competing lineages — trajectory algorithms
  like Palantir serve that purpose and cannot resolve the transition timing.
- No candidate lineage-specific regulatory factors can be defined (no differential
  expression between terminal branches), so there is no TF covariation signal for
  DICE to quantify.
- The population is a single continuous lineage with no competing downstream fates,
  so no priming/bifurcation dynamic exists to detect.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DICE
  role: R package providing CompDICE (stage-wise distance covariance entropy with
    useSpear=FALSE, std=TRUE), CompPrimingPval (significance vs randomized matrices),
    CreateDPTbin (pseudotime binning), and the humanTF regulatory-factor list.
  repo: github.com/aet21/DICE
- name: R
  role: Language in which the DICE package is implemented and the stage-wise workflow
    (DE analysis, TF selection, CompDICE per stage, CSV export) is executed.
  repo: github.com/aet21/DICE
- name: BioTIP
  role: DNB-based tipping-point method used as a negative comparison; its criticality
    index failed to show a clear transition where DICE succeeded.
- name: Palantir
  role: Trajectory-inference method mentioned in the article as a way to estimate
    cell-fate probabilities; DICE pinpoints bifurcation timing more precisely than
    such lineage-trajectory inference algorithms.
  repo: dpeerlab/Palantir
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:29233a6cc01bf57eed2c8e767715a88f66dd0764c4c1053f37e683a07d7782a4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aet21/DICE
  tools:
  - BioTIP
  - CompDICE
  - DICE
  - Palantir
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# scrna-seq-stage-wise-dice-dynamics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes distance covariance entropy (DICE) over successive developmental stages or pseudotime bins of a scRNA-Seq time-course to pinpoint the timing of a lineage bifurcation. It outperforms variance-based and trajectory-inference approaches because it tracks covariation entropy of lineage-specific regulatory factors rather than individual gene variance.

## When to use

Use when a time-course or staged single-cell RNA-Seq dataset spans a known or suspected lineage bifurcation (e.g., retinal progenitors splitting into neuronal vs epithelial fates) and the exact transition/tipping stage must be resolved.

## When NOT to use

- The goal is generic trajectory/pseudotime inference or cell clustering rather than localizing a bifurcation point between competing lineages — trajectory algorithms like Palantir serve that purpose and cannot resolve the transition timing.
- No candidate lineage-specific regulatory factors can be defined (no differential expression between terminal branches), so there is no TF covariation signal for DICE to quantify.
- The population is a single continuous lineage with no competing downstream fates, so no priming/bifurcation dynamic exists to detect.

## Inputs

- scRNA-Seq time-course/stage-series expression matrix (e.g., GSE107618 human fetal retina, 5W–24W)
- List of regulatory factors / transcription factors (e.g., humanTF from the DICE package)
- Differentially expressed, lineage-antagonistic TF sets (equal counts per branch, e.g., top-7 per lineage)
- Stage or pseudotime bin assignments for cells (e.g., CreateDPTbin with minBins=7)

## Outputs

- Stage-wise DICE values (distance covariance entropy per developmental stage)
- Inferred bifurcation/tipping-point stage (e.g., week 13 for retina)
- Stage-wise DICE table exported to CSV (e.g., retina_dice_transition_results.csv)

## How to apply

Restrict the expression matrix to regulatory factors using the humanTF list, then run a differential expression analysis between terminal cells of the two competing lineages to identify antagonistic, lineage-specific TFs. Select an EQUAL number of top TFs per branch (top-7 neuronal and top-7 epithelial in the retina case; top-4 per branch in the liver case) so inference is not biased toward one lineage. For each stage (e.g., 5W to 24W), run the DICE package's CompDICE function on the selected TF expression covariation with useSpear=FALSE and std=TRUE, and optionally assess significance against randomized null matrices with CompPrimingPval. Plot DICE as a function of stage and call the bifurcation at the stage where DICE reaches its minimum or begins to rise — in the retina dataset this identified 13W, consistent with prior knowledge, while individual TF variances and BioTIP's criticality index showed no clear transition.

## Related tools

- **DICE** (R package providing CompDICE (stage-wise distance covariance entropy with useSpear=FALSE, std=TRUE), CompPrimingPval (significance vs randomized matrices), CreateDPTbin (pseudotime binning), and the humanTF regulatory-factor list.) — github.com/aet21/DICE
- **R** (Language in which the DICE package is implemented and the stage-wise workflow (DE analysis, TF selection, CompDICE per stage, CSV export) is executed.) — github.com/aet21/DICE
- **BioTIP** (DNB-based tipping-point method used as a negative comparison; its criticality index failed to show a clear transition where DICE succeeded.)
- **Palantir** (Trajectory-inference method mentioned in the article as a way to estimate cell-fate probabilities; DICE pinpoints bifurcation timing more precisely than such lineage-trajectory inference algorithms.) — dpeerlab/Palantir

## Examples

```
library(DICE); dice.v <- CompDICE(tf.expr.m, useSpear=FALSE, std=TRUE); write.csv(data.frame(stage=stages, DICE=dice.v), 'retina_dice_transition_results.csv')
```

## Evaluation signals

- The stage-wise DICE curve is smooth away from the transition and shows a clear minimum/rise at a single stage (e.g., 13W in the retina time-course).
- The inferred transition stage agrees with independent prior knowledge of the developmental event.
- Equal numbers of candidate TFs are selected per lineage branch (e.g., 7 vs 7) so the entropy is not biased toward one branch.
- Individual TF variances and competing tipping-point methods (e.g., BioTIP's criticality index) are checked and expected NOT to show a comparably clear transition.
- The stage-wise DICE table and the inferred transition week are written to the results CSV and are internally consistent.

## Limitations

- Sampling of cells from the transition region is often very sparse in scRNA-Seq data, making tipping-point identification intrinsically difficult.
- In the real datasets analyzed, covariation patterns between regulatory factors did not deviate strongly from monotonic linear dependencies, so DICE's non-linear capability is less exercised on real data.
- Explicit computed DICE statistics and P-values were printed but not reported in the paper, and DICE/dependency version numbers were not documented, complicating exact reproduction.
- TF expression changes are hard to detect due to scRNA-Seq's low sensitivity, so candidate RF selection via differential overexpression is a critical, potentially fragile upstream step.

## Evidence

- [methods] Equal per-branch TF selection avoids biasing the inference: [evidence span withheld — anchor status: not_found]
