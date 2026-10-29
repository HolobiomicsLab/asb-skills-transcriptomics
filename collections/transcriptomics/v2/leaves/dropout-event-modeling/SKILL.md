---
name: dropout-event-modeling
description: Use when a raw scRNA-seq count matrix (genes × cells) contains excessive
  false zeros from dropout events and downstream tasks like dimension reduction, clustering,
  or differential expression analysis are degraded by them.
when_to_use_negative:
- When dropout rates in the raw data are severely high (e.g., droplet-based technologies),
  as the article notes improving imputation efficiency in this regime is a future
  direction
- When cell type labels are available and a supervised (labeled) imputation mode is
  required — the workflow here uses labeled = FALSE
- When a feature table or already-normalized/imputed matrix is the input, since scImpute
  is designed to be applied to the raw read count matrix before downstream analyses
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scImpute
  role: 'Performs the dropout detection and imputation: takes the raw count matrix
    plus Kcluster, drop_thre, labeled, and ncores parameters and produces the imputed
    matrix scimpute_count.csv'
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:4b858779a2fcca82348e46ac689a17448a5bd0ad7e85fc3d3f22124538279d9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - scImpute
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# dropout-event-modeling

## Summary

A skill for detecting and imputing dropout events (false zero expression) in raw scRNA-seq count matrices using scImpute, which identifies likely dropouts and imputes only those values without introducing new biases to the rest of the data.

## When to use

Use when a raw scRNA-seq count matrix (genes × cells) contains excessive false zeros from dropout events and downstream tasks like dimension reduction, clustering, or differential expression analysis are degraded by them.

## When NOT to use

- When dropout rates in the raw data are severely high (e.g., droplet-based technologies), as the article notes improving imputation efficiency in this regime is a future direction
- When cell type labels are available and a supervised (labeled) imputation mode is required — the workflow here uses labeled = FALSE
- When a feature table or already-normalized/imputed matrix is the input, since scImpute is designed to be applied to the raw read count matrix before downstream analyses

## Inputs

- Raw scRNA-seq count matrix (CSV with rows = genes, columns = cells), the only input of scimpute()

## Outputs

- Imputed count matrix of the same dimension, written as scimpute_count.csv in the output directory
- Column indices of outlier cells detected and excluded from imputation
- Evaluation metrics (e.g., PCA within-cluster sum-of-squares, DE precision-recall AUC) summarized as simulation_evaluation_metrics.csv

## How to apply

Run the exported scimpute() function on the raw count matrix (rows = genes, columns = cells), which is the only required input, and set Kcluster to the known or estimated number of cell subpopulations (e.g., Kcluster = 3 for the simulated three-cell-type dataset). Set drop_thre = 0.5 so only values with dropout probability larger than the threshold are imputed, set labeled = FALSE when cell type labels are unavailable, and pass ncores for parallel computation, which the authors highly recommend to significantly reduce computation time. scImpute returns the column indices of outlier cells and writes the imputed matrix of the same dimension to scimpute_count.csv in the output directory. Apply it to the whole-genome count matrix rather than a heavily filtered gene set, since most genes should be present to ensure robust identification of dropouts. Validate downstream by comparing PCA cluster separation (within-cluster sum-of-squares on the first two principal components, e.g., 94 imputed vs 2646 raw) and differential expression precision-recall against a known ground truth of DE genes (e.g., 810 DE genes; AUC >80.0% even at 75% zero counts).

## Related tools

- **scImpute** (Performs the dropout detection and imputation: takes the raw count matrix plus Kcluster, drop_thre, labeled, and ncores parameters and produces the imputed matrix scimpute_count.csv) — https://github.com/Vivianstats/scImpute

## Examples

```
scimpute(count_path = "raw_count.csv", infile = "csv", outfile = "csv", out_dir = "./", labeled = FALSE, drop_thre = 0.5, Kcluster = 3, ncores = 10)
```

## Evaluation signals

- Output scimpute_count.csv exists in out_dir and has the same dimension (genes × cells) as the input raw count matrix
- PCA on the imputed matrix yields markedly smaller within-cluster sum-of-squares on the first two PCs than the raw data (e.g., 94 vs 2646 in the simulated three-cell-type dataset)
- Differential expression analysis on imputed data against a simulated ground truth (e.g., 810 DE genes) achieves AUC of the precision-recall curve >80.0% even when raw data contain ~75% zero counts
- DE accuracy on scImpute-imputed data increases monotonically as the simulated dropout proportion decreases
- The function returns column indices of detected outlier cells, which should be excluded from imputation

## Limitations

- No software version for scImpute is specified in the article or README
- Computation efficiency can be improved by a prior filtering step on cells based on biological knowledge, which must be supplied by the user
- The choice of parameter K (number of cell populations) is left to the user without a definitive selection procedure; it can be chosen based on clustering of the raw data and the desired resolution
- Improving imputation efficiency when dropout rates in raw data are severely high (droplet-based technologies) is a stated future direction
- Users may need additional prior knowledge on similar genes for more complicated models accounting for gene similarities

## Evidence

- [abstract] scImpute automatically identifies likely dropouts, and only perform imputation on these values without introducing new biases to the rest data.: "scImpute automatically identifies likely dropouts, and only perform imputation on these values without introducing new biases to the rest data."
- [methods] Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`.: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
- [methods] We highly recommend using parallel computing with `scImpute`, which will significantly reduce the computation time.: "We highly recommend using parallel computing with `scImpute`, which will significantly reduce the computation time."
- [readme] This function returns the column indices of outlier cells, and creates a new file `scimpute_count.csv` in `out_dir` to store the imputed count matrix.: "This function returns the column indices of outlier cells, and creates a new file `scimpute_count.csv` in `out_dir` to store the imputed count matrix."
- [readme] we recommend applying scImpute on the whole-genome count matrix. A filtering step on genes is acceptable but most genes should be present: [evidence span withheld — anchor status: external_doc]
