---
name: count-matrix-file-io
description: Use when a raw scRNA-seq count matrix (genes as rows, cells as columns,
  e.g. a csv like raw_count.csv) must be read into scImpute and an imputed count matrix
  of the same dimension must be produced for downstream analyses.
when_to_use_negative:
- The input is not a raw count matrix (e.g. normalized or log-transformed data) —
  scImpute expects the raw count matrix before downstream analyses such as normalization
  or dimension reduction
- A gene-filtered matrix has been heavily reduced — most genes should be present to
  ensure robust identification of dropouts
- Only a subset of entries should be modified deliberately — scImpute itself decides
  which values (dropout probability larger than drop_thre) are imputed
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scImpute
  role: 'R package whose scimpute() function performs the file I/O: reads the raw
    count csv and writes the imputed scimpute_count.csv'
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:1522cc2839356c42f5fc43c0d13c83e5ba265bf3cb4df581c583b22a40418cb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - MAGIC
  - SAVER
  - scImpute
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# count-matrix-file-io

## Summary

Handles the file input/output conventions of the scImpute imputation workflow: reading a raw scRNA-seq gene-by-cell count matrix (csv format) and writing the imputed matrix scimpute_count.csv with identical dimensions. This makes scImpute's drop-out imputation a drop-in replacement step in a scRNA-seq pipeline.

## When to use

Use when a raw scRNA-seq count matrix (genes as rows, cells as columns, e.g. a csv like raw_count.csv) must be read into scImpute and an imputed count matrix of the same dimension must be produced for downstream analyses.

## When NOT to use

- The input is not a raw count matrix (e.g. normalized or log-transformed data) — scImpute expects the raw count matrix before downstream analyses such as normalization or dimension reduction
- A gene-filtered matrix has been heavily reduced — most genes should be present to ensure robust identification of dropouts
- Only a subset of entries should be modified deliberately — scImpute itself decides which values (dropout probability larger than drop_thre) are imputed

## Inputs

- Raw scRNA-seq count matrix (rows = genes, columns = cells), e.g. raw_count.csv packaged with scImpute
- Input file format string (infile, e.g. 'csv') and output file format string (outfile, e.g. 'csv')
- Output directory path (out_dir)

## Outputs

- Imputed count matrix written to scimpute_count.csv in out_dir, with the same gene-by-cell dimensions as the input
- Column indices of outlier cells returned by scimpute()

## How to apply

Supply scImpute's only required input: a raw count matrix with rows representing genes and columns representing cells, given as a file path with its format declared via infile (e.g. 'csv'). Set outfile to the desired output format ('csv') and out_dir to the output directory; the function writes a new file scimpute_count.csv there storing the imputed count matrix. Apply scImpute to the whole-genome count matrix — a filtering step on genes is acceptable but most genes should be present to ensure robust identification of dropouts; the output must retain the same gene-by-cell dimensions as the input. The scimpute() call also returns the column indices of outlier cells, which scImpute detected and excluded from imputation. Optionally, when the number of cells is extremely large, perform a filtering step on cells first to save computation time.

## Related tools

- **scImpute** (R package whose scimpute() function performs the file I/O: reads the raw count csv and writes the imputed scimpute_count.csv) — https://github.com/Vivianstats/scImpute

## Examples

```
scimpute(count_path = system.file("extdata", "raw_count.csv", package = "scImpute"), infile = "csv", outfile = "csv", out_dir = "./", labeled = FALSE, drop_thre = 0.5, Kcluster = 2, ncores = 10)
```

## Evaluation signals

- scimpute_count.csv exists in out_dir after the call
- Output matrix has the same gene-by-cell dimensions as the input raw count matrix
- Only entries whose dropout probability exceeds drop_thre were modified; all other values are unchanged
- Returned outlier-cell indices correspond to cells excluded from imputation
- Input was the whole-genome (or nearly whole-genome) count matrix, not a heavily gene-filtered matrix

## Limitations

- Applying scImpute to a heavily gene-filtered matrix is discouraged; a filtering step on genes is acceptable but most genes should be present
- For extremely large cell numbers, a cell filtering step is needed to save computation time; efficiency improves further with parallel computing via ncores
- No scImpute software version number is stated in the paper or README

## Evidence

- [readme] Its only input is the raw count matrix with rows representing genes and columns representing cells.: "Its only input is the raw count matrix with rows representing genes and columns representing cells. It will output an imputed count matrix with the same dimension."
- [readme] scimpute_count.csv in out_dir: "This function returns the column indices of outlier cells, and creates a new file `scimpute_count.csv` in `out_dir` to store the imputed count matrix."
- [readme] we recommend applying scImpute on the whole-genome count matrix: [evidence span withheld — anchor status: external_doc]
- [methods] Only the values that have dropout probability larger than drop_thre are imputed by scImpute.: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
- [methods] When the number of cells is extremely large, a filtering step on the cells can save the computation time: "When the number of cells is extremely large, a filtering step on the cells can save the computation time of `scImpute`."
