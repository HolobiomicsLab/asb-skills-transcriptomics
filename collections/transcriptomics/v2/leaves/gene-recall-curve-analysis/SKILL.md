---
name: gene-recall-curve-analysis
description: Use when you have per-cell-type ranked biomarker lists (e.g. top-100
  from scaLR ClasswiseAbs) and a curated cell-type-specific reference marker database,
  and need to quantify/plot how many validated markers each tool recovers.
when_to_use_negative:
- No curated cell-type-specific reference gene set exists for the cell types of interest
  (e.g. disease-condition markers such as COVID-19/normal, where scaLR authors note
  'there are many possibilities of genes in the case of normal conditions')
- The comparison tool cannot export per-class feature importance (e.g. scBalance,
  scvi, devCellPy, CellPLM), so no ranked biomarker list exists to sweep
- The goal is annotation accuracy or DGE analysis rather than biomarker validation
  against reference markers
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scaLR
  role: Trains the classifier, extracts classwise feature importance (classwise_abs)
    and generates the gene recall curves via the GeneRecallCurve downstream analysis
    stage
  repo: https://github.com/infocusp/scaLR
- name: CellMarker2.0
  role: Source of curated literature-reported cell-type-specific biomarkers used as
    the reference set for computing recall
- name: Python
  role: Runtime environment (Python 3.10.20) in which the scaLR pipeline and gene_recall_curve
    tutorial are executed
  repo: https://github.com/infocusp/scaLR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:f96994966b2aa3fbbf920161883dccee1875dd60c4dafc67055f3224669eac0d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/infocusp/scaLR
  repo_url: https://github.com/infocusp/scaLR
  tools:
  - CellMarker2.0
  - Python
  - gene_recall_curve
  - scaLR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gene-recall-curve-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates a tool's top-ranked cell-type biomarkers by sweeping its ranked gene list against a curated reference set (e.g. CellMarker2.0 literature-reported markers) and plotting recall curves, as done in scaLR to benchmark against CellTypist-Logistic.

## When to use

Use when you have per-cell-type ranked biomarker lists (e.g. top-100 from scaLR ClasswiseAbs) and a curated cell-type-specific reference marker database, and need to quantify/plot how many validated markers each tool recovers.

## When NOT to use

- No curated cell-type-specific reference gene set exists for the cell types of interest (e.g. disease-condition markers such as COVID-19/normal, where scaLR authors note 'there are many possibilities of genes in the case of normal conditions')
- The comparison tool cannot export per-class feature importance (e.g. scBalance, scvi, devCellPy, CellPLM), so no ranked biomarker list exists to sweep
- The goal is annotation accuracy or DGE analysis rather than biomarker validation against reference markers

## Inputs

- Trained scaLR or CellTypist-Logistic model checkpoints (e.g. from PBMCs-SLE and PBMCs-C19-F datasets)
- anndata .h5ad scRNA-seq dataset with cell_type annotations and log1p-normalized expression in adata.X
- Reference marker CSV (e.g. grc_reference_gene.csv) of cell-type-specific biomarkers from CellMarker2.0
- scaLR pipeline config.yaml with analysis.test_samples_downstream_analysis GeneRecallCurve stage

## Outputs

- Per-cell-type recall counts of top-100 biomarkers overlapping the reference set (e.g. T-cell 21 vs 19; megakaryocyte 10 vs 5)
- Gene recall curves plotting recall vs ranked-biomarker position for each cell type and each tool
- Classwise top gene rankings (via ClasswiseAbs features_selector) consumed by the plot

## How to apply

Extract per-class feature importance from the trained models using scaLR's feature selector modules (classwise_abs / abs_mean) to rank biomarkers for each cell type. Prepare a reference-gene CSV mapping cell types to literature-reported markers (e.g. from CellMarker2.0). Configure the GeneRecallCurve downstream step in the scaLR pipeline config, setting reference_genes_path, top_K (README example: 300) and plots_per_row, with a features_selector such as ClasswiseAbs; this sweeps the ranked list and computes recall counts per cell type. For each cell type and tool, count the top-100 biomarkers that overlap the reference set (in the paper: T-cells 21 vs 19 for scaLR vs CellTypist-Logistic; megakaryocytes 10 vs 5 out of 39 references). Compare the resulting gene recall curves across tools to judge which recovers validated markers at higher recall across ranks.

## Related tools

- **scaLR** (Trains the classifier, extracts classwise feature importance (classwise_abs) and generates the gene recall curves via the GeneRecallCurve downstream analysis stage) — https://github.com/infocusp/scaLR
- **CellMarker2.0** (Source of curated literature-reported cell-type-specific biomarkers used as the reference set for computing recall)
- **Python** (Runtime environment (Python 3.10.20) in which the scaLR pipeline and gene_recall_curve tutorial are executed) — https://github.com/infocusp/scaLR

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m
```

## Evaluation signals

- Reproduced overlap counts match the paper's reported values: scaLR 21 vs CellTypist-Logistic 19 T-cell markers against 223 references, and 10 vs 5 megakaryocyte markers against 39 references
- Gene recall curves for scaLR lie above CellTypist-Logistic across the ranked sweep, reproducing the paper's relative ranking
- Recall values are monotonically non-decreasing along the ranked gene list sweep
- The reference_genes_path CSV parses correctly and every cell type in the target column has non-empty reference markers; recall ≤ number of reference markers per cell type

## Limitations

- Feature importance cannot be extracted from several comparison tools (scBalance, scvi, devCellPy, CellPLM), limiting which tools can be included in recall comparisons
- Gene recall curve analysis was not applied to clinical condition biomarkers (COVID-19/normal) because many genes are plausible for normal conditions; it needs a reliable reference gene list
- scaLR performance depends on input data format and preprocessing (anndata .h5ad with log1p-normalized values, range 0-10), which affects the ranked feature list feeding the curve

## Evidence

- [readme] Various visualizations, such as ROC curve of class annotation, feature rank plots, heatmap of top genes per class: [evidence span withheld — anchor status: external_doc]
