---
name: differential-expression-aucpr-assessment
description: Use when evaluating an imputation or preprocessing method's effect on
  differential expression accuracy in scRNA-seq data where a ground-truth set of DE
  genes is known, especially as a function of the zero-count (dropout) proportion.
when_to_use_negative:
- No ground-truth DE gene set is available (e.g., real experimental scRNA-seq data
  without simulated truth) — AUC-PR against truth is undefined
- Data already contain extremely high dropout rates from droplet-based protocols,
  where the paper notes imputation efficiency degrades and improved methods remain
  a future direction
- The question is imputation fidelity of individual expression values rather than
  downstream DE accuracy — AUC-PR on DE calls does not measure per-value recovery
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3517
tools:
- name: scImpute
  role: Imputes dropout values in the raw count matrix (Kcluster = 3, drop_thre =
    0.5, labeled = FALSE, ncores = 10) prior to DE analysis and AUC-PR scoring
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:f78a7f1328540492eabda2fc29e4de145b9de772c3ea7643a65829115d8877ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - scImpute
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# differential-expression-aucpr-assessment

## Summary

Quantifies how well a scRNA-seq imputation method (e.g., scImpute) supports downstream differential expression (DE) analysis by computing the area under the precision-recall curve (AUC-PR) of DE calls against a simulated ground truth, across controlled dropout rates.

## When to use

Use when evaluating an imputation or preprocessing method's effect on differential expression accuracy in scRNA-seq data where a ground-truth set of DE genes is known, especially as a function of the zero-count (dropout) proportion.

## When NOT to use

- No ground-truth DE gene set is available (e.g., real experimental scRNA-seq data without simulated truth) — AUC-PR against truth is undefined
- Data already contain extremely high dropout rates from droplet-based protocols, where the paper notes imputation efficiency degrades and improved methods remain a future direction
- The question is imputation fidelity of individual expression values rather than downstream DE accuracy — AUC-PR on DE calls does not measure per-value recovery

## Inputs

- Simulated raw scRNA-seq count matrix (rows = genes, columns = cells; e.g., 20,000 genes x 150 cells, ~75% zeros) as CSV
- Ground-truth set of differentially expressed genes (e.g., 810 simulated DE genes)
- scImpute-imputed count matrix (scimpute_count.csv)
- Dropout proportion setting per simulation replicate

## Outputs

- Area under the precision-recall curve (AUC-PR) for DE analysis on imputed data vs. ground truth
- Raw vs. imputed count matrices (scimpute_count.csv for the imputed matrix)
- Simulation evaluation summary table (simulation_evaluation_metrics.csv)
- Optional PCA cluster-separation metrics (within-cluster sum-of-squares, e.g., 94 imputed vs. 2646 raw)

## How to apply

Simulate a count matrix per the Methods design — 3 cell subpopulations with 50 cells each, 20,000 genes, 810 DE genes, gene means drawn from Normal(1.8, 0.5) and standard deviations from Normal(0.6, 0.1) — and impose dropout with probability exp(-0.1*mean^2) to reach a target zero fraction (e.g., ~75% zeros). Run scImpute's exported scimute/scimpute() function on the raw count matrix (rows = genes, columns = cells) with Kcluster = 3, drop_thre = 0.5, labeled = FALSE, and parallel computation (ncores = 10), which writes scimpute_count.csv. Optionally verify clustering quality via PCA: retain the first two principal components of raw vs. imputed matrices and compare within-cluster sum-of-squares (reported separation: 94 imputed vs. 2646 raw). Then perform DE analysis on the imputed data, rank genes against the 810-DE-gene ground truth, and compute the AUC-PR at the current dropout setting — the paper's finding is that scImpute retains >80.0% AUC-PR even with 75.0% zero counts, and DE accuracy rises as dropout proportion decreases. Write the imputed matrix and all metrics to files, with the final summary saved as simulation_evaluation_metrics.csv.

## Related tools

- **scImpute** (Imputes dropout values in the raw count matrix (Kcluster = 3, drop_thre = 0.5, labeled = FALSE, ncores = 10) prior to DE analysis and AUC-PR scoring) — https://github.com/Vivianstats/scImpute

## Examples

```
scimpute(count_path="raw_count.csv", infile="csv", outfile="csv", out_dir="./", labeled=FALSE, drop_thre=0.5, Kcluster=3, ncores=10)
```

## Evaluation signals

- AUC-PR of DE calls exceeds 80.0% at the 75% zero-counts setting
- DE accuracy monotonically increases as the dropout proportion decreases across simulated conditions
- PCA within-cluster sum-of-squares drops markedly after imputation (expected ~94 for imputed vs. ~2646 for raw data)
- Imputed matrix has the same dimensions as the input raw count matrix and is written to scimpute_count.csv
- simulation_evaluation_metrics.csv contains the recorded metric values for each dropout setting

## Limitations

- Imputation efficiency degrades when dropout rates in raw data are severely high (droplet-based technologies); improving this is a stated future direction
- Only values with dropout probability larger than drop_thre are imputed, so the drop_thre choice bounds how much of the matrix is altered
- DE accuracy gains depend on the simulation's distributional assumptions (Normal(1.8, 0.5) means, Normal(0.6, 0.1) SDs, dropout probability exp(-0.1*mean^2)), not necessarily real data

## Evidence

- [abstract] scImpute improves DE accuracy as measured by AUC-PR: "scImpute is shown to identify likely dropouts, enhance the clustering of cell subpopulations, improve the accuracy of differential expression analysis, and aid the study of gene expression dynamics."
- [methods] Only values above dropout probability threshold are imputed: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
- [readme] scImpute applied before downstream analyses including differential gene expression analysis: "`scImpute` is developed to accurately and robustly impute the dropout values in scRNA-seq data. `scImpute` can be applied to raw read count matrix before the users perform downstream analyses"
- [methods] Parallel computing recommendation for scImpute: "We highly recommend using parallel computing with `scImpute`, which will significantly reduce the computation time."
