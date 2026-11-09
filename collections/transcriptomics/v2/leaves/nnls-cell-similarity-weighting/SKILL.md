---
name: nnls-cell-similarity-weighting
description: Use when a raw gene-by-cell count matrix contains dropout zeros (gene
  moderate in some cells, undetected in others) and only the likely dropouts — not
  the rest of the data — should be imputed before clustering or differential expression.
when_to_use_negative:
- Input is not a raw whole-genome count matrix — the README recommends applying scImpute
  on the whole-genome count matrix, and heavy gene filtering undermines robust dropout
  identification
- All expression values should be smoothed/adjusted, not just dropouts — MAGIC and
  SAVER alter all gene expression levels, which is precisely what this selective approach
  avoids and is the wrong model if that behavior is desired
- Cell type labels are available and a labeled workflow is intended — set labeled=TRUE
  rather than using the unlabeled Kcluster-based path shown here
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scImpute
  role: R package providing the scimpute() function that computes dropout probabilities,
    fits NNLS cell-similarity coefficients on B_j genes, and imputes only A_j dropout
    entries
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:946456655d1fae49742426f78438a4b150bd875b2c5172eeaacf1c0f39a6f970
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - scImpute
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# nnls-cell-similarity-weighting

## Summary

scImpute's selective imputation step: identify probable dropout zeros in a raw scRNA-seq count matrix and impute only those entries by non-negative least squares (NNLS) weighted expression from similar cells, leaving all non-dropout values untouched.

## When to use

Use when a raw gene-by-cell count matrix contains dropout zeros (gene moderate in some cells, undetected in others) and only the likely dropouts — not the rest of the data — should be imputed before clustering or differential expression.

## When NOT to use

- Input is not a raw whole-genome count matrix — the README recommends applying scImpute on the whole-genome count matrix, and heavy gene filtering undermines robust dropout identification
- All expression values should be smoothed/adjusted, not just dropouts — MAGIC and SAVER alter all gene expression levels, which is precisely what this selective approach avoids and is the wrong model if that behavior is desired
- Cell type labels are available and a labeled workflow is intended — set labeled=TRUE rather than using the unlabeled Kcluster-based path shown here

## Inputs

- Raw scRNA-seq count matrix as CSV (rows = genes, columns = cells), e.g. raw_count.csv from the package extdata
- Parameter values: drop_thre (dropout probability threshold, default 0.5), Kcluster (number of cell subpopulations, e.g. 2), labeled (FALSE when cell labels unavailable), ncores

## Outputs

- Imputed count matrix written to scimpute_count.csv with the same gene-by-cell dimensions as the input
- Returned column indices of detected outlier cells excluded from imputation

## How to apply

Load the raw count matrix (rows = genes, columns = cells, e.g. the packaged raw_count.csv) and run scImpute::scimpute() with infile='csv', outfile='csv', labeled=FALSE, drop_thre=0.5, Kcluster=2, and ncores=10 for parallel computation. For each cell j, scImpute splits the gene set into A_j, entries with dropout probability above drop_thre (candidate dropouts), and B_j, genes not severely affected by dropouts. It learns cell-similarity coefficients by non-negative least squares fitted on the B_j genes, then imputes only A_j entries using NNLS-weighted expression of the same genes from the similar cells. The function returns the column indices of outlier cells (which are excluded from imputation) and writes the imputed matrix to scimpute_count.csv in out_dir.

## Related tools

- **scImpute** (R package providing the scimpute() function that computes dropout probabilities, fits NNLS cell-similarity coefficients on B_j genes, and imputes only A_j dropout entries) — https://github.com/Vivianstats/scImpute

## Examples

```
scimpute(count_path = system.file("extdata", "raw_count.csv", package = "scImpute"), infile = "csv", outfile = "csv", out_dir = "./", labeled = FALSE, drop_thre = 0.5, Kcluster = 2, ncores = 10)
```

## Evaluation signals

- scimpute_count.csv exists in the specified out_dir after the run
- Output matrix has identical gene-by-cell dimensions to the input raw count matrix
- Only entries whose dropout probability exceeded drop_thre were modified; values below the threshold are unchanged
- All imputed values are non-negative, consistent with the NNLS weighting
- The function returns a non-empty set of outlier cell indices, and those cells' columns reflect exclusion from imputation

## Limitations

- Imputation efficiency degrades when dropout rates in the raw data are severely high (e.g. droplet-based technologies); improving this is a stated future direction
- Choice of K (number of cell populations) is left to the user without a definitive selection procedure; it can be chosen from clustering of the raw data and desired resolution
- For extremely large numbers of cells, computation time is significant unless a prior cell-filtering step is performed based on biological knowledge (which the article does not specify)
- No software version number for scImpute is stated in the article

## Evidence

- [abstract] scImpute automatically identifies likely dropouts, and only perform imputation on these values without introducing new biases to the rest data.: "scImpute automatically identifies likely dropouts, and only perform imputation on these values without introducing new biases to the rest data."
- [methods] Only the values that have dropout probability larger than drop_thre are imputed: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
- [readme] This function returns the column indices of outlier cells, and creates a new file scimpute_count.csv in out_dir: "This function returns the column indices of outlier cells, and creates a new file `scimpute_count.csv` in `out_dir` to store the imputed count matrix."
- [readme] we recommend applying scImpute on the whole-genome count matrix: [evidence span withheld — anchor status: external_doc]
