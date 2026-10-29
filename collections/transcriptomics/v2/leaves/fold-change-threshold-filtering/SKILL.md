---
name: fold-change-threshold-filtering
description: Use when comparing inferred cell abundance matrices between two sample
  groups (e.g., LUAD vs LUSC tumor samples) and you need a list of significantly and
  substantially shifted cell types meeting explicit FC and q-value cutoffs.
when_to_use_negative:
- Input is raw bulk RNA-seq expression rather than an inferred per-sample cell abundance
  matrix — run DCNet deconvolution first (with log2(x+1) transformation) before filtering
  abundances
- You need survival clustering or risk-score modules rather than a two-group differential
  abundance list — use the consensus clustering / risk-model workflow instead
- Groups are not binary subtypes (e.g., single cohort without labels), making per-cell-type
  fold change between LUAD and LUSC undefined
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DCNet
  role: Produces the cell landscape (per-sample cell-type abundances) that is the
    substrate for the fold-change and q-value filtering
  repo: https://github.com/xindd/DCNet
- name: Python
  role: Used to compute per-cell-type fold changes, run the statistical tests with
    multiple-testing correction, and write the filtered differential_cells.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:b08dd57c369954b5ac7be9f602a0e6cf365ad603abfa366341905920737c08ba
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# fold-change-threshold-filtering

## Summary

Filters inferred cell-type abundances (e.g., DCNet output for TCGA LUAD vs LUSC) by per-cell-type fold change between subtypes and multiple-testing-adjusted q-values, retaining directional differential cell types for downstream tables or classifiers.

## When to use

Use when comparing inferred cell abundance matrices between two sample groups (e.g., LUAD vs LUSC tumor samples) and you need a list of significantly and substantially shifted cell types meeting explicit FC and q-value cutoffs.

## When NOT to use

- Input is raw bulk RNA-seq expression rather than an inferred per-sample cell abundance matrix — run DCNet deconvolution first (with log2(x+1) transformation) before filtering abundances
- You need survival clustering or risk-score modules rather than a two-group differential abundance list — use the consensus clustering / risk-model workflow instead
- Groups are not binary subtypes (e.g., single cohort without labels), making per-cell-type fold change between LUAD and LUSC undefined

## Inputs

- DCNet-inferred cell-type abundance matrix (samples x cell types, >400 cell types) for TCGA LUAD and LUSC tumor samples
- Sample subtype labels mapping each sample to LUAD or LUSC

## Outputs

- Filtered differential cell table with columns: cell type, fold change, q-value (e.g., differential_cells.csv)
- Directional list of differential cell types (15 in LUSC, 18 in LUAD, 12 common in the reported finding)

## How to apply

Load the DCNet-inferred cell abundance matrix for the two subtype groups along with sample labels, then compute per-cell-type fold change in abundance between groups (e.g., LUAD vs LUSC) in Python. Test abundance differences per cell type between groups and adjust p-values for multiple testing to obtain q-values. Apply the dual cutoff of fold change >2 or <0.5 with q<0.01, keeping directionality (higher in LUSC: Neuroepithelial, Myofibroblast, Basal; higher in LUAD: Granulocyte, Alveolar, Leukocyte, Secretory, MDSC, Ciliated). Write the filtered table with cell type, fold change, and q-value columns (e.g., to differential_cells.csv). In this study, the retained differential cell types then served as features for an SVM lung-cancer subtype classifier that reached 0.975 precision.

## Related tools

- **DCNet** (Produces the cell landscape (per-sample cell-type abundances) that is the substrate for the fold-change and q-value filtering) — https://github.com/xindd/DCNet
- **Python** (Used to compute per-cell-type fold changes, run the statistical tests with multiple-testing correction, and write the filtered differential_cells.csv)

## Examples

```
import pandas as pd; d = pd.read_csv('abundance.csv', index_col=0); fc = d[d.subtype=='LUSC'].mean()/d[d.subtype=='LUAD'].mean(); sig = diff[(diff.fc>2)|(diff.fc<0.5)][diff.q<0.01]; sig.to_csv('differential_cells.csv')
```

## Evaluation signals

- Every retained cell type satisfies fold change >2 or <0.5 AND q<0.01; no cell type outside these cutoffs appears in the output
- Output table contains exactly three columns — cell type, fold change, q-value — plus directionality annotations
- Directionality is consistent with the reported signature: Neuroepithelial, Myofibroblast, Basal higher in LUSC; Granulocyte, Alveolar, Leukocyte, Secretory, MDSC, Ciliated higher in LUAD
- q-values are the adjusted (multiple-testing-corrected) p-values, not raw p-values, for all tested cell types
- Downstream check: using the retained differential cell types as SVM features yields subtype-prediction precision near the reported 0.975

## Limitations

- Cutoffs (FC >2 or <0.5, q<0.01) are study-specific for LUAD vs LUSC on DCNet-inferred abundances and may need recalibration for other cohorts or cancer types
- DCNet was trained on CPU with no random seed or reproducibility controls reported, so inferred abundances (and hence fold changes) may vary across runs
- Only source code is deposited; trained model weights availability and fine-tuning procedure are not fully specified, affecting reproducibility of the abundance inputs

## Evidence

- [full_text] Filter cell types meeting fold change >2 or <0.5 with q<0.01, retaining directionality (higher in LUSC: Neuroepithelial, Myofibroblast, Basal; higher in LUAD: Granulocyte, Alveolar, Leukocyte, Secretory, MDSC, Ciliated).: [evidence span withheld — anchor status: not_found]
- [full_text] Perform a statistical test of abundance differences per cell type between the two subtypes and adjust p-values for multiple testing to obtain q-values.: [evidence span withheld — anchor status: not_found]
