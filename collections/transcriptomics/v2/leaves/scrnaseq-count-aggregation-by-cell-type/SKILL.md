---
name: scrnaseq-count-aggregation-by-cell-type
description: Use when a scRNA-seq anndata (.h5ad) matrix with per-cell metadata (cell_type,
  donor_id, condition) must be reduced to pseudo-bulk counts per sample and cell type
  before calling DEGs between conditions.
when_to_use_negative:
- The input is already bulk/aggregated count data with no per-cell cell_type annotations
  — pseudo-bulk summation over cell_type and donor_id groups is meaningless
- Cells lack sample/donor identifiers in adata.obs, so no per-sample pseudo-bulk profiles
  can be formed
- The question is per-cell differential expression rather than sample-level DGE between
  factor_categories
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scaLR
  role: Provides the DgePseudoBulk (and DgeLMEM) downstream-analysis modules that
    aggregate counts per sample and cell type, then generate volcano plots and associated
    gene lists
  repo: github.com/infocusp/scaLR
- name: Python
  role: Platform in which scaLR is implemented and in which the pseudo-bulk aggregation
    and DGE pipeline are run (tested with Python 3.10.20)
  repo: github.com/infocusp/scaLR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:e009ec9976d0f2f92bd552364c39108f5075eb865dcdd2fa4eb8e1caf54de01c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/infocusp/scaLR
  tools:
  - Python
  - scaLR
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# scrnaseq-count-aggregation-by-cell-type

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates single-cell RNA-seq counts into pseudo-bulk profiles per sample and cell type using scaLR's DgePseudoBulk module, converting cell-level data into sample-level profiles that support differential gene expression (DGE) calling with volcano plots and gene lists.

## When to use

Use when a scRNA-seq anndata (.h5ad) matrix with per-cell metadata (cell_type, donor_id, condition) must be reduced to pseudo-bulk counts per sample and cell type before calling DEGs between conditions.

## When NOT to use

- The input is already bulk/aggregated count data with no per-cell cell_type annotations — pseudo-bulk summation over cell_type and donor_id groups is meaningless
- Cells lack sample/donor identifiers in adata.obs, so no per-sample pseudo-bulk profiles can be formed
- The question is per-cell differential expression rather than sample-level DGE between factor_categories

## Inputs

- anndata .h5ad scRNA-seq matrix (adata.X log1p-normalized counts in 0-10 range; adata.obs with cell_type, donor_id, and condition/target columns; gene names in adata.var)
- Top-K feature list from scaLR feature selection (e.g. top 5,000 features)
- Sample metadata mapping cells to donor/sample IDs and disease/condition categories

## Outputs

- Pseudo-bulk count profiles aggregated per sample (sum_column) and cell type (celltype_column)
- DEG gene lists with upregulated/downregulated genes per approach (fold change ±1.5, P<0.05)
- Volcano plots per approach (DgePseudoBulk and DgeLMEM)
- DEG count comparison table (e.g. dge_deg_counts.csv) tabulating counts per direction for test vs full samples

## How to apply

Run scaLR's DgePseudoBulk downstream analysis from the clinical-condition config (scaLR/tutorials/pipeline/config_clinical.yaml), setting celltype_column: 'cell_type', design_factor: 'disease', factor_categories (e.g. ['COVID-19', 'normal']), and sum_column: 'donor_id' so counts are summed over all cells sharing a donor and cell type, producing one pseudo-bulk profile per sample-by-cell-type combination. Restrict the input to the top-K features from scaLR feature selection (e.g. the top 5,000 features) so DGE is computed over the final model's gene set. Call DEGs at a fold-change cutoff of ±1.5 and P<0.05 for the pseudo-bulk approach, and optionally cross-check with scalr.analysis.dge_lmem, which fits a linear mixed-effects model per gene and calls DEGs at P<0.05. Run the module on both the test samples and the full sample set — aggregating over all samples substantially increases DEG yield (in the myocardial case, pseudo-bulk upregulated genes rose from 41 to 254 in smooth muscle myoblasts). Collect the resulting gene lists and tabulate DEG counts per direction (up/down) per approach, e.g. into a dge_deg_counts.csv comparison table.

## Related tools

- **scaLR** (Provides the DgePseudoBulk (and DgeLMEM) downstream-analysis modules that aggregate counts per sample and cell type, then generate volcano plots and associated gene lists) — github.com/infocusp/scaLR
- **Python** (Platform in which scaLR is implemented and in which the pseudo-bulk aggregation and DGE pipeline are run (tested with Python 3.10.20)) — github.com/infocusp/scaLR

## Examples

```
python pipeline.py --config scaLR/tutorials/pipeline/config_clinical.yaml
```

## Evaluation signals

- DEG counts under the top-5,000-feature model match reported increases: pseudo-bulk upregulated 41→254 and LMEM upregulated 92→130 in the myocardial comparison
- Each pseudo-bulk profile equals the sum of adata.X counts over all cells sharing the same sum_column (donor_id) and celltype_column (cell_type) value
- Volcano plots and associated gene lists are produced for both DgePseudoBulk and DgeLMEM as the scaLR DGE module specifies
- Called DEGs respect the stated thresholds: fold-change cutoff of ±1.5 and P<0.05 (pseudo-bulk), P<0.05 (LMEM)
- Full-sample runs yield more DEGs than test-sample-only runs across cell types and directions, consistent with the reported pattern

## Limitations

- scaLR's memory usage is inversely proportional to execution time — reducing memory consumption may increase processing time, so large matrices must be read in backed mode / batch-wise
- DgeLMEM is slow when multiprocessing is inefficient on few CPUs (e.g. only 2 CPUs in the Colab runtime), so pseudo-bulk is the faster option
- scaLR performance depends on input data format — data must be .h5ad anndata with log1p normalization (range 0-10), requiring user preprocessing for optimal results

## Evidence

- [readme] Two DGE options and their parameters: [evidence span withheld — anchor status: external_doc]
- [readme] Expected input normalization for aggregation: [evidence span withheld — anchor status: external_doc]
- [readme] LMEM runtime caveat: [evidence span withheld — anchor status: external_doc]
