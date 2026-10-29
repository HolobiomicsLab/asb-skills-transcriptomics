---
name: ppi-network-integration-with-transcriptome
description: Use when the goal is to estimate or rank differentiation potency of single
  cells from scRNA-Seq profiles (e.g. discriminating pluripotent hESCs from progenitors)
  where marker expression or timepoint labels are unavailable or unreliable due to
  dropouts.
when_to_use_negative:
- When a reliable marker gene or timepoint label is available for ordering potency
  — SCENT is intended for scenarios where such prior biological knowledge is absent
  or dropout precludes marker use
- When pseudo-time ordering via algorithms like Monocle or MPath is the specific goal
  — the article notes network-entropy-based potency ordering is not readily obtainable
  from those algorithms, and conversely they serve different purposes
- When only bulk transcriptomes without any network context are available and a purely
  transcriptomic entropy measure is desired — SCENT requires PPI integration and network-based
  entropies behave differently from StemID/SLICE-style transcriptomic entropies
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0602
tools:
- name: SCENT
  role: R/Bioconductor package that computes normalized signalling entropy rate (SR)
    per cell by integrating the expression matrix with the PPI network; installed
    via devtools::install_github("aet21/SCENT")
  repo: https://github.com/aet21/SCENT
- name: R
  role: Runtime environment for loading the expression matrix and PPI network and
    executing the SCENT computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:347a3858b60b9f779b3f8fb21306299fc2ca17793d530de58a8634dc1f0c3f6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ppi-network-integration-with-transcriptome

## Summary

Compute signalling entropy (SCENT) by integrating a single-cell RNA-Seq expression matrix with a protein-protein interaction (connectome) network to estimate differentiation potency per cell, without feature selection or prior biological knowledge such as marker genes or timepoints.

## When to use

Use when the goal is to estimate or rank differentiation potency of single cells from scRNA-Seq profiles (e.g. discriminating pluripotent hESCs from progenitors) where marker expression or timepoint labels are unavailable or unreliable due to dropouts.

## When NOT to use

- When a reliable marker gene or timepoint label is available for ordering potency — SCENT is intended for scenarios where such prior biological knowledge is absent or dropout precludes marker use
- When pseudo-time ordering via algorithms like Monocle or MPath is the specific goal — the article notes network-entropy-based potency ordering is not readily obtainable from those algorithms, and conversely they serve different purposes
- When only bulk transcriptomes without any network context are available and a purely transcriptomic entropy measure is desired — SCENT requires PPI integration and network-based entropies behave differently from StemID/SLICE-style transcriptomic entropies

## Inputs

- Single-cell RNA-Seq expression matrix (genes x cells, e.g. 1,018 Chu et al. hESC/progenitor cells)
- Human protein-protein interaction (connectome) network

## Outputs

- Per-cell normalized signalling entropy rate (SR) values
- CSV of per-cell SR values and class labels (e.g. chu_scent_sr.csv)
- Optional ROC AUC / Spearman correlation statistics against potency class labels or scores

## How to apply

Load the scRNA-Seq expression matrix (e.g. the Chu et al. set of 1,018 cells) together with a human PPI (connectome) network in R. Run the SCENT package to compute the normalized signalling entropy rate (SR) for each cell by integrating expression with the PPI network, explicitly without feature selection or prior biological knowledge. Optionally correlate per-cell SR values with an external pluripotency score (Spearman correlation of 0.91 was reported on Chu et al.) and compute a ROC AUC for class discrimination (AUC = 0.96 for pluripotent vs non-pluripotent progenitors, Wilcoxon P<1e-300). Write per-cell SR values and class labels to an output file such as chu_scent_sr.csv. The rationale is that signalling promiscuity of a cell's transcriptome in the context of an interaction network is higher in less differentiated cells, and the subtle positive correlation between transcriptome and connectome makes this measure more accurate and robust than purely transcriptomic entropies (e.g. StemID, SLICE).

## Related tools

- **SCENT** (R/Bioconductor package that computes normalized signalling entropy rate (SR) per cell by integrating the expression matrix with the PPI network; installed via devtools::install_github("aet21/SCENT")) — https://github.com/aet21/SCENT
- **R** (Runtime environment for loading the expression matrix and PPI network and executing the SCENT computation)

## Examples

```
library(devtools); devtools::install_github("aet21/SCENT"); library(SCENT)
```

## Evaluation signals

- Per-cell SR values reproduce the reported discrimination of pluripotent hESCs vs non-pluripotent progenitors with ROC AUC = 0.96 (Wilcoxon test P<1e-300)
- Spearman correlation between per-cell SR and an independent pluripotency score reaches 0.91 on the Chu et al. data
- Output file (e.g. chu_scent_sr.csv) contains one SR value plus class label per cell (1,018 cells for Chu et al.) with no missing entries
- SR computation runs without feature selection or any prior biological knowledge (no marker genes, no timepoint labels) as required by the SCENT design
- Pluripotent cells should exhibit the higher signalling entropy relative to multipotent and differentiated cells, consistent with the published potency ordering

## Limitations

- The precise relation between single-cell signalling entropy and transcriptional entropy (as defined by Richard et al.) is non-trivial and not established
- SCENT does not use transcriptomic entropy alone — unlike StemID and SLICE, it depends on PPI network information, so results differ from purely transcriptomic entropy measures
- Criticality hallmarks (e.g. an SI peak before a cell-fate transition) remain untested pending high-resolution time-course and genome-wide scRNA-Seq data
- No existing assay can measure all single cells in a population, limiting validation of bulk-vs-single-cell synergy

## Evidence

- [abstract] potency approximation via network entropy: "differentiation potency of a single cell can be approximated by computing the signalling promiscuity, or entropy, of a cell's transcriptome in the context of an interaction network, without the need"
- [abstract] accuracy vs other entropy measures: [evidence span withheld — anchor status: not_found]
- [intro] SCENT purpose / no prior knowledge: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
- [discussion] discrimination without training: "signalling entropy can accurately discriminate pluripotent from multipotent and differentiated cells, without the need for feature selection or training"
- [readme] installation command: "devtools::install_github("aet21/SCENT")"
