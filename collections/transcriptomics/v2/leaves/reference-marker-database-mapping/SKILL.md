---
name: reference-marker-database-mapping
description: Use when a trained cell-annotation model has produced ranked per-class
  feature/biomarker lists and you need to quantify how many of the top-K markers are
  literature-validated, or compare marker recall across tools.
when_to_use_negative:
- No reference marker set exists for the target classes — the README notes gene recall
  curve was skipped for COVID-19/normal conditions because 'there are many possibilities
  of genes in the case of normal conditions'.
- A comparison tool does not expose per-class feature importances (scBalance, scvi,
  devCellPy, CellPLM cannot produce ranked top features), making recall comparison
  impossible.
- Goal is differential gene expression (DGE) rather than validated marker recall —
  use DgePseudoBulk/DgeLMEM instead.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scaLR
  role: Provides the feature-selector (ClasswiseAbs/AbsMean), LinearScorer, and GeneRecallCurve
    downstream-analysis modules used to rank biomarkers and sweep them against reference
    markers
  repo: https://github.com/infocusp/scaLR
- name: CellMarker2.0
  role: Curated reference database of cell-specific biomarkers from published scRNA-seq
    studies, used as the recall ground truth
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:f2c9188458c2cf861cdab4fad468e5a5fafd13789e30f2d1b4913f1856ec2aa0
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# reference-marker-database-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps ranked cell-type biomarkers predicted by a trained model (e.g. scaLR) onto a curated reference marker database such as CellMarker2.0, computing per-cell-type overlap (recall) counts and gene recall curves to validate biomarker discovery.

## When to use

Use when a trained cell-annotation model has produced ranked per-class feature/biomarker lists and you need to quantify how many of the top-K markers are literature-validated, or compare marker recall across tools.

## When NOT to use

- No reference marker set exists for the target classes — the README notes gene recall curve was skipped for COVID-19/normal conditions because 'there are many possibilities of genes in the case of normal conditions'.
- A comparison tool does not expose per-class feature importances (scBalance, scvi, devCellPy, CellPLM cannot produce ranked top features), making recall comparison impossible.
- Goal is differential gene expression (DGE) rather than validated marker recall — use DgePseudoBulk/DgeLMEM instead.

## Inputs

- Trained scaLR model checkpoint (from final model training on top-K features)
- anndata .h5ad dataset with cell_type target column (e.g. PBMCs-SLE, PBMCs-C19-F), adata.X log1p-normalized 0-10
- CellMarker2.0 reference gene CSV of cell-specific biomarkers (reference_genes_path)
- Ranked top-100 biomarker lists from comparison tools (e.g. CellTypist-Logistic)

## Outputs

- Per-cell-type overlap/recall counts of predicted biomarkers against CellMarker2.0 references
- Gene recall curves (plots) per cell type and per tool
- Lists of validated overlapping markers per cell type

## How to apply

Extract per-class feature importance from the trained scaLR model using the scalr.feature.selector modules (ClasswiseAbs or AbsMean) with a LinearScorer scoring config, ranking the top-100 biomarkers per cell type for scaLR and for each comparison tool (e.g. CellTypist-Logistic). Load the CellMarker2.0 reference list of cell-specific biomarkers identified from published scRNA-seq studies as a CSV (reference_genes_path) with columns mapping genes to cell types. Run the GeneRecallCurve downstream analysis, sweeping the ranked biomarker list (top_K, e.g. 300) against the reference markers, and record the raw overlap counts per cell type per tool. Validate by checking that recall counts reproduce the reported values (e.g. T-cell 21 vs 19, megakaryocyte 10 vs 5 of 39) and that recall curves preserve the relative ranking between scaLR and the comparator.

## Related tools

- **scaLR** (Provides the feature-selector (ClasswiseAbs/AbsMean), LinearScorer, and GeneRecallCurve downstream-analysis modules used to rank biomarkers and sweep them against reference markers) — https://github.com/infocusp/scaLR
- **CellMarker2.0** (Curated reference database of cell-specific biomarkers from published scRNA-seq studies, used as the recall ground truth)

## Examples

```
analysis.test_samples_downstream_analysis: [{name: GeneRecallCurve, params: {reference_genes_path: 'scaLR/tutorials/pipeline/grc_reference_gene.csv', top_K: 300, features_selector: {name: ClasswiseAbs, params: {}}}}]
```

## Evaluation signals

- Reproduced recall counts match the paper's reported values: 21 vs 19 T-cell markers against 223 reported T-cell biomarkers, and 10 vs 5 megakaryocyte markers against 39 reported biomarkers
- Gene recall curves show the same relative ranking between scaLR and CellTypist-Logistic across PBMCs-SLE and PBMCs-C19-F datasets
- reference_genes_path CSV loads and contains reference genes matching the cell types of the trained model's target column
- Overlap counts are computed on identical top-100 ranked lists for both tools (same classwise_abs/abs_mean selector settings)
- Every recalled marker appears in both the model's ranked list and the CellMarker2.0 reference set (set-intersection invariant)

## Limitations

- Feature importance cannot be extracted from several comparison tools (scBalance, scvi, devCellPy, CellPLM), limiting comparability of biomarker ranking
- Input format dependence: pipeline expects .h5ad anndata with log1p-normalized values in range 0-10; improper preprocessing degrades results
- Comparison tool versions are not stated and custom comparison scripts live in a separate scaLR comparison GitHub repository

## Evidence

- [readme] Gene recall curves as downstream output: [evidence span withheld — anchor status: not_found]
- [readme] GeneRecallCurve configuration in scaLR pipeline: [evidence span withheld — anchor status: external_doc]
