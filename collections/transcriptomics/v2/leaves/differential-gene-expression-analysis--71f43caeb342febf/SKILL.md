---
name: differential-gene-expression-analysis
description: Use when you have a scRNA-seq count matrix (.h5ad) with cell-type and
  condition/disease annotations and need per-cell-type DEG calls between conditions
  (e.g. COVID-19 vs normal, infarct vs control), with ranked gene lists and volcano
  plots.
when_to_use_negative:
- Input lacks per-sample/donor and cell-type annotations required for pseudo-bulk
  aggregation and the mixed-effects model
- The goal is feature importance / biomarker ranking from the DNN model — use scaLR's
  linear_scorer/shap_scorer gene_analysis instead, not DGE
- Only test-set samples should be analyzed when maximum DEG sensitivity is required
  — the paper shows full-samples DGE yields substantially more DEGs
edam_operation: http://edamontology.org/operation_3731
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3307
tools:
- name: scaLR
  role: Provides the DGE module (scalr.analysis.dge_pseudobulk and scalr.analysis.dge_lmem)
    that aggregates counts per sample/cell type, fits per-gene models, and generates
    volcano plots and gene lists
  repo: https://github.com/infocusp/scaLR
- name: Python
  role: Language in which scaLR is implemented and in which the DGE pipeline and downstream
    tabulation of DEG counts are run
  repo: https://github.com/infocusp/scaLR
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:899dd4af678dbc2c2f953e5b584abbd98432debce7f8d8365f53dd2e338f2ea5
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
  edam_operation: http://edamontology.org/operation_3731
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# differential-gene-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

scaLR's differential gene expression (DGE) module identifies condition-associated genes from scRNA-seq data via two complementary approaches — pseudo-bulk aggregation per sample/cell type and per-gene linear mixed-effects models (LMEM) — producing volcano plots and associated DEG gene lists.

## When to use

Use when you have a scRNA-seq count matrix (.h5ad) with cell-type and condition/disease annotations and need per-cell-type DEG calls between conditions (e.g. COVID-19 vs normal, infarct vs control), with ranked gene lists and volcano plots.

## When NOT to use

- Input lacks per-sample/donor and cell-type annotations required for pseudo-bulk aggregation and the mixed-effects model
- The goal is feature importance / biomarker ranking from the DNN model — use scaLR's linear_scorer/shap_scorer gene_analysis instead, not DGE
- Only test-set samples should be analyzed when maximum DEG sensitivity is required — the paper shows full-samples DGE yields substantially more DEGs

## Inputs

- anndata .h5ad scRNA-seq matrix (adata.X normalized log1p counts 0-10, adata.obs with cell_type/condition/donor_id metadata)
- Optional feature-subset restriction (e.g. scaLR top-K selected features, such as top 5,000)
- DGE config: celltype_column, design_factor (e.g. 'disease'), factor_categories (e.g. ['COVID-19', 'normal']), sum_column (e.g. 'donor_id')

## Outputs

- Volcano plots per approach (pseudo-bulk and LMEM)
- DEG gene lists per cell type per approach and direction (upregulated/downregulated)
- DEG count comparison table (e.g. dge_deg_counts.csv with 41→254 pseudo-bulk up, 92→130 LMEM up at top-5,000 features)

## How to apply

Load the annotated anndata (.h5ad) and restrict to scaLR's selected features if applicable (e.g. the top 5,000 features). Run scalr.analysis.dge_pseudobulk, which aggregates single-cell counts into pseudo-bulk profiles per sample and cell type (config keys: celltype_column, design_factor, factor_categories, sum_column), and/or scalr.analysis.dge_lmem, which fits a linear mixed-effects model per gene across cell types/conditions; call DEGs at fold-change cutoff ±1.5 and P<0.05. Configure the analysis section in config.yaml (full_samples_downstream_analysis for all samples, test_samples_downstream_analysis for test-set samples only) and run via pipeline.py. Decision point: the paper showed running DGE on all samples rather than only test-set samples substantially increases detected DEGs (e.g. pseudo-bulk upregulated genes in smooth muscle myoblasts rose from 41 to 254), so prefer full-samples analysis when sample-level coverage matters. Collect the volcano plots and gene lists and tabulate per-direction (up/down) DEG counts to compare approaches.

## Related tools

- **scaLR** (Provides the DGE module (scalr.analysis.dge_pseudobulk and scalr.analysis.dge_lmem) that aggregates counts per sample/cell type, fits per-gene models, and generates volcano plots and gene lists) — https://github.com/infocusp/scaLR
- **Python** (Language in which scaLR is implemented and in which the DGE pipeline and downstream tabulation of DEG counts are run) — https://github.com/infocusp/scaLR

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m   # with analysis.full_samples_downstream_analysis: [DgePseudoBulk(celltype_column='cell_type', design_factor='disease', factor_categories=['COVID-19','normal'], sum_column='donor_id'), DgeLMEM]
```

## Evaluation signals

- DEG counts under a defined feature subset match published magnitudes, e.g. pseudo-bulk upregulated smooth-muscle-myoblast DEGs rising from 41 (test samples) to 254 (all samples) and LMEM up from 92 to 130 at the top-5,000-feature model
- Calls use the stated thresholds: fold-change cutoff ±1.5 and P<0.05 (LMEM DEGs at P<0.05)
- Both volcano plots and associated gene lists are generated per approach, as the module is expected to produce
- Gene lists are directionally split into upregulated and downregulated sets (reported e.g. 22→164 and 256→332 downregulated)
- Config resolves the design_factor categories (e.g. ['COVID-19', 'normal']) and aggregates counts by sum_column 'donor_id' without dropping cell types

## Limitations

- DgeLMEM is slow in low-CPU environments because multiprocessing is inefficient (the README notes this with only 2 CPUs in the Colab runtime)
- scaLR's performance varies with input data format, requiring users to ensure proper preprocessing (normalized log1p values in anndata .h5ad)
- DEG counts depend heavily on the sample scope (test-set vs all samples) and the feature subset (e.g. top 5,000 features), so results are not directly comparable across these settings

## Evidence

- [readme] Two DGE options in scaLR: [evidence span withheld — anchor status: external_doc]
- [readme] LMEM runtime caveat: [evidence span withheld — anchor status: external_doc]
- [full_text] Full-samples vs test-samples DEG increase: [evidence span withheld — anchor status: not_found]
