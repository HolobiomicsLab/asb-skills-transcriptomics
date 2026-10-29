---
name: trait-relevance-score-computation-single-cell
description: Use when per-cell scores for trait association are needed from scRNA-seq
  data given GWAS summary statistics, e.g., to identify trait-relevant cells against
  a scDesign2 ground truth or real disease data.
when_to_use_negative:
- Input is already a per-cell score or significance table (no GWAS-to-gene prioritization
  step is needed).
- The goal is cell-type-level (bulk) enrichment rather than single-cell inference;
  methods like LDSC-SEG, RolyPoly, or EPIC target tissue/cell-type resolution and
  neglect within-cell-type heterogeneity.
- The scRNA-seq data lack normalized RNA assay and enough cells/genes after filtering
  (min_cells=50 genes, min_genes=200 cells), making TRS and background correction
  unstable.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0203
tools:
- name: scPagwas
  role: Prioritizes the top 1,000 trait-relevant genes from GWAS summary statistics
    and scRNA-seq data via pathway activation polygenic regression; the gene set drives
    the TRS.
  repo: https://github.com/dengchunyu/scPagwas
- name: Seurat AddModuleScore function
  role: Computes the per-cell TRS from the prioritized gene set with default parameters,
    using control gene sets matched by expression bins.
  repo: https://github.com/satijalab/seurat
- name: Get_CorrectBg_p
  role: scPagwas function performing the Monte Carlo background correction (iters_singlecell
    = 100) that yields per-cell p-values for TRS significance.
  repo: https://github.com/dengchunyu/scPagwas
- name: scDRS
  role: Comparative cell-scoring method run with MAGMA-prioritized genes as a baseline
    against scPagwas genes in the precision benchmark.
  repo: https://github.com/martinjzhang/scDRS
- name: MAGMA
  role: Gene-based association test providing the alternative top 1,000 trait-relevant
    gene set used as the scDRS baseline.
- name: scDesign2
  role: Simulates the ground-truth scRNA-seq dataset (five cell types matching FACS-sorted
    bulk populations from GSE107011) against which TRS precision is evaluated.
  repo: https://github.com/JSB-UCLA/scDesign2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:0181444b005bdf6e1d449067bb7ac8b8297bc646c9f9140e5eb31a9bfa20b433
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
  - Get_CorrectBg_p
  - MAGMA
  - Seurat AddModuleScore function
  - scDRS
  - scDesign2
  - scPagwas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# trait-relevance-score-computation-single-cell

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute per-cell trait-relevance scores (TRS) from scRNA-seq data using Seurat's AddModuleScore on scPagwas-prioritized trait-relevant genes, with Monte Carlo background correction (Get_CorrectBg_p, iters_singlecell=100) to call trait-relevant cells at p < 0.05.

## When to use

Use when per-cell scores for trait association are needed from scRNA-seq data given GWAS summary statistics, e.g., to identify trait-relevant cells against a scDesign2 ground truth or real disease data.

## When NOT to use

- Input is already a per-cell score or significance table (no GWAS-to-gene prioritization step is needed).
- The goal is cell-type-level (bulk) enrichment rather than single-cell inference; methods like LDSC-SEG, RolyPoly, or EPIC target tissue/cell-type resolution and neglect within-cell-type heterogeneity.
- The scRNA-seq data lack normalized RNA assay and enough cells/genes after filtering (min_cells=50 genes, min_genes=200 cells), making TRS and background correction unstable.

## Inputs

- GWAS summary statistics (e.g., monocyte count, IEU ID ieu-b-31)
- Seurat-format scRNA-seq object with 'RNA' assay and normalized expression
- KEGG pathway gene list (Genes_by_pathway_kegg)
- LD reference (chrom_ld, 1000 Genomes Phase 3 Panel) and block annotation (hg37/hg38)
- Optional ground-truth cell labels from scDesign2 simulation

## Outputs

- Per-cell trait-relevance scores (TRS)
- Background-corrected per-cell p-values (trait-relevant cells called at p < 0.05)
- Top 1,000 trait-relevant gene list from scPagwas
- Precision benchmark table (e.g., precision_benchmark.csv)

