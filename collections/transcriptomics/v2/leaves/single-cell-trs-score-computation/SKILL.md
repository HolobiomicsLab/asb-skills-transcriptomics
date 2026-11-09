---
name: single-cell-trs-score-computation
description: Use when a ground-truth scRNA-seq dataset with cell-type labels and pruned
  GWAS summary statistics (e.g. monocyte count GWAS) must be scored per cell to measure
  how precisely trait-relevant cells are retrieved.
when_to_use_negative:
- The input is a bulk RNA-seq or precomputed feature table without individual-cell
  expression — AddModuleScore and per-cell gPAS require single-cell resolution.
- GWAS summary statistics are unpruned or lack required columns (MAF, SNP positions)
  — maf_filter=0.01 and the 20 kb SNP-to-gene window assume pruned, annotated summary
  stats.
- The goal is causal inference for trait-cell associations — the article states statistical
  association between traits and individual cells does not imply causality.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scPagwas
  role: Runs polygenic regression to estimate per-cell gPAS and prioritize trait-relevant
    genes by Pearson correlation with summed gPAS (scGet_PCC)
  repo: https://github.com/dengchunyu/scPagwas
- name: Seurat AddModuleScore function
  role: Computes the per-cell TRS from the scPagwas-prioritized top 1,000 genes with
    control gene sets matched by expression bins
  repo: https://github.com/satijalab/seurat
- name: scDRS 1.0.2
  role: Comparison cell-scoring method run with MAGMA-derived genes vs scPagwas-prioritized
    top genes to benchmark TRS precision
  repo: https://github.com/martinjzhang/scDRS
- name: MAGMA
  role: Provides the baseline gene set (MAGMA-prioritized disease genes) used for
    the scDRS precision comparison
  repo: https://github.com/gusevlab/fusion_twas
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:501d3b11c0935541554fd4bf5d9588b36aef704ac072d279c82bec2da00b3ad1
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
  - Seurat AddModuleScore function
  - scDRS 1.0.2
  - scPagwas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-trs-score-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-cell trait-relevance scores (TRS) for single-cell RNA-seq data by prioritizing genes via Pearson correlation of expression with summed gPAS from scPagwas polygenic regression, scoring cells with Seurat's AddModuleScore, and assigning background-corrected significance. Enables benchmarking of trait-relevant cell identification against ground-truth labels, e.g. monocyte count-relevant cells in BMMC data.

## When to use

Use when a ground-truth scRNA-seq dataset with cell-type labels and pruned GWAS summary statistics (e.g. monocyte count GWAS) must be scored per cell to measure how precisely trait-relevant cells are retrieved.

## When NOT to use

- The input is a bulk RNA-seq or precomputed feature table without individual-cell expression — AddModuleScore and per-cell gPAS require single-cell resolution.
- GWAS summary statistics are unpruned or lack required columns (MAF, SNP positions) — maf_filter=0.01 and the 20 kb SNP-to-gene window assume pruned, annotated summary stats.
- The goal is causal inference for trait-cell associations — the article states statistical association between traits and individual cells does not imply causality.

## Inputs

- BMMC scRNA-seq Seurat object with 'RNA' assay and normalized expression
- Preprocessed monocyte count GWAS summary statistics (IEU ID: ieu-b-31), pruned tab-delimited text with six columns
- KEGG pathway gene list (Genes_by_pathway_kegg)
- Chromosome LD reference (1,000 Genomes Phase 3 Panel) and block annotation (hg37/hg38)
- Ground-truth cell-type labels (5,000 monocytes vs T, B, DC, NK cells)

## Outputs

- Per-cell trait-relevance scores (TRS) from Seurat AddModuleScore
- Background-corrected per-cell p-values and q-values (Get_CorrectBg_p, 100 iterations)
- Top 1,000 prioritized trait-relevant genes ranked by scGet_PCC Pearson correlation
- Retrieval precision of trait-relevant cell identification at q<0.05 (target ~98.2% for scPagwas; 0.787 vs 0.984 for scDRS with MAGMA vs scPagwas genes)

## How to apply

Run the scPagwas pipeline with singlecell=TRUE (maf_filter=0.01, 20 kb SNP-to-gene window, pathways of 5-300 genes, n_topgenes=1000, iters_singlecell=100, KEGG pathway list) to obtain per-cell gPAS coefficients. Prioritize trait-relevant genes by Pearson correlation of each gene's expression with the summed gPAS (scGet_PCC) and take the top 1,000 genes. Compute the TRS per cell with Seurat's AddModuleScore using control gene sets matched by expression bins, then derive background-corrected per-cell p-values via Get_CorrectBg_p over 100 Monte Carlo iterations against control TRSs. Define positive cells at q<0.05, rank cells by TRS score, and compute retrieval precision against ground-truth labels (e.g. 5,000 monocytes vs non-monocyte T, B, DC, NK cells in BMMC). Optionally feed the scPagwas-prioritized top genes into scDRS 1.0.2 as the gene set to compare against MAGMA-derived disease genes.

## Related tools

- **scPagwas** (Runs polygenic regression to estimate per-cell gPAS and prioritize trait-relevant genes by Pearson correlation with summed gPAS (scGet_PCC)) — https://github.com/dengchunyu/scPagwas
- **Seurat AddModuleScore function** (Computes the per-cell TRS from the scPagwas-prioritized top 1,000 genes with control gene sets matched by expression bins) — https://github.com/satijalab/seurat
- **scDRS 1.0.2** (Comparison cell-scoring method run with MAGMA-derived genes vs scPagwas-prioritized top genes to benchmark TRS precision) — https://github.com/martinjzhang/scDRS
- **MAGMA** (Provides the baseline gene set (MAGMA-prioritized disease genes) used for the scDRS precision comparison) — https://github.com/gusevlab/fusion_twas

## Examples

```
Pagwas_data <- scPagwas_main2(Pagwas=NULL, gwas_data="GWAS_summ.txt", Single_data="BMMC.rds", singlecell=TRUE, iters_singlecell=100, Pathway_list=Genes_by_pathway_kegg, chrom_ld=chrom_ld)
```

## Evaluation signals

- TRS-based retrieval precision against ground-truth monocyte labels should reproduce approximately 98.2% (Figure 3F) on the real BMMC benchmark.
- scDRS precision with scPagwas top 1,000 genes should improve to about 0.984 versus about 0.787 with MAGMA genes (Figures 3G-3H).
- The prioritized gene set contains exactly n_topgenes=1000 genes ranked by scGet_PCC Pearson correlation with summed gPAS.
- Per-cell p-values from Get_CorrectBg_p must come from 100 background iterations, with positive cells defined at q<0.05.
- GWAS filtering invariants: SNPs with MAF < 0.01 and on ChrX-Y removed; pathways limited to 5-300 genes.

## Limitations

- Statistical association between traits and individual cells does not imply causality but may reflect indirect identification of causal associations.
- Default TRS relies on Seurat's AddModuleScore cell-scoring method for Seurat compatibility, which may not be optimal for all datasets.
- Extensive sparsity and technical noise in single-cell data mean top genetic association signals at specific loci may be absent from most cells, making per-cell inference difficult for gene-scoring methods.
- Substantial memory demands arise when running single-cell calculations in R; a Python version (scPagwas_py) synchronized with scPagwas2.0 is available for computational efficiency.

## Evidence

- [readme] Seurat input format: [evidence span withheld — anchor status: external_doc]
