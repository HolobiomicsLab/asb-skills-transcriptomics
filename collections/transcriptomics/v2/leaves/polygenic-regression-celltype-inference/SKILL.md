---
name: polygenic-regression-celltype-inference
description: Use when you have GWAS summary statistics and a cell-type-annotated scRNA-seq
  dataset (Seurat object or AnnData) and need to identify which cell types or single
  cells are statistically associated with a complex trait.
when_to_use_negative:
- When a statistical association between the trait and individual cells is intended
  to be interpreted as causal — the method identifies association only
- When the analysis requires functionally informed SNP-to-gene linking beyond proximal
  distance — scPagwas annotates SNPs solely by a 20 kb window
- When non-KEGG or non-canonical pathway knowledge is required — the default pathway
  analysis is restricted to canonical KEGG pathways
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3517
tools:
- name: scPagwas
  role: 'Core polygenic regression method: pathway activation transformation of scRNA-seq
    data combined with GWAS summary statistics to infer trait-relevant cell types
    and single cells'
  repo: https://github.com/dengchunyu/scPagwas
- name: scPagwas_py
  role: Python implementation synchronized with scPagwas2.0, used to accommodate the
    substantial memory demands of single-cell data calculations
  repo: https://github.com/dengchunyu/scPagwas_py
- name: Seurat
  role: Provides the scRNA-seq object format and the AddModuleScore cell-scoring method
    used to compute the trait-relevance score (TRS)
  repo: https://github.com/satijalab/seurat
- name: Genes_by_pathway_kegg
  role: Package-provided KEGG pathway gene list used to annotate SNP-assigned genes
    to pathways
- name: block_annotation
  role: Package-provided gene position annotation (hg37/hg38) defining genome blocks
    for the block bootstrap
- name: chrom_ld
  role: Package-provided per-chromosome linkage disequilibrium data used in the regression
    and bootstrap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:b47ed814da17f1aad088a89948978fd6a30fdda05852b77918c24c750476a251
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
  - Genes_by_pathway_kegg
  - Seurat
  - block_annotation
  - chrom_ld
  - scPagwas
  - scPagwas_py
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# polygenic-regression-celltype-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply scPagwas, a pathway-based polygenic regression method that integrates GWAS summary statistics with pathway activation transformation of scRNA-seq data, to infer trait-relevant cell types and single cells (e.g., naive CD8+ T cells associated with severe COVID-19).

## When to use

Use when you have GWAS summary statistics and a cell-type-annotated scRNA-seq dataset (Seurat object or AnnData) and need to identify which cell types or single cells are statistically associated with a complex trait.

## When NOT to use

- When a statistical association between the trait and individual cells is intended to be interpreted as causal — the method identifies association only
- When the analysis requires functionally informed SNP-to-gene linking beyond proximal distance — scPagwas annotates SNPs solely by a 20 kb window
- When non-KEGG or non-canonical pathway knowledge is required — the default pathway analysis is restricted to canonical KEGG pathways

## Inputs

- GWAS summary statistics formatted as a six-column tab-delimited txt file
- Preprocessed scRNA-seq data in Seurat format (.rds with 'RNA' assay, normalized) or AnnData for scPagwas_py
- Cell-type annotations assigned to Seurat Idents
- Pathway list (Genes_by_pathway_kegg)
- block_annotation (hg37 or hg38 gene positions, package-provided)
- chrom_ld (per-chromosome LD data, package-provided)

## Outputs

- bootstrap_results data frame with block-bootstrap p-values and confidence intervals per cell type (e.g., covid_b2_celltype_bootstrap_results.csv)
- Single-cell TRS significance p-values
- Per-pathway/per-cell gPAS coefficients
- List of top 1,000 trait-relevant genes

## How to apply

Format the GWAS summary statistics as a six-column tab-delimited txt file and filter SNPs with maf_filter = 0.01, removing SNPs on sex chromosomes (ChrX-Y) and the MHC region (Chr6: 25-35 Mbp). Run scPagwas_main with Pathway_list = Genes_by_pathway_kegg, the package-provided block_annotation and chrom_ld, marg = 10000 (20 kb SNP-to-gene window), and pathway size limits of 5-300 genes, with singlecell = TRUE and celltype = TRUE. The method applies SVD-based pathway activation transformation to the normalized gene-by-cell matrix, fits a method-of-moments polygenic regression to estimate per-pathway per-cell gPAS coefficients, selects the top 1,000 trait-relevant genes by Pearson correlation with summed gPASs, and computes a TRS via Seurat's AddModuleScore. Assess cell-type significance using block bootstrap with iters_celltype = 200 over pathway-based genome blocks, and single-cell significance with iters_singlecell = 100 and n_topgenes = 1000 via Monte Carlo testing against control TRSs. Extract the bootstrap_results data frame with per-cell-type p-values and confidence intervals and call cell types significant at FDR < 0.05.

## Related tools

- **scPagwas** (Core polygenic regression method: pathway activation transformation of scRNA-seq data combined with GWAS summary statistics to infer trait-relevant cell types and single cells) — https://github.com/dengchunyu/scPagwas
- **scPagwas_py** (Python implementation synchronized with scPagwas2.0, used to accommodate the substantial memory demands of single-cell data calculations) — https://github.com/dengchunyu/scPagwas_py
- **Seurat** (Provides the scRNA-seq object format and the AddModuleScore cell-scoring method used to compute the trait-relevance score (TRS)) — https://github.com/satijalab/seurat
- **Genes_by_pathway_kegg** (Package-provided KEGG pathway gene list used to annotate SNP-assigned genes to pathways)
- **block_annotation** (Package-provided gene position annotation (hg37/hg38) defining genome blocks for the block bootstrap)
- **chrom_ld** (Package-provided per-chromosome linkage disequilibrium data used in the regression and bootstrap)

## Examples

```
Pagwas_data <- scPagwas_main2(Pagwas=NULL, gwas_data="GWAS_summ_example.txt", Single_data="scRNAexample.rds", Pathway_list=Genes_by_pathway_kegg, chrom_ld=chrom_ld, block_annotation=block_annotation_hg37, singlecell=TRUE, celltype=TRUE)
```

## Evaluation signals

- Reproduced severe COVID-19 associations recover naive CD8+ T cells (p = 4.6e-17), megakaryocytes (p = 7.8e-6), and CD16+ monocytes (p = 1.14e-4) as significant at FDR < 0.05, matching Table S8 / Figure 5A-D
- bootstrap_results contains a p-value and confidence interval for every annotated cell type
- All analyzed pathways contain between 5 and 300 genes (min.pathway.size = 5, max.pathway.size = 300)
- GWAS input has no SNPs with MAF < 0.01, no sex chromosome (ChrX-Y) SNPs, and no SNPs in the MHC region (Chr6: 25-35 Mbp)
- Cell-type-level bootstrap uses 200 iterations (iters_celltype = 200) and single-cell Monte Carlo testing uses iters_singlecell = 100 with n_topgenes = 1000

## Limitations

- Identification of a statistical association between complex diseases/traits and individual cells does not imply causality but may reflect indirect identification of causal associations
- Pathway analysis is restricted to canonical pathways identified in the KEGG database
- SNPs are annotated to genes solely by proximal distance (20 kb window); functionally informed SNP-to-gene linking is deferred to future work
- For substantial memory demands of single-cell data calculations in R, the Python version scPagwas_py (synchronized with scPagwas2.0) may be needed
- scPagwas2 requires the scPagwas_main2 function to replace the original scPagwas_main

## Evidence

- [results] COVID-19 finding (source card finding, reproduced via workflow): [evidence span withheld — anchor status: not_found]
