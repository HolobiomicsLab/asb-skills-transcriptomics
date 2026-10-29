---
name: scatac-seq-gene-accessibility-quantification
description: Use when scRNA-Seq-derived cell assignments or marker gene lists must
  be validated against matched scATAC-Seq data by testing whether target genes of
  lineage regulators (e.g. GATA1, KLF1, IRF8) show higher chromatin accessibility
  in the predicted cells.
when_to_use_negative:
- No matched scATAC-Seq data for the same cell types is available — the vignette notes
  that joint SHARE-Seq profiles were not available for the liver hematopoiesis study,
  and this validation approach requires separate ATAC profiles of the same cell types
- 'The query factor is not a transcription factor (e.g. UHRF1): it has no motif and
  hence no chromVAR TF activity, so only target-gene accessibility sums apply'
- The goal is peak-level or cluster-level ATAC analysis rather than gene-level accessibility
  validation of predicted primed cells
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: DICE
  role: Provides the scRNA-Seq-derived primed-cell assignments, marker gene lists,
    and the ranzoniATAC data objects (ranzoni.gene.access.m, primedCellMarker.l, ranzoni.TFA.m)
    used for ATAC-side validation
  repo: github.com/aet21/DICE
- name: chromVAR
  role: Source of the precomputed per-cell TF motif activity matrix (ranzoni.TFA.m)
    whose GATA1/KLF1/IRF8 activities are compared between primed groups
  repo: https://github.com/GreenleafLab/chromVAR
- name: Seurat
  role: Used to log-normalize the gene-level accessibility count matrix generated
    from scATAC-Seq peaks
- name: ggplot2/ggpubr
  role: Builds boxplots of summed TFA per group and annotates one-sided Wilcoxon rank-sum
    tests (alternative='greater') via geom_signif
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:6274491ae03ace1b50d1f1bcbbfd35bd56e6a7bdbcf13e120b17a4a234050746
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# scatac-seq-gene-accessibility-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify per-cell gene-level chromatin accessibility from scATAC-Seq peak data (peaks 3kb upstream plus gene body, log-normalized with Seurat) to validate scRNA-Seq-derived primed states, using marker-gene accessibility sums and chromVAR TF motif activity as validation readouts.

## When to use

Use when scRNA-Seq-derived cell assignments or marker gene lists must be validated against matched scATAC-Seq data by testing whether target genes of lineage regulators (e.g. GATA1, KLF1, IRF8) show higher chromatin accessibility in the predicted cells.

## When NOT to use

- No matched scATAC-Seq data for the same cell types is available — the vignette notes that joint SHARE-Seq profiles were not available for the liver hematopoiesis study, and this validation approach requires separate ATAC profiles of the same cell types
- The query factor is not a transcription factor (e.g. UHRF1): it has no motif and hence no chromVAR TF activity, so only target-gene accessibility sums apply
- The goal is peak-level or cluster-level ATAC analysis rather than gene-level accessibility validation of predicted primed cells

## Inputs

- scATAC-Seq peak-level count matrix aggregated to gene-level accessibility (peaks within 3kb upstream of each gene plus gene body), log-normalized with Seurat
- Precomputed chromVAR TF activity matrix for the MPP cells (e.g. ranzoni.TFA.m)
- scRNA-Seq-derived primed-cell assignments and their Wilcoxon marker gene lists (e.g. primedCellMarker.l for GP pair UHRF1-IRF8 and MEMP pair GATA1-KLF1)

## Outputs

- Per-cell summed marker-gene accessibility scores and ATAC-side primed-cell assignments (top-10% by quantile-0.9 cutoff, mutually exclusive between lineages)
- Per-cell summed chromVAR TFA values per lineage (GATA1+KLF1 for MEMP; IRF8 for GP)
- Boxplot figure comparing TFA across primedMEMP, primedGP, and unprimed groups with one-sided Wilcoxon significance bars (tfa_validation_boxplots.png)

