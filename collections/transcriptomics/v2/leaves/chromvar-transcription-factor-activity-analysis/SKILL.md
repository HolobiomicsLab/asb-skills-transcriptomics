---
name: chromvar-transcription-factor-activity-analysis
description: Use when scRNA-Seq-derived lineage-priming predictions (e.g. DICE primed
  cells and their TFs such as GATA1, KLF1, IRF8) need independent validation against
  matched scATAC-Seq accessibility and chromVAR TF activity (TFA) data.
when_to_use_negative:
- No matched scATAC-Seq data exists for the scRNA-Seq study (e.g. SHARE-Seq joint
  profiles were not available for the liver hematopoiesis data), so ATAC-side primed
  cells cannot be defined from marker accessibility
- The candidate regulator is not a TF (e.g. UHRF1, an epigenetic regulator) — chromVAR
  motif activity is undefined for non-TF factors
- The goal is TF activity from scRNA-Seq expression rather than chromatin accessibility;
  chromVAR requires sparse chromatin accessibility data
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3676
tools:
- name: chromVAR
  role: R package for sparse chromatin accessibility data; infers per-cell TF motif
    activity used to test elevation in predicted primed cells
  repo: https://github.com/GreenleafLab/chromVAR
- name: DICE
  role: Supplies the scRNA-Seq-derived primed cell assignments and priming TFs (FindPrimedCells)
    and the ranzoniATAC data objects
  repo: https://github.com/aet21/DICE
- name: Seurat
  role: Log-normalizes the gene-level accessibility count matrix aggregated from scATAC-Seq
    peaks 3kb upstream and over gene bodies
- name: ggplot2/ggpubr
  role: Boxplot visualization with ggpubr geom_signif one-sided Wilcoxon rank-sum
    tests comparing primedMEMP, primedGP, and unprimed groups
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:ee215ad60e8d1c9c3fefb2f47b5953eadfcb097b0b807a358a791ef0eeb86b6e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aet21/DICE
  repo_url: https://github.com/GreenleafLab/chromVAR
  tools:
  - DICE
  - R
  - Seurat
  - chromVAR
  - ggplot2
  - ggplot2/ggpubr
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3676
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# chromvar-transcription-factor-activity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates scRNA-Seq-predicted primed cell states by testing whether chromVAR-inferred transcription factor (TF) motif activity is elevated in the corresponding primed cells defined from matched scATAC-Seq chromatin accessibility. In the DICE workflow, GATA1/KLF1 activity must be higher in MEMP-primed neighborhoods and IRF8 activity higher in GP-primed neighborhoods.

## When to use

Use when scRNA-Seq-derived lineage-priming predictions (e.g. DICE primed cells and their TFs such as GATA1, KLF1, IRF8) need independent validation against matched scATAC-Seq accessibility and chromVAR TF activity (TFA) data.

## When NOT to use

- No matched scATAC-Seq data exists for the scRNA-Seq study (e.g. SHARE-Seq joint profiles were not available for the liver hematopoiesis data), so ATAC-side primed cells cannot be defined from marker accessibility
- The candidate regulator is not a TF (e.g. UHRF1, an epigenetic regulator) — chromVAR motif activity is undefined for non-TF factors
- The goal is TF activity from scRNA-Seq expression rather than chromatin accessibility; chromVAR requires sparse chromatin accessibility data

## Inputs

- Precomputed Ranzoni scATAC-Seq objects from data(ranzoniATAC): gene-level accessibility matrix ranzoni.gene.access.m (peaks 3kb upstream of marker genes and gene body, log-normalized with Seurat)
- Primed-cell marker-gene lists primedCellMarker.l (Wilcoxon-derived markers of primedGP and primedMEMP MPPs)
- Precomputed chromVAR TF activity matrix ranzoni.TFA.m for MPPs
- scRNA-Seq-derived primed-GP and primed-MEMP MPP cell assignments from DICE FindPrimedCells (GP pair UHRF1-IRF8, MEMP pair GATA1-KLF1)

## Outputs

- ATAC-side primed cell assignments (top-10% by summed marker accessibility, mutually exclusive across lineages)
- Per-cell summed TF activity scores (GATA1+KLF1 for MEMP; IRF8 for GP)
- Boxplot figure tfa_validation_boxplots.png with ggpubr geom_signif one-sided Wilcoxon rank-sum P-values

## How to apply

