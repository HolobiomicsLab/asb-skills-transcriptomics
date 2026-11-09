---
name: lineage-antagonistic-tf-selection
description: 'Use when preparing the regulatory-factor input for DICE/CompDICE bifurcation-timing
  analysis: after differential expression between terminal cells of competing lineages,
  to pick an equal number of antagonistic lineage-specific TFs per branch.'
when_to_use_negative:
- When the goal is trajectory inference or pseudotime estimation itself — use Palantir
  instead; DICE-based TF selection quantifies bifurcation timing, not cell orderings.
- When candidate regulatory factors have not yet been filtered by differential overexpression
  between lineages — applying entropy computation to an unbalanced or arbitrary TF
  set can bias the inferred transition point toward one lineage.
- When the population is not multipotent/bifurcating (no competing downstream lineages)
  — the antagonistic-TF logic assumes two competing fates.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DICE
  role: R package whose humanTF list restricts the expression matrix to regulatory
    factors and whose CompDICE function computes distance covariance entropy over
    the selected lineage-antagonistic TFs (useSpear=FALSE, std=TRUE)
  repo: https://github.com/aet21/DICE
- name: R
  role: Implementation environment for the differential expression, TF selection,
    and CompDICE pipeline
  repo: https://github.com/aet21/DICE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:bc8f76032ec23ee964620cb0ce020eb6938417152669577e8cf06a3734dd7523
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
  - CompDICE
  - DICE
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# lineage-antagonistic-tf-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects an equal number of top lineage-specific, antagonistically expressed transcription factors (e.g. top-7 neuronal vs top-7 epithelial TFs, or top-4 cholangiocyte vs top-4 hepatocyte factors) as the balanced regulatory-factor input required for DICE bifurcation-timing analysis.

## When to use

Use when preparing the regulatory-factor input for DICE/CompDICE bifurcation-timing analysis: after differential expression between terminal cells of competing lineages, to pick an equal number of antagonistic lineage-specific TFs per branch.

## When NOT to use

- When the goal is trajectory inference or pseudotime estimation itself — use Palantir instead; DICE-based TF selection quantifies bifurcation timing, not cell orderings.
- When candidate regulatory factors have not yet been filtered by differential overexpression between lineages — applying entropy computation to an unbalanced or arbitrary TF set can bias the inferred transition point toward one lineage.
- When the population is not multipotent/bifurcating (no competing downstream lineages) — the antagonistic-TF logic assumes two competing fates.

## Inputs

- scRNA-Seq expression matrix of developmental time-course cells (e.g. GEO: GSE107618 human fetal retina, 5W-24W)
- humanTF transcription factor list (data(humanTF) from the DICE package)
- Cell-type labels distinguishing terminal cells of each competing lineage (e.g. neuronal vs epithelial; hepatocyte vs cholangiocyte)
- Differential expression results between terminal lineage cells identifying antagonistic TFs

## Outputs

- Balanced matrix of lineage-specific TF expression (top-N TFs per branch, equal N for both lineages)
- Per-lineage TF sets used as CompDICE input across developmental stages
- Stage-wise DICE table and inferred bifurcation/transition point (e.g. week 13 in the retina time-course)

## How to apply

Restrict the scRNA-Seq expression matrix to transcription factors using the humanTF list, then perform differential expression between the terminal cells of each competing lineage to identify antagonistic TFs upregulated in each branch (e.g. 17 candidate TFs: 10 neuronal-specific, 7 epithelial-specific in the human fetal retina GSE107618 dataset). Select the top-N TFs per lineage, choosing an EQUAL number of TFs for each branch (top-7 per lineage for retina; top-4 per lineage for liver: LSR, ELF3, IRF6, BGN for cholangiocyte vs NR1I3, NR1I2, FOXA2, TRIM15 for hepatocyte) to avoid biasing the DICE inference toward one lineage. Feed the balanced TF-by-cell expression matrix for each lineage into CompDICE (e.g. useSpear=FALSE, std=TRUE) across each developmental stage or pseudotime bin, and locate the stage at which DICE reaches its minimum/rises as the bifurcation point. This differential-overexpression-first strategy exists because scRNA-Seq has low sensitivity to detect dynamic changes in TF expression, so candidate RFs must be identified before entropy covariation is computed.

## Related tools

- **DICE** (R package whose humanTF list restricts the expression matrix to regulatory factors and whose CompDICE function computes distance covariance entropy over the selected lineage-antagonistic TFs (useSpear=FALSE, std=TRUE)) — https://github.com/aet21/DICE
- **R** (Implementation environment for the differential expression, TF selection, and CompDICE pipeline) — https://github.com/aet21/DICE

## Examples

```
data(humanTF); dice.v <- CompDICE(topTFs.m, useSpear=FALSE, std=TRUE)
```

## Evaluation signals

- The two lineages contribute an equal number of TFs (e.g. top-7 vs top-7, top-4 vs top-4) — asymmetric selection is a known bias risk.
- All selected TFs are contained in the humanTF regulatory-factor list and show lineage-antagonistic differential expression (upregulated in one terminal branch, not the other).
- CompDICE over the selected TF set yields a smooth stage-wise entropy profile with a clear minimum/rise at the known bifurcation (e.g. week 13 of retinal development), whereas individual TF variances and BioTIP criticality show no clear transition.
- Output CSV contains both the stage-wise DICE table and the inferred transition week, and the transition week is consistent with prior developmental knowledge.

## Limitations

- scRNA-Seq has low sensitivity to detect dynamic changes in TF expression, so candidate identification depends on differential overexpression between terminal lineages rather than on dynamics along the trajectory.
- The number of top TFs per lineage is a practitioner choice (top-4 in liver, top-7 in retina); equal numbers per branch must be enforced to avoid biasing DICE toward one lineage.
- In the analyzed real datasets, covariation patterns between RFs did not deviate strongly from monotonic linear dependencies, so the non-linear capability of DICE was less exercised; explicit computed DICE values/P-values may be printed but not reported in text.

## Evidence
