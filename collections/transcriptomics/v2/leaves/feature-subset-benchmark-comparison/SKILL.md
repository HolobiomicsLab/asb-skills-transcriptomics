---
name: feature-subset-benchmark-comparison
description: Use when an scaLR-trained model on top-K features must be evaluated for
  DGE sensitivity, specifically when asked whether running dge_pseudobulk/dge_lmem
  on all samples rather than test samples changes DEG counts.
when_to_use_negative:
- The goal is cell-type annotation accuracy benchmarking (precision/recall/f1) rather
  than DGE sensitivity to sample scope
- The input dataset lacks per-sample donor grouping (sum_column/donor_id) or condition
  labels required for pseudo-bulk aggregation and LMEM
- Features have not yet been selected and you need the full genome-scale training
  pipeline instead of a downstream comparison
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scaLR
  role: Platform whose DGE module (dge_pseudobulk, dge_lmem) generates the volcano
    plots and gene lists compared across test-sample and full-sample runs
  repo: https://github.com/infocusp/scaLR
- name: Python
  role: Execution environment for scaLR (tested in Python 3.10.20) used to run the
    pipeline and tabulate DEG counts
  repo: https://github.com/infocusp/scaLR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:b8b6cc9c2ad9624c2720e4930a43122231ba2c101ebf133e7835207b3134aaf8
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# feature-subset-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark differential gene expression (DGE) results from scaLR computed on all samples versus only test-set samples, using the top-K (e.g. 5,000) selected features, to quantify how sample scope inflates DEG detection. The comparison relies on scaLR's two DGE approaches — pseudo-bulk aggregation and linear mixed-effects modeling (LMEM) — with volcano plots and gene lists as outputs.

## When to use

Use when an scaLR-trained model on top-K features must be evaluated for DGE sensitivity, specifically when asked whether running dge_pseudobulk/dge_lmem on all samples rather than test samples changes DEG counts.

## When NOT to use

- The goal is cell-type annotation accuracy benchmarking (precision/recall/f1) rather than DGE sensitivity to sample scope
- The input dataset lacks per-sample donor grouping (sum_column/donor_id) or condition labels required for pseudo-bulk aggregation and LMEM
- Features have not yet been selected and you need the full genome-scale training pipeline instead of a downstream comparison

## Inputs

- AnnData .h5ad count matrix with cells in adata.obs (incl. target/cell_type, condition, donor_id columns) and genes in adata.var
- Top-K feature list from scaLR feature selection (e.g. k=5000 via AbsMean selector on LinearScorer scores)
- scaLR pipeline config YAML (config_clinical.yaml style) with test_samples_downstream_analysis and full_samples_downstream_analysis sections

## Outputs

- Volcano plots and DEG gene lists for DgePseudoBulk and DgeLMEM under both sample scopes
- DEG count comparison table (up/down per approach, test vs all samples), e.g. dge_deg_counts.csv

## How to apply

Load the scRNA-seq count matrix as an .h5ad AnnData object (cells in obs, genes in var, log1p-normalized X), restrict to the top-K features selected by scaLR's LinearScorer/AbsMean feature selection, and annotate cells with cell-type and condition columns (e.g. cell_type, disease, donor_id). Run scalr.analysis.dge_pseudobulk, which aggregates counts into pseudo-bulk profiles per sample and cell type using celltype_column, design_factor, factor_categories, and sum_column, calling DEGs at a fold-change cutoff of ±1.5 and P<0.05. Separately run scalr.analysis.dge_lmem, which fits a linear mixed-effects model per gene across cell types/conditions with DEGs called at P<0.05. Execute each DGE module twice — once under the test_samples_downstream_analysis config section and once under full_samples_downstream_analysis — then tabulate upregulated/downregulated DEG counts per approach and diff the gene lists. In the scaLR myocardial benchmark, the all-samples run on the top-5,000-feature model raised pseudo-bulk upregulated genes in smooth muscle myoblasts from 41 to 254 and LMEM upregulated genes from 92 to 130, with downregulated counts rising 22→164 (pseudo-bulk) and 256→332 (LMEM).

## Related tools

- **scaLR** (Platform whose DGE module (dge_pseudobulk, dge_lmem) generates the volcano plots and gene lists compared across test-sample and full-sample runs) — https://github.com/infocusp/scaLR
- **Python** (Execution environment for scaLR (tested in Python 3.10.20) used to run the pipeline and tabulate DEG counts) — https://github.com/infocusp/scaLR

## Examples

```
python pipeline.py --config scaLR/tutorials/pipeline/config_clinical.yaml -l -m
```

## Evaluation signals

- Reproduced pseudo-bulk upregulated DEG counts on the top-5,000-feature myocardial model rise from 41 (test samples) to 254 (all samples)
- Reproduced LMEM upregulated DEG counts rise from 92 (test samples) to 130 (all samples)
- Downregulated DEG counts match reported increases: 22→164 (pseudo-bulk) and 256→332 (LMEM)
- Both DGE runs use the same fold-change cutoff (±1.5) and P<0.05 so only sample scope differs
- Output gene lists parse into a dge_deg_counts.csv table with one row per approach × direction × sample scope

## Limitations

- DgeLMEM can be slow because multiprocessing is inefficient on low-CPU runtimes (e.g. 2-CPU Colab)
- Benchmark result tables referenced in the discussion (Tables 2–4) and supplementary biomarker tables S10–S14 are not present in the extracted text, so exact tolerances must be taken from the source card
- scaLR's memory usage is inversely proportional to execution time, so full-sample DGE on 191k cells × 28k genes may trade runtime for memory

## Evidence

- [readme] Two DGE options in scaLR config: [evidence span withheld — anchor status: external_doc]
- [readme] Full-samples vs test-samples downstream config sections: [evidence span withheld — anchor status: external_doc]
