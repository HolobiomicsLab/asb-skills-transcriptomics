---
name: single-cell-multiomic-data-integration
description: Use when a scRNA-Seq analysis has predicted primed cells and candidate
  regulatory factors (e.g. GATA1, KLF1, IRF8 in fetal liver MPPs) and matched scATAC-Seq
  data from the same cell types is available for epigenomic validation.
when_to_use_negative:
- No matched scATAC-Seq data exists for the cell types under study (e.g. the Ranzoni
  study lacked SHARE-Seq joint profiles, forcing this indirect validation)
- The candidate regulator is not a transcription factor with a motif (e.g. UHRF1),
  so no chromVAR motif activity can be computed for it
- The scRNA-Seq result is a cluster/trajectory annotation without defined marker genes,
  since the ATAC-side cell assignment depends on summing accessibility over lineage
  marker genes
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3169
tools:
- name: DICE
  role: R package that identifies primed cells and candidate regulatory factors from
    scRNA-Seq (FindPrimedCells), whose outputs are transferred to the ATAC data
  repo: github.com/aet21/DICE
- name: chromVAR
  role: Computes per-cell TF motif activity from the sparse scATAC-Seq accessibility
    data for validation of TF regulators
  repo: GreenleafLab/chromVAR
- name: Seurat
  role: Log-normalizes the gene-level accessibility count matrix built from ATAC peaks
- name: ggplot2/ggpubr
  role: Boxplot visualization with geom_signif one-sided Wilcoxon rank-sum comparisons
- name: R
  role: Implementation environment for the DICE/chromVAR validation workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:9818fc516e4465226a5a73661164833c1150a4decfdb239a0ab6bf99a4518df1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aet21/DICE
  tools:
  - DICE
  - R
  - Seurat
  - chromVAR
  - ggplot2
  - ggplot2/ggpubr
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# single-cell-multiomic-data-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates scRNA-Seq-derived primed states and their regulatory factors by projecting them onto matched scATAC-Seq data: transfer RNA-derived cell assignments and marker genes to a gene-level accessibility matrix, then test chromVAR TF motif activity differences between primed cell groups.

## When to use

Use when a scRNA-Seq analysis has predicted primed cells and candidate regulatory factors (e.g. GATA1, KLF1, IRF8 in fetal liver MPPs) and matched scATAC-Seq data from the same cell types is available for epigenomic validation.

## When NOT to use

- No matched scATAC-Seq data exists for the cell types under study (e.g. the Ranzoni study lacked SHARE-Seq joint profiles, forcing this indirect validation)
- The candidate regulator is not a transcription factor with a motif (e.g. UHRF1), so no chromVAR motif activity can be computed for it
- The scRNA-Seq result is a cluster/trajectory annotation without defined marker genes, since the ATAC-side cell assignment depends on summing accessibility over lineage marker genes

## Inputs

- Gene-level scATAC-Seq accessibility matrix (peaks aggregated 3kb upstream of marker genes plus gene body, log-normalized with Seurat), e.g. ranzoni.gene.access.m
- scRNA-Seq-derived primed-cell assignments per lineage (from FindPrimedCells) and their Wilcoxon marker-gene lists (primedCellMarker.l)
- Precomputed chromVAR TF activity matrix for MPPs, e.g. ranzoni.TFA.m

## Outputs

- ATAC-side primed cell sets (primedMEMP, primedGP, unprimed) with mutually exclusive assignments
- Boxplots of summed TF activity per group with one-sided Wilcoxon significance bars (tfa_validation_boxplots.png)
- Per-cell summed TF activity values for lineage-specific TF sets

## How to apply

Load the matched scATAC-Seq objects (here data(ranzoniATAC): ranzoni.gene.access.m, primedCellMarker.l, ranzoni.TFA.m). Transfer the scRNA-Seq-derived primed-cell assignments (FindPrimedCells pairs: GP = UHRF1-IRF8, MEMP = GATA1-KLF1) and their Wilcoxon marker genes to the ATAC side by intersecting each marker list with rownames of the gene-level accessibility matrix, summing accessibility per cell, and calling the top-10% (quantile 0.9) cells as ATAC-side primed; exclude cells primed for both lineages with intersect/setdiff so each cell has a single assignment. Then test whether chromVAR TF activity is enriched as expected: sum GATA1 and KLF1 TFA per cell for the MEMP lineage, use IRF8 TFA alone for GP (UHRF1 is not a TF and has no motif activity), and compare groups with one-sided Wilcoxon rank-sum tests (alternative='greater') in ggpubr::geom_signif boxplots. Rationale: priming detected at the transcriptomic level should be reflected in higher chromatin accessibility of the regulators' targets in the corresponding primed cells.

## Related tools

- **DICE** (R package that identifies primed cells and candidate regulatory factors from scRNA-Seq (FindPrimedCells), whose outputs are transferred to the ATAC data) — github.com/aet21/DICE
- **chromVAR** (Computes per-cell TF motif activity from the sparse scATAC-Seq accessibility data for validation of TF regulators) — GreenleafLab/chromVAR
- **Seurat** (Log-normalizes the gene-level accessibility count matrix built from ATAC peaks)
- **ggplot2/ggpubr** (Boxplot visualization with geom_signif one-sided Wilcoxon rank-sum comparisons)
- **R** (Implementation environment for the DICE/chromVAR validation workflow)

## Examples

```
library(DICE); data(ranzoniATAC); overlap.v <- intersect(atac.MEMP.prime.cell.v, atac.GP.prime.cell.v)
```

## Evaluation signals

- Summed GATA1/KLF1 TFA is significantly higher in primedMEMP cells (one-sided Wilcoxon P < 0.05 vs both primedGP and unprimed)
- IRF8 TFA is significantly higher in primedGP cells (one-sided Wilcoxon, alternative='greater')
- No cell is assigned to both primedMEMP and primedGP after intersect/setdiff overlap removal
- ATAC-side primed sets contain exactly the top-10% of cells by summed marker-gene accessibility (quantile 0.9 cutoff), and all marker genes used are present in rownames of ranzoni.gene.access.m
- The saved figure tfa_validation_boxplots.png reproduces the vignette's significance pattern

## Limitations

- No SHARE-Seq (joint scRNA-Seq & scATAC-Seq) profiles were available for the fetal liver hematopoiesis study, so primed states had to be assigned in the scATAC-Seq data by an invented, approximate (though rigorous) method
- Validation was purely computational: no experimental work was performed to demonstrate the importance of the identified regulatory factors in priming
- In real datasets the covariation patterns between regulatory factors did not deviate strongly from monotonic linear dependencies, so conclusions rest on accessibility enrichment rather than non-linear DICE behavior

## Evidence

- [methods] # exclude overlapping primed cells
overlap.v <- intersect(atac.MEMP.prime.cell.v,atac.GP.prime.cell.v): "# exclude overlapping primed cells
overlap.v <- intersect(atac.MEMP.prime.cell.v,atac.GP.prime.cell.v)"
- [readme] chromVAR is an R package for the analysis of sparse chromatin accessibility data from single cell or bulk ATAC or DNAse-seq data.: [evidence span withheld — anchor status: external_doc]
