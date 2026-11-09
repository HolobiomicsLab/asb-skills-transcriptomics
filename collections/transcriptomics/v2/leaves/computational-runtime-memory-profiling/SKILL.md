---
name: computational-runtime-memory-profiling
description: Use when comparing network inference methods' computational efficiency
  on scRNA-seq data, or validating scalability claims such as a 1000-gene x 8000-cell
  10X network finishing in under 100 seconds.
when_to_use_negative:
- Input is bulk RNA-seq data — the article warns imputed/processed single-cell data
  may remain sparse, so bulk-oriented benchmarking conditions do not transfer
- The analysis goal is directional or signed regulatory edges — scLink (and hence
  its efficiency profile) yields undirected statistical dependency edges only
- The dataset is a single small gene set where runtime differences between methods
  are negligible and not the question of interest
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3518
tools:
- name: scLink
  role: Primary method being profiled; sclink_norm preprocesses counts (library size
    1e6) and sclink_net infers the sparse co-expression network whose runtime and
    memory are recorded
  repo: https://github.com/Vivianstats/scLink
- name: PIDC
  role: Comparison method using partial information decomposition; its runtime and
    memory are measured on identical inputs for the efficiency benchmark
- name: GENIE3
  role: Comparison method that decomposes the p-gene network into p tree-based regression
    problems; its runtime and memory are measured for benchmarking
- name: R
  role: Implementation environment; scLink is distributed as an R package on CRAN
  repo: https://github.com/Vivianstats/scLink
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:4f405b22d812c1f558e5fc827c7c4a6ec6df8329876b91b00f668bc93bfe008c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - GENIE3
  - PIDC
  - R
  - scLink
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# computational-runtime-memory-profiling

## Summary

Profiles wall-clock runtime and peak memory of gene network inference methods (scLink, PIDC, GENIE3) across cell types and gene-set sizes, to quantify scalability and robustness trade-offs on single-cell data.

## When to use

Use when comparing network inference methods' computational efficiency on scRNA-seq data, or validating scalability claims such as a 1000-gene x 8000-cell 10X network finishing in under 100 seconds.

## When NOT to use

- Input is bulk RNA-seq data — the article warns imputed/processed single-cell data may remain sparse, so bulk-oriented benchmarking conditions do not transfer
- The analysis goal is directional or signed regulatory edges — scLink (and hence its efficiency profile) yields undirected statistical dependency edges only
- The dataset is a single small gene set where runtime differences between methods are negligible and not the question of interest

## Inputs

- Smart-seq2 Tabula Muris count matrices for late pro-B, bladder urothelial, myeloid, and microglial cell types (306-4394 cells)
- 10X Genomics count matrix of 10,085 B cells
- Gene sets of 100/200/500 (Tabula Muris) and 500/1000 (10X) genes selected for downstream analysis
- Method implementations: scLink (R package), PIDC, GENIE3

## Outputs

- efficiency_robustness_summary.csv with one row per (method, dataset, cell type, gene-set size) containing averaged wall-clock runtime, peak memory, and robustness score RS
- Per-method runtime/memory profiles across gene-set sizes (100-1000 genes) and cell numbers (306-10,085 cells)
- RS values in [0, 1] for scLink, PIDC, and GENIE3

## How to apply

Run each method (scLink via sclink_net with candidate regularization lda = seq(0.5, 0.1, -0.05), plus PIDC and GENIE3) on identical preprocessed inputs produced by sclink_norm (library size 1e6) restricted to fixed gene sets — 100/200/500 genes for Tabula Muris Smart-seq2 cell types (306-4394 cells) and 500/1000 genes for the 10X Genomics 10,085-cell B-cell matrix. Repeat inference over 10 random half-samplings of cells per cell type to compute robustness scores RS as agreement of inferred edges, so runtime/memory measurements accompany stability estimates. For each (method, dataset, cell type, gene-set size), record averaged wall-clock time and peak memory, and include the maximal 1000-gene x 8000-cell 10X run. Rationale: robustness and resource cost are joint criteria — scLink wins on both, so profiling a method only on accuracy without runtime/memory would miss the efficiency advantage shown in Figures S14-S15.

## Related tools

- **scLink** (Primary method being profiled; sclink_norm preprocesses counts (library size 1e6) and sclink_net infers the sparse co-expression network whose runtime and memory are recorded) — https://github.com/Vivianstats/scLink
- **PIDC** (Comparison method using partial information decomposition; its runtime and memory are measured on identical inputs for the efficiency benchmark)
- **GENIE3** (Comparison method that decomposes the p-gene network into p tree-based regression problems; its runtime and memory are measured for benchmarking)
- **R** (Implementation environment; scLink is distributed as an R package on CRAN) — https://github.com/Vivianstats/scLink

## Examples

```
dat.norm <- sclink_norm(counts, lib.size = 1e6); net <- sclink_net(dat.norm, lda = seq(0.5, 0.1, -0.05))
```

## Evaluation signals

- efficiency_robustness_summary.csv exists with rows for all three methods and all tested conditions (4 Tabula Muris cell types x 3 gene-set sizes, plus 10X at 500 and 1000 genes)
- All recorded robustness scores RS fall within [0, 1]
- scLink's 1000-gene x 8000-cell 10X Genomics run completes in fewer than 100 seconds
- scLink's averaged runtime and peak memory are lower than PIDC and GENIE3 for every tested condition
- Each (method, dataset, cell type, gene-set size) combination has runtime averaged over the same 10 random half-samplings used for RS

## Limitations

- Runtime and memory figures are hardware- and implementation-dependent; the article reports no software version for the scLink R package or the comparison methods
- Robustness (RS) is estimated from 10 random half-samplings, so it and any runtime figures tied to those repeats carry sampling variability
- The profiling covers gene sets of up to 1000 genes; the article suggests dividing large gene lists into major modules via sclink_cor and applying scLink per module rather than scaling naively
- scLink's edges are undirected statistical dependencies, so efficiency gains cannot be compared against direction-aware benchmarks of regulatory network inference

## Evidence

- [methods] normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis: "normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis"
- [intro] GENIE3 decomposes the prediction of a gene network between p genes into p different regression problems, and uses tree-based ensemble methods: "GENIE3 decomposes the prediction of a gene network between p genes into p different regression problems, and uses tree-based ensemble methods"
- [intro] PIDC uses partial information decomposition based on the multivariate information theory to quantify the statistical dependencies between genes: "PIDC uses partial information decomposition based on the multivariate information theory to quantify the statistical dependencies between genes"
- [discussion] first divide the genes into a few major modules based on scLink's correlation, and then separately apply scLink to each module: [evidence span withheld — anchor status: not_found]
- [readme] scLink has three main functions: `sclink_norm` for pre-processing gene expression data: "-   `sclink_norm` for pre-processing gene expression data
-   `sclink_cor` for calculating the co-expression matrix by scLink
-   `sclink_net` for constructing the gene co-expression network by scLink"
