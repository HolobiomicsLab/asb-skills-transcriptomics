---
name: scrnaseq-data-formatting
description: Use when raw scRNA-seq data (e.g. GEO accession GSE116753 or similar)
  must be reshaped into MDIC3's exp.txt/metadata.txt input format before GRN inference
  with GNIPLR and communication scoring.
when_to_use_negative:
- Data is already in MDIC3's exp.txt/metadata.txt format and validated — no reformatting
  needed
- Analysis targets cell-cell communication methods with other input requirements (e.g.
  CellChat expects R objects/CellChatDB, not exp.txt)
- The expression file lacks cell-type labels in metadata, since MDIC3 requires existing
  cell classification labels to run
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MDIC3
  role: Provides lucky.readexp and lucky.readlabel functions to load and validate
    the formatted exp.txt and metadata.txt inputs
  repo: https://github.com/LYxiaotai/MDIC3
- name: GNIPLR
  role: Downstream consumer of the formatted expression matrix for gene regulatory
    network inference with block-wise calculation
  repo: https://github.com/zyllluck/GNIPLR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2023.100911
    title: 'MDIC3: Matrix decomposition to infer cell-cell communication'
  source_content_hash: sha256:16b7b56f3ac71e89ccff9d327fbe4d5634059f34f3268a5607954d53335e85e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LYxiaotai/MDIC3
  tools:
  - CellChat
  - CellPhoneDB
  - GNIPLR
  - MDIC3
  - Python
  - iTALK
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# scrnaseq-data-formatting

## Summary

Formats scRNA-seq expression matrices and cell-type metadata into the exact *.txt layouts MDIC3 requires (rows = genes, columns = cells; cells in metadata matching the expression header), so cell-cell communication inference can run downstream.

## When to use

Use when raw scRNA-seq data (e.g. GEO accession GSE116753 or similar) must be reshaped into MDIC3's exp.txt/metadata.txt input format before GRN inference with GNIPLR and communication scoring.

## When NOT to use

- Data is already in MDIC3's exp.txt/metadata.txt format and validated — no reformatting needed
- Analysis targets cell-cell communication methods with other input requirements (e.g. CellChat expects R objects/CellChatDB, not exp.txt)
- The expression file lacks cell-type labels in metadata, since MDIC3 requires existing cell classification labels to run

## Inputs

- Raw scRNA-seq count/expression matrix (e.g. GEO GSE116753 human islet dataset)
- Cell metadata with cell-type labels
- Target formats: exp.txt (genes x cells, no header field in first row) and metadata.txt (cell name, cell label)

## Outputs

- exp.txt: single-cell gene expression *.txt file, rows = genes, columns = cells, first row = cell names only
- metadata.txt: *.txt file, first column = cells, second column = corresponding cell-type labels
- In-memory objects from lucky.readexp (AA, gene_exp, cellname) and lucky.readlabel (labels, label_index, label_cell)

## How to apply

Download the scRNA-seq dataset (e.g. human islet data from GEO: GSE116753) and write the expression matrix as a *.txt file where each row is a gene and each column is a cell, ensuring the first row contains only cell names with no header field. Write a metadata *.txt with the first column as cell names and the second column as cell-type labels, matching the first row of the expression file exactly. Use the lucky.readexp('exp.txt') and lucky.readlabel('metadata.txt') helpers from MDIC3 to load and validate the formatted files, returning AA, gene_exp, cellname and labels, label_index, label_cell respectively. Include at least 10 cells per input, as the README recommends, and verify cell-name correspondence before running GNIPLR (step size ≤3000, e.g. step=500, with multiprocessing) and MDIC3_score.

## Related tools

- **MDIC3** (Provides lucky.readexp and lucky.readlabel functions to load and validate the formatted exp.txt and metadata.txt inputs) — https://github.com/LYxiaotai/MDIC3
- **GNIPLR** (Downstream consumer of the formatted expression matrix for gene regulatory network inference with block-wise calculation) — https://github.com/zyllluck/GNIPLR

## Examples

```
AA, gene_exp, cellname = lucky.readexp('exp.txt'); labels, label_index, label_cell = lucky.readlabel('metadata.txt')
```

## Evaluation signals

- First row of exp.txt contains only cell names with no header field; rows are genes, columns are cells
- First column of metadata.txt matches exactly with the first row of the expression file
- Input contains at least 10 cells, per the README recommendation
- lucky.readexp returns three objects (AA, gene_exp, cellname) and lucky.readlabel returns (labels, label_index, label_cell) without errors
- Formatted human islet inputs reproduce the benchmark (MDIC3 AUC 0.801 vs CellChat 0.247, iTALK 0.449, CellPhoneDB 0.417) when run through the full pipeline

## Limitations

- MDIC3 is not limited to GNIPLR — any GRN tool can be used — but the GRN adjacency matrix supplied via -grn must be a *.txt with only numerical values
- When the number of cells exceeds the number of genes there may be some missing information in the right submatrix W
- GNIPLR block calculation should use a step size of 3000 or less; large gene counts require multiprocessing tuned to the machine configuration

## Evidence

- [readme] The single-cell gene expression data must be a *.txt file, while each row represents a gene and each column represents a cell.: "The single-cell gene expression data must be a *.txt file, while each row represents a gene and each column represents a cell."
- [readme] first row must only contain cell names but not contain a header field: "Note that the first row of the single-cell gene expression data must only contains cell names but not contains a header field."
- [readme] first column of the scRNAseq metadata should match exactly with the first row of the single-cell gene expression data: "Note that the first column of the scRNAseq metadata should match exactly with the first row of the single-cell gene expression data."
- [methods] AA, gene_exp, cellname = lucky.readexp('exp.txt'): "AA, gene_exp, cellname = lucky.readexp('exp.txt')"
- [methods] labels, label_index, label_cell = lucky.readlabel('metadata.txt'): "labels, label_index, label_cell = lucky.readlabel('metadata.txt')"
- [readme] we suggest users set the '-step' parameter to a value of 3000 or less: "When the number of genes is less than 3000, we suggest users set the '-step' parameter to a value of 3000 or less."
