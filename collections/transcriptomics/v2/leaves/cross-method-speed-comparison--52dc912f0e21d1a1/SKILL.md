---
name: cross-method-speed-comparison
description: Use when comparing the computational efficiency of scDist, Augur, and
  nDEG on the same scRNA-seq data — full datasets, subsamples of varying size, or
  large simulated cohorts (~1M cells) — to record timing ratios.
when_to_use_negative:
- The goal is to compare detection accuracy or biological findings rather than computational
  runtime — scDist's statistical validation (Fig. S20/S21 timings) does not establish
  superiority of perturbation calls.
- The compared methods cannot run on the same input, e.g. data are raw counts, since
  scDist requires normalized expression (scTransform Pearson residuals recommended)
  and cannot currently be applied directly to raw counts.
- Only a single cell type or single sample is available, making replicate-aware timing
  comparisons and mixed-effects fitting (fixed.effects="response", random.effects="patient")
  uninformative.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scDist
  role: Method under test; mixed-effects-model perturbation detector timed against
    Augur and nDEG on COVID-19, subsampled, and 1M-cell simulated data
  repo: github.com/phillipnicol/scDist
- name: Augur
  role: Machine-learning-based comparator whose ~5 min runtime on the 13-sample COVID-19
    dataset is benchmarked against scDist's ~50 s
- name: nDEG
  role: Comparator on the 1,000,000-cell simulated dataset against which scDist's
    >60x speedup is recorded
- name: lme4
  role: Fits the per-PC linear mixed-effects models underlying scDist; its fitting
    cost contributes to the measured runtime
- name: lmerTest
  role: Provides Satterthwaite's approximation of degrees of freedom used in scDist's
    Wald tests during benchmarked runs
- name: irlba
  role: Computes the low-dimensional PCA embedding whose loadings serve as the plug-in
    estimator for U in scDist, affecting benchmarked runtime
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:cbe05940817a851af7610d6d0bc729a047bc5cecd7c29d067174ef1631b3e917
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Augur
  - irlba
  - lme4
  - lmerTest
  - nDEG
  - scDist
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-method-speed-comparison

## Summary

Benchmark the wall-clock runtime of scDist against competing cell-type perturbation methods (Augur, nDEG) on the same scRNA-seq inputs, including a 13-sample COVID-19 dataset, size-varied subsamples, and a 1,000,000-cell simulation, to quantify speedups (~50 s vs ~5 min vs Augur; ~5-fold average across subsamples; >60x vs nDEG; ~10 min for 1M cells).

## When to use

Use when comparing the computational efficiency of scDist, Augur, and nDEG on the same scRNA-seq data — full datasets, subsamples of varying size, or large simulated cohorts (~1M cells) — to record timing ratios.

## When NOT to use

- The goal is to compare detection accuracy or biological findings rather than computational runtime — scDist's statistical validation (Fig. S20/S21 timings) does not establish superiority of perturbation calls.
- The compared methods cannot run on the same input, e.g. data are raw counts, since scDist requires normalized expression (scTransform Pearson residuals recommended) and cannot currently be applied directly to raw counts.
- Only a single cell type or single sample is available, making replicate-aware timing comparisons and mixed-effects fitting (fixed.effects="response", random.effects="patient") uninformative.

## Inputs

- 13-sample COVID-19 scRNA-seq dataset (normalized expression matrix plus per-cell condition/patient/cluster metadata)
- Size-varied subsamples of the COVID-19 dataset
- 1,000,000-cell simulated dataset generated with scDist::simData(nct=10, N1=5, N2=5)

## Outputs

- runtime_benchmark.csv with wall-clock timings for scDist, Augur, and nDEG on each input
- Speedup ratio table (~5-fold vs Augur across subsamples, >60x vs nDEG, ~10 min for 1M cells)

## How to apply

Install scDist from GitHub with devtools::install_github("phillipnicol/scDist") and load it alongside Augur and nDEG. Run all three methods on the 13-sample COVID-19 scRNA-seq dataset with a timing wrapper, reproducing ~50 s for scDist versus ~5 min for Augur (Fig. S20). Generate size-varied subsamples of the same dataset, rerun all three methods per subsample, and average the speedup to reproduce the reported ~5-fold advantage. Separately, generate a 1,000,000-cell simulated dataset with simData(nct=10, N1=5, N2=5), time scDist (~10 min, Fig. S21), time nDEG on the identical data, and record the >60x speedup. Compile all wall-clock timings and speedup ratios into runtime_benchmark.csv and validate against the four reported targets.

## Related tools

- **scDist** (Method under test; mixed-effects-model perturbation detector timed against Augur and nDEG on COVID-19, subsampled, and 1M-cell simulated data) — github.com/phillipnicol/scDist
- **Augur** (Machine-learning-based comparator whose ~5 min runtime on the 13-sample COVID-19 dataset is benchmarked against scDist's ~50 s)
- **nDEG** (Comparator on the 1,000,000-cell simulated dataset against which scDist's >60x speedup is recorded)
- **lme4** (Fits the per-PC linear mixed-effects models underlying scDist; its fitting cost contributes to the measured runtime)
- **lmerTest** (Provides Satterthwaite's approximation of degrees of freedom used in scDist's Wald tests during benchmarked runs)
- **irlba** (Computes the low-dimensional PCA embedding whose loadings serve as the plug-in estimator for U in scDist, affecting benchmarked runtime)

## Examples

```
library(scDist); sim <- simData(nct=10,N1=5,N2=5); system.time(out <- scDist(sim$Y,sim$meta.data,fixed.effects="response",random.effects="patient",clusters="clusters"))
```

## Evaluation signals

- Full COVID-19 timing table shows scDist ~50 s versus Augur ~5 min (matching Fig. S20)
- Average speedup of scDist over Augur across size-varied subsamples is ~5-fold
- scDist runtime on the 1,000,000-cell simData(nct=10, N1=5, N2=5) dataset is ~10 minutes (matching Fig. S21)
- nDEG runtime on the same 1M-cell data yields a >60x scDist speedup
- runtime_benchmark.csv exists and contains wall-clock timings and speedup ratios for all three methods on all tested inputs

## Limitations

- Reported timings were obtained on the paper's hardware and specific 13-sample COVID-19 and simulated datasets; absolute runtimes will vary with cell count, gene count, and machine.
- scDist's speed advantage depends on its low-dimensional PCA approximation and fixed heuristic number of PCs (20 or 50); an adaptive PC-selection procedure does not exist, and results are sensitive to the choice of K.
- Benchmarking does not speak to statistical accuracy: naive approaches can yield false positives from individual-to-individual and cohort-to-cohort variability, and over-clustering can bias scDist toward false negatives.

## Evidence

- [supplementary] On the COVID-19 dataset with 13 samples, scDist completed the analysis in around 50 seconds, while Augur required 5 minutes.: [evidence span withheld — anchor status: not_found]
- [methods] simData generation of 1M-cell dataset: "Generate simulated data with 10 cell types and 5 patients in each group"
- [methods] Install scDist from GitHub via devtools: "From the R console, `devtools::install_github("phillipnicol/scDist")`"
- [discussion] scDist is computationally fast: "We present scDist, a statistically rigorous and computationally fast method for detecting cell-type specific differences across multiple groups or conditions."
- [readme] README demo invocation: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response",
              random.effects="patient",
              clusters="clusters")"
