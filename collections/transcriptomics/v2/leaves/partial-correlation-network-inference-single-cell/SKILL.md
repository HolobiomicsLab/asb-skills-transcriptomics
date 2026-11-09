---
name: partial-correlation-network-inference-single-cell
description: Use when the goal is to construct a gene co-expression network from a
  raw single-cell expression matrix (e.g., MCA, >5000 cells) via GGM/pcor methods,
  and to identify gene expression programs (GEPs) from it.
when_to_use_negative:
- The dataset is small-scale or has few cells (<5000 recommended by the README for
  SingleCellGGM) where existing pcor methods like ppcor/SILGGM were originally tested
- The goal requires integration of non-transcriptomic single-cell modalities (ATAC-seq,
  proteomics) — SingleCellGGM relies exclusively on transcriptome data
- Dropout-imputation or metacell-based co-expression approaches (MAGIC, scImpute,
  MetaCell, scCorr) are the intended analysis route rather than GGM/pcor network inference
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: SingleCellGGM
  role: 'Primary MATLAB algorithm: iteratively samples gene pairs (~20,000 iterations)
    to compute pcors and constructs the single-cell GGM co-expression network from
    a log-normalized expression matrix'
  repo: https://github.com/MaShisongLab/SingleCellGGM
- name: ppcor
  role: Comparator R package that infers a partial-correlation co-expression network
    (MCA_ppcor) on the identical input matrix for benchmarking
- name: SILGGM
  role: Comparator R package for large-scale Gaussian graphical model network inference
    (MCA_SILGGM) on the same input matrix, benchmarked against SingleCellGGM
- name: ModuleEnrichmentAnalysis
  role: R pipeline that performs GO (and MP) term enrichment on mouse gene modules/GEPs
    to count GO-enriched and significantly enriched programs (p <= 1E-20)
  repo: https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:cf13b5dd06fc28920623c69adf9fe771bd7d66ae90b5a59ce47bde8398c44ea2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/MaShisongLab/CellGEP
  repo_url: https://github.com/MaShisongLab/SingleCellGGM
  tools:
  - ModuleEnrichmentAnalysis
  - SILGGM
  - SingleCellGGM
  - ppcor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# partial-correlation-network-inference-single-cell

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infers single-cell gene co-expression networks using partial correlation (pcor)-based graphical Gaussian model approaches — primarily SingleCellGGM, with ppcor and SILGGM as comparators — and evaluates the resulting GEPs by module counts, GO enrichment, and significance criteria.

## When to use

Use when the goal is to construct a gene co-expression network from a raw single-cell expression matrix (e.g., MCA, >5000 cells) via GGM/pcor methods, and to identify gene expression programs (GEPs) from it.

## When NOT to use

- The dataset is small-scale or has few cells (<5000 recommended by the README for SingleCellGGM) where existing pcor methods like ppcor/SILGGM were originally tested
- The goal requires integration of non-transcriptomic single-cell modalities (ATAC-seq, proteomics) — SingleCellGGM relies exclusively on transcriptome data
- Dropout-imputation or metacell-based co-expression approaches (MAGIC, scImpute, MetaCell, scCorr) are the intended analysis route rather than GGM/pcor network inference

## Inputs

- Log-normalized single-cell gene expression matrix (double/sparse format, cells in rows, genes in columns), e.g., MCA 'Figure2-batch-removed.txt' (25,133 genes, 61,637 cells)
- Ensembl gene ID list (e.g., 'data/MCA.ensembl.gene.ids.txt') for gene filtering
- Number of iterations (typically 20000) and gene names as SingleCellGGM inputs

## Outputs

- GGM co-expression network of significant gene-pair pcors (ggm.SigEdges, e.g., 127,229 edges for MCA), writable as a tab-delimited file such as 'mca.ggm.network.txt'
- FDR table over pcor cutoffs from fdr_control (e.g., fdr.fdr)
- Gene modules/GEPs from MCL clustering (e.g., 137 GEPs for SingleCellGGM on MCA) and per-GEP GO enrichment results from ModuleEnrichmentAnalysis
- Per-method comparison table (benchmark_comparison.csv) with module counts, enrichment counts, significance counts, and runtime/memory

