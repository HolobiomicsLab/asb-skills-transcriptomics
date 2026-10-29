---
name: pca-based-clustering-evaluation
description: Use when you need to demonstrate that imputation or preprocessing improves
  cell-subpopulation separation in scRNA-seq count matrices, by comparing PCA-based
  cluster compactness of raw vs imputed data against a known ground-truth clustering.
when_to_use_negative:
- When cell-type labels are unknown and no ground-truth clustering exists to define
  within-cluster sum-of-squares — cluster compactness cannot be computed without labels
- When the goal is imputation itself rather than evaluation of its effect on clustering;
  use scimpute() directly instead of this evaluation wrapper
- When data dimensionality or nonlinearity makes two PCs a poor summary (e.g., trajectories
  needing time-series analysis rather than subpopulation separation)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3525
- http://edamontology.org/topic_0203
tools:
- name: scImpute
  role: Produces the imputed count matrix (via scimpute(), Kcluster = 3, drop_thre
    = 0.5) whose cluster separation is compared against the raw matrix by PCA
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:43b08ca18f40e8d73fe6d1e37161ef1f5749c57ad8a03bd632e0ab9f7a4ca591
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - scImpute
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3525
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# pca-based-clustering-evaluation

## Summary

Quantify how well cell subpopulations separate in scRNA-seq count matrices by projecting raw and scImpute-imputed data onto the first two principal components and comparing within-cluster sum-of-squares, as done in the scImpute paper's simulated three-cell-type benchmark (within-cluster SS 94 after imputation vs 2646 for raw data).

## When to use

Use when you need to demonstrate that imputation or preprocessing improves cell-subpopulation separation in scRNA-seq count matrices, by comparing PCA-based cluster compactness of raw vs imputed data against a known ground-truth clustering.

## When NOT to use

- When cell-type labels are unknown and no ground-truth clustering exists to define within-cluster sum-of-squares — cluster compactness cannot be computed without labels
- When the goal is imputation itself rather than evaluation of its effect on clustering; use scimpute() directly instead of this evaluation wrapper
- When data dimensionality or nonlinearity makes two PCs a poor summary (e.g., trajectories needing time-series analysis rather than subpopulation separation)

## Inputs

- Raw scRNA-seq count matrix (rows = genes, columns = cells), e.g. CSV
- scImpute-imputed count matrix (scimpute_count.csv) with same dimensions
- Ground-truth cell-subpopulation labels (e.g., 3 simulated cell types, 50 cells each)
- Optional: ground-truth set of differentially expressed genes (810 in the simulation)

## Outputs

- First two principal component scores for raw and imputed matrices
- Within-cluster sum-of-squares per data version (e.g., 94 imputed vs 2646 raw)
- Precision-recall curve and AUPRC for DE analysis on imputed data (expected >80.0%)
- Summary metrics file, e.g., simulation_evaluation_metrics.csv

## How to apply

Start from a raw gene-by-cell count matrix (in the scImpute simulation: 3 subpopulations x 50 cells, 20,000 genes, 810 DE genes, gene means from Normal(1.8, 0.5), SDs from Normal(0.6, 0.1), with dropout imposed until ~75% of entries are zeros) and its scImpute-imputed counterpart produced by scimpute() with Kcluster = 3, drop_thre = 0.5, labeled = FALSE, ncores = 10 (output: scimpute_count.csv). Apply PCA to both matrices, retain the first two principal components, and compute the within-cluster sum-of-squares using the known cell-subpopulation labels. The rationale is that dropout zeros inflate within-cluster dispersion; imputation should shrink it substantially — the paper reports 94 (imputed) vs 2646 (raw). Optionally link the same benchmark to downstream DE analysis by computing the area under the precision-recall curve against the 810 simulated DE genes (expected >80.0% AUPRC even at 75% zeros). Write metrics such as simulation_evaluation_metrics.csv to record the comparison.

## Related tools

- **scImpute** (Produces the imputed count matrix (via scimpute(), Kcluster = 3, drop_thre = 0.5) whose cluster separation is compared against the raw matrix by PCA) — https://github.com/Vivianstats/scImpute

## Examples

```
scimpute(count_path="raw_count.csv", infile="csv", outfile="csv", out_dir="./", labeled=FALSE, drop_thre=0.5, Kcluster=3, ncores=10)
```

## Evaluation signals

- Within-cluster sum-of-squares on the first two PCs should drop markedly after imputation (paper reports 94 imputed vs 2646 raw)
- Imputed matrix must have the same gene-by-cell dimensions as the raw input matrix
- AUPRC of DE analysis on imputed data should exceed 80.0% even when raw data contain up to 75% zero counts
- DE accuracy should increase monotonically as the simulated dropout proportion decreases
- All metric values should be persisted (e.g., simulation_evaluation_metrics.csv) and reproducible from the same raw count CSV

## Limitations

- K (number of cell populations) must be supplied by the user; no definitive selection procedure is given — it 'can be chosen based on the clustering result of the raw data and the resolution level desired by the users'
- Imputation efficiency degrades when dropout rates in the raw data are severely high, as with droplet-based technologies
- In this benchmark the method was tuned to a simulated three-cell-type dataset with a known DE-gene ground truth; on real data without labels, within-cluster sum-of-squares cannot be computed

## Evidence

- [methods] PCA separation and DE precision-recall reproduce reported cluster separation: [evidence span withheld — anchor status: not_found]
- [methods] Parameter settings for scimpute run: [evidence span withheld — anchor status: not_found]
- [methods] Dropout threshold semantics from scImpute docs: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
- [readme] scimpute() usage from README: "It will output an imputed count matrix with the same dimension. In the simplest case, the imputation task can be done with one single function `scimpute`"
- [abstract] scImpute improves clustering of subpopulations: "scImpute is shown to identify likely dropouts, enhance the clustering of cell subpopulations, improve the accuracy of differential expression analysis"
