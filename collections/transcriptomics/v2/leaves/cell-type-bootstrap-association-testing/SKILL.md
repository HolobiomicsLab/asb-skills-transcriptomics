---
name: cell-type-bootstrap-association-testing
description: Use when GWAS summary statistics and an annotated single-cell Seurat
  object are available and the goal is to rank predefined cell types for trait/disease
  association at FDR < 0.05 (e.g., AD vs. OPCs/microglia, severe COVID-19 vs. PBMC
  subsets).
when_to_use_negative:
- The scRNA-seq object lacks cell-type annotations or clusters fall below min_clustercells
  = 10, so cell-type-level t-statistics cannot be estimated
- The analysis target is single-cell-level p-values or trait-relevant gene prioritization
  (use scPagwas' singlecell scoring via Get_CorrectBg_p and scGet_PCC instead of the
  cell-type bootstrap)
- GWAS and scRNA-seq come from mismatched genome builds without the matching block_annotation
  (hg37 vs hg38), so SNP-to-gene/pathway block assignment would be invalid
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: scPagwas
  role: Runs the pathway-based polygenic regression and the block bootstrap (iters_celltype
    = 200) that yields per-cell-type t-statistics, p-values and confidence intervals;
    Boot_evaluate and Merge_celltype_p summarize results
  repo: https://github.com/dengchunyu/scPagwas
- name: scPagwas_py
  role: Python implementation synchronized with scPagwas 2.0 for memory-demanding
    single-cell data; consumes chrom_ld CSVs per chromosome (1.csv-22.csv) and block_annotation.csv
  repo: https://github.com/dengchunyu/scPagwas_py
- name: Seurat
  role: Holds the annotated Seurat object (Idents = cell types) whose normalized RNA
    assay feeds the cell-type association test; supplies the AddModuleScore TRS used
    alongside the bootstrap
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:e8a5b41f29a9b9d1f70f01549796ab41514ade25bc5f4cacb0a54ae0ee2b3d35
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
  - chrom_ld
  - scPagwas
  - scPagwas_py
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# cell-type-bootstrap-association-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

scPagwas cell-type-level association testing: after pathway SVD transformation and polygenic regression of GWAS summary statistics against pathway-annotated scRNA-seq data, a block bootstrap over pathway-based genome blocks (200 iterations by default) estimates per-cell-type significance for trait associations.

## When to use

Use when GWAS summary statistics and an annotated single-cell Seurat object are available and the goal is to rank predefined cell types for trait/disease association at FDR < 0.05 (e.g., AD vs. OPCs/microglia, severe COVID-19 vs. PBMC subsets).

## When NOT to use

