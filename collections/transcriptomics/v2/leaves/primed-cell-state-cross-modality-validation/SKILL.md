---
name: primed-cell-state-cross-modality-validation
description: Use when you have DICE-predicted primed cells and regulator TFs from
  scRNA-Seq plus a matched scATAC-Seq dataset for the same cell types, and need epigenomic
  confirmation that primed cells show higher accessibility/motif activity of the predicted
  TFs' targets.
when_to_use_negative:
- No matched scATAC-Seq data exists for the same cell types (e.g. no SHARE-Seq or
  matched epigenomic dataset) — cross-modality validation is impossible and only the
  scRNA-Seq evidence stands
- The predicted priming regulator is not a transcription factor (e.g. UHRF1), since
  chromVAR motif activity cannot be computed for non-TF chromatin modifiers
- You need experimental/functional proof that the regulatory factors drive priming
  — this skill only provides computational epigenomic corroboration, not perturbation
  evidence
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2229
tools:
- name: DICE
  role: Supplies FindPrimedCells primed-GP/primed-MEMP assignments, Wilcoxon marker-gene
    lists, and the ranzoniATAC data objects used for validation
  repo: https://github.com/aet21/DICE
- name: chromVAR
  role: Provides the TF motif activity (TFA) matrix per single ATAC cell that is summed
    across lineage regulators and compared between primed groups
  repo: https://github.com/GreenleafLab/chromVAR
- name: Seurat
  role: Used to log-normalize the gene-level accessibility count matrix built from
    scATAC-Seq peaks 3kb upstream of marker genes plus gene body
- name: ggpubr/ggplot2
  role: Builds the boxplots with geom_signif one-sided Wilcoxon rank-sum annotations
    saved as tfa_validation_boxplots.png
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:f873092e3faa06dbc221573f8bf573d844bd98cdd549fbeb469211e17468497f
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
  - DICE
  - R
  - Seurat
  - chromVAR
  - ggplot2
  - ggpubr/ggplot2
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# primed-cell-state-cross-modality-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates scRNA-Seq-predicted primed cell states (e.g. GP and MEMP primed MPPs from DICE) in matched scATAC-Seq data by showing that chromVAR transcription factor activity of the predicted regulators (GATA1, KLF1 for MEMP; IRF8 for GP) is elevated in scATAC cells assigned as primed via summed marker-gene accessibility.

## When to use

Use when you have DICE-predicted primed cells and regulator TFs from scRNA-Seq plus a matched scATAC-Seq dataset for the same cell types, and need epigenomic confirmation that primed cells show higher accessibility/motif activity of the predicted TFs' targets.

## When NOT to use

- No matched scATAC-Seq data exists for the same cell types (e.g. no SHARE-Seq or matched epigenomic dataset) — cross-modality validation is impossible and only the scRNA-Seq evidence stands
- The predicted priming regulator is not a transcription factor (e.g. UHRF1), since chromVAR motif activity cannot be computed for non-TF chromatin modifiers
- You need experimental/functional proof that the regulatory factors drive priming — this skill only provides computational epigenomic corroboration, not perturbation evidence

## Inputs

- Precomputed scATAC-Seq gene-level accessibility matrix (e.g. ranzoni.gene.access.m from data(ranzoniATAC)), log-normalized with Seurat from peaks 3kb upstream of each marker gene plus gene body
- scRNA-Seq-derived primed-GP and primed-MEMP MPP cell assignments from DICE FindPrimedCells and their Wilcoxon-derived marker-gene lists (primedCellMarker.l)
- Precomputed chromVAR TF activity matrix for MPPs (e.g. ranzoni.TFA.m)

## Outputs

- Per-cell summed TF activity scores for each lineage's regulators (GATA1+KLF1 for MEMP; IRF8 for GP)
- ATAC-side primed-cell assignments (top-10% accessibility cells, mutually exclusive between lineages)
- Boxplot figure (e.g. tfa_validation_boxplots.png) with one-sided Wilcoxon P-values comparing primed vs primed and primed vs unprimed groups

