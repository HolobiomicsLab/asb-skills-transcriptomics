---
name: hesc-npc-potency-population-comparison
description: Use when analysing snapshot scRNA-Seq populations (e.g. hESC vs progenitor/NPC)
  where differentiation potency must be inferred without marker genes or timepoint
  labels, and population-level potency heterogeneity (Shannon index) is the target
  output.
when_to_use_negative:
- When a reliable marker gene or timepoint label is available and potency ordering
  can be done directly — SCENT is intended for cases lacking prior biological knowledge
  such as marker expression or timepoint.
- When the research question is trajectory reconstruction rather than potency-state
  assignment and population heterogeneity — the paper notes this insight is not readily
  obtainable with Monocle or MPath, but those tools serve a different purpose.
- When the dataset is a feature table already summarised per population (bulk), since
  single-cell potency states and the Shannon index over single-cell potency distributions
  require per-cell resolution.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SCENT
  role: 'Core package: computes single-cell signalling entropy rates (logit-SR), fits
    the mixture-of-Gaussians potency-state model, and yields Shannon index of potency
    heterogeneity'
  repo: https://github.com/aet21/SCENT
- name: R
  role: Analysis environment in which the SCENT pipeline is executed and results exported
    to CSV
- name: BiocStyle
  role: Bioconductor styling package used in the SCENT tutorial/vignette accompanying
    the workflow
- name: CCAT
  role: Companion tool used alongside SCENT in the tutorial for estimating differentiation
    potency of single cells
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:25a48b29d4d32a0f5912dedfc7de52396ddc165e7f6437202f616995a4d3acc3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - BiocStyle
  - CCAT
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# hesc-npc-potency-population-comparison

## Summary

Reproduce the SCENT (single-cell entropy) potency-state analysis on the Chu et al. hESC/neural-progenitor scRNA-Seq dataset: fit a mixture-of-Gaussians model to single-cell signalling entropy rates, assign cells to discrete potency states, and quantify population-level differentiation heterogeneity with the Shannon index.

## When to use

Use when analysing snapshot scRNA-Seq populations (e.g. hESC vs progenitor/NPC) where differentiation potency must be inferred without marker genes or timepoint labels, and population-level potency heterogeneity (Shannon index) is the target output.

## When NOT to use

- When a reliable marker gene or timepoint label is available and potency ordering can be done directly — SCENT is intended for cases lacking prior biological knowledge such as marker expression or timepoint.
- When the research question is trajectory reconstruction rather than potency-state assignment and population heterogeneity — the paper notes this insight is not readily obtainable with Monocle or MPath, but those tools serve a different purpose.
- When the dataset is a feature table already summarised per population (bulk), since single-cell potency states and the Shannon index over single-cell potency distributions require per-cell resolution.

## Inputs

- Chu et al. hESC/progenitor (hESC and NPC) scRNA-Seq expression matrix (gene x cell counts)
- SCENT signalling interaction-network gene set used for gene mapping
- SCENT R package installed from aet21/SCENT

## Outputs

- Per-cell discrete potency-state assignments from the mixture-of-Gaussians fit on logit-SR values
- Low-potency fraction per population (hESC expected ~4%)
- Per-population Shannon index (SI) of potency heterogeneity
- scent_chu_results.csv with per-cell assignments, low-potency fractions, and Shannon indices

## How to apply

Load the Chu et al. expression matrix (hESC and NPC populations) in R with the SCENT package, mapping genes onto SCENT's signalling interaction-network dataset. Compute the single-cell signalling entropy rate for each cell and transform values to logit-SR scale, since SCENT operates on this transformed scale. Fit a mixture-of-Gaussians model to the logit-SR values to assign each cell to a discrete potency state; the correct model should be parsimonious (two states: high-potency pluripotent and lower-potency non-pluripotent progenitor-like). Estimate the fraction of hESC cells in the low-potency state (expected ~4%), reflecting cells primed for differentiation. Compute the Shannon index over the distribution of potency states per population and confirm the NPC population attains the highest SI, then write per-cell assignments, low-potency fractions, and Shannon indices to scent_chu_results.csv.

## Related tools

- **SCENT** (Core package: computes single-cell signalling entropy rates (logit-SR), fits the mixture-of-Gaussians potency-state model, and yields Shannon index of potency heterogeneity) — https://github.com/aet21/SCENT
- **R** (Analysis environment in which the SCENT pipeline is executed and results exported to CSV)
- **BiocStyle** (Bioconductor styling package used in the SCENT tutorial/vignette accompanying the workflow)
- **CCAT** (Companion tool used alongside SCENT in the tutorial for estimating differentiation potency of single cells)

## Examples

```
library(devtools); devtools::install_github("aet21/SCENT")
```

## Evaluation signals

- The mixture-of-Gaussians fit on logit-SR values recovers a parsimonious two-state model rather than a fragmented multi-state solution
- Approximately 4% of hESC cells are assigned to the low-potency state
- The NPC population attains the highest Shannon index across populations, reflecting its two distinct potency subsets
- The high-potency state corresponds to the pluripotent hESC population and the lower-potency state to non-pluripotent progenitor-like cells
- scent_chu_results.csv contains per-cell potency-state assignments, low-potency fractions, and per-population Shannon indices

## Limitations

- Signalling entropy results depend on mapping genes to an interaction network; the paper notes entropy-based measures that do not use network information (StemID, SLICE) are less accurate and robust.
- The precise relation between single-cell signalling entropy and transcriptional entropy (as defined by Richard et al.) is non-trivial and not established.
- Criticality hallmarks (e.g. an SI peak before a cell-fate transition) remain untested pending high-resolution time-course and genome-wide scRNA-Seq data.
- High dropout rates can preclude marker-based approaches, which is a motivation for SCENT, but network coverage of detected genes still constrains the entropy estimate.

## Evidence

- [discussion] SCENT correctly predicted that a hESC population contains a small fraction of cells of lower potency primed for differentiation: "using SCENT we were able to correctly predict that a hESC population contains a small fraction of cells of lower potency which are primed for differentiation"
- [discussion] SCENT inferred an NPC population consisted of two distinct subsets, with only the lower potency subset representing bona fide NPCs: "SCENT inferred that an assayed NPC population was made up two distinct subsets, correctly predicting that only the lower potency subset represents bona fide NPCs"
- [discussion] assign single cells into potency states, then estimate a SI over the whole cell population: "we can assign single cells into potency states, from which a SI over the whole cell population (that is, using the distribution of potency states over single cells) can then be estimated"
- [readme] The main purpose of the SCENT package is to estimate differentiation potency without prior biological knowledge: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
- [readme] SCENT installation from the aet21/SCENT GitHub repository: "devtools::install_github("aet21/SCENT")"