- The scRNA-seq object lacks cell-type annotations or clusters fall below min_clustercells = 10, so cell-type-level t-statistics cannot be estimated
- The analysis target is single-cell-level p-values or trait-relevant gene prioritization (use scPagwas' singlecell scoring via Get_CorrectBg_p and scGet_PCC instead of the cell-type bootstrap)
- GWAS and scRNA-seq come from mismatched genome builds without the matching block_annotation (hg37 vs hg38), so SNP-to-gene/pathway block assignment would be invalid

## Inputs

- GWAS summary statistics formatted as a six-column tab-delimited txt file (e.g., ieu-b-2 AD stats or severe COVID-19 stats)
- Seurat-format scRNA-seq object with cell-type annotations assigned to Idents (e.g., Grubman entorhinal cortex snRNA-seq, 11,786 cells; or a 469,453-cell PBMC dataset)
- Package-provided Genes_by_pathway_kegg pathway list, block_annotation (hg37/hg38), and 1,000 Genomes Phase 3 chrom_ld LD data

## Outputs

- bootstrap_results data frame of per-cell-type bootstrap p-values and confidence intervals (e.g., covid_b2_celltype_bootstrap_results.csv / *_cellytpes_bootstrap_results.csv)
- Merged_celltype_pvalue from Merge_celltype_p (merged cell-type p-value CSV)
- FDR < 0.05 call set of trait-associated cell types (e.g., OPCs and microglia for AD; naive CD8+ T cells, megakaryocytes, CD16+ monocytes for severe COVID-19)

## How to apply

Preprocess the GWAS file with maf_filter = 0.01, removing sex-chromosome SNPs and the MHC region (Chr6: 25-35 Mbp), and restrict pathway annotation to KEGG pathways with 5-300 genes (min.pathway.size = 5, max.pathway.size = 300). Map SNPs to genes with a 20 kb window (marg = 10000), link pathway blocks to the package-provided 1,000 Genomes Phase 3 LD data (chrom_ld), then run Link_pathway_blocks_gwas (singlecell = TRUE, celltype = TRUE) and Pagwas_perform_regression. Run the block bootstrap (iters_celltype = 200) which estimates the standard error and computes a t-statistic with a corresponding P value for each cell type, then apply FDR correction; scPagwas' Boot_evaluate and Merge_celltype_p produce the bootstrap results and merged cell-type p-values. Decision point: declare a cell type trait-associated only when bootstrap p (or FDR < 0.05) passes, e.g., naive CD8+ T cells (p = 4.6e-17) and megakaryocytes for severe COVID-19, or OPCs and microglia for Alzheimer's disease.

## Related tools

- **scPagwas** (Runs the pathway-based polygenic regression and the block bootstrap (iters_celltype = 200) that yields per-cell-type t-statistics, p-values and confidence intervals; Boot_evaluate and Merge_celltype_p summarize results) — https://github.com/dengchunyu/scPagwas
- **scPagwas_py** (Python implementation synchronized with scPagwas 2.0 for memory-demanding single-cell data; consumes chrom_ld CSVs per chromosome (1.csv-22.csv) and block_annotation.csv) — https://github.com/dengchunyu/scPagwas_py
- **Seurat** (Holds the annotated Seurat object (Idents = cell types) whose normalized RNA assay feeds the cell-type association test; supplies the AddModuleScore TRS used alongside the bootstrap) — https://github.com/satijalab/seurat

## Examples

```
Pagwas_data <- scPagwas_main2(Pagwas=NULL, gwas_data="GWAS_summ_example.txt", Single_data="scRNAexample.rds", Pathway_list=Genes_by_pathway_kegg, chrom_ld=chrom_ld, block_annotation=block_annotation_hg37, iters_celltype=200, singlecell=TRUE, celltype=TRUE)
```

## Evaluation signals

- bootstrap_results contains a per-cell-type p-value and confidence interval for every annotated cell type after 200 block bootstrap iterations
- Significant calls meet FDR < 0.05 and match reported magnitudes (severe COVID-19: naive CD8+ T cells p = 4.6e-17, megakaryocytes p = 7.8e-6, CD16+ monocytes p = 1.14e-4; AD: OPCs and microglia)
- Pre-filters hold: MAF >= 0.01, no sex-chromosome SNPs, no MHC region (Chr6: 25-35 Mbp), pathways limited to 5-300 genes
- Exported artifacts exist: *_cellytpes_bootstrap_results.csv and merged cell-type p-value CSV, plus the Seurat result object (e.g., AD_scPagwas_results.RData)
- Cell-type results are consistent between the single-cell gPAS/TRS layer and the cell-type bootstrap layer (in scPagwas2 the synchronization issue is addressed)

## Limitations

- Statistical association between the trait and a cell type does not imply causality; it may reflect indirect identification of causal associations
- SNPs are annotated to genes solely by proximal distance (20 kb window); functionally informed SNP-to-gene linking is deferred to future work
- Pathway annotation is restricted to canonical KEGG pathways (5-300 genes) because these are experimentally validated
- Original R implementation has substantial memory demands on large single-cell data; the Python scPagwas_py was created for this reason

## Evidence

- [discussion] the block bootstrap method45 is used to estimate the standard error and compute a t-statistic with a corresponding P value for each cell type: "the block bootstrap method45 is used to estimate the standard error and compute a t-statistic with a corresponding P value for each cell type"
- [results] scPagwas identified that naive CD8+ T cells (p = 4.6 × 10−17), megakaryocytes (p = 7.8 × 10−6), and CD16+ monocytes (p = 1.14 × 10−4) demonstrated significant associations with severe COVID-19 (FDR < 0.05).: [evidence span withheld — anchor status: not_found]
- [readme] scPagwas employs a polygenic regression model to prioritize a set of trait-relevant genes and uncover trait-relevant cell subpopulations by incorporating pathway activity transformed single-cell RNA sequencing: "scPagwas employs a polygenic regression model to prioritize a set of trait-relevant genes and uncover trait-relevant cell subpopulations"
- [methods] SNPs with minor allele frequencies smaller than 0.01 or on the sex chromosomes (ChrX-Y) were removed.: "SNPs with minor allele frequencies smaller than 0.01 or on the sex chromosomes (ChrX-Y) were removed."
