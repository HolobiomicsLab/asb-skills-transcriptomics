---
name: cell-type-annotation-mapping
description: Use when a single-cell expression matrix (genes x cells) must be paired
  with a cell-annotation metafile before running NATMI ExtractEdges.py, or when cell-type
  labels need to be validated so edges are attributed to the right cell types.
when_to_use_negative:
- The input is bulk transcriptomics or proteomic data — the cell-labels metafile is
  required only for single-cell analysis, where cell-to-type mapping applies
- The expression matrix is spatial-resolved data — NATMI has not been specifically
  designed for spatial data, so mapping cells to histological locations is out of
  scope
- Cell annotations are unavailable or untrustworthy — the paper notes using original
  authors' annotations 'can potentially affect the accuracy of predicted networks'
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NATMI
  role: Consumes the --annFile cell-labels metafile with the --emFile expression matrix
    in ExtractEdges.py to assign cell-type labels and extract ligand-receptor edges
    between annotated cell types
  repo: https://github.com/forrest-lab/NATMI/
- name: pandas
  role: Used to load the expression matrix and cell-labels metafile, and to map each
    cell barcode to its cell-type annotation when preparing the aggregated per-cell-type
    table
  repo: https://pandas.pydata.org/
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:37097fbbd890d535bd260ca952b7eab7a1d28bf60acfbac96ad2ce362fe8d9a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cell-type-annotation-mapping

## Summary

Prepare the cell-labels metafile that maps each cell barcode in a single-cell expression matrix to a cell-type label, so NATMI's ExtractEdges.py can aggregate expression per cell type and predict cell-to-cell communication edges. Correct barcode-to-label mapping is a prerequisite for computing per-cell-type detection rates, mean-expression weights, and specificity weights.

## When to use

Use when a single-cell expression matrix (genes x cells) must be paired with a cell-annotation metafile before running NATMI ExtractEdges.py, or when cell-type labels need to be validated so edges are attributed to the right cell types.

## When NOT to use

- The input is bulk transcriptomics or proteomic data — the cell-labels metafile is required only for single-cell analysis, where cell-to-type mapping applies
- The expression matrix is spatial-resolved data — NATMI has not been specifically designed for spatial data, so mapping cells to histological locations is out of scope
- Cell annotations are unavailable or untrustworthy — the paper notes using original authors' annotations 'can potentially affect the accuracy of predicted networks'

## Inputs

- Cell-labels metafile (--annFile) mapping each cell barcode/column to a cell-type label (e.g. toy.sc.ann.txt, annotation.txt)
- Single-cell normalised expression matrix (--emFile) with gene identifiers as rows and cell identifiers as columns (e.g. toy.sc.em.txt)
- Optionally, cell-type annotation metadata from public atlases (Tabula Muris cell-type annotation metafile)

## Outputs

- Per-cell-type aggregated expression/detection summary used by ExtractEdges.py
- Annotated cell-type labels attributed to each predicted ligand-receptor edge in the edge output (e.g. detection_expression_table.csv, edge_weights.csv sender/receiver columns)

## How to apply

Load the cell-labels metafile (--annFile) alongside the expression matrix (--emFile, genes as rows, cell identifiers as columns) with pandas and confirm every cell column maps to exactly one cell-type label; NATMI requires this mapping for single-cell analysis to aggregate expression per cell type. Use annotations over all cell types in the atlas (e.g. the 117 Tabula Muris cell types), since NATMI uses these labels for sender-receiver edge extraction with connectomeDB2020 (lrc2p) pairs. The manuscript relied on the original authors' cell annotations, and NATMI accepts official gene symbols plus the label file without further conversion beyond species homology matching (--species mouse maps mouse symbols to the human pair list). After mapping, verify that per-cell-type detection rates (>20% of cells, --detectionThreshold 0.2) and mean expression (10 CPM, --expressionThreshold) can be computed per label before edge-weight computation.

## Related tools

- **NATMI** (Consumes the --annFile cell-labels metafile with the --emFile expression matrix in ExtractEdges.py to assign cell-type labels and extract ligand-receptor edges between annotated cell types) — https://github.com/forrest-lab/NATMI/
- **pandas** (Used to load the expression matrix and cell-labels metafile, and to map each cell barcode to its cell-type annotation when preparing the aggregated per-cell-type table) — https://pandas.pydata.org/

## Examples

```
python ExtractEdges.py --emFile toy.sc.em.txt --annFile toy.sc.ann.txt --species mouse --interDB lrc2p --out outfolder
```

## Evaluation signals

- Every cell identifier (column) in the expression matrix has exactly one cell-type label in the --annFile metafile, with no unmapped barcodes
- The number of distinct labels matches the expected atlas granularity (e.g. 117 cell types for the Tabula Muris atlas)
- Per-cell-type detection rates can be computed such that a gene is 'expressed' only when detected in more than 20% of that cell type's cells (--detectionThreshold 0.2)
- Retained edges carry valid sender/receiver cell-type labels, and for Tabula Muris the aggregated autocrine analysis yields a median of ~54 co-detected pairs per cell type and 719/2293 (~31%) co-detected connectomeDB2020 pairs
- Per-cell-type mean-expression and specificity weights (0-1, 1 meaning the pair is detected only in that cell-type combination) are computable from the label mapping

## Limitations

- Predicted networks rely on the original authors' cell annotations, which 'can potentially affect the accuracy of predicted networks'
- The metafile is only used for single-cell analysis; bulk datasets bypass cell-type mapping entirely
- Specificity weights computed after label aggregation are dataset-specific (local), not global; a global reference (e.g. Human Cell Atlas) is not yet available

## Evidence

- [readme] metafile requirements: "the metafile describing mapping between each cell and a cell-type label"
- [discussion] ExtractEdges invocation: "python ExtractEdges.py --emFile expression.matrix.txt --annFile annotation.txt --out outfolder"
- [discussion] annotation provenance caveat: "we have used the original cellular annotations provided by the authors of each manuscript11,24,64, which can potentially affect the accuracy of predicted networks"
- [discussion] detection threshold applied per cell type: "we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene"
