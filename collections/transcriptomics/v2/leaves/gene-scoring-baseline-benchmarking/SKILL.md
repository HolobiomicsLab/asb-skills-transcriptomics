---
name: gene-scoring-baseline-benchmarking
description: Use when you need to show that genes prioritized by a new GWAS-by-scRNA-seq
  method are more functionally enriched than top genes from established gene-scoring
  baselines such as MAGMA or eQTL-based TWAS methods.
when_to_use_negative:
- Your goal is to identify trait-relevant cell types or individual cells rather than
  to compare prioritized gene lists across methods — use the scPagwas single-cell/cell-type
  workflow instead
- You lack per-gene scores from the baselines (MAGMA, TWAS, S-PrediXcan, S-MultiXcan)
  for the same traits, so no matched gene-set comparison is possible
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scPagwas
  role: Prioritizes trait-relevant genes via Pearson correlation of gene expression
    with summed gPASs (scGet_PCC); top 1,000 genes form the method's gene sets under
    test
  repo: https://github.com/dengchunyu/scPagwas
- name: Seurat
  role: Provides the Seurat object format for the BMMC single-cell data (Seu_Hema_data.rds)
    and the AddModuleScore-based TRS used in the scPagwas pipeline
  repo: https://github.com/satijalab/seurat
- name: MAGMA
  role: Gene-based association baseline whose top genes are rebuilt into trait gene
    sets and run through the identical GO enrichment pipeline for comparison
- name: TWAS
  role: eQTL-based gene-scoring baseline for rebuilding trait gene sets in the comparison
  repo: https://github.com/gusevlab/fusion_twas
- name: S-PrediXcan
  role: eQTL-based gene-scoring baseline for rebuilding trait gene sets in the comparison
  repo: https://github.com/hakyimlab/MetaXcan
- name: S-MultiXcan
  role: eQTL-based gene-scoring baseline for rebuilding trait gene sets in the comparison
  repo: https://github.com/hakyimlab/MetaXcan
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:a53a0c671c0ea583044e8420254943e5c43d315546b370953c172ca25c6d1750
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
  - S-MultiXcan
  - S-PrediXcan
  - Seurat
  - TWAS
  - scPagwas
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gene-scoring-baseline-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark scPagwas-prioritized trait-relevant genes against gene-scoring baselines (MAGMA, TWAS, S-PrediXcan, S-MultiXcan) by running an identical GO biological-process over-representation pipeline on each method's top 1,000 genes per trait and comparing counts of significant BP terms.

## When to use

Use when you need to show that genes prioritized by a new GWAS-by-scRNA-seq method are more functionally enriched than top genes from established gene-scoring baselines such as MAGMA or eQTL-based TWAS methods.

## When NOT to use

- Your goal is to identify trait-relevant cell types or individual cells rather than to compare prioritized gene lists across methods — use the scPagwas single-cell/cell-type workflow instead
- You lack per-gene scores from the baselines (MAGMA, TWAS, S-PrediXcan, S-MultiXcan) for the same traits, so no matched gene-set comparison is possible

## Inputs

- BMMC Seurat object (Seu_Hema_data.rds)
- Per-trait scPagwas results containing scPagwas.gPAS.score
- GWAS summary statistics for 10 blood cell traits (e.g., IEU IDs ieu-b-31, ebi-a-GCST004627, ieu-b-34, ieu-b-30, ieu-b-33, ieu-b-29, ebi-a-GCST90002335, ebi-a-GCST004632, ebi-a-GCST90002329, ebi-a-GCST90002311)
- Baseline gene scores from MAGMA, TWAS, S-PrediXcan, and S-MultiXcan
- GO biological-process gene sets and blood-trait gene-set backgrounds

## Outputs

- scPagwas_GO_enrichment_TableS3.csv: significant BP-term counts per method per trait
- Top 1,000 trait-relevant gene sets per trait per method
- GO BP over-representation results with FDR values

## How to apply

For each of the 10 blood traits, load the BMMC Seurat object (Seu_Hema_data.rds) and the per-trait scPagwas results containing scPagwas.gPAS.score, then compute each gene's Pearson correlation with the genetic score using scPagwas::scGet_PCC and take the top 1,000 genes (n_topgenes = 1000) as the trait-relevant gene set. Run GO biological-process over-representation enrichment on each top-gene set against blood-trait gene-set backgrounds, counting significant BP terms among over-expressed genes at FDR < 0.01. Rebuild the same trait gene sets using the four baselines — MAGMA and the eQTL-based TWAS, S-PrediXcan, and S-MultiXcan — and run the identical enrichment pipeline so the only varying factor is the gene-scoring method. Tabulate significant-term counts per method per trait and save as scPagwas_GO_enrichment_TableS3.csv; the rationale is that scPagwas's pathway-activation-based gene prioritization should yield gene sets more enriched in blood-trait functional gene sets than LD- or eQTL-based gene scores.

## Related tools

- **scPagwas** (Prioritizes trait-relevant genes via Pearson correlation of gene expression with summed gPASs (scGet_PCC); top 1,000 genes form the method's gene sets under test) — https://github.com/dengchunyu/scPagwas
- **Seurat** (Provides the Seurat object format for the BMMC single-cell data (Seu_Hema_data.rds) and the AddModuleScore-based TRS used in the scPagwas pipeline) — https://github.com/satijalab/seurat
- **MAGMA** (Gene-based association baseline whose top genes are rebuilt into trait gene sets and run through the identical GO enrichment pipeline for comparison)
- **TWAS** (eQTL-based gene-scoring baseline for rebuilding trait gene sets in the comparison) — https://github.com/gusevlab/fusion_twas
- **S-PrediXcan** (eQTL-based gene-scoring baseline for rebuilding trait gene sets in the comparison) — https://github.com/hakyimlab/MetaXcan
- **S-MultiXcan** (eQTL-based gene-scoring baseline for rebuilding trait gene sets in the comparison) — https://github.com/hakyimlab/MetaXcan

## Examples

```
Rscript -e 'library(scPagwas); pcc <- scGet_PCC(Single_data=Seu_Hema_data, Pagwas=trait_results, n_topgenes=1000)'
```

## Evaluation signals

- The comparison table reproduces the Table S3 pattern: scPagwas top genes are substantially more enriched in blood-trait gene sets than all four baselines across the 10 blood traits
- Lymphocyte count over-expression genes yield 35 significant BP terms (FDR < 0.01) for scPagwas versus 0 for MAGMA — the acceptance check
- Exactly 1,000 genes are selected per trait per method (n_topgenes = 1000) and each set is enriched against the same blood-trait gene-set background
- Output file scPagwas_GO_enrichment_TableS3.csv contains one row per method per trait with significant BP-term counts

## Limitations

- Enrichment conclusions depend on the blood-trait gene-set backgrounds chosen; the comparison is over-representation analysis, not a direct association test
- MAGMA-based baselines rely on disease/trait-specific genes from GWAS gene-based association tests, whose accuracy the article notes scDRS-like approaches depend heavily on
- scPagwas gene prioritization is itself dependent on the KEGG pathway annotation (canonical pathways, 5–300 gene size limits), which bounds which genes can be prioritized

## Evidence

- [results] scPagwas-identified trait-relevant genes were more highly enriched in functional gene sets related to blood cell traits than genes identified by the four other gene-scoring methods: [evidence span withheld — anchor status: not_found]
- [results] for lymphocyte count over-expression genes, scPagwas enriched 35 significant BP terms (FDR < 0.01) versus 0 for MAGMA: [evidence span withheld — anchor status: not_found]
