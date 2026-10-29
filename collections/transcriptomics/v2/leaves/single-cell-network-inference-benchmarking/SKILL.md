---
name: single-cell-network-inference-benchmarking
description: Use when comparing gene regulatory/co-expression network inference methods
  on scRNA-seq count matrices, especially for quantifying robustness via half-sampling
  agreement and wall-clock runtime/memory on 10X or Smart-seq2 data.
when_to_use_negative:
- The goal is to infer directional regulatory, co-regulatory, or functionally coordinated
  edge types — scLink edges are based on statistical dependencies and do not have
  directions.
- The input is bulk RNA-seq or already-imputed single-cell data — imputed data may
  still be much sparser than bulk data, so bulk-data network methods may perform poorly,
  and imputation itself can be time-consuming.
- You only need to explore correlation structure without constructing a network —
  sclink_cor should be used first rather than the full sclink_net benchmark.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scLink
  role: Primary method under benchmark; sclink_norm normalizes to library size 1e6,
    sclink_cor investigates correlation structures, sclink_net infers the sparse co-expression
    network with candidate lda = seq(0.5, 0.1, -0.05)
  repo: https://github.com/Vivianstats/scLink
- name: PIDC
  role: Comparison method; uses partial information decomposition based on multivariate
    information theory to quantify gene dependencies, run on the same normalized inputs
- name: GENIE3
  role: Comparison method; decomposes network prediction between p genes into p regression
    problems using tree-based ensemble methods, run on the same normalized inputs
- name: R
  role: Implementation environment; the scLink R package is installed via install.packages("scLink")
    from CRAN
  repo: https://github.com/Vivianstats/scLink
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:1f0a78b94dbc8a7860f96c7e202483144a256ae493ab783daf639e37163b8ea9
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# single-cell-network-inference-benchmarking

## Summary

Benchmarks scLink's sparse gene co-expression network inference against PIDC and GENIE3 on Smart-seq2 (Tabula Muris) and 10X Genomics scRNA-seq data, comparing computational efficiency (runtime, peak memory) and robustness (edge agreement across random half-samplings).

## When to use

Use when comparing gene regulatory/co-expression network inference methods on scRNA-seq count matrices, especially for quantifying robustness via half-sampling agreement and wall-clock runtime/memory on 10X or Smart-seq2 data.

## When NOT to use

- The goal is to infer directional regulatory, co-regulatory, or functionally coordinated edge types — scLink edges are based on statistical dependencies and do not have directions.
- The input is bulk RNA-seq or already-imputed single-cell data — imputed data may still be much sparser than bulk data, so bulk-data network methods may perform poorly, and imputation itself can be time-consuming.
- You only need to explore correlation structure without constructing a network — sclink_cor should be used first rather than the full sclink_net benchmark.

## Inputs

- Smart-seq2 Tabula Muris count matrices for late pro-B, bladder urothelial, myeloid, and microglial cell types (306-4394 cells)
- 10X Genomics count matrix of 10,085 B cells
- Gene lists of 100/200/500 (Tabula Muris) and 500/1000 (10X) genes

## Outputs

- efficiency_robustness_summary.csv with one row per (method, dataset, cell type, gene-set size)
- Per-method robustness scores (RS) in [0, 1] and averaged wall-clock runtime and peak memory per condition

## How to apply

Preprocess each count matrix with scLink's sclink_norm, normalizing to a library size of 10^6 and restricting to gene sets of 100/200/500 genes (Tabula Muris: late pro-B, bladder urothelial, myeloid, microglial; 306-4394 cells) and 500/1000 genes (10X Genomics B cells, 10,085 cells). Run scLink via sclink_net with candidate regularization values lda = seq(0.5, 0.1, -0.05), and run PIDC and GENIE3 on the identical normalized inputs. Repeat each network inference on 10 random half-samplings of cells per cell type and compute the robustness score RS as agreement of inferred edges between half-samples. Record averaged wall-clock runtime and peak memory per method, gene-set size, and cell type, then aggregate one row per (method, dataset, cell type, gene-set size) into efficiency_robustness_summary.csv.

## Related tools

- **scLink** (Primary method under benchmark; sclink_norm normalizes to library size 1e6, sclink_cor investigates correlation structures, sclink_net infers the sparse co-expression network with candidate lda = seq(0.5, 0.1, -0.05)) — https://github.com/Vivianstats/scLink
- **PIDC** (Comparison method; uses partial information decomposition based on multivariate information theory to quantify gene dependencies, run on the same normalized inputs)
- **GENIE3** (Comparison method; decomposes network prediction between p genes into p regression problems using tree-based ensemble methods, run on the same normalized inputs)
- **R** (Implementation environment; the scLink R package is installed via install.packages("scLink") from CRAN) — https://github.com/Vivianstats/scLink

## Examples

```
library(scLink); norm <- sclink_norm(counts, lib.size = 1e6, genes = genes); net <- sclink_net(norm, lda = seq(0.5, 0.1, -0.05))
```

## Evaluation signals

- efficiency_robustness_summary.csv exists with rows for all three methods (scLink, PIDC, GENIE3) and all tested conditions
- All robustness score (RS) values fall within [0, 1]
- scLink finishes the 1000-gene x 8000-cell 10X Genomics network in fewer than 100 seconds
- scLink's runtime and peak memory are lower than both PIDC and GENIE3 across tested conditions
- scLink's robustness (RS) exceeds that of PIDC and GENIE3 on the repeated half-samplings

## Limitations

- Edges identified by scLink are undirected — it is not feasible to directly distinguish regulatory, co-regulatory, and functionally coordinated edge types using only gene expression data
- Imputation of gene expression can be time-consuming depending on cell number, and imputed data may still be much sparser than bulk data
- For very large gene lists, scLink should be applied per module — first divide genes into major modules based on scLink's correlation, then apply scLink separately to each module
- No software version specified for the scLink R package in the article

## Evidence

- [results] scLink achieved higher robustness than PIDC and GENIE3 while requiring much less computation time and memory usage: [evidence span withheld — anchor status: not_found]
- [methods] normalize the read count matrix with a library size of 10^6: "normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis"
- [methods] use the function `sclink_net` to calculate the robust correlation matrix: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink"
- [intro] PIDC uses partial information decomposition based on the multivariate information theory: "PIDC uses partial information decomposition based on the multivariate information theory to quantify the statistical dependencies between genes"
- [readme] The package is available on CRAN: "The package is available on CRAN. For installation please use the following codes in `R`

install.packages("scLink")"
