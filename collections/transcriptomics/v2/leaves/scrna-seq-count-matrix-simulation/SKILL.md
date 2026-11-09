---
name: scrna-seq-count-matrix-simulation
description: Use when you need to assess how dropout prevalence affects an imputation
  method's performance — e.g., benchmarking scImpute on simulated count matrices with
  known cell subpopulations and DE gene ground truth.
when_to_use_negative:
- Your data are real scRNA-seq reads without a known DE-gene ground truth — precision-recall
  evaluation requires simulated labels
- The count matrix is already imputed or normalized; scImpute is designed to be applied
  to the raw count matrix before downstream analyses
- The number of cell subpopulations is unknown and cannot be supplied as Kcluster
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scImpute
  role: Imputes dropout values in the simulated raw count matrix; called via scimpute()
    with Kcluster = 3 and drop_thre = 0.5 before PCA and DE evaluation
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:e4c6d3ff17041123ad928585db1fa65edb4d48c81a94308008386e84b4f5ea9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - scImpute
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# scrna-seq-count-matrix-simulation

## Summary

Simulate scRNA-seq count matrices with controlled dropout to benchmark imputation: build a three-cell-type count matrix (3 subpopulations × 50 cells, 20,000 genes, 810 DE genes), impose dropout with probability exp(-0.1*mean^2) up to ~75% zeros, then evaluate scImpute-imputed data via PCA cluster separation and DE precision-recall AUC.

## When to use

Use when you need to assess how dropout prevalence affects an imputation method's performance — e.g., benchmarking scImpute on simulated count matrices with known cell subpopulations and DE gene ground truth.

## When NOT to use

- Your data are real scRNA-seq reads without a known DE-gene ground truth — precision-recall evaluation requires simulated labels
- The count matrix is already imputed or normalized; scImpute is designed to be applied to the raw count matrix before downstream analyses
- The number of cell subpopulations is unknown and cannot be supplied as Kcluster

## Inputs

- Simulation design parameters: 3 cell subpopulations × 50 cells, 20,000 genes, 810 DE genes
- Gene mean distribution Normal(1.8, 0.5) and standard deviation distribution Normal(0.6, 0.1)
- scImpute call parameters: Kcluster = 3, drop_thre = 0.5, labeled = FALSE, ncores = 10
- Raw count matrix (rows = genes, columns = cells) in CSV format

## Outputs

- scimpute_count.csv — imputed count matrix in the output directory
- simulation_evaluation_metrics.csv — within-cluster sum-of-squares (PCA PC1/PC2) and DE precision-recall AUC values
- Simulated ground-truth DE gene labels (810 genes) for PR-curve evaluation

## How to apply

In R, simulate the count matrix per the Methods design: 3 cell subpopulations with 50 cells each, 20,000 genes, 810 differentially expressed genes, gene means drawn from Normal(1.8, 0.5) and standard deviations from Normal(0.6, 0.1), imposing dropout with probability exp(-0.1*mean^2) until ~75% of entries are zeros. Run the exported scimpute() function on the raw count matrix (rows = genes, columns = cells) with Kcluster = 3, drop_thre = 0.5, labeled = FALSE, and ncores = 10, which writes scimpute_count.csv to the output directory; only values with dropout probability larger than drop_thre are imputed. Apply PCA to both raw and imputed matrices, retain the first two principal components, and compute within-cluster sum-of-squares (expected ~94 imputed vs ~2646 raw at high dropout). Perform differential expression on the imputed data against the 810-DE-gene ground truth and compute area under the precision-recall curve. Save the imputed matrix and metrics to simulation_evaluation_metrics.csv; DE accuracy should increase as dropout proportion decreases, with AUC >80.0% even at 75% zeros.

## Related tools

- **scImpute** (Imputes dropout values in the simulated raw count matrix; called via scimpute() with Kcluster = 3 and drop_thre = 0.5 before PCA and DE evaluation) — https://github.com/Vivianstats/scImpute

## Examples

```
scimpute(count_path = "raw_count.csv", infile = "csv", outfile = "csv", out_dir = "./", labeled = FALSE, drop_thre = 0.5, Kcluster = 3, ncores = 10)
```

## Evaluation signals

- At the ~75% zeros setting, within-cluster sum-of-squares drops from ~2646 (raw) to ~94 (imputed) on the first two principal components
- Area under the DE precision-recall curve on imputed data exceeds 80.0% at 75% zero counts
- Imputed matrix (scimpute_count.csv) has the same gene × cell dimensions as the raw input count matrix
- DE accuracy on imputed data increases monotonically as the simulated dropout proportion decreases
- Fraction of zero entries in the raw simulated matrix is approximately 75%

## Limitations

- Imputation efficiency degrades when dropout rates in the raw data are severely high (e.g., droplet-based technologies), noted as a future direction
- Choice of parameter Kcluster (number of cell populations) is left to the user without a definitive selection procedure
- No software version for scImpute is specified; a filtering step on genes is acceptable but most genes should be present to ensure robust identification of dropouts

## Evidence

- [full_text] Simulation workflow: [evidence span withheld — anchor status: not_found]
- [methods] Imputation threshold: "Only the values that have dropout probability larger than `drop_thre` are imputed by `scImpute`."
- [abstract] Benchmark finding: "Evaluation based on both simulated and real human and mouse scRNA-seq data suggests that scImpute is an effective tool to recover transcriptome dynamics masked by dropouts."
- [readme] Tool usage from README: "Its only input is the raw count matrix with rows representing genes and columns representing cells. It will output an imputed count matrix with the same dimension."
