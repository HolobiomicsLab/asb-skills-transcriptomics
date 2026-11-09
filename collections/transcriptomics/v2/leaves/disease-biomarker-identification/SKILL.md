---
name: disease-biomarker-identification
description: Use when the input is disease-vs-control scRNA-seq data in .h5ad (anndata)
  format with a 'disease' target column, and the goal is to rank condition-specific
  biomarker genes rather than annotate cell types.
when_to_use_negative:
- The task is cell type annotation/classification on cell_type labels rather than
  disease-vs-control biomarker discovery (use the cell type classification pipeline
  instead).
- Comparison tools such as scBalance, scvi, devCellPy, or CellPLM are used as the
  model — feature importance cannot be extracted from these, so this top-K biomarker
  ranking workflow is not applicable.
- Input data is not in anndata .h5ad format or adata.X is not log1p-normalized in
  range 0-10; the pipeline expects proper preprocessing and performance may vary with
  input format.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scaLR
  role: End-to-end Python pipeline providing data processing, feature extraction (LinearScorer/SHAP
    scoring, AbsMean/ClasswiseAbs selectors), DNN training on top-K features, evaluation,
    and gene_analysis/DGE downstream analysis for biomarker discovery.
  repo: https://github.com/infocusp/scaLR
- name: Python
  role: Implementation language for the scaLR platform; tested with Python 3.10.20,
    installable via conda/pip (pyscaLR).
  repo: https://github.com/infocusp/scaLR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:e7ada505456e8791d62af6782c06cacd306f1015f6b47c17ff6624f20fc982f6
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
  - Python
  - scaLR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# disease-biomarker-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies disease-vs-control biomarker genes from single-cell RNA-seq using scaLR's low-resource DNN pipeline: train per-condition models, score feature importance (linear weights or SHAP), select top-K genes, and export ranked disease-specific gene lists (top-100 per class) for downstream functional annotation.

## When to use

Use when the input is disease-vs-control scRNA-seq data in .h5ad (anndata) format with a 'disease' target column, and the goal is to rank condition-specific biomarker genes rather than annotate cell types.

## When NOT to use

- The task is cell type annotation/classification on cell_type labels rather than disease-vs-control biomarker discovery (use the cell type classification pipeline instead).
- Comparison tools such as scBalance, scvi, devCellPy, or CellPLM are used as the model — feature importance cannot be extracted from these, so this top-K biomarker ranking workflow is not applicable.
- Input data is not in anndata .h5ad format or adata.X is not log1p-normalized in range 0-10; the pipeline expects proper preprocessing and performance may vary with input format.

## Inputs

- anndata .h5ad single-cell expression matrix (adata.X log1p-normalized counts, adata.obs with target/disease and donor_id columns, adata.var gene names index)
- config.yaml specifying target='disease', model layers, epochs, splitter, scorer, and feature selector
- CellMarker2.0-derived disease/biomarker reference gene sets for functional annotation
- Optional reference gene CSV for gene recall curves

## Outputs

- Top-100 disease-specific biomarker gene list per class (ClasswisePromoters/ClasswiseAbs, k=100)
- Feature importance scores from LinearScorer (or SHAP for multi-layer models) with rank plots and top-genes-per-class heatmaps
- Model evaluation metrics on test set: precision, recall, f1-score, accuracy, ROC-AUC curve
- DGE analysis outputs: volcano plots and associated gene lists (DgePseudoBulk / DgeLMEM)

## How to apply

Load the normal-vs-disease anndata (.h5ad, adata.X log1p-normalized 0-10), set the config target to the disease column, and run the scaLR pipeline: sample_norm/standard_scale preprocessing, stratified group splitting (stratify by donor_id), then feature selection with LinearScorer scoring and AbsMean/ClasswiseAbs or ClasswisePromoters selectors (e.g. k=5000 for selection, k=100 for gene analysis). Train a SequentialModel DNN sized to the number of classes (e.g. layers [5000, 2] for disease/normal) for 10 epochs during feature selection and up to 100 epochs with EarlyStopping (patience 3) for final training. Run gene_analysis with ClasswisePromoters (k=100) to export per-class top-100 gene lists, optionally adding DgePseudoBulk or DgeLMEM DGE analysis (design_factor 'disease', celltype_column 'cell_type', sum_column 'donor_id') with volcano plots and gene lists. Validate by evaluating precision/recall/f1/accuracy on the held-out test set and functionally annotating top-ranked genes against CellMarker2.0-derived disease reference sets (e.g. expecting the reported HBSFG vs HBPFC pattern of 7/10 vs 2/10 neurological-disorder-linked top-10 genes).

## Related tools

- **scaLR** (End-to-end Python pipeline providing data processing, feature extraction (LinearScorer/SHAP scoring, AbsMean/ClasswiseAbs selectors), DNN training on top-K features, evaluation, and gene_analysis/DGE downstream analysis for biomarker discovery.) — https://github.com/infocusp/scaLR
- **Python** (Implementation language for the scaLR platform; tested with Python 3.10.20, installable via conda/pip (pyscaLR).) — https://github.com/infocusp/scaLR

## Examples

```
python pipeline.py --config config_clinical.yaml -l -m
```

## Evaluation signals

- Trained model yields high precision/recall/f1/accuracy on the stratified held-out test set (split stratified on donor_id, e.g. split_ratio [7, 1, 2.5]).
- Top-100 gene lists are exported for each disease-vs-control class (e.g. for both HBSFG and HBPFC datasets).
- Functional annotation against CellMarker2.0-derived disease gene sets reproduces the expected enrichment pattern (e.g. 7/10 top-10 HBSFG genes vs 2/10 HBPFC genes linked to neurological disorders).
- DGE module produces volcano plots and gene lists consistent between pseudo-bulk and LMEM approaches across cell types.
- Feature scoring is stable under sensitivity analyses altering epochs and DNN layers.

## Limitations

- scaLR's memory usage is inversely proportional to execution time — reducing memory consumption may increase processing time.
- Performance may vary based on the input data format, requiring users to ensure proper data preprocessing (log1p 0-10 adata.X) for optimal results.
- Supplementary biomarker tables (S10–S14) and specific final-run hyperparameters (layers, epochs values) are not fully reported; sensitivity analyses are referenced without reported values.

## Evidence

- [readme] README clinical config: target disease, ClasswisePromoters k=100, DgePseudoBulk/DgeLMEM: [evidence span withheld — anchor status: external_doc]
- [readme] README run command: [evidence span withheld — anchor status: external_doc]
