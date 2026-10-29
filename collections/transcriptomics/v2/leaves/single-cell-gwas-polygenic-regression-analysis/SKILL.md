---
name: single-cell-gwas-polygenic-regression-analysis
description: Use when you have GWAS summary statistics (e.g., IEU ID ieu-b-2 AD GWAS)
  plus a cell-annotated snRNA-seq Seurat object (e.g., Grubman entorhinal cortex,
  11,786 cells) and need trait-relevant cell types and single cells.
when_to_use_negative:
- You only have a pre-computed feature/gene-set table without per-cell expression
  or individual-level GWAS summary statistics — scPagwas requires both raw inputs
- You need causal inference for cell-trait associations — the article explicitly notes
  statistical association between traits and individual cells does not imply causality
- Your biological question depends on non-canonical or non-KEGG pathway definitions
  — the method is restricted to canonical KEGG pathways (5-300 genes)
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: scPagwas
  role: 'Core polygenic regression pipeline: pathway SVD transformation, SNP-to-gene/pathway
    block annotation, LD linking, regression, bootstrap cell-type testing, TRS scoring'
  repo: https://github.com/dengchunyu/scPagwas
- name: scPagwas_py
  role: Python implementation fully synchronized with scPagwas2.0 for memory-heavy
    single-cell calculations
  repo: https://github.com/dengchunyu/scPagwas_py
- name: Seurat AddModuleScore function
  role: Computes the trait-relevant score (TRS) per cell with control gene sets matched
    by expression bins
  repo: https://github.com/satijalab/seurat
- name: Seurat
  role: Reads and holds the snRNA-seq Seurat object (RNA assay, normalized) consumed
    by scPagwas
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:f4e8e2a57ba414eeffda4fa22691a210bfc1a04949b65d1932be4d8237a9db68
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
  - scPagwas_py
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-gwas-polygenic-regression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply scPagwas, a pathway-based polygenic regression method, to integrate GWAS summary statistics with pathway-activity-transformed scRNA-seq data to identify trait-relevant cell types and individual cells, as demonstrated for Alzheimer's disease (OPCs and microglia, FDR < 0.05).

## When to use

Use when you have GWAS summary statistics (e.g., IEU ID ieu-b-2 AD GWAS) plus a cell-annotated snRNA-seq Seurat object (e.g., Grubman entorhinal cortex, 11,786 cells) and need trait-relevant cell types and single cells.

## When NOT to use

- You only have a pre-computed feature/gene-set table without per-cell expression or individual-level GWAS summary statistics — scPagwas requires both raw inputs
- You need causal inference for cell-trait associations — the article explicitly notes statistical association between traits and individual cells does not imply causality
- Your biological question depends on non-canonical or non-KEGG pathway definitions — the method is restricted to canonical KEGG pathways (5-300 genes)

## Inputs

