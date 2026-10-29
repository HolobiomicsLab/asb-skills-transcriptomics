---
name: contaminated-cell-marker-based-detection
description: Use when building the SINCERA S4 object from a raw expression matrix
  (genes x cells, FPKM/TPM) and you suspect mixed-cell wells or droplets, e.g. cells
  coexpressing markers of two lineages such as Epcam and Pecam1.
when_to_use_negative:
- Input is a normalized matrix that has already passed cell filtering — filterContaminatedCells
  operates on the SINCERA S4 object during preprocessing, before z-score normalization
  via normalization.zscore.
- No lineage-specific marker genes are known for the tissue; the function requires
  highly specific markers supplied via markers.1 and markers.2 to be meaningful.
- The analysis goal is clustering or cell type identification itself rather than data
  filtering — this skill is explicitly a preprocessing step preceding clustering.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3518
tools:
- name: SINCERA
  role: R S4 pipeline providing the filterContaminatedCells function used to remove
    cells based on coexpression of known marker genes (markers.1/markers.2), after
    construct() and filterLowQualityCells()
  repo: github.com/xu-lab/SINCERA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-1-4939-7710-9_15
    title: Single-Cell Transcriptome Analysis Using SINCERA Pipeline
  source_content_hash: sha256:107338efd8d85364969c4a171b6e4715b862344373844a164551885b7618c7d1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - SINCERA
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Contaminated-cell marker-based detection

## Summary

Detects and removes potentially contaminated cells from a scRNA-Seq dataset in SINCERA by testing coexpression of known marker genes (e.g., epithelial vs endothelial markers) using the filterContaminatedCells function, as part of the preprocessing stage before clustering.

## When to use

Use when building the SINCERA S4 object from a raw expression matrix (genes x cells, FPKM/TPM) and you suspect mixed-cell wells or droplets, e.g. cells coexpressing markers of two lineages such as Epcam and Pecam1.

## When NOT to use

- Input is a normalized matrix that has already passed cell filtering — filterContaminatedCells operates on the SINCERA S4 object during preprocessing, before z-score normalization via normalization.zscore.
- No lineage-specific marker genes are known for the tissue; the function requires highly specific markers supplied via markers.1 and markers.2 to be meaningful.
- The analysis goal is clustering or cell type identification itself rather than data filtering — this skill is explicitly a preprocessing step preceding clustering.

## Inputs

- SINCERA S4 object created by construct() from an expression matrix (exprfile: rows = genes, columns = cells, FPKM/TPM values) and a single-column sample table (samplefile)
- Two sets of highly specific, mutually exclusive marker genes supplied via markers.1 and markers.2 (e.g., Epcam vs Pecam1)

## Outputs

- SINCERA S4 object with contaminated cells removed
- Per-cell QC summary entry recording cells removed by filterContaminatedCells

## How to apply

After installing SINCERA (devtools::install_github("xu-lab/SINCERA")) and running construct on the expression matrix (exprfile, rows = genes, columns = cells) and the sample table (samplefile) to create the S4 object, run filterLowQualityCells with defaults min.expression = 1 FPKM/TPM and min.genes = 500 first. Then call filterContaminatedCells, supplying two highly specific marker sets via markers.1 and markers.2; cells coexpressing both marker sets (e.g., Epcam for epithelium and Pecam1 for endothelium in the mouse lung E16.5 demo) are flagged as potentially contaminated and dropped. The rationale is that a genuine cell should not express exclusive markers of unrelated cell types, so coexpression signals doublets or multiplexed/well-mixed captures. Record which cells are removed at this step in the per-cell QC summary so that every dropped cell is accounted for downstream.

## Related tools

- **SINCERA** (R S4 pipeline providing the filterContaminatedCells function used to remove cells based on coexpression of known marker genes (markers.1/markers.2), after construct() and filterLowQualityCells()) — github.com/xu-lab/SINCERA

## Examples

```
object <- filterContaminatedCells(object, markers.1="Epcam", markers.2="Pecam1")
```

## Evaluation signals

- Only cells surviving filterLowQualityCells (min.expression = 1 FPKM/TPM, min.genes = 500) and filterContaminatedCells remain in the expression matrix written to normalized_expression_matrix.tsv.
- No cell in the filtered object coexpresses both supplied marker sets (e.g., Epcam above detection and Pecam1 above detection simultaneously).
- The QC summary accounts for every removed cell, listing cells dropped by each filter including filterContaminatedCells.
- Per-gene values in the final output are z-score transformed (from normalization.zscore), confirming the filter ran upstream of normalization.
- Marker sets used are demonstrably highly specific (e.g., epithelial Epcam vs endothelial Pecam1 in the mouse lung E16.5 demo), not pan-expressed genes.

## Limitations

- Detection depends entirely on the choice of markers: only 'highly specific markers' (e.g., Epcam vs Pecam1) are appropriate; non-specific markers will misclassify cells.
- The step is optional and catches contamination only between the two marker-defined lineages supplied via markers.1 and markers.2; contamination involving other cell types is not assessed.
- SINCERA's preprocessing assumes FPKM/TPM-scale expression (filters at 1 FPKM/TPM for cells, >5 FPKM/TPM in <2 cells for genes, 0.01 FPKM/TPM minimum), so count-matrix inputs would need transformation first.

## Evidence
