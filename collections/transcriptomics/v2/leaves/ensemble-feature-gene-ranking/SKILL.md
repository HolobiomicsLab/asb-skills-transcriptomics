---
name: ensemble-feature-gene-ranking
description: Use when a simulated or real scRNA-seq count matrix needs a ranked list
  of feature genes via EDGE, and detection accuracy must be scored against known differentially
  expressed genes (top-15/top-30 counts).
when_to_use_negative:
- The goal is only a 2D cell embedding for visualization (EDGE-as-dimensionality-reduction
  or UMAP/t-SNE is the appropriate path, not the feature-gene accuracy evaluation).
- Input is already a ranked or binary feature-gene list, so no ranking from raw counts
  is needed.
- The data are not scRNA-seq counts and there is no notion of differentially expressed
  genes associated with cell types to validate against.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: EDGE
  role: Runs endr(dat_log, custom_defs) to produce the ensemble feature gene ranking;
    installed via install_github('shawnstat/EDGE')
  repo: https://github.com/shawnstat/EDGE
- name: Splatter
  role: Simulates the two-group scRNA-seq count matrices with known DE (feature) genes
    under low- and high-dropout settings via splatSimulate(method='groups')
- name: RSpectra
  role: Dependency used by EDGE to perform the large-scale eigenvalue decomposition
    during spectral embedding
- name: Rcpp
  role: Build dependency (>= 1.0.1) required to install and run the EDGE package
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:ef1441ae939e062f668cfcf94926a1dbbc197ef7dfa53498f6dab2695b3f5b16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shawnstat/EDGE
  tools:
  - EDGE
  - RSpectra
  - Rcpp
  - Splatter
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# ensemble-feature-gene-ranking

## Summary

Rank candidate feature (differentially expressed) genes in scRNA-seq data using EDGE's ensemble of weak similarity learners, then quantify detection accuracy by counting true DE genes among the top 15 and top 30 ranked genes. This reproduces Table 2 of the EDGE paper on Splatter-simulated two-cell-type datasets.

## When to use

Use when a simulated or real scRNA-seq count matrix needs a ranked list of feature genes via EDGE, and detection accuracy must be scored against known differentially expressed genes (top-15/top-30 counts).

## When NOT to use

- The goal is only a 2D cell embedding for visualization (EDGE-as-dimensionality-reduction or UMAP/t-SNE is the appropriate path, not the feature-gene accuracy evaluation).
- Input is already a ranked or binary feature-gene list, so no ranking from raw counts is needed.
- The data are not scRNA-seq counts and there is no notion of differentially expressed genes associated with cell types to validate against.

## Inputs

- Gene-by-cell raw count matrix from splatSimulate(method='groups') (e.g. 500 genes × 1000 cells, two groups via group.prob)
- splatParams object (newSplatParams with nGenes, batchCells, seed, group.prob) including low- and high-dropout settings
- custom endr_defs configuration (n_wl=1000, n_dm, n_neigs=15, opt=TRUE, seed=2020, H)
- Labels of true simulated differentially expressed genes per replication

## Outputs

- EDGE feature gene ranking from endr(dat_log, custom_defs)
- Per-replication counts of true DE genes among top 15 and top 30 ranked genes
- table2_feature_genes.csv with averaged counts per dropout scenario (Table 2 values)

## How to apply

Simulate a two-group scRNA-seq dataset with Splatter's splatSimulate(method='groups') using newSplatParams with nGenes=500, batchCells=1000, a fixed seed (e.g. 2020), and group.prob yielding a rare and a common cell type, under both low- and high-dropout settings. Normalize counts by sweeping library sizes (rs/rs_med) to the median and log2-transform after adding a pseudocount to get dat_log. Run endr(dat_log, custom_defs) with an endr_defs object configured to n_wl=1000 weak learners, n_dm genes per weak learner, n_neigs=15, opt=TRUE, seed=2020, and hash table size H (package default or the memory-light 101,107). The rationale is that EDGE's massive weak learners learn cell similarities while the genes making significant contributions are selected as feature genes, with the top-gene cutoff set at μ − 1.5 × σ of averaged entropy values. For each of 100 replications, count how many of the top 15 and top 30 ranked genes are the true simulated DE genes, then average separately per dropout scenario and write the values to table2_feature_genes.csv.

## Related tools

- **EDGE** (Runs endr(dat_log, custom_defs) to produce the ensemble feature gene ranking; installed via install_github('shawnstat/EDGE')) — https://github.com/shawnstat/EDGE
- **Splatter** (Simulates the two-group scRNA-seq count matrices with known DE (feature) genes under low- and high-dropout settings via splatSimulate(method='groups'))
- **RSpectra** (Dependency used by EDGE to perform the large-scale eigenvalue decomposition during spectral embedding)
- **Rcpp** (Build dependency (>= 1.0.1) required to install and run the EDGE package)

## Examples

```
custom_defs <- endr_defs; custom_defs$n_wl <- 1000; custom_defs$n_dm <- 10; custom_defs$n_neigs <- 15; custom_defs$seed <- 2020; simu.endr <- endr(dat_log, custom_defs)
```

## Evaluation signals

- Averaged true-gene counts across 100 replications reproduce Table 2: 15.00 and 27.06 among top 15/30 in the low-dropout scenario and 14.79 and 25.74 in the high-dropout scenario, within Monte Carlo variability.
- Normalization invariance: dat_log must equal t(log2(sweep(counts_dt, 2, rs/rs_med, '/') + 1)) with rs = colSums(counts_dt).
- The endr_defs configuration matches the paper settings (n_wl=1000, n_neigs=15, opt=TRUE, seed=2020) so rankings are reproducible under the fixed seed.
- Every gene counted as a detected feature gene must belong to the true simulated DE gene set of that replication (no false positives counted in the top-15/top-30 tallies).
- table2_feature_genes.csv contains four values (two cutoffs × two dropout scenarios) and nothing else.

## Limitations

- The optimal hash table size is 1,017,881 but requires large memory; the README uses a reduced H of 101,107, which 'works well' most of the time but may affect results.
- The number of genes B per weak learner follows a heuristic (typically < 0.05*G) and exact per-dataset values are not stated; hyperparameters a and b are not reported.
- Version numbers for R, Splatter, RSpectra, and EDGE are unspecified, and results are subject to Monte Carlo sampling variability across 100 replications.

## Evidence

- [discussion] Feature gene cutoff via entropy: "We choose μ −  1.5 × σ as the cutoff value to select top feature genes"
- [intro] Weak learners select feature genes: "massive weak learners are allowed in EDGE to learn cell similarities accurately, while the genes that make significant contributions in the process are selected as feature genes."
- [readme] Splatter simulation setup: "params <- newSplatParams(nGenes=GC, seed=myseed, batchCells=BC, group.prob=gp)"
- [readme] Normalization and log transform: "dat_norm <- sweep(counts_dt,2,rs/rs_med,"/")"
- [methods] endr call with custom definitions: "simu.endr <- endr(dat_log,custom_defs)"
- [discussion] Gene count for simulated feature gene identification: "We selected 1000 top variable genes for real datasets and 500 for simulated datasets"
