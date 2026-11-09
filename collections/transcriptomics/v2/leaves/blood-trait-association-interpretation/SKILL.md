---
name: blood-trait-association-interpretation
description: Use when GWAS summary statistics for a blood cell trait (e.g., lymphocyte
  count, ieu-b-31 monocyte count) have already been run through scPagwas and you need
  to prioritize trait-relevant genes and quantify their GO-BP enrichment versus MAGMA/TWAS-based
  gene sets.
when_to_use_negative:
- The goal is single-cell or cell-type-level trait association (TRS scoring, Monte
  Carlo testing, block bootstrap) rather than gene-level prioritization and enrichment
  interpretation
- The trait lacks a blood-trait gene-set background against which GO over-representation
  can be evaluated, undermining the scPagwas-vs-baseline contrast
- Input is already a curated disease-gene list, in which case the PCC-based top-1,000-gene
  prioritization step is redundant
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scPagwas
  role: Computes Pearson correlation (scGet_PCC) between gene expression and summed
    gPASs to rank and prioritize trait-relevant genes; provides KEGG pathway list,
    chrom_ld, and block_annotation
  repo: https://github.com/sulab-wmu/scPagwas
- name: Seurat
  role: Reads and handles the BMMC Seurat object (Seu_Hema_data.rds) whose normalized
    RNA assay supplies gene expression for the PCC computation
  repo: https://github.com/satijalab/seurat
- name: MAGMA
  role: Baseline gene-scoring method whose top-1,000-gene trait sets are rebuilt and
    run through the identical GO-BP enrichment pipeline for comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:49ce8d4f9a6b7ddb2ae3847a1c3c9454623bd8c45fe874f0eeb725c56ec70f9b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dengchunyu/scPagwas
  repo_url: https://github.com/sulab-wmu/scPagwas
  tools:
  - MAGMA
  - Seurat
  - scPagwas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# blood-trait-association-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interprets blood-trait GWAS signals at gene level by ranking scPagwas-derived PCC scores to select the top 1,000 trait-relevant genes per trait, then testing GO biological-process over-representation of those gene sets against gene-scoring baselines (MAGMA, TWAS, S-PrediXcan, S-MultiXcan). It turns cellular-level scPagwas associations into biologically interpretable gene-set evidence.

## When to use

Use when GWAS summary statistics for a blood cell trait (e.g., lymphocyte count, ieu-b-31 monocyte count) have already been run through scPagwas and you need to prioritize trait-relevant genes and quantify their GO-BP enrichment versus MAGMA/TWAS-based gene sets.

## When NOT to use

- The goal is single-cell or cell-type-level trait association (TRS scoring, Monte Carlo testing, block bootstrap) rather than gene-level prioritization and enrichment interpretation
- The trait lacks a blood-trait gene-set background against which GO over-representation can be evaluated, undermining the scPagwas-vs-baseline contrast
- Input is already a curated disease-gene list, in which case the PCC-based top-1,000-gene prioritization step is redundant

## Inputs

- GWAS summary statistics for blood cell traits (IEU IDs: ieu-b-30 white blood cell count, ieu-b-31 monocyte count, ieu-b-34 neutrophil count, ieu-b-33 eosinophil count, ieu-b-29 basophil count, ebi-a-GCST004627 lymphocyte count, ebi-a-GCST004632 lymphocyte percent, ebi-a-GCST90002335 mean corpus volume, ebi-a-GCST90002329 MCHC, ebi-a-GCST90002311 hemoglobin concentration)
- BMMC scRNA-seq Seurat object (Seu_Hema_data.rds)
- Per-trait scPagwas results containing scPagwas.gPAS.score
- KEGG pathway list (Genes_by_pathway_kegg), chrom_ld, and block_annotation supplied with the scPagwas package

## Outputs

- Per-trait ranked PCC gene lists with top 1,000 trait-relevant gene sets
- Per-trait GO biological-process over-representation results (significant BP terms at FDR < 0.01) for scPagwas and the four baselines
- Method-by-trait comparison table of significant-term counts (scPagwas_GO_enrichment_TableS3.csv)

