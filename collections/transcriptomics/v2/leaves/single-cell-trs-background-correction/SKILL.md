---
name: single-cell-trs-background-correction
description: Use when scoring individual cells for GWAS trait association from a Seurat
  scRNA-seq object — i.e., after trait-relevant genes (top 1,000 by Pearson correlation
  with gPASsum) are prioritized and per-cell p-values are needed (scPagwas singlecell
  = TRUE).
when_to_use_negative:
- The goal is bulk cell-type enrichment from GWAS summary statistics alone without
  per-cell scoring — methods such as LDSC-SEG or EPIC apply instead
- Only trait-relevant gene prioritization is needed; the gPAS/Pearson-correlation
  step suffices and Monte Carlo TRS testing is unnecessary overhead
- The scRNA-seq data is unnormalized or lacks a 'RNA' assay, since AddModuleScore-based
  TRS computation assumes a normalized Seurat object
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0203
tools:
- name: scPagwas
  role: Runs the polygenic regression that yields trait-relevant genes and orchestrates
    TRS computation, Monte Carlo single-cell testing, and cell-type block bootstrap
  repo: https://github.com/dengchunyu/scPagwas
- name: Seurat AddModuleScore
  role: Default cell-scoring method used to compute the TRS with expression-bin-matched
    control gene sets
  repo: https://github.com/satijalab/seurat
- name: scPagwas_py
  role: Python/Scanpy alternative for the same TRS pipeline when single-cell memory
    demands exceed R's capacity
  repo: https://github.com/dengchunyu/scPagwas_py
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:81ddbfffba224c1202e35ba3dc7a74fdf4dbca047d3e1246b75933b9b1394407
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
  - Seurat AddModuleScore
  - chrom_ld
  - scPagwas
  - scPagwas_py
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# single-cell-trs-background-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes a per-cell trait relevance score (TRS) from scPagwas-prioritized trait-relevant genes using Seurat's AddModuleScore with expression-bin-matched control gene sets, then assesses single-cell significance by a Monte Carlo test against the empirical distribution of control TRSs. This background correction guards against expression sparsity, technical noise, and cell-level confounders.

## When to use

Use when scoring individual cells for GWAS trait association from a Seurat scRNA-seq object — i.e., after trait-relevant genes (top 1,000 by Pearson correlation with gPASsum) are prioritized and per-cell p-values are needed (scPagwas singlecell = TRUE).

## When NOT to use

- The goal is bulk cell-type enrichment from GWAS summary statistics alone without per-cell scoring — methods such as LDSC-SEG or EPIC apply instead
- Only trait-relevant gene prioritization is needed; the gPAS/Pearson-correlation step suffices and Monte Carlo TRS testing is unnecessary overhead
- The scRNA-seq data is unnormalized or lacks a 'RNA' assay, since AddModuleScore-based TRS computation assumes a normalized Seurat object

## Inputs

- Six-column tab-delimited GWAS summary statistics file (MAF > 0.01, sex chromosomes and Chr6:25-35 Mbp MHC region removed)
- Seurat object (.rds) with 'RNA' assay and normalized expression matrix, cell-type annotations in Idents
- Pathway list (Genes_by_pathway_kegg) with pathways of 5-300 genes
- chrom_ld LD data and block_annotation (hg37/hg38) supplied by the scPagwas package

## Outputs

- Per-cell TRS values (scPagwas TRS) overlaid on the Seurat object
- Single-cell Monte Carlo p-values against the empirical distribution of control TRSs
- Cell-type-level bootstrap results (bootstrap p-values and confidence intervals per cell type)

## How to apply

First identify trait-relevant genes by Pearson correlation of each gene's expression with the summed gPASs (gPASsum) across individual cells, retaining the top 1,000 genes. Compute the TRS with the Seurat AddModuleScore cell-scoring method at default parameters, which builds control gene sets matched by expression bins so that background correction accounts for expression-level bias. Determine single-cell statistical significance via a Monte Carlo test comparing the scPagwas TRS to the empirical distribution of control TRSs; within scPagwas_main/scPagwas_main2 this is controlled by iters_singlecell (e.g., 100 or 200 iterations) and n_topgenes = 1000, with min_clustercells = 10 filtering sparse clusters. The rationale is that top genetic association signals may be absent from most cells due to extensive sparsity and technical noise in scRNA-seq data, so pathway-transformed gene prioritization plus bin-matched control scores yields stable per-cell inference that Seurat's normalized 'RNA' assay can directly consume.

## Related tools

- **scPagwas** (Runs the polygenic regression that yields trait-relevant genes and orchestrates TRS computation, Monte Carlo single-cell testing, and cell-type block bootstrap) — https://github.com/dengchunyu/scPagwas
- **Seurat AddModuleScore** (Default cell-scoring method used to compute the TRS with expression-bin-matched control gene sets) — https://github.com/satijalab/seurat
- **scPagwas_py** (Python/Scanpy alternative for the same TRS pipeline when single-cell memory demands exceed R's capacity) — https://github.com/dengchunyu/scPagwas_py

## Examples

```
Pagwas_data <- scPagwas_main2(Pagwas = NULL, gwas_data = "GWAS_summ_example.txt", Single_data = "scRNAexample.rds", output.prefix = "test", singlecell = T, celltype = T)
```

## Evaluation signals

- Trait-relevant gene set size matches n_topgenes = 1000 and each gene's Pearson correlation with gPASsum was computed across individual cells
- Single-cell p-values come from a Monte Carlo comparison against the empirical distribution of control TRSs, with iters_singlecell >= 100 iterations completed
- Control gene sets are expression-bin matched (AddModuleScore defaults), so TRS distributions are not dominated by high-expression cells
- In the severe COVID-19 PBMC reproduction, the pipeline recovers naive CD8+ T cells (p = 4.6e-17), megakaryocytes (p = 7.8e-6), and CD16+ monocytes (p = 1.14e-4) as significant at FDR < 0.05
- Clusters with fewer than min_clustercells = 10 cells are excluded before scoring

## Limitations

- Statistical association between the trait and individual cells does not imply causality; it may reflect indirect identification of causal associations
- The default TRS relies on Seurat's AddModuleScore cell-scoring method for Seurat compatibility, though scDRS, AUCell, and VISION also showed good and robust performance
- Top genetic association signals at specific loci may be absent from most cells because of extensive sparsity and technical noise in single-cell data, making per-cell inference sensitive to normalization choices
- Single-cell TRS calculations are memory-intensive in R; a fully synchronized Python version (scPagwas_py) is provided for large datasets

## Evidence

- [readme] Seurat input requirements: [evidence span withheld — anchor status: external_doc]
