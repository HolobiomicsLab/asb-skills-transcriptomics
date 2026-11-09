---
name: cross-method-network-comparison
description: Use when evaluating a gene co-expression network inference method against
  alternatives on scRNA-seq data and you need quantitative computational-efficiency
  and robustness comparisons rather than only accuracy metrics.
when_to_use_negative:
- The goal is only network accuracy against a gold-standard edge list rather than
  computational efficiency and robustness comparison across methods.
- Input data are not sparse single-cell read counts (e.g., bulk RNA-seq), where scLink's
  zero-inflation-aware normalization is unnecessary.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scLink
  role: Method under comparison; sclink_norm preprocesses counts (library size 1e6)
    and sclink_net infers the sparse co-expression network with candidate lda = seq(0.5,
    0.1, -0.05)
  repo: https://github.com/Vivianstats/scLink
- name: PIDC
  role: Comparison method using partial information decomposition to quantify statistical
    dependencies between genes
- name: GENIE3
  role: Comparison method that decomposes network prediction of p genes into p regression
    problems using tree-based ensemble methods
- name: R
  role: Environment in which scLink, PIDC, and GENIE3 runs and the timing/memory/robustness
    aggregation are executed
  repo: https://github.com/Vivianstats/scLink
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:7468e925debedaea63cfa53d61ec811308325520719c88f4197550d5437b6d7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - GENIE3
  - PIDC
  - R
  - scLink
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cross-method-network-comparison

## Summary

Benchmark a scRNA-seq gene-network inference method (e.g., scLink) against competitors such as PIDC and GENIE3 by measuring runtime, peak memory, and robustness (RS, edge agreement across random half-samplings) on Smart-seq2 Tabula Muris and 10X Genomics datasets.

## When to use

Use when evaluating a gene co-expression network inference method against alternatives on scRNA-seq data and you need quantitative computational-efficiency and robustness comparisons rather than only accuracy metrics.

## When NOT to use

- The goal is only network accuracy against a gold-standard edge list rather than computational efficiency and robustness comparison across methods.
- Input data are not sparse single-cell read counts (e.g., bulk RNA-seq), where scLink's zero-inflation-aware normalization is unnecessary.

## Inputs

- Smart-seq2 Tabula Muris count matrices for late pro-B, bladder urothelial, myeloid, and microglial cell types (306-4394 cells)
- 10X Genomics count matrix of 10,085 B cells
- Gene sets of 100/200/500 genes (Tabula Muris) and 500/1000 genes (10X)

## Outputs

- efficiency_robustness_summary.csv with one row per (method, dataset, cell type, gene-set size) containing averaged wall-clock runtime, peak memory, and robustness score RS
- Inferred gene networks from scLink, PIDC, and GENIE3 per condition and half-sample

## How to apply

Normalize each count matrix with scLink's sclink_norm (library size 1e6) and restrict to fixed gene sets (100/200/500 genes for Smart-seq2 Tabula Muris cell types such as late pro-B, bladder urothelial, myeloid, and microglial; 500/1000 genes for 10X B cells). Run scLink via sclink_net with candidate regularization lda = seq(0.5, 0.1, -0.05), and run PIDC and GENIE3 on identical inputs. Repeat each inference on 10 random half-samplings of cells per cell type and compute the robustness score RS as the agreement of inferred edges between half-samples. Record averaged wall-clock runtime and peak memory per method, dataset, cell type, and gene-set size, including the 1000-gene x 8000-cell 10X run, then aggregate into a single CSV with one row per (method, dataset, cell type, gene-set size).

## Related tools

- **scLink** (Method under comparison; sclink_norm preprocesses counts (library size 1e6) and sclink_net infers the sparse co-expression network with candidate lda = seq(0.5, 0.1, -0.05)) — https://github.com/Vivianstats/scLink
- **PIDC** (Comparison method using partial information decomposition to quantify statistical dependencies between genes)
- **GENIE3** (Comparison method that decomposes network prediction of p genes into p regression problems using tree-based ensemble methods)
- **R** (Environment in which scLink, PIDC, and GENIE3 runs and the timing/memory/robustness aggregation are executed) — https://github.com/Vivianstats/scLink

## Examples

```
library(scLink); norm <- sclink_norm(counts, lib.size=1e6, genes=genes); net <- sclink_net(norm, lda=seq(0.5, 0.1, -0.05))
```

## Evaluation signals

- efficiency_robustness_summary.csv exists with rows for all three methods (scLink, PIDC, GENIE3) and all tested conditions (dataset, cell type, gene-set size).
- All RS values fall within [0, 1].
- scLink's 1000-gene x 8000-cell 10X Genomics run completes in fewer than 100 seconds.
- scLink shows lower runtime and peak memory than PIDC and GENIE3 for the same inputs.
- Each condition is replicated on 10 random half-samplings and metrics are averaged per (method, dataset, cell type, gene-set size).

## Limitations

- RS measures agreement of edges between half-samples only; it captures robustness, not accuracy against a known regulatory truth.
- No software version is specified for the scLink R package in the article, which can affect exact reproducibility of timings.
- scLink edges are undirected statistical dependencies; they do not distinguish regulatory, co-regulatory, or functionally coordinated edge types.

## Evidence

- [results] scLink achieved higher robustness than PIDC and GENIE3 while requiring much less computation time and memory usage, and finished a 1000-gene, 8000-cell 10X Genomics network in fewer than 100 seconds.: [evidence span withheld — anchor status: not_found]
- [intro] GENIE3 decomposes the prediction of a gene network between p genes into p different regression problems, and uses tree-based ensemble methods: "GENIE3 decomposes the prediction of a gene network between p genes into p different regression problems, and uses tree-based ensemble methods"
- [intro] PIDC uses partial information decomposition based on the multivariate information theory to quantify the statistical dependencies between genes: "PIDC uses partial information decomposition based on the multivariate information theory to quantify the statistical dependencies between genes"
- [methods] normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis: "normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis"
- [readme] The scLink R package is available at https://github.com/Vivianstats/scLink.: "The package is available on CRAN. For installation please use the
following codes in `R`

install.packages("scLink")"