## How to apply

Load the matched scATAC-Seq objects (e.g. data(ranzoniATAC): ranzoni.gene.access.m, primedCellMarker.l, ranzoni.TFA.m). Transfer the scRNA-Seq primed-cell assignments (from FindPrimedCells, e.g. GP pair UHRF1-IRF8 and MEMP pair GATA1-KLF1) by intersecting the Wilcoxon-derived primed-cell marker genes with the rownames of the gene-level accessibility matrix, summing accessibility per cell, and calling the top-10% (quantile 0.9) of cells as ATAC-side primed for each lineage. Exclude cells primed for both lineages using intersect/setdiff so each cell has a single assignment. Then compute summed chromVAR TFA per cell — GATA1 plus KLF1 for MEMP, IRF8 alone for GP (UHRF1 is not a TF and has no motif activity) — and compare groups with one-sided Wilcoxon rank-sum tests (alternative='greater') visualized as ggpubr geom_signif boxplots. The rationale is that if the scRNA-Seq primed states are real, their regulatory factors should already show elevated target accessibility/motif activity in the matching epigenomic modality.

## Related tools

- **DICE** (Supplies FindPrimedCells primed-GP/primed-MEMP assignments, Wilcoxon marker-gene lists, and the ranzoniATAC data objects used for validation) — https://github.com/aet21/DICE
- **chromVAR** (Provides the TF motif activity (TFA) matrix per single ATAC cell that is summed across lineage regulators and compared between primed groups) — https://github.com/GreenleafLab/chromVAR
- **Seurat** (Used to log-normalize the gene-level accessibility count matrix built from scATAC-Seq peaks 3kb upstream of marker genes plus gene body)
- **ggpubr/ggplot2** (Builds the boxplots with geom_signif one-sided Wilcoxon rank-sum annotations saved as tfa_validation_boxplots.png)

## Examples

```
library(DICE); data(ranzoniATAC); primedMEMP.tfa.v <- colSums(ranzoni.TFA.m[c('GATA1','KLF1'), atac.MEMP.prime.cell.v]); wilcox.test(primedMEMP.tfa.v, unprimed.tfa.v, alternative='greater')
```

## Evaluation signals

- Summed GATA1/KLF1 TFA is significantly higher in primedMEMP cells (one-sided Wilcoxon P < 0.05) versus both primedGP and unprimed cells
- IRF8 TFA is significantly higher in primedGP cells (one-sided Wilcoxon P < 0.05) versus the other groups, as reported in the vignette
- Primed-cell exclusivity invariant: intersect(atac.MEMP.prime.cell.v, atac.GP.prime.cell.v) is empty after setdiff-based overlap removal
- ATAC-side primed sets are exactly the top-10% of cells (quantile 0.9 cutoff) by summed marker-gene accessibility intersected with rownames of the accessibility matrix
- UHRF1 is excluded from TFA computation because it has no chromVAR motif activity

## Limitations

- The ATAC-side primed-state assignment is an invented, approximate method (top-10% marker accessibility) rather than a directly measured state, since matched SHARE-Seq joint profiles were unavailable for this study
- No experimental work demonstrates the importance of the identified regulatory factors in priming; only computational scATAC-Seq validation was performed
- Numerical results depend on the quantile cutoff (0.9) and on chromVAR/Seurat versions, which are not pinned in the methods

## Evidence

- [results] chromVAR-derived TFA levels of GATA1 and KLF1 were higher in the cellular neighborhoods enriched for MEMP-primed cells, while the converse was true for TFA of IRF8 (higher in GP-primed neighborhoods).: [evidence span withheld — anchor status: not_found]
- [methods] # exclude overlapping primed cells
overlap.v <- intersect(atac.MEMP.prime.cell.v,atac.GP.prime.cell.v): "# exclude overlapping primed cells
overlap.v <- intersect(atac.MEMP.prime.cell.v,atac.GP.prime.cell.v)"
