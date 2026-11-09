---
name: cpm-expression-threshold-application
description: Use when filtering ligand–receptor pairs from single-cell or bulk expression
  data (e.g. Tabula Muris) with NATMI's --detectionThreshold and --expressionThreshold
  arguments before predicting communication edges.
when_to_use_negative:
- The expression matrix has not been normalised to CPM (dividing by total UMIs/reads
  and multiplying by 1,000,000) — the 10 CPM expression threshold is meaningless on
  raw counts
- The goal is modelling heteromeric complexes, co-receptor requirements, or downstream
  signalling consequences — NATMI explicitly does not model these
- The data are spatial transcriptomics — NATMI has not been specifically designed
  for spatial data
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: NATMI
  role: Provides the --detectionThreshold (default 0.2) and --expressionThreshold
    arguments applied during edge extraction with ExtractEdges.py
  repo: https://github.com/forrest-lab/NATMI/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:5b5737ba90452c5cb39bee90354e3157998f0958685b5b252c9ffa770172ba84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cpm-expression-threshold-application

## Summary

Applies NATMI's expression and detection filtering — a ligand/receptor gene counts as 'expressed' in a cell type only if detected in more than 20% of that cell type's cells (detectionThreshold default 0.2) and mean expression levels of ligand and receptor exceed 10 CPM — to decide which ligand–receptor edges are retained for cell-to-cell communication prediction.

## When to use

Use when filtering ligand–receptor pairs from single-cell or bulk expression data (e.g. Tabula Muris) with NATMI's --detectionThreshold and --expressionThreshold arguments before predicting communication edges.

## When NOT to use

- The expression matrix has not been normalised to CPM (dividing by total UMIs/reads and multiplying by 1,000,000) — the 10 CPM expression threshold is meaningless on raw counts
- The goal is modelling heteromeric complexes, co-receptor requirements, or downstream signalling consequences — NATMI explicitly does not model these
- The data are spatial transcriptomics — NATMI has not been specifically designed for spatial data

## Inputs

- Normalised single-cell (or bulk) expression matrix ('--emFile') with gene identifiers as rows and cell identifiers as columns, scaled to counts per million (CPM)
- Cell-labels metafile ('--annFile') mapping each cell to a cell-type label
- Ligand–receptor pair list (connectomeDB2020 or user-supplied)

## Outputs

- Per-cell-type detection/expression summary table (detection_expression_table.csv) listing retained ligand–receptor pairs with detection rates and expression levels
- Filtered ligand–receptor edges (all retained pairs have detection rate > 0.2 and expression > 10 CPM) used for downstream network construction

## How to apply

Load the normalised expression matrix ('--emFile', genes as rows, cells as columns, expression normalised by dividing each profile by total UMI count and multiplying by 1,000,000 to give CPM) and the cell-labels metafile ('--annFile') with pandas. For each cell type, compute each gene's detection rate as the fraction of that cell type's cells with non-zero read counts, and retain a ligand/receptor gene only if its detection rate is greater than 0.2 (the '--detectionThreshold' default). Optionally drop edges whose mean ligand or receptor expression level is not greater than 10 CPM using the '--expressionThreshold' argument. Assemble a per-cell-type table of retained pairs with their detection rates and expression levels (e.g. detection_expression_table.csv) and verify that every retained pair exceeds both thresholds before feeding edges to ExtractEdges.py / VisInteractions.py.

## Related tools

- **NATMI** (Provides the --detectionThreshold (default 0.2) and --expressionThreshold arguments applied during edge extraction with ExtractEdges.py) — https://github.com/forrest-lab/NATMI/

## Examples

```
python ExtractEdges.py --emFile expression.matrix.txt --annFile annotation.txt --out outfolder --detectionThreshold 0.2 --expressionThreshold 10
```

## Evaluation signals

- Output file exists in CSV format and contains detection-rate and expression-level fields
- Every retained ligand/receptor gene has a detection rate strictly greater than 0.2 in the relevant cell type
- Every retained pair has a mean expression level greater than 10 CPM for both ligand and receptor
- Detection rates are computed per cell type as the fraction of that cell type's cells with non-zero read counts
- Downstream edges passed to VisInteractions.py exclude any pair failing either threshold

## Limitations

- Detection and expression thresholds are applied on mRNA expression levels rather than protein concentration, and data are subject to dropouts
- Edges highlighted in the manuscript were based on experiments without replicates
- Specificity weights are dataset-specific (local), not global; no global specificity reference is yet available

## Evidence

- [discussion] Detection threshold >20% of cells: "we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene"
- [methods] detectionThreshold argument: "--detectionThreshold DETECTIONTHRESHOLD do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
- [methods] expressionThreshold argument: "--expressionThreshold EXPRESSIONTHRESHOLD do not draw the edges in which expression levels of the ligand and the receptor are not greater than the threshold"
- [methods] CPM normalisation: "normalized each expression profile by dividing it by the total number of unique molecular identifiers and multiplying by 1,000,000"
- [discussion] mRNA vs protein caveat: "the predictions are necessarily based on the mRNA expression levels rather than on protein concentration and the data is subject to dropouts"
