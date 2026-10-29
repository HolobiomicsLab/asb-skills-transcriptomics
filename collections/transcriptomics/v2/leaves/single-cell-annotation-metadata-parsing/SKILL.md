---
name: single-cell-annotation-metadata-parsing
description: Use when a single-cell gene expression matrix must be paired with a cell-label
  metafile so that expression can be aggregated per cell type prior to cell-to-cell
  communication analysis.
when_to_use_negative:
- Input is already aggregated per cell type (e.g. bulk expression profiles or FANTOM5-style
  samples) — no cell barcode-to-label mapping is needed
- The expression data are not accompanied by any cell-type annotation metafile, so
  per-cell-type grouping is impossible
- The goal is spatial transcriptomics analysis, which NATMI has not been specifically
  designed for
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NATMI
  role: Provides ExtractEdges.py, which consumes the expression matrix and annotation
    file (--emFile/--annFile) to build the cell-type-by-gene profiles used for edge
    weights
  repo: https://github.com/forrest-lab/NATMI/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:18b73f7b11ae2fdf3b2ecb862154fe6ab4c798c009c68bf06dfe6603182adb50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-cell-annotation-metadata-parsing

## Summary

Parses a single-cell expression matrix (e.g. toy.sc.em.txt) together with its cell-label metafile (e.g. toy.sc.ann.txt) to map each cell barcode to its cell-type annotation and aggregate expression per cell type, as required before NATMI's ExtractEdges.py can compute ligand–receptor edge weights.

## When to use

Use when a single-cell gene expression matrix must be paired with a cell-label metafile so that expression can be aggregated per cell type prior to cell-to-cell communication analysis.

## When NOT to use

- Input is already aggregated per cell type (e.g. bulk expression profiles or FANTOM5-style samples) — no cell barcode-to-label mapping is needed
- The expression data are not accompanied by any cell-type annotation metafile, so per-cell-type grouping is impossible
- The goal is spatial transcriptomics analysis, which NATMI has not been specifically designed for

## Inputs

- Normalised single-cell gene expression matrix (e.g. toy.sc.em.txt, CPM-normalised, official gene symbols)
- Cell labels metafile mapping each cell barcode to a cell-type annotation (e.g. toy.sc.ann.txt)

## Outputs

- Per-cell-type aggregated expression profiles keyed to cell-type annotations
- Cell-type-restricted mean expression and detection rates per gene, ready for ExtractEdges.py edge-weight computation

## How to apply

Load the normalised single-cell expression matrix (counts per million, obtained by dividing each profile by total UMIs/reads and multiplying by 1,000,000) with pandas, then load the cell-label metafile that describes the mapping between each cell barcode and a cell-type label. Join the two on cell barcode so every expression column/row carries its cell-type annotation, and group expression values by cell type to produce per-cell-type expression profiles. NATMI expects official gene symbols (HGNC/MGI; HGNC ID and Ensembl ID are additionally supported for human and mouse) and, for non-human species such as mouse, maps gene symbols onto connectomeDB2020 human ligand–receptor pairs via HomoloGene (--species mouse). After aggregation, apply the detection-rate filter — a gene is considered expressed in a cell type only if detected in more than 20% of that cell type's cells (default detectionThreshold 0.2) — before computing mean-expression, specificity, and total-expression edge weights.

## Related tools

- **NATMI** (Provides ExtractEdges.py, which consumes the expression matrix and annotation file (--emFile/--annFile) to build the cell-type-by-gene profiles used for edge weights) — https://github.com/forrest-lab/NATMI/

## Examples

```
python ExtractEdges.py --emFile toy.sc.em.txt --annFile toy.sc.ann.txt --species mouse --out outfolder
```

## Evaluation signals

- Every cell barcode in the expression matrix has a non-null cell-type label after joining with the metafile
- Aggregated per-cell-type expression values are non-negative and consistent with CPM normalisation (each cell profile rescaled by ×1,000,000)
- Downstream ExtractEdges.py runs without errors using the parsed matrix as --emFile and the parsed metafile as --annFile
- Genes failing the >20% detection-rate filter are excluded from expressed-ligand/receptor calls (detectionThreshold 0.2)
- Mean and total edge weights are non-negative and all specificity weights fall within 0–1

## Limitations

- Predictions rely on the original authors' cell annotations, which can potentially affect the accuracy of predicted networks
- Expression data are subject to dropouts and reflect mRNA levels rather than protein concentration
- Specificity weights are local/dataset-specific, computed only from the input dataset, not global across the Human Cell Atlas

## Evidence

- [discussion] normalised by total number of unique molecular identifiers (or reads if the data does not use UMIs) and then rescaled by multiplying by 1,000,000: "normalised by total number of unique molecular identifiers (or reads if the data does not use UMIs) and then rescaled by multiplying by 1,000,000"
- [discussion] we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene: "we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene"
- [readme] the metafile describing mapping between each cell and a cell-type label across the whole dataset: [evidence span withheld — anchor status: not_found]
- [discussion] 'python ExtractEdges.py --emFile expression.matrix.txt --annFile annotation.txt --out outfolder': [evidence span withheld — anchor status: not_found]
- [discussion] the specificity weights were calculated based on the input dataset and thus should be considered local or dataset specific: "the specificity weights were calculated based on the input dataset and thus should be considered local or dataset specific"
