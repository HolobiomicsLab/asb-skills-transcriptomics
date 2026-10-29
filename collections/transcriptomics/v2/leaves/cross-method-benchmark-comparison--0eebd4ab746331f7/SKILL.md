---
name: cross-method-benchmark-comparison
description: Use when a new clonal-inference method must be evaluated head-to-head
  against predecessor methods on identical patient datasets, using a shared downstream
  metric such as cell-to-clone assignment coverage.
when_to_use_negative:
- Only one method is runnable — a cross-method comparison requires the same inputs
  processed by Canopy2, Canopy, and Cardelino
- The comparison target is a simulated-data accuracy metric rather than real-data
  unassigned-cell percentages; simulated validation of Canopy2 is a separate skill
- Input read counts have not passed the pre-processing QC pipeline, so competing methods
  would be evaluated on inconsistent SNV callsets
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3173
tools:
- name: Canopy2
  role: 'Method under benchmark: joint bulk DNA WES + scRNA-seq Bayesian inference
    producing cell-to-clone assignments whose unassigned-cell percentage is compared'
  repo: https://github.com/annweideman/canopy2
- name: Canopy
  role: Predecessor bulk-plus-single-cell method run on the same inputs as a baseline
    competitor
- name: Cardelino
  role: Competing method taking scRNA-seq as input with a guide clonal tree from a
    first-step Canopy run
- name: R
  role: Language in which the canopy2 package functions (get_trees, get_diagnostics,
    get_best_tree) are executed
  repo: https://github.com/annweideman/canopy2
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:8f1aedc1420b23d218dfee23dbfb5b4b004c0528a3179a830e00451f009b59cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/annweideman/canopy2
  tools:
  - Canopy
  - Canopy2
  - Cardelino
  - R
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-method-benchmark-comparison

## Summary

Benchmark a tumor phylogeny method (Canopy2) against competing tools (Canopy, Cardelino) on the same real-data patient inputs by computing a common cell-assignment metric — the percentage of malignant cells left unassigned to subclones — and compiling the results into a comparison table.

## When to use

Use when a new clonal-inference method must be evaluated head-to-head against predecessor methods on identical patient datasets, using a shared downstream metric such as cell-to-clone assignment coverage.

## When NOT to use

- Only one method is runnable — a cross-method comparison requires the same inputs processed by Canopy2, Canopy, and Cardelino
- The comparison target is a simulated-data accuracy metric rather than real-data unassigned-cell percentages; simulated validation of Canopy2 is a separate skill
- Input read counts have not passed the pre-processing QC pipeline, so competing methods would be evaluated on inconsistent SNV callsets

## Inputs

- Post-processed Rs (mutation x single-cell alternative read counts) and Xs (total read counts) matrices from scRNA-seq
- Post-processed Rb and Xb bulk DNA WES read count matrices
- Mutation-specific bursting-kinetics estimates alpha and beta (1 x M vectors)
- Guide clonal tree from a first-step Canopy run (for Cardelino comparison)
- Datasets for patients BC03, BC07 (breast cancer, GSE75688/SRP067248) and GBM9, GBM10 (glioblastoma, EGAS00001001880)

## Outputs

- unassigned_cells_benchmark.csv: per-patient, per-method percentages of unassigned malignant cells (e.g., BC03 15.6/27.3/26.0% for Canopy2/Canopy/Cardelino)
- Per-method cell-to-clone assignment matrices (Ps) and selected clonal tree configurations

## How to apply

Run each competing method on the same post-processed inputs (Rs, Xs, Rb, Xb read count matrices plus mutation-specific bursting-kinetics estimates alpha and beta, with default sequencing-error hyperparameters kappa=1 and tau=999). For Canopy2, run get_trees() MCMC over Klist=3:10 with >=10 chains, thinning (e.g., thin=20), burn-in (e.g., pburn=0.2), and 50k-100k iterations, verify convergence with get_diagnostics(), and select the minimum-modified-BIC configuration via get_best_tree(). For Cardelino, supply a guide clonal tree produced from a first-step Canopy run, mirroring its published usage. For each patient (BC03, BC07, GBM9, GBM10) and each method, compute the percentage of malignant cells unassigned to any subclone and compile the results into unassigned_cells_benchmark.csv; lower unassigned percentages indicate better cell-to-clone assignment coverage.

