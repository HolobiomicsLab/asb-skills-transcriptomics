---
name: alzheimer-scrnaseq-dataset-analysis
description: Use when the task is to identify disease-specific (e.g., Alzheimer's
  vs normal) biomarkers from brain-region scRNA-seq datasets with scaLR's DNN-based
  pipeline and validate top-ranked genes against known neurological-disorder marker
  sets.
when_to_use_negative:
- 'The analysis goal is cell-type annotation rather than disease-vs-control biomarker
  discovery (use scaLR''s cell-type classification config with target: cell_type instead)'
- The dataset is not a two-class disease-vs-control design — DGE analysis via DgePseudoBulk/DgeLMEM
  and ClasswisePromoters selection assume distinguishable condition categories
- Comparison tool outputs lack feature-importance extraction (e.g., scBalance, scvi,
  devCellPy, CellPLM), so biomarker ranking cannot be reproduced there
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scaLR
  role: End-to-end pipeline for preprocessing, DNN training on top-K features, feature
    scoring (linear_scorer, shap_scorer), and biomarker gene ranking via eval_and_analysis_pipeline
  repo: https://github.com/infocusp/scaLR
- name: Python
  role: Runtime platform for scaLR (tested on Python 3.10.20); pipeline executed via
    pipeline.py with config.yaml
  repo: https://github.com/infocusp/scaLR
- name: CellMarker2.0
  role: Source of cell/disease-specific biomarker reference gene sets used to functionally
    annotate top-ranked Alzheimer's genes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:5f86fb11835c803dc9dbab6d0f0e3e0445e74c7c59854d35c239aa514f67afdd
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
  - scaLR
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# alzheimer-scrnaseq-dataset-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces scaLR's disease-vs-control biomarker discovery on Alzheimer's disease brain scRNA-seq datasets (human brain superior frontal gyrus, HBSFG, and prefrontal cortex, HBPFC), ranking disease-associated genes and validating the top hits against neurological-disorder gene references from CellMarker2.0.

## When to use

Use when the task is to identify disease-specific (e.g., Alzheimer's vs normal) biomarkers from brain-region scRNA-seq datasets with scaLR's DNN-based pipeline and validate top-ranked genes against known neurological-disorder marker sets.

## When NOT to use

- The analysis goal is cell-type annotation rather than disease-vs-control biomarker discovery (use scaLR's cell-type classification config with target: cell_type instead)
- The dataset is not a two-class disease-vs-control design — DGE analysis via DgePseudoBulk/DgeLMEM and ClasswisePromoters selection assume distinguishable condition categories
- Comparison tool outputs lack feature-importance extraction (e.g., scBalance, scvi, devCellPy, CellPLM), so biomarker ranking cannot be reproduced there

## Inputs

- HBSFG (human brain superior frontal gyrus) normal-vs-Alzheimer scRNA-seq dataset as .h5ad anndata (adata.X log1p-normalized 0-10, adata.obs with 'target' disease column, adata.var gene names)
- HBPFC (human brain prefrontal cortex) normal-vs-Alzheimer scRNA-seq dataset as .h5ad
- CellMarker2.0-derived neurological-disorder/biomarker reference gene sets

## Outputs

- Ranked disease-associated gene list per dataset from eval_and_analysis_pipeline
- Top-100 biomarker gene list for HBSFG and for HBPFC
- Functional annotation counts of the top-10 genes linked to neurological disorders (expected 7/10 HBSFG, 2/10 HBPFC)
- Feature importance scores from linear_scorer and shap_scorer (abs_mean / classwise_abs selections)

## How to apply

Load the HBSFG and HBPFC normal-vs-Alzheimer .h5ad datasets through scalr.data_ingestion_pipeline, then preprocess each with sample_norm and standard_scale modules and partition cells into train/test sets with the stratified group splitter. Train the scaLR DNN (two-class disease-vs-control target) per dataset and score feature importance with the linear_scorer and shap_scorer modules, selecting top-K genes via the abs_mean and classwise_abs selectors (e.g., ClasswiseAbs with k: 100 for the top-100 biomarker list). Run the eval_and_analysis_pipeline to rank disease-associated genes and export the top-100 biomarker gene list for each of HBSFG and HBPFC. Finally, functionally annotate each dataset's top-10 ranked genes against CellMarker2.0-derived neurological-disorder biomarker sets and count how many are linked to neurological disorders; correct reproduction yields 7/10 neurological-disorder-linked genes in HBSFG versus 2/10 in HBPFC.

## Related tools

- **scaLR** (End-to-end pipeline for preprocessing, DNN training on top-K features, feature scoring (linear_scorer, shap_scorer), and biomarker gene ranking via eval_and_analysis_pipeline) — https://github.com/infocusp/scaLR
- **Python** (Runtime platform for scaLR (tested on Python 3.10.20); pipeline executed via pipeline.py with config.yaml) — https://github.com/infocusp/scaLR
- **CellMarker2.0** (Source of cell/disease-specific biomarker reference gene sets used to functionally annotate top-ranked Alzheimer's genes)

## Examples

```
python pipeline.py --config /path/to/config_clinical.yaml -l -m
```

## Evaluation signals

- Top-100 biomarker gene lists are produced for both HBSFG and HBPFC datasets
- Annotation of the top-10 genes reproduces 7/10 neurological-disorder-linked genes in HBSFG versus 2/10 in HBPFC
- Model evaluation metrics (precision, recall, f1-score, accuracy) are computed on the held-out test set for each disease-vs-control class
- Selected genes derive from documented selectors (abs_mean for top-K feature selection, classwise_abs/ClasswisePromoters with k: 100 for per-class biomarkers)
- Input .h5ad files satisfy the expected schema: log1p-normalized values in range 0-10, 'target' column in obs, gene names in var

## Limitations

- Memory usage is inversely proportional to execution time; reducing memory consumption may increase processing time
- Performance varies with input data format — users must ensure proper preprocessing (e.g., log1p normalization, range 0-10) for optimal results
- Specific DNN hyperparameters (epochs, layer values used in final published runs) are not reported, only sensitivity analyses
- Feature importance cannot be extracted from several comparison tools (scBalance, scvi, devCellPy, CellPLM), limiting comparability of biomarker rankings

## Evidence

- [results] Functional analysis of the top 10 genes shows scaLR detects Alzheimer's/neurological disorder biomarkers in both brain datasets, with 7 of 10 top genes linked to neurological disorders in HBSFG versus only 2 of 10 in HBPFC.: [evidence span withheld — anchor status: not_found]
- [discussion] we have used the HBSFG and prefrontal cortex (HBPFC) (Table 1) datasets of normal and Alzheimer patients: "we have used the HBSFG and prefrontal cortex (HBPFC) (Table 1) datasets of normal and Alzheimer patients"
- [discussion] cell-specific biomarkers identified by various studies based on scRNA-seq and downloaded from CellMarker2.0 [75]: "cell-specific biomarkers identified by various studies based on scRNA-seq and downloaded from CellMarker2.0"
- [discussion] The trained model is then evaluated on the test set using metrics such as precision, recall, f1-score, and accuracy scores: "The trained model is then evaluated on the test set using metrics such as precision, recall, f1-score, and accuracy scores"
- [readme] adata.X: contains normalized gene counts/expression values (log1p normalization with range 0-10 expected): [evidence span withheld — anchor status: external_doc]
- [abstract] At the end of the training of all subsets, the top-K features are selected based on their importance.: "At the end of the training of all subsets, the top-K features are selected based on their importance"