## How to apply

First prioritize trait-relevant genes by running scPagwas_main on GWAS summary statistics (e.g., monocyte count, IEU ID ieu-b-31) with the KEGG pathway list (Genes_by_pathway_kegg), a 20 kb SNP-to-gene window (marg = 10000), MAF filter 0.01, pathway size limits of 5-300 genes, and n_topgenes = 1000; SNPs with MAF < 0.01 and on sex chromosomes are removed. Then compute a per-cell TRS with Seurat's AddModuleScore function using the top 1,000 prioritized genes, with control gene sets matched by expression bins as Seurat default parameters. Assess single-cell significance via a Monte Carlo test (Get_CorrectBg_p with iters_singlecell = 100) comparing the TRS to an empirical distribution of control TRSs, and call cells trait-relevant at p < 0.05. The rationale is that pathway-activation-transformed polygenic regression optimizes trait-relevant gene selection, which scDRS with MAGMA genes alone does not; the same TRS framework lets you benchmark alternative gene sets (MAGMA vs scPagwas) against a ground truth.

## Related tools

- **scPagwas** (Prioritizes the top 1,000 trait-relevant genes from GWAS summary statistics and scRNA-seq data via pathway activation polygenic regression; the gene set drives the TRS.) — https://github.com/dengchunyu/scPagwas
- **Seurat AddModuleScore function** (Computes the per-cell TRS from the prioritized gene set with default parameters, using control gene sets matched by expression bins.) — https://github.com/satijalab/seurat
- **Get_CorrectBg_p** (scPagwas function performing the Monte Carlo background correction (iters_singlecell = 100) that yields per-cell p-values for TRS significance.) — https://github.com/dengchunyu/scPagwas
- **scDRS** (Comparative cell-scoring method run with MAGMA-prioritized genes as a baseline against scPagwas genes in the precision benchmark.) — https://github.com/martinjzhang/scDRS
- **MAGMA** (Gene-based association test providing the alternative top 1,000 trait-relevant gene set used as the scDRS baseline.)
- **scDesign2** (Simulates the ground-truth scRNA-seq dataset (five cell types matching FACS-sorted bulk populations from GSE107011) against which TRS precision is evaluated.) — https://github.com/JSB-UCLA/scDesign2

## Examples

```
Pagwas <- scPagwas_main(Pagwas=NULL, gwas_data='ieu-b-31.txt', Single_data='sim_scRNA.rds', Pathway_list=Genes_by_pathway_kegg, marg=10000, maf_filter=0.01, min.pathway.size=5, max.pathway.size=300, n_topgenes=1000, iters_singlecell=100, singlecell=T); Seurat_obj <- AddModuleScore(Seurat_obj, features=pagwas_topgenes); pvals <- Get_CorrectBg_p(Seurat_obj, iters_singlecell=100)
```

## Evaluation signals

- precision_benchmark.csv exists and contains a precision column; scPagwas precision is within a few percentage points of the reported 95.9%.
- scDRS precision with scPagwas-prioritized genes (0.957) exceeds scDRS precision with MAGMA-prioritized genes (0.940).
- The number of trait-relevant cells called (p < 0.05) is consistent with the held-out ground-truth cell type in the scDesign2 simulation.
- TRS computation uses exactly n_topgenes = 1000 genes and the Seurat AddModuleScore defaults, and per-cell p-values come from iters_singlecell = 100 Monte Carlo iterations.
- GWAS input satisfies filter settings: MAF >= 0.01, no ChrX-Y SNPs, and only KEGG pathways of 5-300 genes retained.

## Limitations

- Pathway analysis is restricted to canonical KEGG pathways only.
- SNPs are annotated to genes solely by proximal distance using a 20 kb window; functionally informed SNP-to-gene linking is deferred to future work.
- The default TRS relies on Seurat's AddModuleScore cell-scoring method for Seurat compatibility.
- Statistical association between traits and individual cells does not imply causality.
- The TRS relies on prior gene prioritization accuracy; trait-relevant genes must be re-derived per trait (here monocyte count via scPagwas or MAGMA).

## Evidence