## How to apply

Load the BMMC Seurat object (Seu_Hema_data.rds) and the per-trait scPagwas results containing scPagwas.gPAS.score. For each of the 10 blood traits, use scPagwas::scGet_PCC to compute the Pearson correlation of each gene's expression with the summed gPASs (gPASsum) across cells, and rank genes by PCC — this is the paper's method for identifying genes whose expression is highly correlated with the polygenic signal. Select the top 1,000 genes per trait (paper default n_topgenes = 1000) as the trait-relevant gene set. Run GO biological-process over-representation enrichment on each top-gene set against blood-trait gene-set backgrounds, counting significant BP terms among over-expressed genes at FDR < 0.01. Rebuild trait gene sets with the four gene-scoring baselines (MAGMA, TWAS, S-PrediXcan, S-MultiXcan) and run the identical enrichment pipeline, then tabulate significant-term counts per method per trait (e.g., lymphocyte count: 35 BP terms for scPagwas vs 0 for MAGMA) and save as scPagwas_GO_enrichment_TableS3.csv.

## Related tools

- **scPagwas** (Computes Pearson correlation (scGet_PCC) between gene expression and summed gPASs to rank and prioritize trait-relevant genes; provides KEGG pathway list, chrom_ld, and block_annotation) — https://github.com/sulab-wmu/scPagwas
- **Seurat** (Reads and handles the BMMC Seurat object (Seu_Hema_data.rds) whose normalized RNA assay supplies gene expression for the PCC computation) — https://github.com/satijalab/seurat
- **MAGMA** (Baseline gene-scoring method whose top-1,000-gene trait sets are rebuilt and run through the identical GO-BP enrichment pipeline for comparison)

## Examples

```
PCC <- scPagwas::scGet_PCC(Single_data = Seu_Hema_data, Pagwas = Pagwas_data)  # rank genes by PCC, take top n_topgenes = 1000 per trait
```

## Evaluation signals

- Lymphocyte-count acceptance check: scPagwas over-expression genes yield 35 significant BP terms (FDR < 0.01) versus 0 for MAGMA
- scPagwas top genes show more significant BP terms than all four baselines across the 10 blood traits, matching the Table S3 pattern
- Exactly 1,000 genes per trait are retained after ranking by PCC (n_topgenes = 1000)
- Output table scPagwas_GO_enrichment_TableS3.csv contains one row per method per trait with significant-term counts as integers
- Same enrichment pipeline (backgrounds, FDR < 0.01) applied uniformly to scPagwas and baseline gene sets so counts are comparable

## Limitations

- Identification of a statistical association between complex traits and cells/genes does not imply causality but may reflect indirect identification of causal associations
- Pathway analysis is restricted to canonical KEGG pathways; GO-BP enrichment inherits the scope of these curated sets
- scPagwas SNPs are annotated to genes by proximal distance (20 kb window), so gene relevance is distance-based, not functionally informed
- The default TRS/cell-scoring machinery relies on Seurat's AddModuleScore for Seurat compatibility, which may constrain gene-scoring flexibility

## Evidence

- [discussion] we determine which gene g exhibits expression that is highly correlated with the summed gPASs (gPASsum) across individual cells using the Pearson correlation method: "we determine which gene g exhibits expression that is highly correlated with the summed gPASs (gPASsum) across individual cells using the Pearson correlation method"
- [results] for lymphocyte count over-expression genes, scPagwas enriched 35 significant BP terms (FDR < 0.01) versus 0 for MAGMA: [evidence span withheld — anchor status: not_found]
- [readme] Code for reproducing the analysis from the paper is available: "Code for reproducing the analysis from the paper is available
[here](https://github.com/dengchunyu/scPagwas_reproduce)"
- [full_text] scPagwas-identified trait-relevant genes were more highly enriched in functional gene sets related to blood cell traits than genes identified by the four other gene-scoring methods: [evidence span withheld — anchor status: not_found]
