---
name: splatter-simulated-data-handling
description: Use when a simulated single-cell RNA-seq benchmark is needed to evaluate
  EDGE or other embedding methods (t-SNE, UMAP), or when a raw gene-by-cell count
  matrix must be log-normalized before calling endr().
when_to_use_negative:
- Input is already a log-normalized cell-by-gene matrix (e.g. dat_log.rds) — re-normalizing
  and re-transforming would distort the values
- The task requires real experimental scRNA-seq data rather than simulated benchmarks
  with known rare/common group proportions
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3577
tools:
- name: Splatter
  role: Simulates the groups-method single-cell RNA-seq count matrix with 500 cells,
    200 genes, and a rare group (2% of cells) for benchmarking
  repo: github.com/shawnstat/EDGE
- name: R
  role: Runtime environment in which splatSimulate, sweep, and log2 normalization
    are executed
  repo: github.com/shawnstat/EDGE
- name: EDGE
  role: Consumer of the preprocessed dat_log matrix via the endr() entry point
  repo: github.com/shawnstat/EDGE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:556778f31e1b1914ce789741afae91e532901ad3ec1b3bd371fbb2f650ea084d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EDGE
  - R
  - Splatter
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# splatter-simulated-data-handling

## Summary

Generate a Splatter-simulated scRNA-seq count matrix and preprocess it (median-based library-size normalization followed by log2(x+1) transformation) so it is in the cell-by-gene log matrix format required by EDGE's endr pipeline.

## When to use

Use when a simulated single-cell RNA-seq benchmark is needed to evaluate EDGE or other embedding methods (t-SNE, UMAP), or when a raw gene-by-cell count matrix must be log-normalized before calling endr().

## When NOT to use

- Input is already a log-normalized cell-by-gene matrix (e.g. dat_log.rds) — re-normalizing and re-transforming would distort the values
- The task requires real experimental scRNA-seq data rather than simulated benchmarks with known rare/common group proportions

## Inputs

- Splatter simulation parameters (newSplatParams with nGenes, seed, batchCells, group.prob)
- Raw gene-by-cell count matrix (counts_dt) from counts(splatSimulate(..., method="groups"))

## Outputs

- Log-normalized cell-by-gene matrix dat_log (500 cells x 200 genes in the README example)
- Simulated count matrix counts_dt and cell-type labels labs from colData(simu_dt)[,3]

## How to apply

Simulate a groups-method Splatter dataset with newSplatParams (e.g. nGenes=200, batchCells=500, group.prob=c(0.98,0.02) so a rare group of ~15 cells exists, seed=2020) and extract counts(simu_dt) as counts_dt. Normalize by computing per-cell library sizes rs <- colSums(counts_dt), taking their median rs_med, and scaling each cell via sweep(counts_dt,2,rs/rs_med,"/") to obtain dat_norm. Apply log2(dat_norm+1) and transpose so rows are cells and columns are genes, yielding dat_log for downstream embedding with endr(dat_log, custom_defs). Rationale: EDGE's endr expects a log-transformed, cell-by-gene matrix; the median-normalization step removes cell-specific capture-depth differences inherent in simulated counts.

## Related tools

- **Splatter** (Simulates the groups-method single-cell RNA-seq count matrix with 500 cells, 200 genes, and a rare group (2% of cells) for benchmarking) — github.com/shawnstat/EDGE
- **R** (Runtime environment in which splatSimulate, sweep, and log2 normalization are executed) — github.com/shawnstat/EDGE
- **EDGE** (Consumer of the preprocessed dat_log matrix via the endr() entry point) — github.com/shawnstat/EDGE

## Examples

```
simu_dt <- splatSimulate(params=params,method="groups"); counts_dt <- counts(simu_dt); rs <- colSums(counts_dt); dat_log <- t(log2(sweep(counts_dt,2,rs/median(rs),"/")+1))
```

## Evaluation signals

- Output matrix dat_log has 500 rows (cells) and 200 columns (genes) matching the Splatter parameters batchCells=500 and nGenes=200
- All entries of dat_log are non-negative and finite (since counts are non-negative integers, log2(x+1) >= 0)
- Column sums of dat_norm are all equal to the median library size rs_med, confirming successful median normalization
- Simulated labels labs match the specified group.prob proportions (approximately 490 cells in the major group and ~15 in the rare group)
- Calling endr(dat_log, custom_defs) runs without a dimensionality error, confirming cell-by-gene orientation

## Limitations

- No version numbers are reported for R or Splatter in the article or README
- Edge-case counts: if a cell has zero library size, the rs/rs_med size factor is zero and sweep would divide by zero; median normalization assumes every simulated cell has non-zero counts
- The simulation uses a fixed seed (2020); different seeds change rare-group composition and can alter benchmark reproducibility

## Evidence

- [methods] splatSimulate with groups method: "We will use the simulator, Splatter, to simulate a single cell RNA-seq dataset with 500 cells and 200 genes."
- [readme] median normalization and log transform: "dat_norm <- sweep(counts_dt,2,rs/rs_med,"/")
    dat_log <- t(log2(dat_norm+1))"
- [readme] simulation parameters: "params <- newSplatParams(nGenes=GC, seed=myseed, batchCells=BC, group.prob=gp)"
- [methods] endr consumes log matrix: "simu.endr <- endr(dat_log,custom_defs)"
