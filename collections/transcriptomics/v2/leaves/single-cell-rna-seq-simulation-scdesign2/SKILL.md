---
name: single-cell-rna-seq-simulation-scdesign2
description: Use when you need a synthetic scRNA-seq dataset with known ground-truth
  trait-relevant cells (e.g., monocyte count) to quantify precision of per-cell scoring
  methods against GWAS-prioritized gene sets.
when_to_use_negative:
- When real scRNA-seq data with experimentally validated trait-relevant labels are
  already available — simulation introduces modeling assumptions rather than resolving
  biology
- When the goal is cell-type-level (aggregate) association rather than per-cell identification,
  since ground-truth simulation is designed to evaluate single-cell-resolution calls
tools:
- name: scDesign2
  role: Simulates the ground-truth single-cell RNA-seq dataset (five cell types matching
    bulk-sorted profiles) used to benchmark trait-relevant cell identification
  repo: https://github.com/JSB-UCLA/scDesign2
- name: scPagwas
  role: Prioritizes the top 1,000 trait-relevant genes from GWAS summary statistics
    and sim-data to compute per-cell TRS; its precision against the simulated ground
    truth is the reference value (95.9%)
  repo: https://github.com/dengchunyu/scPagwas
- name: Seurat
  role: Provides AddModuleScore to compute per-cell TRS on the simulated data, with
    Get_CorrectBg_p background correction (iters_singlecell = 100)
  repo: https://github.com/satijalab/seurat
- name: scDRS
  role: Competing single-cell scoring method (v1.0.2) run with MAGMA- and scPagwas-prioritized
    top 1,000 genes for precision comparison on the simulated data
  repo: https://github.com/martinjzhang/scDRS
- name: MAGMA
  role: Baseline gene-based association test supplying the top 1,000 MAGMA-prioritized
    genes used as the comparison input to scDRS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2023.100383
    title: Polygenic regression uncovers trait-relevant cellular contexts through
      pathway activation transformation of single-cell RNA sequencing data
  source_content_hash: sha256:3232f8f2aec687a12f1776464c88950edd4fcdae05202dd4e19506e147b4c944
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/dengchunyu/scPagwas
  repo_url: https://github.com/JSB-UCLA/scDesign2
  tools:
  - MAGMA
  - Seurat
  - scDRS
  - scDesign2
  - scPagwas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-rna-seq-simulation-scdesign2

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulates a ground-truth scRNA-seq dataset with scDesign2 (v1.0.0) in which cell-type profiles match FACS-sorted bulk populations, holding out the trait-relevant cell type so that per-cell identification methods (scPagwas TRS via Seurat AddModuleScore, scDRS) can be benchmarked with a defined precision metric.

## When to use

Use when you need a synthetic scRNA-seq dataset with known ground-truth trait-relevant cells (e.g., monocyte count) to quantify precision of per-cell scoring methods against GWAS-prioritized gene sets.

## When NOT to use

- When real scRNA-seq data with experimentally validated trait-relevant labels are already available — simulation introduces modeling assumptions rather than resolving biology
- When the goal is cell-type-level (aggregate) association rather than per-cell identification, since ground-truth simulation is designed to evaluate single-cell-resolution calls

## Inputs

- FACS-sorted bulk hematopoietic expression data (GEO accession GSE107011)
- GWAS summary statistics for monocyte count (IEU ID: ieu-b-31)
- KEGG pathway list (Genes_by_pathway_kegg)
- Reference SNP-gene mapping / LD blocks (e.g., block_annotation from scPagwas package)

## Outputs

- Simulated ground-truth scRNA-seq dataset with five cell types and known trait-relevant cell labels (scDesign2, version 1.0.0)
- Trait-relevant cell calls per method at p < 0.05
- Precision benchmark table (precision_benchmark.csv): scPagwas = 95.9%, scDRS(MAGMA genes) = 0.940, scDRS(scPagwas genes) = 0.957

## How to apply

Download FACS-sorted bulk hematopoietic expression data (GEO GSE107011) and fit scDesign2 (version 1.0.0) to generate a simulated dataset containing five cell types whose profiles match the bulk-sorted populations, holding out the trait-relevant cell type for benchmarking. Run scPagwas_main on the simulated data with the monocyte count GWAS (IEU ID: ieu-b-31), the KEGG pathway list (Genes_by_pathway_kegg), a 20 kb SNP-to-gene window (marg = 10000), maf_filter = 0.01, pathway size limits of 5-300 genes, and n_topgenes = 1000 to derive trait-relevant genes. Score per-cell trait relevance with Seurat's AddModuleScore and background-corrected p-values (Get_CorrectBg_p, iters_singlecell = 100), calling trait-relevant cells at p < 0.05. For comparison, run scDRS (1.0.2) twice — once with top 1,000 MAGMA-prioritized genes and once with top 1,000 scPagwas-prioritized genes — and compute precision against the scDesign2 ground truth. The rationale is that simulation gives an unambiguous ground-truth label per cell, enabling direct precision comparison (reported: scPagwas 95.9%; scDRS 0.940 vs 0.957).

## Related tools

- **scDesign2** (Simulates the ground-truth single-cell RNA-seq dataset (five cell types matching bulk-sorted profiles) used to benchmark trait-relevant cell identification) — https://github.com/JSB-UCLA/scDesign2
- **scPagwas** (Prioritizes the top 1,000 trait-relevant genes from GWAS summary statistics and sim-data to compute per-cell TRS; its precision against the simulated ground truth is the reference value (95.9%)) — https://github.com/dengchunyu/scPagwas
- **Seurat** (Provides AddModuleScore to compute per-cell TRS on the simulated data, with Get_CorrectBg_p background correction (iters_singlecell = 100)) — https://github.com/satijalab/seurat
- **scDRS** (Competing single-cell scoring method (v1.0.2) run with MAGMA- and scPagwas-prioritized top 1,000 genes for precision comparison on the simulated data) — https://github.com/martinjzhang/scDRS
- **MAGMA** (Baseline gene-based association test supplying the top 1,000 MAGMA-prioritized genes used as the comparison input to scDRS)

## Evaluation signals

- precision_benchmark.csv exists and contains a precision column
- scPagwas precision is within a few percentage points of the reported 95.9%
- scDRS precision with scPagwas-prioritized genes exceeds precision with MAGMA-prioritized genes (0.957 vs 0.940)
- Simulated dataset contains exactly five cell types and the held-out trait-relevant type matches the bulk-sorted monocyte population profile
- Trait-relevant cells are called at p < 0.05 with Get_CorrectBg_p background correction (iters_singlecell = 100)

## Limitations

- Simulated ground truth depends on scDesign2 fitting quality; conclusions are only as good as the match between simulated and bulk-sorted profiles
- Per-cell significance relies on Seurat's AddModuleScore default TRS, which the paper notes is a compatibility-driven choice
- Statistical association of cells with the trait does not imply causality, so simulated precision does not guarantee biological causal relevance

## Evidence
