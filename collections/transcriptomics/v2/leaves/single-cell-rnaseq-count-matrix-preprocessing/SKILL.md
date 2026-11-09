---
name: single-cell-rnaseq-count-matrix-preprocessing
description: Use when raw or processed scRNA-Seq expression matrices (e.g. hESC/progenitor
  populations like Chu et al.) must be prepared for network-based potency estimation
  with SCENT, requiring gene mapping to a signalling interaction network before computing
  per-cell signalling entropy.
when_to_use_negative:
- The analysis goal is trajectory or pseudo-time construction with tools like Monocle,
  MPath, StemID, or SLICE rather than network-based potency estimation — the article
  notes SCENT's insight is 'not readily obtainable using other algorithms such as
  Monocle or MPath'
- No suitable gene mapping exists between the expression matrix's gene identifiers
  and SCENT's signalling interaction-network dataset, since signalling entropy is
  defined over the interaction network
- Prior biological knowledge such as marker expression or timepoint labels is available
  and preferred — although SCENT is specifically designed for settings where such
  knowledge is absent, e.g. high-dropout data or snapshot datasets
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SCENT
  role: R package used to map genes onto the signalling interaction network, compute
    single-cell signalling entropy rates, transform them to logit-SR, and fit a mixture-of-Gaussians
    model for discrete potency-state assignment
  repo: https://github.com/aet21/SCENT
- name: R
  role: Runtime environment in which the SCENT package is installed and the Chu et
    al. expression matrix is loaded and processed
- name: BiocStyle
  role: Bioconductor styling package used in the SCENT package vignette/tutorial (BiocStyle::html_document
    output)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:068b9362683773c5311104d3a7c7a104b250f4245fd7dc5ed68f22b2c61652cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - BiocStyle
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-rnaseq-count-matrix-preprocessing

## Summary

Prepare a single-cell RNA-Seq expression (count) matrix so it can be consumed by the SCENT signalling-entropy pipeline: load the matrix (e.g. Chu et al. hESC/NPC data) in R, map genes onto the SCENT signalling interaction-network dataset, and derive per-cell logit signalling-entropy-rate (logit-SR) values for downstream potency-state inference.

## When to use

Use when raw or processed scRNA-Seq expression matrices (e.g. hESC/progenitor populations like Chu et al.) must be prepared for network-based potency estimation with SCENT, requiring gene mapping to a signalling interaction network before computing per-cell signalling entropy.

## When NOT to use

- The analysis goal is trajectory or pseudo-time construction with tools like Monocle, MPath, StemID, or SLICE rather than network-based potency estimation — the article notes SCENT's insight is 'not readily obtainable using other algorithms such as Monocle or MPath'
- No suitable gene mapping exists between the expression matrix's gene identifiers and SCENT's signalling interaction-network dataset, since signalling entropy is defined over the interaction network
- Prior biological knowledge such as marker expression or timepoint labels is available and preferred — although SCENT is specifically designed for settings where such knowledge is absent, e.g. high-dropout data or snapshot datasets

## Inputs

- scRNA-seq expression matrix (count or normalized expression) with genes as rows and single cells as columns, e.g. the Chu et al. hESC/NPC dataset
- SCENT signalling interaction-network dataset used for gene mapping
- Sample/population labels (e.g. hESC vs NPC) for per-population heterogeneity summaries

## Outputs

- Per-cell signalling entropy rates on the logit-SR scale
- Per-cell discrete potency-state assignments from the mixture-of-Gaussians fit
- Per-population low-potency cell fractions (e.g. ~4% of hESC cells expected in the low-potency state)
- Per-population Shannon index (SI) of potency heterogeneity, e.g. written to scent_chu_results.csv

## How to apply

Load the scRNA-seq expression matrix (e.g. the Chu et al. hESC and NPC populations) in R with the SCENT package, and map the genes onto SCENT's signalling interaction-network dataset, since signalling entropy is defined over a protein interaction network rather than the raw transcriptome. Compute the single-cell signalling entropy rate for every cell, then transform the values to the logit-SR scale; this normalization is what makes potency values comparable across cells without feature selection or training data. Fit a mixture-of-Gaussians model to the logit-SR values using SCENT's potency-state inference to assign each cell to a discrete potency state — verify the fit selects a parsimonious solution (e.g. a two-state model with a high-potency pluripotent state and a lower-potency progenitor-like state). Finally, derive per-population summaries such as low-potency cell fractions and the Shannon index (SI) of potency heterogeneity computed from the distribution of potency states over single cells.

## Related tools

- **SCENT** (R package used to map genes onto the signalling interaction network, compute single-cell signalling entropy rates, transform them to logit-SR, and fit a mixture-of-Gaussians model for discrete potency-state assignment) — https://github.com/aet21/SCENT
- **R** (Runtime environment in which the SCENT package is installed and the Chu et al. expression matrix is loaded and processed)
- **BiocStyle** (Bioconductor styling package used in the SCENT package vignette/tutorial (BiocStyle::html_document output))

## Examples

```
library(devtools); devtools::install_github("aet21/SCENT")
```

## Evaluation signals

- The mixture-of-Gaussians fit recovers a parsimonious two-state model on the Chu et al. data: a high-potency pluripotent state and a lower-potency non-pluripotent progenitor-like state
- The estimated fraction of hESC cells assigned to the low-potency state is small, around ~4%, consistent with cells primed for differentiation
- The NPC population attains the highest Shannon index (SI) of potency heterogeneity among the assayed populations, reflecting its two distinct potency-state subsets
- All genes used in the entropy computation map successfully onto the SCENT signalling interaction-network dataset (network mapping step completes without excessive gene loss)
- Per-cell logit-SR values are computed for every cell and potency-state assignments, low-potency fractions, and per-population SIs are fully populated in the output file (e.g. scent_chu_results.csv)

## Limitations

- The precise relation between single-cell signalling entropy and transcriptional entropy (as defined by Richard et al.) is non-trivial and not established
- No existing assay can measure all single cells in a population, limiting validation of bulk-vs-single-cell synergy
- Criticality hallmarks (e.g. an SI peak before a cell-fate transition) remain untested, pending upcoming high-resolution time-course and genome-wide scRNA-Seq data

## Evidence

- [other] Load the Chu et al. scRNA-seq expression matrix (hESC and NPC populations) in R with the SCENT package, mapping genes onto the SCENT signalling interaction-network dataset.: [evidence span withheld — anchor status: not_found]
- [intro] The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
- [abstract] differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network, without the need: "differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network, without the need"
- [discussion] SCENT correctly predicted that a hESC population contains a small fraction of cells of lower potency which are primed for differentiation: "using SCENT we were able to correctly predict that a hESC population contains a small fraction of cells of lower potency which are primed for differentiation"
- [readme] devtools::install_github("aet21/SCENT"): "devtools::install_github("aet21/SCENT")"