## How to apply

Load the precomputed ATAC objects (here data(ranzoniATAC): the gene-level accessibility matrix ranzoni.gene.access.m, primed-cell marker lists primedCellMarker.l, and precomputed chromVAR TF activity matrix ranzoni.TFA.m). Intersect each scRNA-Seq-derived Wilcoxon marker list with the rownames of the gene accessibility matrix, sum accessibility per cell, and define ATAC-side primed cells as the top-10% of cells exceeding the quantile-0.9 summed-accessibility cutoff. Exclude cells assigned as primed for both lineages using intersect/setdiff so each cell carries a single lineage label. Then compute per-cell summed chromVAR TFA of the lineage TFs — GATA1 plus KLF1 for the MEMP lineage, IRF8 alone for GP (UHRF1 is not a TF and has no motif activity) — and test group differences with ggplot2 boxplots annotated by ggpubr geom_signif one-sided Wilcoxon rank-sum tests (alternative='greater'). The rationale is that genuine primed states should exhibit elevated accessibility of the candidate regulators' targets even when the scATAC-Seq cells are distinct from the scRNA-Seq cells used for prediction.

## Related tools

- **DICE** (Provides the scRNA-Seq-derived primed-cell assignments, marker gene lists, and the ranzoniATAC data objects (ranzoni.gene.access.m, primedCellMarker.l, ranzoni.TFA.m) used for ATAC-side validation) — github.com/aet21/DICE
- **chromVAR** (Source of the precomputed per-cell TF motif activity matrix (ranzoni.TFA.m) whose GATA1/KLF1/IRF8 activities are compared between primed groups) — https://github.com/GreenleafLab/chromVAR
- **Seurat** (Used to log-normalize the gene-level accessibility count matrix generated from scATAC-Seq peaks)
- **ggplot2/ggpubr** (Builds boxplots of summed TFA per group and annotates one-sided Wilcoxon rank-sum tests (alternative='greater') via geom_signif)

## Examples

```
devtools::install_github("aet21/DICE"); library(DICE); data(ranzoniATAC); data(ranzoniBranchPr)
```

## Evaluation signals

- One-sided Wilcoxon rank-sum P < 0.05 for primedMEMP vs primedGP and primedMEMP vs unprimed summed GATA1+KLF1 TFA (alternative='greater')
- Significantly higher IRF8 TFA in primedGP cells compared with primedMEMP and unprimed cells
- No cell appears in both primedMEMP and primedGP sets — overlap.v <- intersect(...) must be empty after setdiff exclusion
- ATAC-side primed sets contain exactly the top-10% of cells by summed marker accessibility (quantile 0.9 cutoff) restricted to markers present in rownames of ranzoni.gene.access.m
- Figure saved as tfa_validation_boxplots.png shows the expected ordering of TFA distributions across the three groups

## Limitations

- The method to assign corresponding primed states in the scATAC-Seq data was invented/approximate by the authors because matched same-cell (SHARE-Seq) profiles were unavailable for the Ranzoni fetal liver dataset
- TFs lacking motifs (e.g. the chromatin regulator UHRF1) cannot be assessed via chromVAR TF activity, limiting validation to GATA1, KLF1, and IRF8
- No experimental work demonstrated the importance of the identified regulatory factors; validation is purely computational (scATAC-Seq)
- Software versions of dependencies (chromVAR, Seurat, destiny) and explicit DICE statistic values were not reported, complicating exact reproduction

## Evidence

- [results] chromVAR-derived TFA levels of GATA1 and KLF1 were higher in the cellular neighborhoods enriched for MEMP-primed cells, while the converse was true for TFA of IRF8 (higher in GP-primed neighborhoods).: [evidence span withheld — anchor status: not_found]
- [readme] chromVAR is an R package for the analysis of sparse chromatin accessibility data from single cell or bulk ATAC or DNAse-seq data.: [evidence span withheld — anchor status: external_doc]
