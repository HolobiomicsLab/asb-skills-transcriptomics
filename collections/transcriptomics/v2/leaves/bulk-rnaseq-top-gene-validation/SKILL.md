---
name: bulk-rnaseq-top-gene-validation
description: Use when a single-cell trait-association pipeline (scPagwas gPAS/TRS)
  has ranked candidate risk genes and you need external bulk-expression evidence that
  they are disease-relevant.
when_to_use_negative:
- No independent bulk disease-vs-control expression data exist for the trait, so genes
  can only be assessed via GWAS/single-cell evidence alone
- The goal is cell-type-level or single-cell-level association (FDR via Boot_evaluate),
  not gene-level external validation
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: scPagwas
  role: Produces the ranked trait-relevant genes (via scGet_PCC heritability-expression
    correlation) that are then validated in bulk datasets
  repo: https://github.com/dengchunyu/scPagwas
- name: Seurat
  role: Runs the AddModuleScore-based TRS whose top genes feed into the bulk validation
    step
  repo: https://github.com/satijalab/seurat
- name: Seurat AddModuleScore function
  role: Computes the trait-relevant score used to prioritize genes prior to bulk-expression
    validation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:b1d97129642ca2e7a70f93b0c87ea0b865c7a125f8a91a813afcd490accd7ddd
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
  - Seurat
  - Seurat AddModuleScore function
  - scPagwas
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# bulk-rnaseq-top-gene-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates trait-relevant genes prioritized by scPagwas (e.g., CREB1 and GSK3B ranked by scGet_PCC heritability correlation) by testing their differential expression in independent bulk RNA-seq disease datasets such as GSE15222 and GSE109887 for Alzheimer's disease.

## When to use

Use when a single-cell trait-association pipeline (scPagwas gPAS/TRS) has ranked candidate risk genes and you need external bulk-expression evidence that they are disease-relevant.

## When NOT to use

- No independent bulk disease-vs-control expression data exist for the trait, so genes can only be assessed via GWAS/single-cell evidence alone
- The goal is cell-type-level or single-cell-level association (FDR via Boot_evaluate), not gene-level external validation

## Inputs

- Top-ranked trait-relevant genes from scGet_PCC (Pearson correlation of expression with gPASsum, top 1,000 genes selected)
- Independent bulk RNA-seq/microarray disease expression datasets from GEO (e.g., GSE15222, GSE109887 for Alzheimer's disease)

## Outputs

- Per-gene differential expression significance results in disease vs. control bulk samples
- Validation status for each top-ranked gene (e.g., significant up-regulation of CREB1 and GSK3B in GSE15222 and GSE109887)

## How to apply

After running scPagwas_perform_score to obtain gPAS/TRS and ranking genes with scGet_PCC by the Pearson correlation of gene expression with summed gPASs, extract the top-ranked AD genes (in the AD demonstration these were CREB1 and GSK3B from the top 1,000 correlated genes). Then retrieve the relevant disease bulk expression datasets from GEO — for Alzheimer's disease the paper used GSE15222 and GSE109887 — and test whether each top gene is significantly up-regulated in disease versus control samples. Significant up-regulation of CREB1 and GSK3B in both datasets is treated as confirming that the pathway-based polygenic regression prioritized biologically meaningful, disease-relevant genes rather than statistical artifacts.

## Related tools

- **scPagwas** (Produces the ranked trait-relevant genes (via scGet_PCC heritability-expression correlation) that are then validated in bulk datasets) — https://github.com/dengchunyu/scPagwas
- **Seurat** (Runs the AddModuleScore-based TRS whose top genes feed into the bulk validation step) — https://github.com/satijalab/seurat
- **Seurat AddModuleScore function** (Computes the trait-relevant score used to prioritize genes prior to bulk-expression validation)

## Examples

```
Pagwas <- scPagwas_perform_score(Pagwas, singlecell = TRUE); top_genes <- scGet_PCC(Pagwas); # then validate CREB1 and GSK3B up-regulation in GSE15222 and GSE109887
```

## Evaluation signals

- Top genes extracted correspond to the highest scGet_PCC heritability-expression correlations (top 1,000 gene set)
- CREB1 and GSK3B show significant up-regulation in both GSE15222 and GSE109887 disease-vs-control comparisons
- Validated genes are members of KEGG pathways retained by the 5–300 gene size filter used in the scPagwas run
- Direction of expression change in bulk data (up-regulation in AD) is consistent across independent datasets

## Limitations

- Statistical association between traits and individual cells/genes does not imply causality but may reflect indirect associations
- Pathway analysis is restricted to canonical KEGG pathways only, which constrains which genes can be prioritized and subsequently validated
- Genes are validated for association/differential expression, not for mechanistic causality of disease

## Evidence

- [other] Top-ranked AD genes CREB1 and GSK3B validation: [evidence span withheld — anchor status: not_found]
- [other] Bulk dataset validation: [evidence span withheld — anchor status: not_found]
- [discussion] Gene prioritization method: "we determine which gene g exhibits expression that is highly correlated with the summed gPASs (gPASsum) across individual cells using the Pearson correlation method"
- [abstract] OPC/microglia finding context: "oligodendrocyte progenitor cell and microglia subsets with critical pathways by which genetic variants influence Alzheimer’s disease."
