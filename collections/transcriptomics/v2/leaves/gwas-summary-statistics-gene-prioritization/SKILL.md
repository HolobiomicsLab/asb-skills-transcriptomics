---
name: gwas-summary-statistics-gene-prioritization
description: Use when you have GWAS summary statistics (e.g., monocyte count, IEU
  ID ieu-b-31) and a normalized scRNA-seq matrix, and you need a ranked trait-relevant
  gene set (n_topgenes = 1,000) to feed cell-scoring methods like scDRS or Seurat
  AddModuleScore.
when_to_use_negative:
- The goal is cell-type-level (not single-cell or gene-level) trait association, where
  LDSC-SEG, RolyPoly, MAGMA-based approaches, or EPIC are the appropriate alternatives
- The question is functional interpretation of which biological pathways variants
  regulate, not producing a ranked gene list for cell scoring
- No GWAS summary statistics are available and only a curated disease gene set already
  exists to feed scDRS directly
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: scPagwas
  role: 'Core tool: pathway activation transformation and method-of-moments polygenic
    regression produce gPAS scores from which the top 1,000 trait-relevant genes are
    derived via scGet_PCC'
  repo: github.com/dengchunyu/scPagwas
- name: MAGMA
  role: Baseline gene-prioritization method whose top 1,000 genes serve as the comparison
    arm when benchmarking scPagwas-prioritized genes in scDRS
- name: scDRS
  role: 'Downstream consumer: run twice (once with MAGMA genes, once with scPagwas
    top 1,000 genes) to test whether scPagwas gene prioritization improves per-cell
    disease-score precision'
  repo: github.com/martinjzhang/scDRS
- name: Seurat AddModuleScore function
  role: Computes the TRS per cell using the scPagwas-prioritized gene set, with background-corrected
    p-values from Get_CorrectBg_p (iters_singlecell = 100)
  repo: github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:a8a54c65251215c63d25d1a13c749abf2e6186e3fdea979c1c2d1cf62f26968d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dengchunyu/scPagwas
  tools:
  - MAGMA
  - Seurat AddModuleScore function
  - scDRS
  - scPagwas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# gwas-summary-statistics-gene-prioritization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prioritizes trait-relevant genes from GWAS summary statistics by running scPagwas's pathway activation transformation (polygenic regression yielding per-pathway per-cell gPAS coefficients) and ranking genes by Pearson correlation of their expression with the summed gPAS (gPASsum), returning the top 1,000 genes. These genes are the critical input that downstream per-cell scorers such as scDRS and Seurat AddModuleScore depend on for accurate trait-relevant cell identification.

## When to use

Use when you have GWAS summary statistics (e.g., monocyte count, IEU ID ieu-b-31) and a normalized scRNA-seq matrix, and you need a ranked trait-relevant gene set (n_topgenes = 1,000) to feed cell-scoring methods like scDRS or Seurat AddModuleScore.

## When NOT to use

- The goal is cell-type-level (not single-cell or gene-level) trait association, where LDSC-SEG, RolyPoly, MAGMA-based approaches, or EPIC are the appropriate alternatives
- The question is functional interpretation of which biological pathways variants regulate, not producing a ranked gene list for cell scoring
- No GWAS summary statistics are available and only a curated disease gene set already exists to feed scDRS directly

## Inputs

- GWAS summary statistics file (tab-delimited; e.g., monocyte count IEU ID ieu-b-31, pruned six-column text)
- Normalized scRNA-seq data in Seurat format with an 'RNA' assay (e.g., BMMC Seurat object or scDesign2-simulated five-cell-type dataset)
- KEGG pathway list (Genes_by_pathway_kegg)
- LD reference data (1000 Genomes Phase 3 per-chromosome LD, chrom_ld)
- Gene block annotation (block_annotation_hg37 or hg38)

## Outputs

- Ranked list of top 1,000 trait-relevant genes prioritized from the GWAS
- Per-pathway per-cell gPAS coefficients (polygenic regression output)
- Pathway-by-cell PAS matrix from SVD transformation
- Precision benchmark table (e.g., precision_benchmark.csv) comparing scDRS with scPagwas genes vs MAGMA genes

## How to apply

Run scPagwas_main on the GWAS summary statistics and a normalized Seurat object (RNA assay) with the KEGG pathway list Genes_by_pathway_kegg, maf_filter = 0.01, removal of sex-chromosome SNPs, a 20 kb SNP-to-gene window (marg = 10000), pathway sizes limited to 5-300 genes, and n_topgenes = 1000. The method-of-moments polygenic regression on the SVD-derived pathway-by-cell matrix estimates gPAS coefficients per pathway per cell, which boosts statistical power by collapsing functional actions of genes within pathways. Genes are then ranked by Pearson correlation (scGet_PCC) of each gene's expression with the summed gPASs across individual cells, and the top 1,000 are retained as the trait-relevant gene set. Benchmark this gene set against MAGMA-prioritized genes by scoring cells with scDRS 1.0.2 using each gene set and comparing retrieval precision against a ground truth (e.g., 0.957 vs 0.940 for monocyte count in the scDesign2-simulated dataset; 0.984 vs 0.787 in the real BMMC dataset).

## Related tools

- **scPagwas** (Core tool: pathway activation transformation and method-of-moments polygenic regression produce gPAS scores from which the top 1,000 trait-relevant genes are derived via scGet_PCC) — github.com/dengchunyu/scPagwas
- **MAGMA** (Baseline gene-prioritization method whose top 1,000 genes serve as the comparison arm when benchmarking scPagwas-prioritized genes in scDRS)
- **scDRS** (Downstream consumer: run twice (once with MAGMA genes, once with scPagwas top 1,000 genes) to test whether scPagwas gene prioritization improves per-cell disease-score precision) — github.com/martinjzhang/scDRS
- **Seurat AddModuleScore function** (Computes the TRS per cell using the scPagwas-prioritized gene set, with background-corrected p-values from Get_CorrectBg_p (iters_singlecell = 100)) — github.com/satijalab/seurat

## Examples

```
Pagwas <- scPagwas_main(Pagwas=NULL, gwas_data="GWAS_summ_example.txt", Single_data="scRNAexample.rds", Pathway_list=Genes_by_pathway_kegg, n_topgenes=1000, singlecell=TRUE)
```

## Evaluation signals

- On the scDesign2-simulated monocyte count ground truth, scDRS with scPagwas top 1,000 genes must reach precision ~0.957, exceeding scDRS with MAGMA genes (0.940)
- On the real BMMC ground truth (5,000 monocytes vs T/B/DC/NK cells), scPagwas TRS-based precision should be ~98.2% and scDRS precision should improve from ~0.787 (MAGMA genes) to ~0.984 (scPagwas genes)
- The output gene list contains exactly n_topgenes = 1,000 genes ranked by Pearson correlation of expression with gPASsum
- Filter invariants hold: SNPs with MAF < 0.01 and sex-chromosome SNPs excluded; only pathways of 5-300 genes retained
- precision_benchmark.csv exists and contains a precision column with scPagwas precision within a few percentage points of the reported 95.9%

## Limitations

- SNPs are annotated to genes solely by proximal distance (20 kb window); functionally informed SNP-to-gene linking is deferred to future work
- Pathway analysis is restricted to canonical KEGG pathways only, which may omit biologically relevant gene sets
- Statistical association between traits and cells does not imply causality; identified associations may be indirect

## Evidence
