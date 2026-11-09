---
name: biomarker-comparison-across-annotation-tools
description: Use when two trained cell-annotation models (e.g. scaLR DNN vs CellTypist-Logistic)
  must be compared on how many literature-validated cell-type biomarkers appear in
  their top-K ranked feature lists.
when_to_use_negative:
- When the comparison tool cannot export feature importance (e.g. scBalance, scvi,
  devCellPy, CellPLM) — no top-K biomarker ranking can be extracted
- When the question is differential expression between conditions rather than validated
  cell-type biomarker recall — use the DGE Pseudo bulk/LMEM modules instead
- When no curated reference markers exist for the cell types of interest (e.g. condition-specific
  genes such as COVID-19/normal), recall against CellMarker2.0 is not well defined
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scaLR
  role: Trains the DNN classifier and provides classwise_abs/abs_mean feature selectors,
    LinearScorer, and the GeneRecallCurve downstream module for the comparison
  repo: github.com/infocusp/scaLR
- name: CellTypist
  role: Logistic-regression comparator whose top-100 biomarkers are extracted and
    scored against the same CellMarker2.0 references
- name: CellMarker2.0
  role: Curated reference database of cell-specific biomarkers from published scRNA-seq
    studies used as the recall ground truth
- name: Python
  role: Implementation language for scaLR and the comparison scripts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:8ca1a11b179270e97381e5180809e497588c366bf63b4bfa140f43b104890c6a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/infocusp/scaLR
  tools:
  - CellMarker2.0
  - CellTypist
  - Python
  - scaLR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# biomarker-comparison-across-annotation-tools

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks annotation/biomarker-discovery platforms by extracting each tool's per-class top-K gene rankings (e.g. top-100) and scoring their overlap with curated literature marker references (CellMarker2.0), summarized as per-cell-type recall counts and gene recall curves.

## When to use

Use when two trained cell-annotation models (e.g. scaLR DNN vs CellTypist-Logistic) must be compared on how many literature-validated cell-type biomarkers appear in their top-K ranked feature lists.

## When NOT to use

- When the comparison tool cannot export feature importance (e.g. scBalance, scvi, devCellPy, CellPLM) — no top-K biomarker ranking can be extracted
- When the question is differential expression between conditions rather than validated cell-type biomarker recall — use the DGE Pseudo bulk/LMEM modules instead
- When no curated reference markers exist for the cell types of interest (e.g. condition-specific genes such as COVID-19/normal), recall against CellMarker2.0 is not well defined

## Inputs

- Trained scaLR and CellTypist-Logistic model checkpoints on the same datasets (PBMCs-SLE, PBMCs-C19-F)
- anndata .h5ad expression matrices with cell_type labels in adata.obs and genes in adata.var
- CellMarker2.0 reference marker list as CSV (e.g. scaLR/tutorials/pipeline/grc_reference_gene.csv)

## Outputs

- Per-cell-type recall counts of top-100 biomarkers overlapping CellMarker2.0 references (e.g. T-cell 21 vs 19; megakaryocyte 10 vs 5)
- Gene recall curves per cell type comparing scaLR and CellTypist-Logistic
- Ranked per-class biomarker lists via classwise_abs selection

## How to apply

Train each tool on the same scRNA-seq datasets (here PBMCs-SLE and PBMCs-C19-F, .h5ad/anndata with a cell_type target), then extract per-class feature importance from the trained models using the scaLR selector modules scalr.feature.selector.classwise_abs or abs_mean and the LinearScorer to rank the top-100 biomarkers per cell type. Load the CellMarker2.0 curated reference list of cell-specific biomarkers identified from published scRNA-seq studies (CSV, e.g. grc_reference_gene.csv). For each cell type and tool, count how many of the top-100 ranked genes overlap the CellMarker2.0 reference set to obtain recall counts, and sweep the ranked list to generate gene recall curves (scalr GeneRecallCurve, with top_K set to the sweep depth and plots_per_row controlling layout). Judge success by whether the reproduced recall counts match published values and whether the recall curves preserve the relative ranking between tools.

## Related tools

- **scaLR** (Trains the DNN classifier and provides classwise_abs/abs_mean feature selectors, LinearScorer, and the GeneRecallCurve downstream module for the comparison) — github.com/infocusp/scaLR
- **CellTypist** (Logistic-regression comparator whose top-100 biomarkers are extracted and scored against the same CellMarker2.0 references)
- **CellMarker2.0** (Curated reference database of cell-specific biomarkers from published scRNA-seq studies used as the recall ground truth)
- **Python** (Implementation language for scaLR and the comparison scripts)

## Examples

```
analysis.test_samples_downstream_analysis: - name: GeneRecallCurve; params: reference_genes_path: 'scaLR/tutorials/pipeline/grc_reference_gene.csv', top_K: 300, features_selector: ClasswiseAbs
```

## Evaluation signals

- Reproduced T-cell recall counts equal 21 (scaLR) vs 19 (CellTypist) against 223 reported T-cell biomarkers
- Reproduced megakaryocyte recall counts equal 10 (scaLR) vs 5 (CellTypist) against 39 reported biomarkers
- Gene recall curves show the same relative ranking between scaLR and CellTypist-Logistic across both datasets
- Each tool's top-100 list contains exactly 100 ranked genes per cell type after classwise_abs selection
- Reference-gene CSV loads and matches gene names in adata.var before overlap counting

## Limitations

- Feature importance cannot be extracted from several comparison tools (scBalance, scvi, devCellPy, CellPLM), limiting comparability of biomarker ranking
- Benchmark result tables and supplementary biomarker tables (Tables 2–4, S10–S14) were referenced but not present in the extracted text, so exact values may need verification against the paper SI
- Comparison tool versions are not stated; scaLR's performance may vary with input data format, requiring proper preprocessing (log1p normalization, range 0-10 in adata.X)

## Evidence

- [methods] classwise_abs module reference: "classwise\_abs
--------------

.. automodule:: scalr.feature.selector.classwise\_abs"
- [readme] gene recall curves as downstream output: [evidence span withheld — anchor status: external_doc]
- [discussion] CellMarker2.0 as reference source: "cell-specific biomarkers identified by various studies based on scRNA-seq and downloaded from CellMarker2.0 [75]"
- [other] reproduced recall counts: [evidence span withheld — anchor status: not_found]
- [discussion] feature-extraction limitation in comparisons: "We cannot extract the features, or there is no explanation about how to get the top features for each cell type from the generated models, scBalance, scvi, devcellpy, and cellPLM"
