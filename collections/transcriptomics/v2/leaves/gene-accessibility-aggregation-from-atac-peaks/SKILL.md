---
name: gene-accessibility-aggregation-from-atac-peaks
description: Use when matched peak-level scATAC-Seq counts must be summarized to per-gene
  accessibility (3kb-upstream+gene-body aggregation around RNA-defined marker genes)
  to validate primed states or compare chromatin opening across cell groups.
when_to_use_negative:
- The target is per-cell TF motif variability rather than per-gene accessibility -
  chromVAR computes motif deviations directly from the peak matrix, so aggregating
  peaks to genes is the wrong operation
- No marker-gene list or gene-body/TSS annotations are available to anchor peaks,
  so the 3kb-upstream+gene-body assignment window cannot be defined
- The scATAC-Seq data are neither matched to the same cells nor represent the same
  cell-types as the scRNA-Seq priming calls (the paper validated only in the same
  cells via SHARE-Seq or separate populations of the same cell-types)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0203
tools:
- name: DICE
  role: Hosts the scATAC-Seq validation workflow and matched peak-level datasets (ranzoniATAC);
    supplies scRNA-Seq-derived marker genes and primed-cell assignments that anchor
    the peak-to-gene aggregation
  repo: github.com/aet21/DICE
- name: Seurat
  role: Log-normalizes the aggregated gene accessible count matrix before primed vs
    unprimed comparisons
- name: chromVAR
  role: Computes TF activity scores (Runx2, Zfhx3, Foxo3, Lmo7) from the same peak-level
    scATAC-Seq data and binding motifs as a parallel validation readout to the gene-level
    accessibility matrix
  repo: GreenleafLab/chromVAR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2022.105709
    title: Distance covariance entropy reveals primed states and bifurcation dynamics
      in single-cell RNA-Seq data
  source_content_hash: sha256:06209077028f77626e009d307bfea101157dba195cc590364e705f65fe6e8cfb
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
  - ggpubr
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gene-accessibility-aggregation-from-atac-peaks

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates a gene-level accessibility matrix from peak-level scATAC-Seq data by summing peaks within a region 3kb upstream of each marker gene plus the gene body, then log-normalizes the gene accessible count matrix with Seurat. This converts chromatin peak data into per-gene scores that validate scRNA-Seq-derived DICE primed states in matched or same-cell ATAC profiles (e.g. SHARE-Seq).

## When to use

Use when matched peak-level scATAC-Seq counts must be summarized to per-gene accessibility (3kb-upstream+gene-body aggregation around RNA-defined marker genes) to validate primed states or compare chromatin opening across cell groups.

## When NOT to use

- The target is per-cell TF motif variability rather than per-gene accessibility - chromVAR computes motif deviations directly from the peak matrix, so aggregating peaks to genes is the wrong operation
- No marker-gene list or gene-body/TSS annotations are available to anchor peaks, so the 3kb-upstream+gene-body assignment window cannot be defined
- The scATAC-Seq data are neither matched to the same cells nor represent the same cell-types as the scRNA-Seq priming calls (the paper validated only in the same cells via SHARE-Seq or separate populations of the same cell-types)

## Inputs

- Peak-level scATAC-Seq count matrix (peaks x cells) with genomic coordinates of each peak, e.g. DICE's ranzoniATAC dataset or SHARE-Seq TAC peak data
- Marker-gene list derived from scRNA-Seq (genes with significantly higher expression in primed vs unprimed cells by Wilcoxon rank-sum test) plus gene/TSS coordinate annotations
- Per-cell priming assignments per lineage (primed vs unprimed), including identification of overlapping primed cells between competing lineages for exclusion

## Outputs

- Gene-level accessibility count matrix (marker genes x cells), each entry being the sum of peaks within 3kb upstream of the gene plus its gene body
- Log-normalized gene accessibility matrix (Seurat) ready for group comparisons
- One-sided Wilcoxon rank-sum statistics and P-values (alternative='greater') for aggregated accessibility in primed vs unprimed cells per lineage

## How to apply

Start from the peak-level scATAC-Seq count matrix with peak coordinates for the matched cells (e.g. DICE's ranzoniATAC data or SHARE-Seq TAC peaks) and a marker-gene set obtained from scRNA-Seq Wilcoxon rank-sum tests comparing primed versus unprimed populations (not restricted to regulatory factors). For each cell, aggregate the counts of peaks falling within a region 3kb upstream of each marker gene and its gene body into a gene-level accessibility count matrix (genes x cells). Log-normalize this gene accessible count matrix with Seurat before any group comparison. Before testing, exclude overlapping primed cells assigned as primed to both competing lineages (e.g. intersect of MEMP- and GP-primed cells, or HS- and IRS-primed TACs). Compare primed versus unprimed cells with one-sided Wilcoxon rank-sum tests (alternative='greater'); the aggregation is judged successful when primed cells show significantly higher accessibility toward their assigned lineage, concordant with chromVAR TF-activity and Palantir fate-probability readouts on the same peaks.

## Related tools

- **DICE** (Hosts the scATAC-Seq validation workflow and matched peak-level datasets (ranzoniATAC); supplies scRNA-Seq-derived marker genes and primed-cell assignments that anchor the peak-to-gene aggregation) — github.com/aet21/DICE
- **Seurat** (Log-normalizes the aggregated gene accessible count matrix before primed vs unprimed comparisons)
- **chromVAR** (Computes TF activity scores (Runx2, Zfhx3, Foxo3, Lmo7) from the same peak-level scATAC-Seq data and binding motifs as a parallel validation readout to the gene-level accessibility matrix) — GreenleafLab/chromVAR

## Examples

```
library(DICE); data(ranzoniATAC); data(humanTF); # aggregate peaks 3kb upstream of marker genes + gene body, then log-normalize with Seurat
```

## Evaluation signals

- The output matrix is marker genes x cells and each gene's value equals the summed counts of peaks overlapping the 3kb-upstream+gene-body window; genes with no overlapping peaks are zero prior to normalization
- Values are Seurat log-normalized (non-negative log scale), not raw peak counts, ensuring group comparisons are not confounded by library-size differences
- Overlapping primed cells (primed for both competing lineages) are removed before testing, as in the intersect of MEMP- and GP-primed cell vectors
- One-sided Wilcoxon tests (alternative='greater') report significantly higher aggregated accessibility in lineage-primed vs unprimed cells, concordant with chromVAR TF activity (Runx2/Zfhx3 in HS-primed; Foxo3/Lmo7 in IRS-primed) and Palantir fate probabilities

## Limitations

- The paper's own caveat: assignment of corresponding primed states in the scATAC-Seq data required an invented, approximate method rather than a joint-profile ground truth
- Aggregation only captures peaks within 3kb upstream of the marker gene and the gene body, so distal enhancers beyond that window are excluded by construction
- Matched joint profiles are not always available (SHARE-Seq profiles were not available for the liver hematopoiesis study), so validation may have to rely on separate cell populations of the same cell-types
- No experimental perturbation of the identified regulatory factors was performed; validation is purely computational (accessibility and TF activity), and dependency version numbers (Seurat, chromVAR) were not recorded

## Evidence

- [other] SHARE-Seq application: gene-level accessibility aggregated over marker-gene-associated peaks: [evidence span withheld — anchor status: not_found]
