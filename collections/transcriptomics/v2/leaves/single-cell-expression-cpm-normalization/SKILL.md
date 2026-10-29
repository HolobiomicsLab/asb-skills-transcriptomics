---
name: single-cell-expression-cpm-normalization
description: Use when preparing a raw single-cell UMI count matrix (e.g. Tabula Muris
  or Tabula Muris Senis) as input to NATMI's ExtractEdges.py, which expects CPM-normalized
  expression profiles with a cell-annotation metafile.
when_to_use_negative:
- The expression matrix is already CPM-normalized (re-normalizing pre-scaled data
  would distort values)
- The downstream tool expects raw counts or a different normalization (e.g. library-size
  log-transformed counts for Seurat/Scanpy workflows rather than NATMI's CPM input)
- The data is proteomic or bulk abundance data normalized by a method already accounted
  for upstream
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NATMI
  role: Consumes the CPM-normalized expression matrix (--emFile) together with the
    annotation metafile (--annFile) in ExtractEdges.py to extract ligand-receptor
    edges using the connectomeDB2020 (lrc2p) pair list
  repo: https://github.com/forrest-lab/NATMI/
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-18873-z
    title: Predicting cell-to-cell communication networks using NATMI
  source_content_hash: sha256:e0fb8730a7d2fd07137fbafb752c5c5f8caf7cec58a6642a06cc95000fe24e43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/forrest-lab/NATMI
  tools:
  - NATMI
  - Python
  - connectomeDB2020
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-expression-cpm-normalization

## Summary

Normalize single-cell gene expression profiles to counts per million (CPM) by dividing each cell's profile by its total UMI (or read) count and multiplying by 1,000,000, as required by NATMI before running ExtractEdges.py to predict cell-to-cell communication networks.

## When to use

Use when preparing a raw single-cell UMI count matrix (e.g. Tabula Muris or Tabula Muris Senis) as input to NATMI's ExtractEdges.py, which expects CPM-normalized expression profiles with a cell-annotation metafile.

## When NOT to use

- The expression matrix is already CPM-normalized (re-normalizing pre-scaled data would distort values)
- The downstream tool expects raw counts or a different normalization (e.g. library-size log-transformed counts for Seurat/Scanpy workflows rather than NATMI's CPM input)
- The data is proteomic or bulk abundance data normalized by a method already accounted for upstream

## Inputs

- Raw single-cell UMI count matrix (genes x cells) in text/CSV format
- Cell-type annotation metafile mapping each cell barcode to a cell-type label
- Optionally: pre-normalized CPM expression matrices (e.g. 3m.upm.em.csv, 18m.upm.em.csv from the figshare deposit)

## Outputs

- CPM-normalized expression matrix (e.g. .upm.em.csv) where each cell's values sum to 1,000,000
- Normalized matrix ready as --emFile input to NATMI ExtractEdges.py

## How to apply

For each cell, sum the total number of unique molecular identifiers (or reads if the data does not use UMIs), divide every gene's raw count by that total, and multiply by 1,000,000 to rescale to CPM. Apply this normalization separately to every expression profile before combining with the cell-type annotation metafile that maps each cell barcode to its cell type. NATMI's downstream detection threshold (a gene counts as expressed if detected in more than 20% of cells of a cell type) and the 10 CPM expression threshold operate on the CPM-scaled values, so skipping or altering this rescaling changes which ligand-receptor edges survive. The Tabula Muris Senis mammary gland workflow uses pre-normalized 3- and 18-month matrices from the figshare deposit that are already counts-per-million-normalized, so verify whether the input is raw counts (normalize yourself) or already CPM (use as-is).

## Related tools

- **NATMI** (Consumes the CPM-normalized expression matrix (--emFile) together with the annotation metafile (--annFile) in ExtractEdges.py to extract ligand-receptor edges using the connectomeDB2020 (lrc2p) pair list) — https://github.com/forrest-lab/NATMI/

## Examples

```
python ExtractEdges.py --species mouse --emFile 3m.upm.em.csv --annFile 3m.ann.csv --interDB lrc2p --coreNum 4 --out 3m.mg
```

## Evaluation signals

- Column-wise (per-cell) sums of the normalized matrix each equal 1,000,000 (up to floating-point tolerance)
- All values in the normalized matrix are non-negative and no gene exceeds 1,000,000
- Downstream ExtractEdges.py with --interDB lrc2p and detection threshold 0.2 reproduces reported edge counts (e.g. 126,738 edges for the Skelly cardiac dataset; 2045/1247/340 differential edges for 3- vs 18-month mammary gland)
- Tabula Muris self-signalling validation: median ~54 co-detected pairs per cell type and 719/2293 (~31%) co-detected connectomeDB2020 pairs

## Limitations

- Normalization is by total UMI count only; predictions are based on mRNA expression rather than protein concentration and are subject to dropouts, which can affect edge detection
- The 10 CPM expression threshold and 20% detection threshold interact with CPM scaling, so results are sensitive to the normalization being applied consistently across conditions being compared

## Evidence

- [discussion] CPM normalization of single-cell profiles before edge extraction: "normalised by total number of unique molecular identifiers (or reads if the data does not use UMIs) and then rescaled by multiplying by 1,000,000"
- [methods] Methods description of the per-cell normalization procedure: "normalized each expression profile by dividing it by the total number of unique molecular identifiers and multiplying by 1,000,000"
- [discussion] Detection threshold applied to CPM-normalized data: "we consider a gene as expressed in a given cell type if more than 20% of cells of the cell type had non-zero read counts for that gene"
- [readme] Pre-normalized inputs in the Tabula Muris Senis workflow: [evidence span withheld — anchor status: not_found]
- [discussion] mRNA-based prediction caveat: "the predictions are necessarily based on the mRNA expression levels rather than on protein concentration and the data is subject to dropouts"
