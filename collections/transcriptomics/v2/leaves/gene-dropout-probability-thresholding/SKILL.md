---
name: gene-dropout-probability-thresholding
description: Use when a raw gene-by-cell count matrix (csv) from scRNA-seq contains
  dropout zeros — genes observed at moderate expression in some cells but undetected
  in others — and only high-confidence dropouts should be imputed without altering
  unaffected values.
when_to_use_negative:
- The goal is to impute or smooth all expression values, not just dropouts — MAGIC
  and SAVER alter all gene expression levels and may suit that intent instead
- The count matrix has been heavily gene-filtered — scImpute recommends applying it
  to the whole-genome count matrix, since most genes should be present to ensure robust
  identification of dropouts
- Dropout rates in the raw data are severely high (e.g. droplet-based technologies),
  where the authors note imputation efficiency still needs improvement
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scImpute
  role: Computes per-entry dropout probabilities, thresholds them at drop_thre, learns
    cell-similarity NNLS coefficients on reliable genes (B_j), and imputes only candidate
    dropout entries (A_j) in the raw count matrix
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:b4b1ed75712a0aaa6fc8ac8a315e1cf1c3053b2e7600498db28cb38b1298a527
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - scImpute
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# gene-dropout-probability-thresholding

## Summary

Identify likely dropout zeros in a scRNA-seq raw count matrix by computing each entry's dropout probability and imputing only values above a probability threshold (default drop_thre = 0.5) via scImpute's NNLS cell-similarity regression, leaving reliable values untouched.

## When to use

Use when a raw gene-by-cell count matrix (csv) from scRNA-seq contains dropout zeros — genes observed at moderate expression in some cells but undetected in others — and only high-confidence dropouts should be imputed without altering unaffected values.

## When NOT to use

- The goal is to impute or smooth all expression values, not just dropouts — MAGIC and SAVER alter all gene expression levels and may suit that intent instead
- The count matrix has been heavily gene-filtered — scImpute recommends applying it to the whole-genome count matrix, since most genes should be present to ensure robust identification of dropouts
- Dropout rates in the raw data are severely high (e.g. droplet-based technologies), where the authors note imputation efficiency still needs improvement

## Inputs

- Raw scRNA-seq count matrix in csv format (rows = genes, columns = cells), e.g. raw_count.csv from scImpute's extdata
- Optional cell-type labels (if labeled=TRUE); default run assumes labels are not available

## Outputs

- Imputed count matrix written to scimpute_count.csv in out_dir, with the same gene-by-cell dimensions as the input
- Column indices of detected outlier cells (returned by scimpute())

## How to apply

Load the raw count matrix (rows = genes, columns = cells) and run scImpute's scimpute() with infile='csv', outfile='csv', labeled=FALSE, drop_thre=0.5, Kcluster=2, and ncores=10 for parallel computation. For each cell j, scImpute partitions the gene set into A_j (entries with dropout probability > drop_thre, the highly probable dropouts) and B_j (values not severely affected by dropouts); it learns cell-similarity coefficients by non-negative least squares on the B_j genes and imputes only the A_j entries with NNLS-weighted expression borrowed from similar cells. Outlier cells are detected and excluded from imputation, and their indices are returned. The rationale is that imputing every value, as MAGIC and SAVER do, can introduce new biases into values unaffected by dropouts, whereas thresholding on dropout probability restricts modification to likely false zeros.

## Related tools

- **scImpute** (Computes per-entry dropout probabilities, thresholds them at drop_thre, learns cell-similarity NNLS coefficients on reliable genes (B_j), and imputes only candidate dropout entries (A_j) in the raw count matrix) — https://github.com/Vivianstats/scImpute

## Examples

```
scimpute(count_path = system.file("extdata", "raw_count.csv", package = "scImpute"), infile = "csv", outfile = "csv", out_dir = "./", labeled = FALSE, drop_thre = 0.5, Kcluster = 2, ncores = 10)
```

## Evaluation signals

- scImpute_count.csv exists in the specified out_dir
- Output matrix has the same gene-by-cell dimensions as the input raw count matrix
- Only entries whose dropout probability exceeds drop_thre (0.5) were modified; reliable (B_j) values are unchanged relative to the input
- scimpute() returns a vector of outlier-cell column indices, and those cells are excluded from imputation
- Downstream clustering of cell subpopulations improves relative to clustering on the raw data

## Limitations

- No software version for scImpute is specified in the article
- Choice of K (number of cell populations) is left to the user without a definitive selection procedure; it can be chosen from clustering of the raw data and desired resolution
- For extremely large numbers of cells, a filtering step on cells based on biological knowledge is recommended to save computation time, but such prior knowledge is not specified
- Imputation efficiency degrades when dropout rates in raw data are severely high (droplet-based technologies); noted as a future direction

## Evidence

- [methods] Only the values that have dropout probability larger than drop_thre are imputed by scImpute.: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
- [abstract] scImpute automatically identifies likely dropouts and only imputes those values.: "scImpute automatically identifies likely dropouts, and only perform imputation on these values without introducing new biases to the rest data."
- [readme] README quick-start scimpute() call with csv input/output, labeled=FALSE, drop_thre=0.5, Kcluster=2, ncores=10.: "infile = "csv",           # format of input file
         outfile = "csv",          # format of output file"
- [readme] README: scimpute returns outlier cell indices and writes scimpute_count.csv.: "This function returns the column indices of outlier cells, and creates a new file `scimpute_count.csv` in `out_dir` to store the imputed count matrix."