Load the precomputed scATAC-Seq objects via data(ranzoniATAC): the gene-level accessibility matrix ranzoni.gene.access.m, the primed-cell marker-gene lists primedCellMarker.l, and the precomputed chromVAR TFA matrix ranzoni.TFA.m for MPPs. Take the scRNA-Seq-derived primed-GP and primed-MEMP MPP assignments (GP pair UHRF1-IRF8, MEMP pair GATA1-KLF1 from FindPrimedCells) and their Wilcoxon-derived marker genes, intersect each marker list with the rownames of ranzoni.gene.access.m, sum accessibility per cell, and define ATAC-side primed cells as the top-10% of cells (quantile 0.9 cutoff) with highest summed marker accessibility. Exclude cells classified as primed for both lineages using intersect/setdiff so each cell has a single assignment. For the MEMP lineage compute the summed chromVAR TFA of GATA1 and KLF1 per cell; for the GP lineage use IRF8 TFA alone (UHRF1 is not a TF and has no motif activity). Build ggplot2 boxplots with ggpubr geom_signif one-sided Wilcoxon rank-sum tests (alternative='greater') comparing primedMEMP vs primedGP and vs unprimed (and analogously for IRF8 in primedGP), saving the figure as tfa_validation_boxplots.png. The rationale is that chromVAR identifies motifs associated with variability in chromatin accessibility between individual cells, so elevated target accessibility in predicted primed cells should translate into elevated TFA.

## Related tools

- **chromVAR** (R package for sparse chromatin accessibility data; infers per-cell TF motif activity used to test elevation in predicted primed cells) — https://github.com/GreenleafLab/chromVAR
- **DICE** (Supplies the scRNA-Seq-derived primed cell assignments and priming TFs (FindPrimedCells) and the ranzoniATAC data objects) — https://github.com/aet21/DICE
- **Seurat** (Log-normalizes the gene-level accessibility count matrix aggregated from scATAC-Seq peaks 3kb upstream and over gene bodies)
- **ggplot2/ggpubr** (Boxplot visualization with ggpubr geom_signif one-sided Wilcoxon rank-sum tests comparing primedMEMP, primedGP, and unprimed groups)

## Examples

```
library(DICE); data(ranzoniATAC); atac.MEMP.prime.cell.v <- names(which(apply(ranzoni.gene.access.m[intersect(primedCellMarker.l$MEMP, rownames(ranzoni.gene.access.m)),], 2, sum) > quantile(colSums(ranzoni.gene.access.m), 0.9)))
```

## Evaluation signals

- Boxplot shows significantly higher summed GATA1/KLF1 TFA in primedMEMP cells (one-sided Wilcoxon P < 0.05 vs primedGP and vs unprimed)
- Boxplot shows significantly higher IRF8 TFA in primedGP cells, reproducing the vignette result
- ATAC-side primed cells are defined as the top-10% (quantile 0.9) of cells by summed marker-gene accessibility, with no cell assigned to both lineages (overlap removed via intersect/setdiff)
- Each marker-gene list is restricted to genes present in rownames(ranzoni.gene.access.m) before summing accessibility
- Figure file tfa_validation_boxplots.png is produced with ggpubr significance brackets from alternative='greater' tests

## Limitations

- Validation of primed states in scATAC-Seq relied on an invented, approximate method for assigning corresponding primed states, since SHARE-Seq (joint scRNA-Seq & scATAC-Seq) profiles were not available for the liver hematopoiesis study
- No experimental work was performed to demonstrate the importance of the identified regulatory factors in priming; only scATAC-Seq validation was done
- UHRF1, part of the GP priming pair, has no chromVAR motif activity because it is not a TF, so GP validation rests on IRF8 alone

## Evidence

- [results] chromVAR-derived TFA levels of GATA1 and KLF1 were higher in the cellular neighborhoods enriched for MEMP-primed cells, while the converse was true for TFA of IRF8: [evidence span withheld — anchor status: not_found]
- [readme] chromVAR is an R package for the analysis of sparse chromatin accessibility data from single cell or bulk ATAC or DNAse-seq data: [evidence span withheld — anchor status: external_doc]
- [methods] Generate gene-level accessibility matrix from scATAC-Seq peaks 3kb upstream of marker genes and gene body, log-normalized with Seurat: "use the scATAC-Seq peak-level data to generate a gene-level accessibility count matrix by aggregating peaks within a region 3kb upstream of each marker gene and gene body, and then log-normalize the"
- [discussion] Validation of primed states in scATAC-Seq data relied on an invented, approximate method for assigning corresponding primed states: "to validate the primed states as identified from the scRNA-Seq data, we had to invent a method to assign corresponding primed states in the scATAC-Seq data."
