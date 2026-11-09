---
name: go-biological-process-enrichment-analysis
description: Use when comparing trait-relevant gene sets from GWAS-to-single-cell
  gene prioritization methods and you need GO BP over-representation term counts (FDR
  < 0.01) as the functional-quality benchmark.
when_to_use_negative:
- Your goal is single-cell association itself (TRS/cell-type inference) rather than
  evaluating the quality of the prioritized gene sets — use the scPagwas_main pipeline
  instead.
- Input gene sets are pathway-level (KEGG, 5–300 genes) results from the PAS step,
  not whole-gene-set over-representation — the GO BP ORA here is defined on top-1,000
  gene lists.
- No matched baseline gene scores (MAGMA/TWAS/S-PrediXcan/S-MultiXcan) are available,
  since the skill's purpose is the head-to-head enrichment comparison.
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0625
tools:
- name: scPagwas
  role: Prioritizes trait-relevant genes via Pearson correlation of gene expression
    with summed gPAS scores (scGet_PCC); supplies the top-1,000 gene sets under test
  repo: https://github.com/dengchunyu/scPagwas
- name: Seurat
  role: Provides the Seurat object format for the single-cell data (AddModuleScore-based
    TRS pipeline that generates the gPAS context for gene ranking)
  repo: https://github.com/satijalab/seurat
- name: MAGMA
  role: Baseline gene-scoring method whose top-gene sets are run through the identical
    GO BP enrichment pipeline for comparison
- name: scPagwas_py
  role: Python implementation of scPagwas, fully synchronized with scPagwas2.0, for
    memory-heavy single-cell computations
  repo: https://github.com/dengchunyu/scPagwas_py
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:fcb72e274232804feafaeb4f14d4b082da04f7ad38992a2954f0b3c43ff0d3a0
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
  - scPagwas_py
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# go-biological-process-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs GO biological-process over-representation enrichment on scPagwas-prioritized trait-relevant gene sets (top 1,000 genes per blood trait) and benchmarks the enrichment yield against MAGMA, TWAS, S-PrediXcan, and S-MultiXcan baselines. This evaluates whether scPagwas's Pearson-correlation gene prioritization recovers functionally coherent blood-trait biology better than eQTL-based gene-scoring methods.

## When to use

Use when comparing trait-relevant gene sets from GWAS-to-single-cell gene prioritization methods and you need GO BP over-representation term counts (FDR < 0.01) as the functional-quality benchmark.

## When NOT to use

- Your goal is single-cell association itself (TRS/cell-type inference) rather than evaluating the quality of the prioritized gene sets — use the scPagwas_main pipeline instead.
- Input gene sets are pathway-level (KEGG, 5–300 genes) results from the PAS step, not whole-gene-set over-representation — the GO BP ORA here is defined on top-1,000 gene lists.
- No matched baseline gene scores (MAGMA/TWAS/S-PrediXcan/S-MultiXcan) are available, since the skill's purpose is the head-to-head enrichment comparison.

## Inputs

- BMMC Seurat object (Seu_Hema_data.rds) with normalized RNA assay
- Per-trait scPagwas results containing scPagwas.gPAS.score for 10 blood cell traits
- GWAS summary statistics for blood traits (e.g., IEU ID ebi-a-GCST004627 for lymphocyte count, ieu-b-31 for monocyte count)
- MAGMA, TWAS, S-PrediXcan, and S-MultiXcan per-gene association scores for the same traits

## Outputs

- Top 1,000 trait-relevant gene sets per trait per method (ranked by PCC or method-specific gene score)
- Per-trait, per-method counts of significant GO BP terms (FDR < 0.01) among over-expressed genes
- Comparison table scPagwas_GO_enrichment_TableS3.csv reproducing Table S3

## How to apply

Load the BMMC Seurat object (Seu_Hema_data.rds) and the per-trait scPagwas results containing scPagwas.gPAS.score. For each of the 10 blood traits, compute the Pearson correlation of each gene's expression with the summed gPAS genetic score using scPagwas::scGet_PCC, rank genes by PCC, and retain the top 1,000 (paper default n_topgenes = 1000) as the trait-relevant gene set. Run GO biological-process over-representation enrichment on each top-gene set against blood-trait gene-set backgrounds, counting significant BP terms among over-expressed genes at FDR < 0.01. Rebuild the trait gene sets with MAGMA, TWAS, S-PrediXcan, and S-MultiXcan scores and run the identical enrichment pipeline so term counts are directly comparable across methods. Tabulate significant-term counts per method per trait (e.g., lymphocyte count: 35 significant BP terms for scPagwas vs 0 for MAGMA) and save as scPagwas_GO_enrichment_TableS3.csv.

## Related tools

- **scPagwas** (Prioritizes trait-relevant genes via Pearson correlation of gene expression with summed gPAS scores (scGet_PCC); supplies the top-1,000 gene sets under test) — https://github.com/dengchunyu/scPagwas
- **Seurat** (Provides the Seurat object format for the single-cell data (AddModuleScore-based TRS pipeline that generates the gPAS context for gene ranking)) — https://github.com/satijalab/seurat
- **MAGMA** (Baseline gene-scoring method whose top-gene sets are run through the identical GO BP enrichment pipeline for comparison)
- **scPagwas_py** (Python implementation of scPagwas, fully synchronized with scPagwas2.0, for memory-heavy single-cell computations) — https://github.com/dengchunyu/scPagwas_py

## Examples

```
pcc <- scPagwas::scGet_PCC(data = Seu_Hema_data); top_genes <- names(sort(pcc, decreasing = TRUE))[1:1000]
```

## Evaluation signals

- Lymphocyte-count acceptance check: scPagwas over-expression genes yield 35 significant BP terms (FDR < 0.01) vs 0 for MAGMA.
- Comparison table scPagwas_GO_enrichment_TableS3.csv reproduces the Table S3 pattern where scPagwas top genes out-enrich all four baselines across blood traits.
- Each top-gene set contains exactly 1,000 genes (n_topgenes = 1000) ranked by PCC against the summed gPAS score.
- Same GO BP over-representation pipeline and blood-trait gene-set background applied identically to scPagwas and all four baseline gene sets, so term counts are directly comparable.
- Reproduction code available at github.com/dengchunyu/scPagwas_reproduce (Zenodo DOI 10.5281/zenodo.8137370).

## Limitations

- Statistical association between traits and genes/cells does not imply causality but may reflect indirect identification of causal associations.
- Pathway analysis restricted to canonical KEGG pathways; SNP-to-gene annotation uses only proximal distance (20 kb window), limiting functional linking.
- The reproducibility paper comparison depends on per-trait GWAS summary statistic quality; only 10 blood cell traits were benchmarked.

## Evidence
