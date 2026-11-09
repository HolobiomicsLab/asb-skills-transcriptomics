---
name: tcga-lung-cancer-subtype-comparison
description: Use when bulk RNA-seq samples from two cancer subtypes (e.g., TCGA LUAD
  vs LUSC) have DCNet-inferred cell abundance matrices and the goal is to identify
  cell types significantly enriched or depleted between subtypes.
when_to_use_negative:
- Samples lack subtype labels or span more than two groups — the workflow is a two-group
  (LUAD vs LUSC) contrast with directional fold changes, not a multi-class or unsupervised
  clustering analysis
- Abundances come from a different deconvolution method (CIBERSORTx, EPIC, MCP-counter,
  quanTIseq, TIMER, xCell) with a non-overlapping cell-type panel — cell-type names
  and fold-change interpretation would not match the DCNet 434-cell landscape
- The goal is per-gene differential expression rather than deconvolved cell-type abundance
  differences
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: DCNet
  role: Produces the cell landscape (cell abundance per sample) from bulk RNA-seq
    that is contrasted between LUAD and LUSC
  repo: https://github.com/xindd/DCNet
- name: Python
  role: Computes per-cell-type fold changes, statistical tests, q-value adjustment,
    and writes differential_cells.csv
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Framework in which the DCNet model used to generate the abundance matrix was
    coded and trained
  repo: https://github.com/xindd/DCNet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:7f4194c99039f5ad6bc9c33700e0b5999b384c3651c7941d067b5ad7c6924e78
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  - mxnet
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# tcga-lung-cancer-subtype-comparison

## Summary

Differentially abundant cell-type analysis between TCGA lung adenocarcinoma (LUAD) and lung squamous cell carcinoma (LUSC) using DCNet-inferred cell landscapes, testing per-cell-type abundance differences with fold change and multiple-testing correction.

## When to use

Use when bulk RNA-seq samples from two cancer subtypes (e.g., TCGA LUAD vs LUSC) have DCNet-inferred cell abundance matrices and the goal is to identify cell types significantly enriched or depleted between subtypes.

## When NOT to use

- Samples lack subtype labels or span more than two groups — the workflow is a two-group (LUAD vs LUSC) contrast with directional fold changes, not a multi-class or unsupervised clustering analysis
- Abundances come from a different deconvolution method (CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, xCell) with a non-overlapping cell-type panel — cell-type names and fold-change interpretation would not match the DCNet 434-cell landscape
- The goal is per-gene differential expression rather than deconvolved cell-type abundance differences

## Inputs

- DCNet-inferred cell abundance matrix for TCGA LUAD and LUSC tumor samples (rows = cell types, columns = samples)
- Sample subtype labels distinguishing LUAD from LUSC
- Bulk RNA-seq expression matrix from GDC (rows = human gene symbols, columns = samples), log2(x+1) transformed, used upstream by DCNet

## Outputs

- differential_cells.csv with cell type, fold change, and q-value columns and directionality (higher in LUAD vs LUSC)
- List of significantly differential cell types usable as SVM classifier features for lung cancer subtype prediction

## How to apply

Load the DCNet-inferred cell abundance matrix (one column per sample, one row per cell type) for TCGA LUAD and LUSC tumor samples together with subtype labels; DCNet landscapes are derived from log2(x+1)-transformed bulk RNA-seq expression matrices whose rows are human gene symbols. Compute per-cell-type fold change in abundance between LUAD and LUSC in Python, then run a per-cell-type statistical test of abundance differences with p-values adjusted for multiple testing to q-values. Retain cell types passing fold change >2 or <0.5 with q<0.01, preserving directionality — the paper found 15 differentially expressed cells in LUSC and 18 in LUAD with 12 common types (Neuroepithelial, Myofibroblast and Basal higher in LUSC; Granulocyte, Alveolar, Leukocyte, Secretory, MDSC and Ciliated higher in LUAD). These differential cell types were subsequently used as features for an SVM subtype classifier that reached precision 0.975. Write the filtered table (cell type, fold change, q-value) to differential_cells.csv.

## Related tools

- **DCNet** (Produces the cell landscape (cell abundance per sample) from bulk RNA-seq that is contrasted between LUAD and LUSC) — https://github.com/xindd/DCNet
- **Python** (Computes per-cell-type fold changes, statistical tests, q-value adjustment, and writes differential_cells.csv) — https://github.com/xindd/DCNet
- **mxnet** (Framework in which the DCNet model used to generate the abundance matrix was coded and trained) — https://github.com/xindd/DCNet

## Examples

```
python differential_cells.py --abundance dcnet_luad_lusc.csv --labels subtypes.csv --fc-hi 2 --fc-lo 0.5 --q 0.01 --out differential_cells.csv
```

## Evaluation signals

- Filtered cell types all satisfy fold change >2 or <0.5 with q<0.01, and each row carries directionality consistent with the group means
- Directionality of top hits matches the published signature: Neuroepithelial, Myofibroblast, Basal higher in LUSC; Granulocyte, Alveolar, Leukocyte, Secretory, MDSC, Ciliated higher in LUAD
- differential_cells.csv contains the required columns (cell type, fold change, q-value) and no row with missing q-value
- The 12 common differential cell types (overlap of LUSC- and LUAD-significant sets) are reconciled against the 15 (LUSC) and 18 (LUAD) counts reported in the finding
- Upstream abundance matrix derives from a log2(x+1)-transformed gene-by-sample matrix with rows as human gene symbols, as DCNet requires

## Limitations

- The paper does not name the specific statistical test or multiple-testing correction used for the LUAD vs LUSC abundance comparison, so exact reproduction of q-values is not guaranteed
- No random seed or reproducibility controls are reported for DCNet training, and trained model weights are not stated as available, only source code; fine-tuning of the dropout-0.1 model on user data is not specified
- Cell-type inference is bounded by the DCNet panel of 434 cell types embedded from CellMarker marker genes; cell types whose marker genes are undetected in TCGA were discarded upstream

## Evidence

- [results] Neuroepithelial, Myofibroblast and Basal cell are higher in LUSC, while Granulocyte, Alveolar, Leukocyte, Secretory, Myeloid-derived suppressor and Ciliated cells are higher in LUAD.: [evidence span withheld — anchor status: not_found]
