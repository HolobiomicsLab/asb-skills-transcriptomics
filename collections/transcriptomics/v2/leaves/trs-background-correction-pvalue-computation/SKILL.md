---
name: trs-background-correction-pvalue-computation
description: Use when scPagwas has produced a gPAS/TRS per cell from GWAS summary
  statistics plus scRNA-seq data and you need per-cell p-values corrected against
  a background of control TRSs to call trait-relevant individual cells.
when_to_use_negative:
- When the goal is cell-type-level (not single-cell-level) association — use the block
  bootstrap cell-type inference (Boot_evaluate, 200 iterations) instead of per-cell
  p-values
- When no GWAS summary statistics are available to derive the trait-relevant gene
  set, since control TRSs and background correction require a genetically informed
  TRS
- When the scRNA-seq object is raw counts without normalization/VST pathway activation
  transformation, as AddModuleScore-based TRS computation requires normalized data
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scPagwas
  role: Provides scPagwas_perform_score (gPAS/TRS) and Get_CorrectBg_p (iters_singlecell
    = 100, n_topgenes = 1000) for background-corrected per-cell p-values
  repo: https://github.com/dengchunyu/scPagwas
- name: Seurat AddModuleScore function
  role: Computes the TRS by cell scoring against the trait-relevant gene set with
    default parameters
  repo: https://github.com/satijalab/seurat
- name: scPagwas_py
  role: Python implementation synchronized with scPagwas 2.0 for memory-heavy single-cell
    calculations
  repo: https://github.com/dengchunyu/scPagwas_py
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:883aff836623ef83f08a8a33b2acea4596106029a665a536ea59bd5c8187d973
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dengchunyu/scPagwas
  repo_url: https://github.com/dengchunyu/scPagwas
  tools:
  - Seurat AddModuleScore function
  - scPagwas
  - scPagwas_py
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# trs-background-correction-pvalue-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-cell significance of trait-relevant scores (TRS) in scPagwas by Monte Carlo comparison of each cell's Seurat AddModuleScore-based TRS against control TRSs matched by expression bins, then background-corrects the p-values (Get_CorrectBg_p with iters_singlecell = 100, n_topgenes = 1000) to identify individual trait-associated cells.

## When to use

Use when scPagwas has produced a gPAS/TRS per cell from GWAS summary statistics plus scRNA-seq data and you need per-cell p-values corrected against a background of control TRSs to call trait-relevant individual cells.

## When NOT to use

- When the goal is cell-type-level (not single-cell-level) association — use the block bootstrap cell-type inference (Boot_evaluate, 200 iterations) instead of per-cell p-values
- When no GWAS summary statistics are available to derive the trait-relevant gene set, since control TRSs and background correction require a genetically informed TRS
- When the scRNA-seq object is raw counts without normalization/VST pathway activation transformation, as AddModuleScore-based TRS computation requires normalized data

## Inputs

- Normalized Seurat object (Seurat 4.3.0, 'RNA' assay) with per-cell gPAS/TRS from scPagwas_perform_score
- Top-ranked trait-relevant gene set (n_topgenes = 1000) selected by Pearson correlation with summed gPASs
- GWAS summary statistics and single-cell context from prior scPagwas pipeline steps

## Outputs

- Per-cell background-corrected p-values for the TRS
- Single-cell score/p-value CSV export
- Seurat result object with scPagwas results (e.g., AD_scPagwas_results.RData)

## How to apply

First run scPagwas_perform_score to obtain gPAS and the TRS, where the TRS is computed with Seurat's AddModuleScore using the top 1,000 trait-relevant genes ranked by scGet_PCC Pearson correlation of gene expression with summed gPASs. Then call Get_CorrectBg_p (iters_singlecell = 100, n_topgenes = 1000): the method-of-moments polygenic regression coefficients define the gene set, and Monte Carlo iteration builds an empirical distribution of control TRSs (control gene sets matched by expression bins) against which each cell's observed TRS is compared to yield a raw p-value that is background-corrected for single-cell sparsity and technical noise. The rationale is that top genetic association signals may be absent from most cells because of single-cell sparsity, so raw module scores are inflated; the MC background correction calibrates per-cell significance. Downstream, cells with corrected p-values below significance thresholds are reported as trait-relevant, and the single-cell scores/p-values are exported as CSV alongside the Seurat result object (e.g., AD_scPagwas_results.RData).

## Related tools

- **scPagwas** (Provides scPagwas_perform_score (gPAS/TRS) and Get_CorrectBg_p (iters_singlecell = 100, n_topgenes = 1000) for background-corrected per-cell p-values) — https://github.com/dengchunyu/scPagwas
- **Seurat AddModuleScore function** (Computes the TRS by cell scoring against the trait-relevant gene set with default parameters) — https://github.com/satijalab/seurat
- **scPagwas_py** (Python implementation synchronized with scPagwas 2.0 for memory-heavy single-cell calculations) — https://github.com/dengchunyu/scPagwas_py

## Examples

```
Pagwas <- Get_CorrectBg_p(Pagwas = Pagwas, iters_singlecell = 100, n_topgenes = 1000)
```

## Evaluation signals

- Per-cell p-values fall in [0, 1] and cells with high TRS receive lower background-corrected p-values than cells with low TRS
- Corrected p-values change relative to raw TRS ranks only where background control TRSs are inflated, confirming Monte Carlo calibration occurred (iters_singlecell = 100 replicates completed)
- The gene set used for scoring contains exactly n_topgenes = 1000 top-ranked genes by scGet_PCC heritability correlation (e.g., CREB1 and GSK3B among the top AD genes)
- Per-cell calls are consistent with the independently computed cell-type-level block bootstrap results (OPC and microglia FDR < 0.05 in the AD case)
- Exported single-cell score/p-value CSV rows match the cell barcodes in the saved Seurat result object

## Limitations

- Identification of a statistical association between complex diseases/traits and individual cells does not imply causality but may reflect indirect identification of causal associations
- The default TRS relies on Seurat's AddModuleScore cell-scoring method for Seurat compatibility, which constrains the scoring approach
- Per-cell inference is difficult because top genetic association signals at specific loci may be absent from most cells due to extensive sparsity and technical noise in single-cell data, making background correction essential but dependent on expression-bin-matched control TRSs

## Evidence
