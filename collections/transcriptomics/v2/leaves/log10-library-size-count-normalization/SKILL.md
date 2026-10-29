---
name: log10-library-size-count-normalization
description: Use when a raw gene-by-cell count matrix (e.g. raw_count.csv) must be
  normalized before fitting scImpute's mixture model or comparing expression across
  cells with differing library sizes.
when_to_use_negative:
- Input is already normalized or log-transformed expression data rather than a raw
  count matrix
- The counts have not been generated genome-wide — a heavy prior gene filtering removes
  most genes, undermining robust dropout identification
edam_operation: http://edamontology.org/operation_3231
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scImpute
  role: Performs the library-size normalization and log10 transformation internally
    as a prerequisite to fitting the Gamma-Normal mixture model on the normalized
    matrix
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-03405-7
    title: An accurate and robust imputation method scImpute for single-cell RNA-seq
      data
  source_content_hash: sha256:50a029e0b37bed5a5a27d3172e4d67e3bd0530e62792ac000d277598e4e9ba58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scImpute
  tools:
  - scImpute
  edam_operation: http://edamontology.org/operation_3231
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# log10-library-size-count-normalization

## Summary

Normalizes raw scRNA-seq counts by each cell's library size and applies a log10 transformation, producing the expression matrix on which scImpute's Gamma-Normal mixture model is fit to estimate dropout probabilities.

## When to use

Use when a raw gene-by-cell count matrix (e.g. raw_count.csv) must be normalized before fitting scImpute's mixture model or comparing expression across cells with differing library sizes.

## When NOT to use

- Input is already normalized or log-transformed expression data rather than a raw count matrix
- The counts have not been generated genome-wide — a heavy prior gene filtering removes most genes, undermining robust dropout identification

## Inputs

- raw scRNA-seq count matrix (rows = genes, columns = cells), e.g. raw_count.csv

## Outputs

- log10-transformed library-size-normalized expression matrix (same gene-by-cell dimensions) used for mixture model fitting

## How to apply

Take the raw count matrix (rows = genes, columns = cells) and divide each gene's count by the total count (library size) of the same cell to remove cell-specific sequencing-depth effects. Apply a log10 transformation to the library-size-normalized values to obtain the normalized expression matrix used for model fitting. In the scImpute pipeline this normalization is a prerequisite step performed inside the scimpute function after loading the raw counts via count_path (infile = 'csv'), before partitioning cells into Kcluster subpopulations and fitting the per-gene Gamma (dropout) and Normal (true expression) mixture components by EM. The output is a log10-normalized expression matrix of the same gene-by-cell dimensions that feeds downstream posterior dropout probability estimation and imputation of entries with dropout probability above drop_thre = 0.5.

## Related tools

- **scImpute** (Performs the library-size normalization and log10 transformation internally as a prerequisite to fitting the Gamma-Normal mixture model on the normalized matrix) — https://github.com/Vivianstats/scImpute

## Examples

```
scimpute(count_path = system.file("extdata", "raw_count.csv", package = "scImpute"), infile = "csv", outfile = "csv", out_dir = "./", labeled = FALSE, drop_thre = 0.5, Kcluster = 2, ncores = 10)
```

## Evaluation signals

- Normalized matrix preserves the input dimensions: one row per gene and one column per cell
- Each cell's values are scaled so they reflect library-size-corrected, log10-transformed counts rather than raw counts
- No negative values are produced (log10 applied to non-negative normalized counts)
- Downstream per-gene mixture fitting runs on the normalized matrix and yields a dropout probability matrix of matching gene-by-cell dimensions

## Limitations

- Normalization must be applied to a raw whole-genome count matrix; gene filtering is acceptable but most genes should be present to ensure robust identification of dropouts
- Library-size normalization alone does not address dropout rates that are severely high (e.g. droplet-based protocols), which degrades downstream imputation efficiency

## Evidence

- [methods] workflow quote on normalization: [evidence span withheld — anchor status: not_found]
- [readme] readme quote on input expectations: [evidence span withheld — anchor status: external_doc]
- [readme] readme quote on downstream steps: "It will output an imputed count matrix with the same dimension."
- [methods] article quote on normalization usage: "- normalization of scRNA-seq data"
