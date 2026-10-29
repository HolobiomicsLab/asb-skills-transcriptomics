---
name: scrnaseq-normalized-expression-processing
description: Use when constructing ligand-receptor communication edges from a normalised
  single-cell expression matrix and need an explicit, reproducible definition of gene
  'expression' per cell type before edge extraction.
when_to_use_negative:
- Input is bulk transcriptomics or proteomic data where per-cell detection rates are
  not defined
- Heteromeric complex-mediated interactions are required, since NATMI does not model
  them
- Spatially resolved communication inference is the goal, as NATMI has not been specifically
  designed for spatial data
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: NATMI
  role: Provides the detectionThreshold (default 0.2) and expressionThreshold (10
    CPM) filters applied during ExtractEdges.py to define expressed ligand/receptor
    genes per cell type
  repo: https://github.com/forrest-lab/NATMI/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:922a09059428edb7a46ba858639093fd49354e47d56b951631ac0ec0e5d5c97e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# scrnaseq-normalized-expression-processing

## Summary

Applies NATMI's gene-expression filtering convention to single-cell data: a ligand/receptor counts as expressed in a cell type only if detected in more than 20% of that type's cells and its mean expression exceeds 10 counts-per-million (CPM), yielding a per-cell-type detection/expression table used to gate cell-to-cell communication edges.

## When to use

Use when constructing ligand-receptor communication edges from a normalised single-cell expression matrix and need an explicit, reproducible definition of gene 'expression' per cell type before edge extraction.

## When NOT to use

- Input is bulk transcriptomics or proteomic data where per-cell detection rates are not defined
- Heteromeric complex-mediated interactions are required, since NATMI does not model them
- Spatially resolved communication inference is the goal, as NATMI has not been specifically designed for spatial data

## Inputs

- Normalised single-cell gene expression matrix ('--emFile', gene identifiers as rows, cell identifiers as columns, CPM-normalised)
- Cell labels metafile ('--annFile') mapping each cell to a cell-type label
- Ligand-receptor pair list (connectomeDB2020 or user-supplied)

## Outputs

- Per-cell-type detection/expression summary table (detection_expression_table.csv) listing each retained ligand/receptor pair with detection rate (>0.2) and expression level (>10 CPM)

## How to apply

Load the normalised expression matrix ('--emFile', genes as rows, cells as columns) and the cell-labels metafile ('--annFile') with pandas. For each cell type, compute each gene's detection rate as the fraction of that type's cells with non-zero counts, and retain ligand/receptor genes whose detection rate is greater than 0.2 (NATMI's '--detectionThreshold' default). Optionally apply the '--expressionThreshold' filter, dropping edges whose ligand or receptor mean expression is not greater than 10 CPM; input data must first be normalised by dividing each profile by total UMI count and multiplying by 1,000,000. Assemble a per-cell-type table of retained ligand/receptor pairs with their detection rate and expression level (e.g., median 54 pairs co-detected in Tabula Muris) and write it to detection_expression_table.csv.

## Related tools

- **NATMI** (Provides the detectionThreshold (default 0.2) and expressionThreshold (10 CPM) filters applied during ExtractEdges.py to define expressed ligand/receptor genes per cell type) — https://github.com/forrest-lab/NATMI/

## Examples

```
python ExtractEdges.py --emFile expression.matrix.txt --annFile annotation.txt --out outfolder --detectionThreshold 0.2 --expressionThreshold 10
```

## Evaluation signals

- Output file exists in csv format and contains detection-rate and expression-level fields
- Every retained ligand/receptor pair has detection rate strictly greater than 0.2 in the relevant cell type
- Every retained pair has mean expression level greater than 10 CPM after normalisation (total UMI × 1,000,000)
- Sanity check against reported scale: Tabula Muris showed a median of 54 ligand-receptor pairs co-detected per cell type
- Expression matrix was CPM-normalised before thresholding (values rescaled by multiplying by 1,000,000)

## Limitations

- Predictions are based on mRNA expression levels rather than protein concentration and scRNA-seq data is subject to dropouts
- Detection-rate and expression-threshold cutoffs are heuristic; edges discarded at these cutoffs may represent real signalling
- Results depend on the original authors' cell annotations, which can potentially affect the accuracy of predicted networks

## Evidence

- [discussion] Detection threshold discussion: "we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene"
- [methods] detectionThreshold argument: "--detectionThreshold DETECTIONTHRESHOLD
                        do not draw the interactions in which detection rates of the ligand and the receptor are lower than the threshold (default 0.2)"
- [methods] expressionThreshold argument: "--expressionThreshold EXPRESSIONTHRESHOLD
                        do not draw the edges in which expression levels of the ligand and the receptor are not greater than the threshold"
- [methods] CPM normalisation: "normalized each expression profile by dividing it by the total number of unique molecular identifiers and multiplying by 1,000,000"
- [discussion] mRNA-based prediction caveat: "the predictions are necessarily based on the mRNA expression levels rather than on protein concentration and the data is subject to dropouts"