- GWAS summary statistics file (e.g., ieu-b-2 Alzheimer's disease GWAS, tab/space-delimited)
- Cell-annotated scRNA-seq data in Seurat format with normalized 'RNA' assay (e.g., Grubman entorhinal cortex snRNA-seq, 11,786 cells)
- KEGG pathway gene list (Genes_by_pathway_kegg)
- Block annotation of gene positions (block_annotation_hg37 or hg38, from the package)
- 1,000 Genomes Phase 3 LD reference per chromosome (chrom_ld)

## Outputs

- Cell-type bootstrap association results (*_cellytpes_bootstrap_results.csv) with FDR-significant cell types
- Merged cell-type p-value table (Merged_celltype_pvalue via Merge_celltype_p)
- Gene PCC heritability-correlation ranking (*_gene_PCC.csv) with top trait-relevant genes
- Per-cell TRS (gPAS) scores and background-corrected p-values (single-cell score/p-value CSV)
- Saved Seurat result object (e.g., AD_scPagwas_results.RData)

## How to apply

Preprocess the GWAS file with GWAS_summary_input (maf_filter = 0.01, removing sex-chromosome SNPs) and the Seurat object with Single_data_input, then compute pathway SVD activity scores over the Genes_by_pathway_kegg list keeping pathways of 5-300 genes (Pathway_pcascore_run). Map SNPs to genes with SnpToGene using marg = 10000 (20 kb TSS window), drop the MHC region (Chr6: 25-35 Mbp), annotate SNP-gene-pathway blocks with Pathway_annotation_input, and link blocks to 1,000 Genomes Phase 3 LD data (chrom_ld) via Link_pathway_blocks_gwas(singlecell = TRUE, celltype = TRUE) before Pagwas_perform_regression. Estimate cell-type associations with Boot_evaluate (bootstrap_iters = 200; significance at FDR < 0.05), then run scPagwas_perform_score to obtain gPAS and TRS and Get_CorrectBg_p (iters_singlecell = 100, n_topgenes = 1000) for per-cell Monte Carlo p-values against control TRSs built with Seurat's AddModuleScore. Rank genes by scGet_PCC heritability correlation to extract top trait-relevant genes (e.g., CREB1 and GSK3B for AD) and validate their up-regulation in independent bulk datasets such as GSE15222 and GSE109887.

## Related tools

- **scPagwas** (Core polygenic regression pipeline: pathway SVD transformation, SNP-to-gene/pathway block annotation, LD linking, regression, bootstrap cell-type testing, TRS scoring) — https://github.com/dengchunyu/scPagwas
- **scPagwas_py** (Python implementation fully synchronized with scPagwas2.0 for memory-heavy single-cell calculations) — https://github.com/dengchunyu/scPagwas_py
- **Seurat AddModuleScore function** (Computes the trait-relevant score (TRS) per cell with control gene sets matched by expression bins) — https://github.com/satijalab/seurat
- **Seurat** (Reads and holds the snRNA-seq Seurat object (RNA assay, normalized) consumed by scPagwas) — https://github.com/satijalab/seurat

## Examples

```
Pagwas_data <- scPagwas_main2(Pagwas = NULL, gwas_data = "AD_gwas.txt", Single_data = "entorhinal_snRNA.rds", output.prefix = "AD", block_annotation = block_annotation_hg37, Pathway_list = Genes_by_pathway_kegg, chrom_ld = chrom_ld, singlecell = TRUE, celltype = TRUE)
```

## Evaluation signals

- OPCs and microglia reach cell-type significance at FDR < 0.05 in the bootstrap results (bootstrap_iters = 200) when run on AD GWAS (ieu-b-2) with the entorhinal cortex dataset
- GWAS filtering removes SNPs with MAF < 0.01 and sex-chromosome SNPs, and the MHC region (Chr6: 25-35 Mbp) is excluded from pathway block annotation
- Only KEGG pathways with 5-300 genes are retained after Pathway_pcascore_run (min.pathway.size = 5, max.pathway.size = 300)
- Top-ranked genes by scGet_PCC heritability correlation include CREB1 and GSK3B, and these show significant up-regulation in GSE15222 and GSE109887 AD bulk expression datasets
- Output files present and schema-consistent: *_cellytpes_bootstrap_results.csv, *_gene_PCC.csv, single-cell score/p-value CSV, merged cell-type p-value CSV, and AD_scPagwas_results.RData

## Limitations

- Identification of a statistical association between complex diseases/traits and individual cells does not imply causality but may reflect indirect identification of causal associations
- Pathway analysis is restricted to canonical KEGG pathways only; SNPs are annotated to genes solely by proximal distance (20 kb window), deferring functionally informed SNP-to-gene linking to future work
- Substantial memory demands for single-cell data calculations in R; the Python version (scPagwas_py) is recommended for large datasets
- The default TRS relies on Seurat's AddModuleScore for Seurat compatibility, which may constrain scoring flexibility

## Evidence

- [readme] wrapper invocation: [evidence span withheld — anchor status: external_doc]
