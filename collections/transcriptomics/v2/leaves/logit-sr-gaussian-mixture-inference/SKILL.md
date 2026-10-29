---
name: logit-sr-gaussian-mixture-inference
description: Use when scRNA-Seq data must be stratified into discrete potency states
  using signalling entropy, e.g. to detect a low-potency subpopulation (like the ~4%
  hESC fraction) or compute a Shannon index of heterogeneity.
when_to_use_negative:
- Cells should be ordered by known marker expression or timepoint — SCENT's purpose
  is potency estimation without assuming prior biological knowledge such as marker
  expression or timepoint
- Pseudo-time ordering by alternative trajectory algorithms (Monocle, MPath, StemID,
  SLICE) is already sufficient for the question
- Transcriptomic entropy measures without network information are desired — signalling
  entropy is specifically a network-based measure
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCENT
  role: 'Core R package: computes single-cell signalling entropy rate, maps genes
    to the interaction network, fits the mixture-of-Gaussians potency-state model,
    and derives Shannon indices'
  repo: aet21/SCENT
- name: R
  role: Runtime environment in which the SCENT tutorial and potency-state inference
    workflow are executed
- name: BiocStyle
  role: Bioconductor styling used by the SCENT tutorial vignette (require(BiocStyle))
- name: CCAT
  role: Companion tool from the SCENT tutorial for estimating differentiation potency
    of single cells
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:27116ab7ad5c321e9fb9c888e4a46fcbbd71f838162452e6649771f8166c1b3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - BiocStyle
  - CCAT
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# logit-sr-gaussian-mixture-inference

## Summary

Infers discrete differentiation-potency states by computing single-cell signalling entropy rate (SR) from an interaction network, transforming values to logit-SR scale, and fitting a mixture-of-Gaussians model to assign each cell to a potency state without prior marker or timepoint knowledge.

## When to use

Use when scRNA-Seq data must be stratified into discrete potency states using signalling entropy, e.g. to detect a low-potency subpopulation (like the ~4% hESC fraction) or compute a Shannon index of heterogeneity.

## When NOT to use

- Cells should be ordered by known marker expression or timepoint — SCENT's purpose is potency estimation without assuming prior biological knowledge such as marker expression or timepoint
- Pseudo-time ordering by alternative trajectory algorithms (Monocle, MPath, StemID, SLICE) is already sufficient for the question
- Transcriptomic entropy measures without network information are desired — signalling entropy is specifically a network-based measure

## Inputs

- scRNA-Seq expression matrix (gene x cell counts, e.g. Chu et al. hESC and NPC populations)
- SCENT signalling interaction-network dataset for gene mapping
- R environment with SCENT, devtools, BiocStyle

## Outputs

- Per-cell discrete potency-state assignments from the mixture-of-Gaussians fit
- Logit-SR values per cell
- Fraction of cells in the low-potency state per population (expected ~4% for hESC)
- Per-population Shannon index of potency heterogeneity
- Results table (e.g. scent_chu_results.csv)

## How to apply

Load the scRNA-seq expression matrix (e.g. the Chu et al. hESC/NPC dataset) in R with the SCENT package, mapping genes onto the SCENT signalling interaction network. Compute the single-cell signalling entropy rate for each cell — signalling promiscuity of the transcriptome in the context of an interaction network — and transform values to logit-SR scale. Fit a mixture-of-Gaussians model to the logit-SR values using SCENT's potency-state inference, preferring the parsimonious solution (the expected answer on Chu et al. is a two-state model: a high-potency pluripotent state and a lower-potency non-pluripotent progenitor-like state). Assign each cell to a discrete potency state, then estimate the Shannon index (SI) over the whole population from the distribution of potency states to quantify heterogeneity. Validate by checking that the fitted number of states is minimal and biologically coherent — e.g. roughly 4% of hESC cells should fall in the low-potency state, and the NPC population should attain the highest SI. Write per-cell state assignments, low-potency fractions, and per-population Shannon indices to an output table (e.g. scent_chu_results.csv).

## Related tools

- **SCENT** (Core R package: computes single-cell signalling entropy rate, maps genes to the interaction network, fits the mixture-of-Gaussians potency-state model, and derives Shannon indices) — aet21/SCENT
- **R** (Runtime environment in which the SCENT tutorial and potency-state inference workflow are executed)
- **BiocStyle** (Bioconductor styling used by the SCENT tutorial vignette (require(BiocStyle)))
- **CCAT** (Companion tool from the SCENT tutorial for estimating differentiation potency of single cells)

## Examples

```
library(devtools); devtools::install_github("aet21/SCENT"); library(SCENT)  # then compute logit-SR per cell and fit the mixture-of-Gaussians potency-state model on the Chu et al. matrix
```

## Evaluation signals

- Mixture-of-Gaussians fit recovers a parsimonious two-state solution on the Chu et al. data (high-potency pluripotent state plus lower-potency progenitor-like state)
- Approximately 4% of hESC cells are assigned to the low-potency state, consistent with SCENT's prediction of a small fraction of cells primed for differentiation
- NPC population attains the highest Shannon index of potency heterogeneity across populations
- Per-cell potency-state assignments, low-potency fractions, and per-population Shannon indices are all present and complete in scent_chu_results.csv
- Signalling entropy correctly discriminates pluripotent from multipotent and differentiated cells without feature selection or training

## Limitations

- Signalling entropy does not use transcriptomic entropy measures without network information (as in StemID or SLICE), so results depend on the quality and coverage of the signalling interaction network
- Pseudo-time ordering via signalling entropy is a distinct but related use; the relation between signalling entropy and transcriptional entropy (as defined by Richard et al.) is non-trivial and not established
- SCENT's insight (e.g. identifying subsets of varying potency) is not readily obtainable using other algorithms such as Monocle or MPath, so it cannot be swapped in directly

## Evidence

- [intro] The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
- [readme] mixture model on logit-SR: "A brief tutorial for estimating differentiation potency of single cells using SCENT and CCAT"
- [discussion] potency states to SI: "we can assign single cells into potency states, from which a SI over the whole cell population (that is, using the distribution of potency states over single cells) can then be estimated"
- [discussion] hESC low-potency fraction: "using SCENT we were able to correctly predict that a hESC population contains a small fraction of cells of lower potency which are primed for differentiation"
- [discussion] discrimination without feature selection: "signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training"
- [abstract] network-based entropy abstract: "differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network, without the need"
