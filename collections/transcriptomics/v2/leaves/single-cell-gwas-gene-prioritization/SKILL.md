---
name: single-cell-gwas-gene-prioritization
description: Use when GWAS summary statistics and a normalized single-cell Seurat
  object are available and the goal is to rank trait-relevant genes and compare their
  functional enrichment to gene-scoring baselines.
when_to_use_negative:
- The goal is causal inference — the paper states identification of statistical association
  between traits and cells 'does not imply causality'
- Functionally informed SNP-to-gene linking is required — default scPagwas annotates
  SNPs to genes solely by proximal distance within a 20 kb window
- Input is bulk rather than single-cell expression — the skill depends on per-cell
  expression correlated with per-cell gPAS scores
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_3308
tools:
- name: scPagwas
  role: Computes gPAS scores via polygenic regression and provides scGet_PCC for gene
    prioritization
  repo: https://github.com/dengchunyu/scPagwas
- name: Seurat
  role: Loads and holds the normalized single-cell object whose gene expression is
    correlated with gPASsum; AddModuleScore computes TRS
  repo: https://github.com/satijalab/seurat
- name: MAGMA
  role: Baseline gene-scoring method whose top-gene sets are compared against scPagwas
    prioritization under an identical GO enrichment pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:a462adce02638838f005ec9c18d3f5c2a116fe1b1b1d3d156c483a11fdf56be8
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
  - MAGMA
  - Seurat
  - scPagwas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# single-cell-gwas-gene-prioritization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prioritize trait-relevant genes by correlating per-gene single-cell expression with scPagwas genetic pathway-activation scores (gPAS), then benchmark the top 1,000 genes against MAGMA/TWAS/S-PrediXcan/S-MultiXcan gene sets using GO biological-process over-representation.

## When to use

Use when GWAS summary statistics and a normalized single-cell Seurat object are available and the goal is to rank trait-relevant genes and compare their functional enrichment to gene-scoring baselines.

## When NOT to use

- The goal is causal inference — the paper states identification of statistical association between traits and cells 'does not imply causality'
- Functionally informed SNP-to-gene linking is required — default scPagwas annotates SNPs to genes solely by proximal distance within a 20 kb window
- Input is bulk rather than single-cell expression — the skill depends on per-cell expression correlated with per-cell gPAS scores

## Inputs

- GWAS summary statistics per trait (e.g., IEU IDs ieu-b-31 monocyte count, ebi-a-GCST004627 lymphocyte count)
- Normalized scRNA-seq Seurat object with 'RNA' assay (e.g., Seu_Hema_data.rds BMMC data)
- KEGG pathway gene list (Genes_by_pathway_kegg) and chromosome LD reference (1,000 Genomes Phase 3)
- Per-trait scPagwas results containing scPagwas.gPAS.score

## Outputs

- Per-trait ranked gene list by PCC with gPASsum; top 1,000 trait-relevant gene set per trait
- Comparison table of significant GO BP term counts per method per trait (scPagwas_GO_enrichment_TableS3.csv)

## How to apply

Run scPagwas_main2 on the GWAS summary statistics and single-cell data: SNPs are assigned to genes in KEGG pathways via a 20 kb window, filtered for MAF < 0.01, non-autosomal loci, and the MHC region (Chr6: 25-35 Mbp), and pathways limited to 5-300 genes. After the polygenic regression produces per-cell gPAS scores, compute the Pearson correlation of each gene's expression with the summed gPASs using scPagwas::scGet_PCC and select the top 1,000 genes (paper default n_topgenes = 1000) per trait. For a fair comparison, rebuild the trait gene sets with baseline gene-scoring methods (MAGMA, TWAS, S-PrediXcan, S-MultiXcan) and run the identical GO BP over-representation pipeline on each top-gene set. Count significant BP terms among over-expressed genes at FDR < 0.01 and tabulate per method per trait.

## Related tools

- **scPagwas** (Computes gPAS scores via polygenic regression and provides scGet_PCC for gene prioritization) — https://github.com/dengchunyu/scPagwas
- **Seurat** (Loads and holds the normalized single-cell object whose gene expression is correlated with gPASsum; AddModuleScore computes TRS) — https://github.com/satijalab/seurat
- **MAGMA** (Baseline gene-scoring method whose top-gene sets are compared against scPagwas prioritization under an identical GO enrichment pipeline)

## Examples

```
Pagwas_data <- scPagwas_main2(gwas_data = "GWAS_summ_example.txt", Single_data = "scRNAexample.rds", iters_singlecell = 10, chrom_ld = chrom_ld, singlecell = TRUE, celltype = TRUE)
```

## Evaluation signals

- Exactly 1,000 genes selected per trait (paper default n_topgenes = 1000) ranked by descending PCC with gPASsum
- For lymphocyte count over-expression genes, scPagwas should yield 35 significant BP terms (FDR < 0.01) versus 0 for MAGMA
- The output table scPagwas_GO_enrichment_TableS3.csv reproduces the Table S3 pattern: scPagwas top genes more enriched than all four baselines across the 10 blood traits
- Baselines (MAGMA, TWAS, S-PrediXcan, S-MultiXcan) run through the identical GO BP over-representation pipeline and background gene sets
- gPAS computation used default filters: MAF >= 0.01, autosomal SNPs only, MHC region removed, pathways of 5-300 genes

## Limitations

- Pathway analysis restricted to canonical KEGG pathways only, which may miss non-KEGG biology
- SNPs are annotated to genes solely by proximal distance (20 kb window); functionally informed SNP-to-gene linking deferred to future work
- Statistical association between traits and genes/cells does not imply causality and may reflect indirect associations
- Large single-cell datasets can exceed R memory limits; the authors provide the Python scPagwas_py implementation for computational efficiency

## Evidence
