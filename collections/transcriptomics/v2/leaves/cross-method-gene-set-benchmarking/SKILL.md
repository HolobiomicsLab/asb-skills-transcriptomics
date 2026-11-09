---
name: cross-method-gene-set-benchmarking
description: Use when you have a ground-truth scRNA-seq dataset with known trait-relevant
  cells and want to test whether one method's prioritized gene set improves a second
  scorer (e.g., scDRS) relative to a baseline gene set (e.g., MAGMA-derived).
when_to_use_negative:
- No ground-truth cell labels exist, so retrieval precision cannot be computed and
  only permutation-based or simulated benchmarks (e.g., scDesign2-simulated data)
  are possible
- Input is already a ranked gene set from a single method and the question is only
  cell-type-level association (use scPagwas celltype mode or MAGMA/LDSC-SEG instead)
- The GWAS trait has no biologically plausible cell-type ground truth in the scRNA-seq
  data, making precision-at-q meaningless
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: scPagwas
  role: Prioritizes top 1,000 trait-relevant genes via Pearson correlation of expression
    with summed gPAS (scGet_PCC) and computes TRS with Seurat AddModuleScore; its
    gene set is the treatment arm of the benchmark
  repo: https://github.com/dengchunyu/scPagwas
- name: scDRS 1.0.2
  role: Held-constant downstream cell scorer run twice — once with MAGMA genes (baseline
    precision 0.787), once with scPagwas genes (precision 0.984)
  repo: https://github.com/martinjzhang/scDRS
- name: MAGMA
  role: Source of the baseline GWAS-prioritized disease gene set fed to scDRS for
    comparison
  repo: https://github.com/gusevlab/fusion_twas
- name: Seurat AddModuleScore
  role: Default cell-scoring engine used by scPagwas to compute the trait-relevance
    score (TRS) on the BMMC object
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:b7759266839df99bac25a62e5aa6424630e7dc10496ddee97300a3b0a1c6c0a3
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
  - Seurat AddModuleScore
  - scDRS 1.0.2
  - scPagwas
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cross-method-gene-set-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a cell-scoring method's prioritized gene set (scPagwas top 1,000 trait-relevant genes) against an alternative gene-discovery source (MAGMA) by feeding both into the same downstream scorer (scDRS 1.0.2) and measuring retrieval precision against ground-truth cell labels on the real BMMC monocyte count dataset.

## When to use

Use when you have a ground-truth scRNA-seq dataset with known trait-relevant cells and want to test whether one method's prioritized gene set improves a second scorer (e.g., scDRS) relative to a baseline gene set (e.g., MAGMA-derived).

## When NOT to use

- No ground-truth cell labels exist, so retrieval precision cannot be computed and only permutation-based or simulated benchmarks (e.g., scDesign2-simulated data) are possible
- Input is already a ranked gene set from a single method and the question is only cell-type-level association (use scPagwas celltype mode or MAGMA/LDSC-SEG instead)
- The GWAS trait has no biologically plausible cell-type ground truth in the scRNA-seq data, making precision-at-q meaningless

## Inputs

- BMMC scRNA-seq Seurat object with ground-truth cell-type labels (5,000 monocytes vs T, B, DC, NK cells)
- Preprocessed monocyte count GWAS summary statistics (pruned tab-delimited text, six columns; IEU ID ieu-b-31)
- MAGMA-derived disease gene set for scDRS baseline
- KEGG pathway list (Genes_by_pathway_kegg) and chromosome LD/block annotation

## Outputs

- scPagwas TRS per-cell scores with background-corrected p-values (Get_CorrectBg_p, 100 iterations)
- Two scDRS 1.0.2 score vectors per cell: one from MAGMA genes, one from scPagwas top 1,000 genes
- Retrieval precision at q<0.05 for each method/gene-set combination against ground-truth monocyte labels

## How to apply

Run scPagwas with singlecell=TRUE on the BMMC Seurat object with monocyte count GWAS (maf_filter=0.01, 20 kb SNP-to-gene window, pathways of 5-300 genes, n_topgenes=1000, iters_singlecell=100), prioritizing genes by Pearson correlation of expression with summed gPAS (scGet_PCC) and computing TRS via Seurat AddModuleScore with Get_CorrectBg_p background correction (100 iterations). Then run scDRS 1.0.2 twice on the same cell subset: once with the MAGMA-derived disease gene set and once with the scPagwas top 1,000 genes. Define positive cells at q<0.05, rank cells by TRS/scDRS score, and compute retrieval precision against ground-truth labels (5,000 monocytes vs non-monocyte T, B, DC, NK cells). The rationale is that scDRS accuracy depends heavily on the input disease-specific genes, so holding the scorer constant isolates the gene-set contribution.

## Related tools

- **scPagwas** (Prioritizes top 1,000 trait-relevant genes via Pearson correlation of expression with summed gPAS (scGet_PCC) and computes TRS with Seurat AddModuleScore; its gene set is the treatment arm of the benchmark) — https://github.com/dengchunyu/scPagwas
- **scDRS 1.0.2** (Held-constant downstream cell scorer run twice — once with MAGMA genes (baseline precision 0.787), once with scPagwas genes (precision 0.984)) — https://github.com/martinjzhang/scDRS
- **MAGMA** (Source of the baseline GWAS-prioritized disease gene set fed to scDRS for comparison) — https://github.com/gusevlab/fusion_twas
- **Seurat AddModuleScore** (Default cell-scoring engine used by scPagwas to compute the trait-relevance score (TRS) on the BMMC object) — https://github.com/satijalab/seurat

## Examples

```
Pagwas <- scPagwas_main2(Pagwas=NULL, gwas_data="monocyte_count_pruned.txt", Single_data="BMMC.rds", singlecell=T, celltype=T, Pathway_list=Genes_by_pathway_kegg, n_topgenes=1000, iters_singlecell=100)
```

## Evaluation signals

- scPagwas TRS-based retrieval precision on the real BMMC ground truth should reproduce approximately 98.2% (Figure 3F)
- scDRS precision should improve from about 0.787 with the MAGMA gene set to about 0.984 with the scPagwas top 1,000 gene set (Figures 3G-3H)
- Positive cells are defined at q<0.05 for both scorers; identical q threshold and identical cell subset must be used for a fair comparison
- The scPagwas run must use the default parameters: maf_filter=0.01, marg=10000 (20 kb window), pathway size 5-300, n_topgenes=1000, iters_singlecell=100
- Gene prioritization must come from scGet_PCC (Pearson correlation with summed gPAS), not from raw GWAS p-values

## Limitations

- Statistical association between traits and individual cells does not imply causality
- Pathway analysis is restricted to canonical KEGG pathways only
- scDRS accuracy depends heavily on the input disease-specific gene set, so the benchmark measures gene-set quality conditional on the scorer rather than absolute method performance
- MAGMA-prioritized genes come from bulk gene-based association tests that neglect within-cell-type heterogeneity

## Evidence
