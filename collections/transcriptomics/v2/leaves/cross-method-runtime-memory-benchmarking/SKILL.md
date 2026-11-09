---
name: cross-method-runtime-memory-benchmarking
description: Use when two or more gene co-expression network inference methods must
  be compared on the same single-cell expression matrix, with runtime/memory and downstream
  module quality (GEP counts, GO enrichment) as head-to-head metrics.
when_to_use_negative:
- When only one method is being run — runtime/memory logging alone does not need the
  full cross-method comparison table
- When the compared methods cannot accept the same input matrix (e.g. methods requiring
  different normalizations), since the comparison depends on an identical log-transformed
  input
- When the goal is single-cell label transfer (CellGEP/cellgep) rather than co-expression
  method benchmarking
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SingleCellGGM
  role: Reference GGM-based single-cell GCN method whose runtime, memory, GEP counts
    (137 total, 5 with >=50 genes, 6 at p<=1E-20) anchor the benchmark
  repo: https://github.com/MaShisongLab/SingleCellGGM
- name: ppcor
  role: Baseline pcor-based method run on the same MCA matrix; produced 224 GEPs,
    98 GO-enriched, 64 with >=50 genes, 32 at p<=1E-20
- name: SILGGM
  role: Baseline large-scale GGM method run on the same MCA matrix; produced 258 GEPs,
    175 GO-enriched, 1 with >=50 genes, 2 at p<=1E-20
- name: ModuleEnrichmentAnalysis
  role: R pipeline for GO enrichment analysis of each method's gene modules (GEPs)
    used to count functional significance per method
  repo: https://github.com/MaShisongLab/ModuleEnrichmentAnalysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:333c684b12c02640160ddfbd3da913d0cd2bfab010dafd666d547692a95644a1
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cross-method-runtime-memory-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking workflow for comparing partial-correlation/GGM co-expression network methods (SingleCellGGM, ppcor, SILGGM) on the identical log-transformed MCA matrix, recording elapsed runtime, peak memory, module counts, GO-enrichment counts, and significant-GEP counts per method in a single comparison table.

## When to use

Use when two or more gene co-expression network inference methods must be compared on the same single-cell expression matrix, with runtime/memory and downstream module quality (GEP counts, GO enrichment) as head-to-head metrics.

## When NOT to use

- When only one method is being run — runtime/memory logging alone does not need the full cross-method comparison table
- When the compared methods cannot accept the same input matrix (e.g. methods requiring different normalizations), since the comparison depends on an identical log-transformed input
- When the goal is single-cell label transfer (CellGEP/cellgep) rather than co-expression method benchmarking

## Inputs

- log2-normalized MCA gene expression matrix (cells x genes, shared across all compared methods)
- Ensembl gene ID list (e.g. data/MCA.ensembl.gene.ids.txt)
- network inference methods: SingleCellGGM (MATLAB), ppcor (R), SILGGM (R)
- Markov clustering (MCL) parameters used for the reference SingleCellGGM network
- ModuleEnrichmentAnalysis GO annotation files (Mouse.GO.annotation.txt, GO.names.txt, Mouse.gene.symbl.txt)

## Outputs

- benchmark_comparison.csv: per-method table of module counts, GO-enriched GEP counts, significant GEP counts (>=50 genes; p <= 1E-20), runtime, and peak memory
- per-method GEP/module assignments (e.g. 137 GEPs SingleCellGGM, 224 ppcor, 258 SILGGM)
- runtime and peak-memory measurements for each network inference step

## How to apply

Prepare one shared log2-transformed MCA expression matrix (log2(TPM/10^4 + 1) normalization, genes with Ensembl IDs, low-expression genes expressed in <10 cells filtered out) as the identical input for all compared methods. Run ppcor and SILGGM on the same matrix, recording elapsed runtime and peak memory alongside the SingleCellGGM run (20000 iterations, MATLAB). Cluster each resulting network into gene modules with the Markov clustering (MCL) algorithm using parameters identical to those used for the SingleCellGGM network, yielding e.g. 224 GEPs for ppcor and 258 for SILGGM. Perform GO enrichment analysis on each method's GEPs with the ModuleEnrichmentAnalysis pipeline and count significant GEPs per the paper's criteria: GEPs with >=50 genes and GEPs with GO enrichment p <= 1E-20. Assemble module counts, enrichment counts, significance counts, and runtime/memory figures into a per-method comparison table written as benchmark_comparison.csv.

## Related tools

- **SingleCellGGM** (Reference GGM-based single-cell GCN method whose runtime, memory, GEP counts (137 total, 5 with >=50 genes, 6 at p<=1E-20) anchor the benchmark) — https://github.com/MaShisongLab/SingleCellGGM
- **ppcor** (Baseline pcor-based method run on the same MCA matrix; produced 224 GEPs, 98 GO-enriched, 64 with >=50 genes, 32 at p<=1E-20)
- **SILGGM** (Baseline large-scale GGM method run on the same MCA matrix; produced 258 GEPs, 175 GO-enriched, 1 with >=50 genes, 2 at p<=1E-20)
- **ModuleEnrichmentAnalysis** (R pipeline for GO enrichment analysis of each method's gene modules (GEPs) used to count functional significance per method) — https://github.com/MaShisongLab/ModuleEnrichmentAnalysis

## Examples

```
expression_matrix = double(readtable('Figure2-batch-removed.txt','ReadRowNames',true)); expression_matrix = log2(expression_matrix ./ sum(expression_matrix) * 10000 + 1); ggm = SingleCellGGM(sparse(expression_matrix'), 20000, gene, 'mca')
```

## Evaluation signals

- Reproduced module counts match reported values: 137 GEPs for SingleCellGGM, 224 for ppcor, 258 for SILGGM
- GO-enriched GEP counts match: 123 (SingleCellGGM), 98 (ppcor), 175 (SILGGM)
- Significant GEP counts under both criteria match: >=50 genes gives 5/64/1 and p<=1E-20 gives 6/32/2 for SingleCellGGM/ppcor/SILGGM
- Runtime and peak-memory rows are populated (non-empty) for all three methods in benchmark_comparison.csv
- All three methods were run on the identical log-transformed MCA matrix and clustered with identical MCL parameters

## Limitations

- SingleCellGGM relies exclusively on transcriptome data and does not integrate other single-cell modalities (ATAC-seq, single-cell proteomics)
- Existing pcor-based methods (ppcor, SILGGM) have only been tested on small-scale datasets and have not been widely adopted by the single-cell research community
- Low co-expression signals and frequent dropout values in single-cell data obscure Pearson correlation calculation, confounding comparisons based on raw correlation

## Evidence

- [results] SingleCellGGM identified 64 significant GEPs with enriched GO terms and containing >=50 genes, while ppcor and SILGGM only identified 1 and 5, respectively, from the same MCA dataset.: [evidence span withheld — anchor status: not_found]
- [readme] expression_matrix = log2( expression_matrix ./ sum( expression_matrix ) * 10000 + 1 );: [evidence span withheld — anchor status: external_doc]
- [readme] Filter out genes expressed in less than 10 cells: [evidence span withheld — anchor status: external_doc]
- [readme] ggm = SingleCellGGM(`expression_matrix`,`number_of_iterations`,`gene_names`,`dataset_name`): [evidence span withheld — anchor status: external_doc]
