---
name: method-of-moments-polygenic-regression
description: Use when you need per-cell, pathway-level estimates of genetic association
  from GWAS summary statistics and a normalized single-cell Seurat object, e.g. to
  compute scPagwas.gPAS.score for trait-relevant cell identification.
when_to_use_negative:
- When the goal is cell-type-level (not single-cell) enrichment — use the block-bootstrap
  celltype branch or tools like LDSC-SEG/EPIC instead
- When disease-specific gene prioritization via MAGMA gene-based tests (as in scDRS)
  is preferred over pathway-based regression
- When the single-cell data are not normalized or pathway sizes fall outside 5-300
  genes, since the SVD-based PAS and pathway filtering assumptions are violated
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scPagwas
  role: Implements the method-of-moments per-pathway regression (Pagwas_perform_regression)
    and gPAS aggregation (scPagwas_perform_score)
  repo: https://github.com/dengchunyu/scPagwas
- name: Seurat
  role: Provides the single-cell data container and normalization; AddModuleScore
    used downstream for TRS scoring
  repo: https://github.com/satijalab/seurat
- name: scPagwas_py
  role: Python reimplementation, fully synchronized with scPagwas2.0, for memory-heavy
    single-cell calculations
  repo: https://github.com/dengchunyu/scPagwas_py
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:7555f983a802aca54e63c13797a4c5053ff5ffa17a629aa4125edf3ddeac1b7d
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
  - Pagwas_perform_regression
  - Seurat
  - SnpToGene
  - scPagwas
  - scPagwas_py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# method-of-moments-polygenic-regression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Per-pathway polygenic regression from scPagwas that links SVD-derived pathway activity scores (PAS) of single cells to GWAS summary statistics, estimating each pathway's genetically associated polygenic score (gPAS) per cell via a method-of-moments regression on SNP-pathway LD blocks.

## When to use

Use when you need per-cell, pathway-level estimates of genetic association from GWAS summary statistics and a normalized single-cell Seurat object, e.g. to compute scPagwas.gPAS.score for trait-relevant cell identification.

## When NOT to use

- When the goal is cell-type-level (not single-cell) enrichment — use the block-bootstrap celltype branch or tools like LDSC-SEG/EPIC instead
- When disease-specific gene prioritization via MAGMA gene-based tests (as in scDRS) is preferred over pathway-based regression
- When the single-cell data are not normalized or pathway sizes fall outside 5-300 genes, since the SVD-based PAS and pathway filtering assumptions are violated

## Inputs

- Normalized single-cell Seurat object (.rds) with an 'RNA' assay
- GWAS summary statistics text file (e.g. GWAS_summ_example.txt)
- KEGG pathway list (Genes_by_pathway_kegg), pathways of 5-300 genes
- Block annotation (block_annotation_hg37 or hg38) of gene positions
- Per-chromosome LD matrices (chrom_ld) from the 1000 Genomes Phase 3 Panel

## Outputs

- Pathway_single_results: pathways-by-cells matrix of per-pathway genetic contributions (tau-hat x PAS)
- Per-cell genetic score table (_singlecell_scPagwas_score_pvalue.Result.csv) with scPagwas.gPAS.score and pathway rank p-values
- Per-pathway regression output file (*_Pathway_singlecell_lm_results.txt)

## How to apply

First build the SVD-based pathway-by-cell score matrix with Pathway_pcascore_run on a Seurat object (via Single_data_input, min_clustercells=10), restricting to KEGG pathways of 5-300 genes. Map GWAS SNPs (maf_filter=0.01) to genes with SnpToGene using marg=10000 (a 20 kb window), annotate SNP-pathway blocks with Pathway_annotation_input, and link blocks to GWAS data via Link_pathway_blocks_gwas to build Pathway_ld_gwas_data with LD from the 1000 Genomes Phase 3 Panel. Fit the method-of-moments per-pathway regression with Pagwas_perform_regression to obtain Pathway_single_results, a pathways-by-cells matrix of tau-hat times PAS; the rationale is that method-of-moments improves computational efficiency and estimated uniform convergence compared to likelihood-based polygenic regression. Aggregate per-pathway contributions into per-cell gPAS by column sums, then compute pathway rank p-values with scPagwas_perform_score (remove_outlier=TRUE). Write the per-pathway regression matrix (*_Pathway_singlecell_lm_results.txt) and per-cell score table (_singlecell_scPagwas_score_pvalue.Result.csv) containing scPagwas.gPAS.score.

## Related tools

- **scPagwas** (Implements the method-of-moments per-pathway regression (Pagwas_perform_regression) and gPAS aggregation (scPagwas_perform_score)) — https://github.com/dengchunyu/scPagwas
- **Seurat** (Provides the single-cell data container and normalization; AddModuleScore used downstream for TRS scoring) — https://github.com/satijalab/seurat
- **scPagwas_py** (Python reimplementation, fully synchronized with scPagwas2.0, for memory-heavy single-cell calculations) — https://github.com/dengchunyu/scPagwas_py

## Examples

```
Pagwas_data <- scPagwas_main2(Pagwas=NULL, gwas_data="GWAS_summ_example.txt", Single_data="scRNAexample.rds", output.prefix="test", output.dirs="scPagwastest_output", block_annotation=block_annotation_hg37, Pathway_list=Genes_by_pathway_kegg, chrom_ld=chrom_ld, singlecell=TRUE)
```

## Evaluation signals

- Pathway_single_results is a pathways-by-cells matrix with one row per retained pathway (5-300 genes) and one column per cell
- scPagwas.gPAS.score is defined (non-NA) for every cell passing min_clustercells=10 filtering
- The output files *_Pathway_singlecell_lm_results.txt and _singlecell_scPagwas_score_pvalue.Result.csv exist in the output directory
- GWAS input satisfies maf_filter=0.01; SNPs were mapped to genes within the marg=10000 (20 kb) window
- Per-cell gPAS equals the column sums of Pathway_single_results, so recomputing column sums from the regression matrix must reproduce the reported scores

## Limitations

- Statistical association between traits and individual cells does not imply causality
- Pathway analysis is restricted to canonical KEGG pathways only
- SNPs are annotated to genes solely by proximal distance (20 kb window); functionally informed SNP-to-gene linking is deferred to future work
- Extensive LD regions such as the MHC (Chr6: 25-35 Mbp) require removal, and SNPs on sex chromosomes (ChrX-Y) or with MAF < 0.01 are excluded

## Evidence

- [methods] scPagwas computes the genetically associated PAS (gPAS) for each pathway as the product of τ-hat and the PAS: [evidence span withheld — anchor status: not_found]
- [readme] install and run scPagwas: [evidence span withheld — anchor status: external_doc]