## Related tools

- **Canopy2** (Method under benchmark: joint bulk DNA WES + scRNA-seq Bayesian inference producing cell-to-clone assignments whose unassigned-cell percentage is compared) — https://github.com/annweideman/canopy2
- **Canopy** (Predecessor bulk-plus-single-cell method run on the same inputs as a baseline competitor)
- **Cardelino** (Competing method taking scRNA-seq as input with a guide clonal tree from a first-step Canopy run)
- **R** (Language in which the canopy2 package functions (get_trees, get_diagnostics, get_best_tree) are executed) — https://github.com/annweideman/canopy2

## Examples

```
trees<-get_trees(Rs=Rs,Xs=Xs,Rb=Rb,Xb=Xb,alpha=alpha,beta=beta,kappa=1,tau=999,Klist=3:10,thin=20,pburn=0.2); best<-get_best_tree(trees)
```

## Evaluation signals

- unassigned_cells_benchmark.csv contains unassigned-cell percentages for Canopy2, Canopy, and Cardelino for all four patients (BC03, BC07, GBM9, GBM10)
- Reproduced values match the published pattern: BC03 15.6/27.3/26.0%, BC07 0/29.4/11.8%, GBM9 14.7/36.4/30.2%, GBM10 0/2.7/16.2% (Canopy2/Canopy/Cardelino)
- Canopy2 unassigned percentage is lowest (or tied) in all four patients, consistent with the finding that Canopy2 assigned more cells to clones
- Canopy2 configuration used for the benchmark is the minimum-modified-BIC tree from get_best_tree() with passing MCMC diagnostics (lag ACF below 0.3, aligned posterior densities, noise-like trace plots)
- Cardelino was run with the same guide-tree construction protocol (first-step Canopy run) for each patient, not with ad-hoc trees

## Limitations

- Canopy2 is the slowest of the four evaluated methods (microbenchmarked on a Windows laptop with a 12th Gen Intel Core i9-12900HK, ncores=20), so cluster resources are needed for 50k-100k MCMC iterations across Klist=3:10
- Metropolis-within-Gibbs sampling makes convergence dependent on proposal distributions; poor proposals can cause settling on a local rather than global optimum, potentially biasing the benchmark
- The unassigned-cell metric only reflects assignment coverage, not tree-topology accuracy; other aspects of performance are not captured by this benchmark

## Evidence

- [intro] Cardelino [13] takes scRNA-seq as input and indirectly utilizes bulk DNA-seq data through a guide clonal tree produced from a first-step run of Canopy [21]: "Cardelino [13] takes scRNA-seq as input and indirectly utilizes bulk DNA-seq data through a guide clonal tree produced from a first-step run of Canopy [21]"
- [abstract] We further assess the performance of Canopy2 through application to breast cancer and glioblastoma data, benchmarking against existing methods.: "We further assess the performance of Canopy2 through application to breast cancer and glioblastoma data, benchmarking against existing methods."
- [methods] Select the optimal number of subclones per patient with get_best_tree(), which minimizes a modified Bayesian information criterion (BIC) across the K range.: [evidence span withheld — anchor status: not_found]
- [readme] The average error rate of next-generation sequencing is reported to be 0.1% per nucleotide, so these parameters have been set to default values of kappa=1 and tau = 999: [evidence span withheld — anchor status: external_doc]
- [discussion] Microbenchmarking on a Windows laptop with a 12th Gen Intel Core i9-12900HK processor (ncores=20) showed that Canopy2 is the slowest of the four methods evaluated: "Microbenchmarking on a Windows laptop with a 12th Gen Intel Core i9-12900HK processor (ncores=20) showed that Canopy2 is the slowest of the four methods evaluated"
