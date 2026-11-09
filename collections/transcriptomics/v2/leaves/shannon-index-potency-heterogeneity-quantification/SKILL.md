---
name: shannon-index-potency-heterogeneity-quantification
description: Use when scRNA-seq populations (e.g. hESC vs NPC) must be stratified
  into discrete potency states and their heterogeneity quantified as a Shannon index,
  without marker genes or timepoints.
when_to_use_negative:
- When the goal is pseudo-time trajectory ordering rather than potency-state stratification
  — algorithms such as Monocle or MPath do not readily provide the potency insight
  SCENT gives
- When data cannot be mapped onto SCENT's interaction-network (connectome) gene sets,
  or a purely transcriptomic entropy measure (StemID/SLICE-style) would suffice for
  the question
- When prior biological knowledge such as marker expression or timepoints is already
  trusted and a knowledge-driven potency assignment is acceptable
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCENT
  role: Computes per-cell signalling entropy rate, performs mixture-of-Gaussians potency-state
    inference, and provides the state distribution from which the Shannon index is
    estimated
  repo: https://github.com/aet21/SCENT
- name: R
  role: Execution environment for the SCENT workflow and CSV export of potency assignments
    and Shannon indices
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:825ebd6498d27bf6bde3a3e91bfa58659b4e77dad148b309637a2f665939b8cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - BiocStyle
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# shannon-index-potency-heterogeneity-quantification

## Summary

Quantifies potency heterogeneity within a scRNA-seq population by assigning single cells to discrete potency states with SCENT's mixture-of-Gaussians inference on logit-transformed signalling entropy rate (SR), then computing a Shannon index (SI) over the distribution of potency states. This separates pluripotent from primed-for-differentiation subsets without prior marker knowledge.

## When to use

Use when scRNA-seq populations (e.g. hESC vs NPC) must be stratified into discrete potency states and their heterogeneity quantified as a Shannon index, without marker genes or timepoints.

## When NOT to use

- When the goal is pseudo-time trajectory ordering rather than potency-state stratification — algorithms such as Monocle or MPath do not readily provide the potency insight SCENT gives
- When data cannot be mapped onto SCENT's interaction-network (connectome) gene sets, or a purely transcriptomic entropy measure (StemID/SLICE-style) would suffice for the question
- When prior biological knowledge such as marker expression or timepoints is already trusted and a knowledge-driven potency assignment is acceptable

## Inputs

- scRNA-seq gene-by-cell expression matrix (e.g. Chu et al. hESC/NPC counts)
- SCENT signalling interaction-network gene sets used to map input genes

## Outputs

- Per-cell potency-state assignments from the mixture-of-Gaussians fit on logit-SR
- Per-population low-potency cell fractions (e.g. ~4% for hESC)
- Per-population Shannon index (SI) of potency heterogeneity
- Results table (e.g. scent_chu_results.csv)

## How to apply

Load the scRNA-seq expression matrix (e.g. Chu et al. hESC and NPC populations) in R with the SCENT package, mapping genes onto SCENT's signalling interaction-network dataset. Compute the single-cell signalling entropy rate for each cell and transform values to logit-SR scale; this network-based entropy can discriminate pluripotent from multipotent and differentiated cells without feature selection or training. Fit a mixture-of-Gaussians model to the logit-SR values to assign each cell to a discrete potency state, preferring the most parsimonious solution (for Chu et al., a two-state model: high-potency pluripotent and lower-potency progenitor-like states). Verify expected fractions — e.g. ~4% of hESC cells assigned to the low-potency state primed for differentiation — and confirm the more heterogeneous population (NPC) attains the highest Shannon index. Assign single cells into potency states, from which the SI is estimated over the whole cell population using the distribution of potency states, and export per-cell state assignments, low-potency fractions, and per-population SI values (e.g. to a CSV).

## Related tools

- **SCENT** (Computes per-cell signalling entropy rate, performs mixture-of-Gaussians potency-state inference, and provides the state distribution from which the Shannon index is estimated) — https://github.com/aet21/SCENT
- **R** (Execution environment for the SCENT workflow and CSV export of potency assignments and Shannon indices)

## Examples

```
library(devtools); devtools::install_github("aet21/SCENT")
```

## Evaluation signals

- The mixture-of-Gaussians fit selects a parsimonious solution; on the Chu et al. set it should recover exactly two potency states (high-potency pluripotent and lower-potency progenitor-like)
- The fraction of hESC cells assigned to the low-potency state is small, on the order of ~4%
- The NPC population attains the highest Shannon index among compared populations, reflecting its two distinct subsets
- Per-cell potency-state assignments, low-potency fractions, and per-population SI values are coherently written to the results CSV (row counts match input cells)
- Logit-SR values are on a bounded logit scale and each cell carries exactly one discrete potency-state label

## Limitations

- The precise relation between single-cell signalling entropy and transcriptional entropy (as defined by Richard et al.) is non-trivial and not established
- No existing assay can measure all single cells in a population, limiting validation of bulk-vs-single-cell potency/plasticity estimates
- Criticality hallmarks (e.g. an SI peak before a cell-fate transition) remain untested, pending high-resolution time-course and genome-wide scRNA-seq data
- High dropout rates may preclude marker-based checks; the method is intended for scenarios where marker expression is unreliable or hierarchies are not well-established

## Evidence

- [abstract] potency from network entropy: "differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network"
- [discussion] SI estimation from state distribution: "we can assign single cells into potency states, from which a SI over the whole cell population (that is, using the distribution of potency states over single cells) can then be estimated"
- [discussion] hESC low-potency fraction recovered: "using SCENT we were able to correctly predict that a hESC population contains a small fraction of cells of lower potency which are primed for differentiation"
- [discussion] NPC two-subset inference: "SCENT inferred that an assayed NPC population was made up two distinct subsets, correctly predicting that only the lower potency subset represents bona fide NPCs"
- [readme] SCENT purpose without prior knowledge: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
