---
name: ligand-receptor-expression-filtering
description: Use when building ligand–receptor communication edges from single-cell
  expression data and you need to decide which genes count as 'expressed' per cell
  type before edge extraction.
when_to_use_negative:
- Input data are not normalised to counts per million — detection/expression thresholds
  assume CPM-scaled values, so apply NATMI's normalisation (divide by total UMIs/reads,
  multiply by 1,000,000) first
- You need heteromeric complex modelling or co-receptor requirements — NATMI does
  not model these, so filtering simple pairs is the wrong fit
- You need spatially resolved communication predictions — NATMI has not been specifically
  designed for spatial data
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: NATMI
  role: Implements the detectionThreshold and expressionThreshold filters within ExtractEdges.py
    to gate which ligand–receptor edges are predicted
  repo: https://github.com/forrest-lab/NATMI/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:a7e527ffc7ec45b58af6d249d81bea0cd3cbe11cffd7b571825b0a1f705e893c
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
n_claims: 7
---

# ligand-receptor-expression-filtering

## Summary

NATMI-style filtering that declares a ligand or receptor 'expressed' in a cell type only when detected in more than 20% of that cell type's cells and optionally requires mean expression above 10 counts-per-million, yielding a per-cell-type detection/expression summary used to gate predicted communication edges.

## When to use

Use when building ligand–receptor communication edges from single-cell expression data and you need to decide which genes count as 'expressed' per cell type before edge extraction.

## When NOT to use

- Input data are not normalised to counts per million — detection/expression thresholds assume CPM-scaled values, so apply NATMI's normalisation (divide by total UMIs/reads, multiply by 1,000,000) first
- You need heteromeric complex modelling or co-receptor requirements — NATMI does not model these, so filtering simple pairs is the wrong fit
- You need spatially resolved communication predictions — NATMI has not been specifically designed for spatial data

## Inputs

- Normalised single-cell expression matrix (--emFile), gene identifiers as rows and cell identifiers as columns, values in CPM
- Cell labels metafile (--annFile) mapping each cell to a cell-type label
- Ligand–receptor pair list (connectomeDB2020 by default, or user-supplied pairs)

## Outputs

- Per-cell-type detection/expression summary table (e.g. detection_expression_table.csv) listing ligand–receptor pairs with detection rate and expression level
- Filtered set of ligand–receptor edges where both partners pass detection (>0.2) and optional expression (>10 CPM) thresholds

## How to apply

Load the normalised (CPM) expression matrix ('--emFile', genes as rows, cells as columns) and the cell-labels metafile ('--annFile') with pandas. For each cell type, compute per-gene detection rate as the fraction of cells with non-zero counts, and retain ligand/receptor genes whose detection rate is greater than the --detectionThreshold (default 0.2). Optionally apply the --expressionThreshold, dropping edges in which the mean expression level of the ligand or receptor is not greater than the threshold (e.g. 10 CPM). Assemble a per-cell-type table of retained pairs with detection rate and expression level (detection_expression_table.csv). The 20% rule is NATMI's stated definition of expression because scRNA-seq data are subject to dropouts, so requiring detection in a majority-minority of cells reduces spurious edges.

## Related tools

- **NATMI** (Implements the detectionThreshold and expressionThreshold filters within ExtractEdges.py to gate which ligand–receptor edges are predicted) — https://github.com/forrest-lab/NATMI/

## Examples

```
python ExtractEdges.py --emFile expression.matrix.txt --annFile annotation.txt --out outfolder --detectionThreshold 0.2 --expressionThreshold 10
```

## Evaluation signals

- Output file exists in csv format and contains both detection-rate and expression-level fields
- Every retained ligand/receptor pair has detection rate strictly greater than 0.2 in the relevant cell type
- Every retained pair has mean expression level greater than the expression threshold (e.g. 10 CPM) when --expressionThreshold is applied
- Detection rates were computed as the fraction of cells with non-zero counts within each annotated cell type, not across the whole dataset
- Dropped-edge rate is consistent with threshold stringency: tightening detectionThreshold toward 1.0 monotonically reduces retained pairs

## Limitations

- Predictions are based on mRNA expression rather than protein concentration and single-cell data are subject to dropouts, so genuine low-level expressers may be filtered out
- NATMI does not model heteromeric complexes, heterodimerization, or requirements for co-receptors
- Cell-type labels come from the original dataset authors' annotations, which can potentially affect the accuracy of predicted networks

## Evidence

- [discussion] Detection threshold: gene expressed if detected in more than 20% of cells of a cell type: "we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene"
- [methods] detectionThreshold argument: "--detectionThreshold DETECTIONTHRESHOLD do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
- [methods] expressionThreshold argument: "--expressionThreshold EXPRESSIONTHRESHOLD do not draw the edges in which expression levels of the ligand and the receptor are not greater than the threshold"
- [discussion] mRNA and dropout caveat: "the predictions are necessarily based on the mRNA expression levels rather than on protein concentration and the data is subject to dropouts"
- [readme] ExtractEdges invocation: [evidence span withheld — anchor status: not_found]