## How to apply

Log-normalize the expression matrix (log2(counts/total*10000 + 1)), transpose to cells-in-rows/genes-in-columns, restrict to genes with Ensembl IDs, filter out genes expressed in fewer than 10 cells, and convert to sparse format. Run SingleCellGGM with ~20,000 iterations (or round(gene_number*(gene_number-1)/39980) so each gene pair is sampled ~100 times on average), then save ggm.SigEdges to a tab-delimited network file and optionally control FDR with fdr_control and refine thresholds via adjust_cutoff (default pcor cutoff 0.03, co-expressed-cell cutoff 10). For benchmarking, run ppcor and SILGGM on the identical log-transformed matrix, recording elapsed runtime and peak memory. Cluster each network into gene modules with Markov clustering (MCL) using identical parameters across methods, then assess GEPs with ModuleEnrichmentAnalysis GO enrichment. Judge significance per the paper's criteria: GEPs with >= 50 genes (e.g., 5 for SingleCellGGM vs 64 ppcor / 1 SILGGM on MCA) and GEPs with GO enrichment p <= 1E-20.

## Related tools

- **SingleCellGGM** (Primary MATLAB algorithm: iteratively samples gene pairs (~20,000 iterations) to compute pcors and constructs the single-cell GGM co-expression network from a log-normalized expression matrix) — https://github.com/MaShisongLab/SingleCellGGM
- **ppcor** (Comparator R package that infers a partial-correlation co-expression network (MCA_ppcor) on the identical input matrix for benchmarking)
- **SILGGM** (Comparator R package for large-scale Gaussian graphical model network inference (MCA_SILGGM) on the same input matrix, benchmarked against SingleCellGGM)
- **ModuleEnrichmentAnalysis** (R pipeline that performs GO (and MP) term enrichment on mouse gene modules/GEPs to count GO-enriched and significantly enriched programs (p <= 1E-20)) — https://github.com/MaShisongLab/ModuleEnrichmentAnalysis

## Examples

```
expression_matrix = log2( expression_matrix ./ sum( expression_matrix ) * 10000 + 1 ); ggm = SingleCellGGM( expression_matrix, 20000, gene, 'mca')
```

## Evaluation signals

- Reproduced per-method GEP counts on MCA match reported values: 137 total / 123 GO-enriched for SingleCellGGM, 224/98 for ppcor, 258/175 for SILGGM
- Significant-GEP counts match the paper's criteria: 5 SingleCellGGM / 64 ppcor / 1 SILGGM GEPs with >= 50 genes; 6 / 32 / 2 GEPs with GO enrichment p <= 1E-20
- ggm object properties are consistent with the MCA example: gene_num 24802 (genes with Ensembl IDs retained after filtering from 25133), samples_num 61637, RoundNumber 20000
- runtime/memory rows for all three methods are populated in benchmark_comparison.csv
- FDR check via fdr_control shows acceptable false-discovery proportions across pcor cutoffs

## Limitations

- SingleCellGGM relies exclusively on transcriptome data and does not integrate other single-cell modalities such as ATAC-seq or single-cell proteomics
- Large memory footprint: the MCA run holds pcor_all as a [24802x24802 single] matrix; sparse conversion is advised and low-expression genes (< 10 cells) must be filtered first
- pcor-based methods have only been tested on small-scale datasets and lack broad community adoption, so large-scale benchmarking (runtime/memory) is essential

## Evidence

- [readme] SingleCellGGM takes a single-cell gene expression matrix as input and uses a process consisting of ~20,000 iterations to calculate partial correlation coefficients between gene pairs.: [evidence span withheld — anchor status: external_doc]
- [readme] Expression matrix log normalization: [evidence span withheld — anchor status: external_doc]
