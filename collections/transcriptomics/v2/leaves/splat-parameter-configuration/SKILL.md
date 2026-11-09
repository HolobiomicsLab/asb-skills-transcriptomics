---
name: splat-parameter-configuration
description: Use when simulated scRNA-seq data with known differentially expressed
  (feature) genes and a rare/common two-cell-type structure is needed to benchmark
  feature gene detection or dimensionality reduction methods.
when_to_use_negative:
- The analysis target is real scRNA-seq data where no ground-truth DE gene labels
  exist — Splatter simulation provides synthetic truth, not empirical validation
- More than two cell types or continuous trajectories are needed — the 'groups' method
  configured here yields exactly two groups (rare and common)
- A fixed dataset size different from the small-scale regime (hundreds of genes, hundreds
  to 1000 cells) is required and parameter re-tuning is out of scope
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: splatter
  role: Simulates two-group single-cell RNA-seq count data with known feature genes
    via newSplatParams and splatSimulate(method='groups')
- name: EDGE
  role: Downstream method evaluated on the Splatter-simulated data for feature gene
    detection
  repo: github.com/shawnstat/EDGE
- name: R
  role: Language in which the Splatter simulation and parameter configuration are
    executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:b0be7ee8345328b46aa0cbf37e3267c19e5fa7e877d784fd22fc64e92d842e39
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EDGE
  - R
  - RSpectra
  - Rcpp
  - splatter
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# splat-parameter-configuration

## Summary

Configure Splatter's newSplatParams/splatSimulate(method='groups') to generate two-group scRNA-seq count matrices with a rare and a common cell type, used as ground-truth input for benchmarking EDGE feature gene detection.

## When to use

Use when simulated scRNA-seq data with known differentially expressed (feature) genes and a rare/common two-cell-type structure is needed to benchmark feature gene detection or dimensionality reduction methods.

## When NOT to use

- The analysis target is real scRNA-seq data where no ground-truth DE gene labels exist — Splatter simulation provides synthetic truth, not empirical validation
- More than two cell types or continuous trajectories are needed — the 'groups' method configured here yields exactly two groups (rare and common)
- A fixed dataset size different from the small-scale regime (hundreds of genes, hundreds to 1000 cells) is required and parameter re-tuning is out of scope

## Inputs

- Simulation specification: number of genes (e.g. nGenes=500), cell count (batchCells=1000 or 500), random seed (e.g. 2020), group probabilities (e.g. c(0.98,0.02))
- Dropout scenario setting (low vs. high dropout) encoded in Splatter parameters

## Outputs

- Simulated gene-by-cell count matrix from counts(simu_dt) with known differentially expressed (feature) genes
- Cell-type label vector from colData(simu_dt)[,3] identifying rare vs. common groups

## How to apply

Build simulation parameters with newSplatParams(nGenes, seed, batchCells, group.prob), fixing a random seed (e.g. 2020) for reproducibility and setting group probabilities such as c(0.98, 0.02) to yield a rare (2%) and a common (98%) cell type. Call splatSimulate(params=params, method='groups') and extract counts with counts(simu_dt); group labels come from colData(simu_dt)[,3]. For the EDGE benchmark in this paper, 500 genes and 1000 cells (batchCells=1000) were used, with low- and high-dropout settings distinguished by Splatter dropout parameters. The simulate step is repeated across replications (100 in the Table 2 benchmark) so that the fraction of true feature genes recovered among EDGE's top-15/top-30 rankings can be averaged. Rationale: the 'groups' method produces genes flagged as differentially expressed between groups, providing ground truth feature genes that no real dataset can offer.

## Related tools

- **splatter** (Simulates two-group single-cell RNA-seq count data with known feature genes via newSplatParams and splatSimulate(method='groups'))
- **EDGE** (Downstream method evaluated on the Splatter-simulated data for feature gene detection) — github.com/shawnstat/EDGE
- **R** (Language in which the Splatter simulation and parameter configuration are executed)

## Examples

```
params <- newSplatParams(nGenes=500, seed=2020, batchCells=1000, group.prob=c(0.98,0.02)); simu_dt <- splatSimulate(params=params, method="groups")
```

## Evaluation signals

- Re-running with the same seed (e.g. myseed <- 2020) reproduces the identical count matrix and labels
- colData(simu_dt)[,3] yields approximately the requested group proportions (e.g. ~2% rare cells for gp = c(0.98,0.02); 15 minor-group cells out of 500 in the README example)
- The count matrix dimensions match nGenes x batchCells (e.g. 500 genes x 1000 cells)
- Averaging EDGE top-15/top-30 true-gene counts over 100 replications lands near the reported Table 2 values (15.00/27.06 low dropout; 14.79/25.74 high dropout) within Monte Carlo variability

## Limitations

- The README example uses a small configuration (500 cells, 200 genes); the paper's feature gene benchmark uses 1000 cells and 500 genes — parameters must match the intended benchmark scale
- Large simulated datasets (e.g. 1000 cells x 1500 genes for dimensionality reduction scenarios) increase runtime and downstream memory needs, and EDGE's hash table size trades memory for accuracy
- No version numbers are given for R or Splatter, so exact reproducibility across versions is not guaranteed

## Evidence

- [readme] params <- newSplatParams(nGenes=GC, seed=myseed, batchCells=BC, group.prob=gp): "params <- newSplatParams(nGenes=GC, seed=myseed, batchCells=BC, group.prob=gp)"
- [methods] simu_dt <- splatSimulate(params=params,method="groups"): "simu_dt <- splatSimulate(params=params,method="groups")"
- [discussion] We utilized the R package Splatter to simulate scRNA-seq datasets: "We utilized the R package Splatter to simulate scRNA-seq datasets"
- [readme] There are two groups in the dataset. The minor group contains 15 cells.: "There are two groups in the dataset. The minor group contains 15 cells."
