---
name: protein-interaction-network-based-potency-scoring
description: Use when scoring snapshot scRNA-Seq cells for differentiation potency
  or identifying cancer stem-cell-like phenotypes (e.g. cancer vs non-malignant cells
  in Tirosh melanoma) without prior marker knowledge or timepoints.
when_to_use_negative:
- When the input is already a feature table, cluster assignment, or pseudo-time trajectory
  rather than a raw gene x cell count matrix
- When marker genes or experimental timepoints are reliably available and the question
  is ordering cells along a known differentiation axis (SCENT's value is not requiring
  such priors)
- When the goal is transcriptome-entropy-only analysis without network context — the
  article argues network-based signalling entropy outperforms StemID/SLICE-style measures
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCENT
  role: Core R package providing the SignallingEntropy routine to compute per-cell
    SR scores against a protein interaction network
  repo: https://github.com/aet21/SCENT
- name: CCAT
  role: Companion tool in the SCENT tutorial for estimating differentiation potency
    of single cells
  repo: https://github.com/aet21/SCENT
- name: R
  role: Runtime environment for running SCENT and summarizing SR scores per cell type
- name: BiocStyle
  role: Bioconductor styling used in the SCENT tutorial vignette
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:a674d1876f90f715eaaa3b88fd731811066ed3182fe1bfc9fd10f3242cab77e3
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# protein-interaction-network-based-potency-scoring

## Summary

Estimates single-cell differentiation potency by computing signalling promiscuity (entropy) of each cell's transcriptome in the context of a protein interaction network, using the SCENT package, without feature selection or training. It captures a subtle positive correlation between transcriptome and connectome that entropy-only methods (StemID, SLICE) miss.

## When to use

Use when scoring snapshot scRNA-Seq cells for differentiation potency or identifying cancer stem-cell-like phenotypes (e.g. cancer vs non-malignant cells in Tirosh melanoma) without prior marker knowledge or timepoints.

## When NOT to use

- When the input is already a feature table, cluster assignment, or pseudo-time trajectory rather than a raw gene x cell count matrix
- When marker genes or experimental timepoints are reliably available and the question is ordering cells along a known differentiation axis (SCENT's value is not requiring such priors)
- When the goal is transcriptome-entropy-only analysis without network context — the article argues network-based signalling entropy outperforms StemID/SLICE-style measures

## Inputs

- scRNA-Seq count matrix (genes x cells) with malignant/non-malignant and cell-type annotations
- Protein interaction network bundled with SCENT

## Outputs

- Per-cell SR (signalling entropy) scores for every cell
- Per-cell-type summary table of mean SR values
- Wilcoxon rank-sum contrast statistics (e.g. cancer vs non-malignant P<1e-500)

## How to apply

Load the scRNA-Seq count matrix with cell-type annotations into R (e.g. 3,256 non-malignant and 1,257 cancer cells from 12 melanoma patients) and install SCENT from GitHub. Run the SCENT SignallingEntropy routine on the expression matrix against a bundled protein interaction network to compute an SR (signalling entropy) score per cell, deliberately avoiding feature selection or training since the method is designed to work without biological priors. Summarize mean SR per cell type and contrast cancer cells against each non-malignant population (ECs, CAFs, T cells, B cells, macrophages, NK cells) using a Wilcoxon rank-sum test. Rank non-malignant cell types by mean SR to verify a potency-graded ordering with high-potency stromal types (ECs, CAFs) at the top, then export per-cell scores (e.g. tirosh_sr_scores.csv).

## Related tools

- **SCENT** (Core R package providing the SignallingEntropy routine to compute per-cell SR scores against a protein interaction network) — https://github.com/aet21/SCENT
- **CCAT** (Companion tool in the SCENT tutorial for estimating differentiation potency of single cells) — https://github.com/aet21/SCENT
- **R** (Runtime environment for running SCENT and summarizing SR scores per cell type)
- **BiocStyle** (Bioconductor styling used in the SCENT tutorial vignette)

## Examples

```
library(devtools); devtools::install_github("aet21/SCENT")
```

## Evaluation signals

- Cancer cells show a dramatic and highly significant signalling entropy increase over non-malignant cells (Wilcoxon rank-sum test P<1e-500)
- Non-malignant cell types display a potency-graded mean SR ordering with ECs and CAFs at the top
- Per-cell SR scores cover all cells in the input matrix (e.g. 4,513 cells) with no feature selection or training step applied
- Output CSV (e.g. tirosh_sr_scores.csv) contains one SR score per cell plus a per-cell-type summary table
- Method discriminates pluripotent from multipotent/differentiated cells without prior biological knowledge, as reported in the source article

## Limitations

- The precise relation between single-cell signalling entropy and transcriptional entropy (as defined by Richard et al.) is non-trivial and not established
- No existing assay can measure all single cells in a population, limiting validation of bulk-vs-single-cell synergy
- Criticality hallmarks (e.g. a signalling-entropy peak before a cell-fate transition) remain untested pending high-resolution time-course and genome-wide scRNA-Seq data
- High dropout rates may preclude marker-based approaches, which is a scenario motivating SCENT rather than a failure mode of the method itself

## Evidence

- [abstract] differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network: "differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network"
- [intro] The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
- [methods] A brief tutorial for estimating differentiation potency of single cells using SCENT and CCAT: "A brief tutorial for estimating differentiation potency of single cells using SCENT and CCAT"
- [discussion] other transcriptomic entropy-based measures (those used in StemID and SLICE) which do not use network information: "other transcriptomic entropy-based measures (those used in StemID and SLICE) which do not use network information"
- [readme] To install:
library(devtools)
devtools::install_github("aet21/SCENT"): "devtools::install_github("aet21/SCENT")"
