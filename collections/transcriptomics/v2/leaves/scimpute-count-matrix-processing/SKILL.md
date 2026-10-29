---
name: scimpute-count-matrix-processing
description: Use when a raw scRNA-seq count matrix contains dropout zeros that must
  be imputed before dimension reduction, normalization, clustering, differential expression,
  or time-series analysis.
when_to_use_negative:
- Input is not a raw count matrix (e.g. already normalized or TPM-transformed data)
  — scImpute is designed to be applied to raw read counts before downstream analyses
- The matrix has been heavily filtered so that most genes are absent — most genes
  should be present to ensure robust identification of dropouts
- Dropout rates in the raw data are severely high (e.g. droplet-based technologies)
  — imputation efficiency is limited in this regime
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scImpute
  role: 'Core imputation method: identifies likely dropouts via dropout probability
    (drop_thre = 0.5), imputes only those values, excludes outlier cells, and writes
    scimpute_count.csv'
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:7cf66e55481325902e7ec17551d5fde52052d53323ec75d0fc8cda97560022f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - scImpute
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# scimpute-count-matrix-processing

## Summary

Apply the scImpute R package to impute dropout zero values in a raw scRNA-seq count matrix (rows = genes, columns = cells), producing an imputed count matrix of identical dimensions for downstream analyses such as clustering and differential expression.

## When to use

Use when a raw scRNA-seq count matrix contains dropout zeros that must be imputed before dimension reduction, normalization, clustering, differential expression, or time-series analysis.

## When NOT to use

- Input is not a raw count matrix (e.g. already normalized or TPM-transformed data) — scImpute is designed to be applied to raw read counts before downstream analyses
- The matrix has been heavily filtered so that most genes are absent — most genes should be present to ensure robust identification of dropouts
- Dropout rates in the raw data are severely high (e.g. droplet-based technologies) — imputation efficiency is limited in this regime

## Inputs

- Raw scRNA-seq count matrix (csv or txt) with rows representing genes and columns representing cells
- Optional ERCC spike-in transcript concentrations for validation (e.g. 57 ERCC transcripts in the GSE60361 mouse cortex dataset)
- Kcluster: number of cell subpopulations (or cell type labels if available)

## Outputs

- scimpute_count.csv — imputed count matrix with the same gene-by-cell dimensions as the raw input, written to out_dir
- Column indices of outlier cells detected and excluded from imputation

## How to apply

Run the single R function `scimpute` on the raw count matrix (genes as rows, cells as columns), setting `type`/`infile` and `outfile` to the file format (e.g. csv), `drop_thre = 0.5` so only values with dropout probability above this threshold are imputed, `Kcluster` to the known or estimated number of cell subpopulations, `labeled` to FALSE when cell type labels are unavailable, and `ncores` to enable parallel computation which significantly reduces runtime. scImpute returns the column indices of outlier cells, which it detects and excludes from imputation, and writes the imputed matrix to `scimpute_count.csv` in `out_dir`. Apply scImpute to the whole-genome count matrix; gene filtering is acceptable but most genes should be present to ensure robust identification of dropouts. Validate the output by checking that scimpute_count.csv has the same gene-by-cell dimensions as the raw input and, if ERCC spike-ins are available, that their count-vs-concentration correlations improve after imputation.

## Related tools

- **scImpute** (Core imputation method: identifies likely dropouts via dropout probability (drop_thre = 0.5), imputes only those values, excludes outlier cells, and writes scimpute_count.csv) — https://github.com/Vivianstats/scImpute

## Examples

```
scimpute(count_path = system.file("extdata", "raw_count.csv", package = "scImpute"), infile = "csv", outfile = "csv", out_dir = "./", labeled = FALSE, drop_thre = 0.5, Kcluster = 2, ncores = 10)
```

## Evaluation signals

- scimpute_count.csv exists in out_dir and has the same gene-by-cell dimensions as the raw input count matrix
- On the 3005-cell mouse cortex data with 57 ERCC transcripts, median ERCC count-vs-concentration correlation rises from 0.92 to 0.95 and minimum from 0.81 to 0.89 after imputation
- Only values with dropout probability larger than drop_thre (0.5) are changed; expression values unaffected by dropouts are left unaltered
- The function returns column indices of outlier cells that were detected and excluded from imputation
- Parallel computation via ncores is active and runtime scales reasonably with matrix size

## Limitations

- Imputation efficiency degrades when dropout rates in the raw data are severely high, such as in droplet microfluidics data, and improving this is a stated future direction
- The number of cell populations K is left to the user without a definitive selection procedure; it can be chosen based on clustering of the raw data and the desired resolution
- When the number of cells is extremely large, a prior filtering step on cells based on biological knowledge is needed to save computation time but is not specified

## Evidence

- [abstract] scimpute: "scImpute automatically identifies likely dropouts, and only perform imputation on these values without introducing new biases to the rest data."
- [methods] drop_thre: "Only the values that have **dropout probability** larger than `drop_thre` are imputed by `scImpute`."
- [methods] parallel: "We highly recommend using parallel computing with `scImpute`, which will significantly reduce the computation time."
- [readme] whole-genome: [evidence span withheld — anchor status: external_doc]
- [readme] output file: "creates a new file `scimpute_count.csv` in `out_dir` to store the imputed count matrix"
