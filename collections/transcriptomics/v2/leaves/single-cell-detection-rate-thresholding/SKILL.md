---
name: single-cell-detection-rate-thresholding
description: Use when filtering ligand-receptor pairs in single-cell expression data
  so a gene counts as 'expressed' in a cell type only if detected in more than 20%
  of its cells (NATMI --detectionThreshold default 0.2).
when_to_use_negative:
- Input is bulk expression or proteomic data where per-cell detection rates cannot
  be computed
- Data are not normalised to counts per million (raw counts would make the 10 CPM
  expressionThreshold meaningless)
- Analysis requires modelling heteromeric complexes — NATMI does not model them, so
  pair-level thresholding is insufficient
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NATMI
  role: Implements the detection-rate (>20% of cells) and expression (>10 CPM) filters
    via the --detectionThreshold and --expressionThreshold arguments of ExtractEdges.py
  repo: https://github.com/forrest-lab/NATMI/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:5374045b47c9b8a30ae8db73581a757ce452562bc4858484f65498ecbd3951a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-cell-detection-rate-thresholding

## Summary

Defines ligand/receptor 'expression' in a cell type for NATMI by requiring detection (non-zero counts) in >20% of that cell type's cells, optionally combined with an expression-level threshold of 10 CPM, before predicting cell-to-cell communication edges.

## When to use

Use when filtering ligand-receptor pairs in single-cell expression data so a gene counts as 'expressed' in a cell type only if detected in more than 20% of its cells (NATMI --detectionThreshold default 0.2).

## When NOT to use

- Input is bulk expression or proteomic data where per-cell detection rates cannot be computed
- Data are not normalised to counts per million (raw counts would make the 10 CPM expressionThreshold meaningless)
- Analysis requires modelling heteromeric complexes — NATMI does not model them, so pair-level thresholding is insufficient

## Inputs

- Normalised single-cell expression matrix (--emFile), genes as rows and cells as columns, in CPM
- Cell-labels metafile (--annFile) mapping each cell to a cell-type label
- Ligand-receptor pair list (default connectomeDB2020 or user-supplied)

## Outputs

- Per-cell-type detection/expression summary table of ligand-receptor pairs passing the 0.2 detection-rate filter (e.g. detection_expression_table.csv)
- Filtered communication network edges retained for downstream ExtractEdges.py output

## How to apply

Load the normalised CPM expression matrix (--emFile, genes as rows, cells as columns) and the cell-labels metafile (--annFile) with pandas. For each cell type, compute each gene's detection rate as the fraction of that cell type's cells with non-zero counts, and retain only ligand/receptor genes whose detection rate is greater than 0.2 (NATMI's --detectionThreshold default). Optionally apply --expressionThreshold (e.g. 10 counts-per-million), dropping edges where the mean expression of the ligand or receptor does not exceed it. Assemble a per-cell-type table of retained pairs with detection rates and expression levels; this filter governs which interactions ExtractEdges.py can propose before edge weighting and specificity scoring.

## Related tools

- **NATMI** (Implements the detection-rate (>20% of cells) and expression (>10 CPM) filters via the --detectionThreshold and --expressionThreshold arguments of ExtractEdges.py) — https://github.com/forrest-lab/NATMI/

## Examples

```
python ExtractEdges.py --emFile expression.matrix.txt --annFile annotation.txt --out outfolder --detectionThreshold 0.2 --expressionThreshold 10
```

## Evaluation signals

- Every retained ligand/receptor pair has a detection rate strictly greater than 0.2 in the relevant cell type
- Every retained pair has mean ligand and receptor expression greater than 10 CPM when --expressionThreshold 10 is applied
- Output is a CSV per-cell-type table containing both detection-rate and expression-level fields
- Edge counts should be consistent with NATMI defaults applied to Tabula Muris (e.g. a median of 54 pairs co-detected per cell type in that dataset)
- Running with default --detectionThreshold reproduces NATMI's stated rule that a gene is 'expressed' if >20% of cells of the cell type have non-zero read counts

## Limitations

- Predictions are based on mRNA expression rather than protein concentration and are subject to single-cell dropouts
- Detection rates and specificity weights are dataset-specific (local), not global reference values
- Heteromeric complexes are not modelled, so complex-member genes are filtered and scored individually

## Evidence

- [discussion] Detection threshold definition: "we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene"
- [methods] detectionThreshold argument: "--detectionThreshold DETECTIONTHRESHOLD do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
- [methods] expressionThreshold argument: "--expressionThreshold EXPRESSIONTHRESHOLD do not draw the edges in which expression levels of the ligand and the receptor are not greater than the threshold"
- [methods] CPM normalisation prerequisite: "normalized each expression profile by dividing it by the total number of unique molecular identifiers and multiplying by 1,000,000"
