---
name: pearson-correlation-gene-ranking
description: Use when you have scPagwas per-cell gPAS scores for a GWAS trait and
  a normalized scRNA-seq matrix, and need to derive a ranked trait-relevant gene set
  (e.g., top 1,000 genes) for downstream enrichment or gene-set comparisons.
when_to_use_negative:
- When no per-cell gPAS scores exist yet — the correlation requires scPagwas's pathway-based
  polygenic regression output, not raw GWAS summary statistics
- When the goal is cell-type-level association (TRS/cell-type inference) rather than
  gene-level prioritization
- When genes must be linked to variants by functionally informed mapping — scPagwas
  annotates SNPs to genes only by proximal distance with a 20 kb window
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: scPagwas
  role: Provides scGet_PCC to compute per-gene Pearson correlation of expression with
    summed gPASs and ranks/extracts top trait-relevant genes
  repo: https://github.com/dengchunyu/scPagwas
- name: Seurat
  role: Supplies the normalized single-cell expression matrix (Seurat object, 'RNA'
    assay) and the AddModuleScore-based TRS used alongside gene prioritization
  repo: https://github.com/satijalab/seurat
- name: MAGMA
  role: Baseline gene-scoring method used to build comparison top-gene sets evaluated
    against the scPagwas PCC-ranked genes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:b9d36a627cd32b15cd2ce8944b0a4870673f6f2c670bdd0bcc56ababbe0aefa5
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pearson-correlation-gene-ranking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prioritize trait-relevant genes from scPagwas output by computing the Pearson correlation (PCC) of each gene's single-cell expression with the summed per-cell genetic pathway association scores (gPASsum), then selecting the top 1,000 genes per trait as the trait-relevant gene set.

## When to use

Use when you have scPagwas per-cell gPAS scores for a GWAS trait and a normalized scRNA-seq matrix, and need to derive a ranked trait-relevant gene set (e.g., top 1,000 genes) for downstream enrichment or gene-set comparisons.

## When NOT to use

- When no per-cell gPAS scores exist yet — the correlation requires scPagwas's pathway-based polygenic regression output, not raw GWAS summary statistics
- When the goal is cell-type-level association (TRS/cell-type inference) rather than gene-level prioritization
- When genes must be linked to variants by functionally informed mapping — scPagwas annotates SNPs to genes only by proximal distance with a 20 kb window

## Inputs

- Per-trait scPagwas results containing scPagwas.gPAS.score (per-cell genetic pathway association scores)
- Normalized scRNA-seq Seurat object (e.g., BMMC Seu_Hema_data.rds with 'RNA' assay)
- GWAS summary statistics for the trait (e.g., IEU IDs ieu-b-31 monocyte count, ebi-a-GCST004627 lymphocyte count)

## Outputs

- Ranked gene list per trait by expression–gPASsum Pearson correlation (PCC)
- Top 1,000 trait-relevant gene set per trait (n_topgenes = 1000)
- Comparison table of GO BP enrichment counts across methods (e.g., scPagwas_GO_enrichment_TableS3.csv)

## How to apply

Run scPagwas_main (or scPagwas_main2) on the GWAS summary statistics and a Seurat-normalized single-cell object to obtain per-cell gPAS scores, then use scPagwas::scGet_PCC to compute, for every gene, the Pearson correlation of its expression across cells with the summed gPASs (gPASsum). Rank genes by PCC and select the top 1,000 per trait (paper default n_topgenes = 1000) as the trait-relevant gene set. Rationale: pathway activation transformation (SVD of the variance-stabilized gene-by-cell matrix into a pathway-by-cell matrix) collapses sparse single-cell signals, so correlating gene expression back with gPASsum recovers genes whose expression tracks polygenic risk better than locus-level gene scoring alone. The resulting top-gene sets can be compared across gene-scoring methods (scPagwas vs MAGMA, TWAS, S-PrediXcan, S-MultiXcan) via GO BP over-representation to validate prioritization quality.

## Related tools

- **scPagwas** (Provides scGet_PCC to compute per-gene Pearson correlation of expression with summed gPASs and ranks/extracts top trait-relevant genes) — https://github.com/dengchunyu/scPagwas
- **Seurat** (Supplies the normalized single-cell expression matrix (Seurat object, 'RNA' assay) and the AddModuleScore-based TRS used alongside gene prioritization) — https://github.com/satijalab/seurat
- **MAGMA** (Baseline gene-scoring method used to build comparison top-gene sets evaluated against the scPagwas PCC-ranked genes)

## Examples

```
Pagwas <- scPagwas_main2(gwas_data='GWAS_summ.txt', Single_data='Seu_Hema_data.rds', singlecell=TRUE); gene_pcc <- scGet_PCC(Pagwas, Seu_Hema_data, n_topgenes=1000)
```

## Evaluation signals

- Correlating step reproduces per-gene PCC values against scPagwas.gPAS.score for all genes in the Seurat object
- Exactly n_topgenes = 1000 genes are selected per trait after PCC ranking
- GO BP over-representation on the top-1000 set yields substantially more significant terms (FDR < 0.01) than MAGMA baseline — acceptance check: lymphocyte count over-expression genes give 35 significant BP terms for scPagwas vs 0 for MAGMA
- Comparison table matches the Table S3 pattern: scPagwas top genes more enriched in blood-trait functional gene sets than MAGMA, TWAS, S-PrediXcan, and S-MultiXcan across the 10 blood traits

## Limitations

- SNPs are annotated to genes solely by proximal distance (20 kb window); functionally informed SNP-to-gene linking is deferred to future work
- Trait-relevant gene identification inherits scPagwas's restriction to canonical KEGG pathways (5–300 genes) used to compute the gPAS scores being correlated against
- Statistical association between traits and genes/cells does not imply causality and may reflect indirect associations

## Evidence

- [results] Baseline comparison workflow: [evidence span withheld — anchor status: not_found]
- [readme] Reproduction code availability: [evidence span withheld — anchor status: external_doc]
