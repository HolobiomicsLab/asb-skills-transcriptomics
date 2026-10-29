---
name: r-function-workflow-execution
description: Use when a raw N*N single-cell Hi-C contact matrix (dat_HiC) must be
  bias-corrected and distance-normalized before feeding scHiCDiff.KS/CVM/NB/ZINB detection
  functions with a count.table.
when_to_use_negative:
- The input is already a normalized count.table (bin pairs x cells) — go directly
  to the detection functions scHiCDiff.KS/CVM/NB/ZINB instead of re-normalizing
- The data are bulk Hi-C contact maps rather than single-cell Hi-C matrices; scHiCDiff
  is designed for interaction comparison at single-cell level with excessive zeros
- The goal is DCI detection itself — this skill covers only the normalization stage,
  not the parametric (NB/ZINB/NBH) or non-parametric (KS/CVM) tests
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: scHiCDiff
  role: R package whose normalization function scHiCDiff.norm(dat_HiC, bias.info.path)
    performs Gaussian-convolution imputation, scHiCNorm NB-hurdle bias removal and
    genomic-distance normalization
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Runtime environment in which the scHiCDiff package is source-installed and
    the normalization workflow (load matrices, call scHiCDiff.norm, saveRDS) is executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:5958601b792a79fb59881df6ee463e43d2054d4ed77a7628af5d88681e1fe4f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# r-function-workflow-execution

## Summary

Execute the scHiCDiff normalization stage in R: load an N*N single-cell Hi-C contact matrix plus a bias-info file, impute sparsity with a Gaussian convolution filter, and call scHiCDiff.norm to remove systematic biases (scHiCNorm NB-hurdle) and apply cell-specific genomic-distance normalization.

## When to use

Use when a raw N*N single-cell Hi-C contact matrix (dat_HiC) must be bias-corrected and distance-normalized before feeding scHiCDiff.KS/CVM/NB/ZINB detection functions with a count.table.

## When NOT to use

- The input is already a normalized count.table (bin pairs x cells) — go directly to the detection functions scHiCDiff.KS/CVM/NB/ZINB instead of re-normalizing
- The data are bulk Hi-C contact maps rather than single-cell Hi-C matrices; scHiCDiff is designed for interaction comparison at single-cell level with excessive zeros
- The goal is DCI detection itself — this skill covers only the normalization stage, not the parametric (NB/ZINB/NBH) or non-parametric (KS/CVM) tests

## Inputs

- dat_HiC: a N*N single-cell Hi-C contact matrix
- bias.info.path: pathway of a file with the three local features of all bins (effective length, GC content, mappability of fragment ends)
- R environment with the developmental scHiCDiff package installed from source (scHiCDiff_1.0.tar.gz)

## Outputs

- Normalized Hi-C matrix (square, N*N, same dimensions as dat_HiC, no missing values)
- Saved RDS file, e.g. normalized_hic_matrix.rds, reusable as the normalized count.table input for scHiCDiff.KS, scHiCDiff.CVM, scHiCDiff.NB and scHiCDiff.ZINB

## How to apply

In R, install and load the developmental scHiCDiff package (source install of scHiCDiff_1.0.tar.gz), then load the raw N*N single-cell Hi-C contact matrix (dat_HiC) and the bias.info.path file holding the three local features of all bins: effective length, GC content, and fragment-end mappability. First apply the fixed preprocessing of the normalization stage — imputation of the sparse contact matrix with a Gaussian convolution filter to tackle zero inflation. Then call scHiCDiff.norm(dat_HiC, bias.info.path), which removes systematic biases via the scHiCNorm Negative Binomial Hurdle procedure and applies cell-specific genomic-distance normalization; the function contract states it 'returns the normalized Hi-C matrix'. Save the returned matrix (e.g. as normalized_hic_matrix.rds); downstream, rows are bin pairs and columns are cells, matching the count.table input required by the detection functions. The bias-info items themselves must be generated beforehand (generation is available at http://dna.cs.miami.edu/scHiCNorm).

## Related tools

- **scHiCDiff** (R package whose normalization function scHiCDiff.norm(dat_HiC, bias.info.path) performs Gaussian-convolution imputation, scHiCNorm NB-hurdle bias removal and genomic-distance normalization) — https://github.com/wmalab/scHiCDiff
- **R** (Runtime environment in which the scHiCDiff package is source-installed and the normalization workflow (load matrices, call scHiCDiff.norm, saveRDS) is executed)

## Examples

```
library(scHiCDiff); normalized <- scHiCDiff.norm(dat_HiC, bias.info.path); saveRDS(normalized, "normalized_hic_matrix.rds")
```

## Evaluation signals

- The output file normalized_hic_matrix.rds exists and loads as a square matrix of the same N*N dimensions as the input contact matrix
- All entries of the returned normalized Hi-C matrix are defined (no missing/NA values), per the function contract that it 'returns the normalized Hi-C matrix'
- The normalized matrix is non-negative and shaped as bin pairs x cells, matching the count.table schema required by the detection functions
- The bias.info file contains exactly the three local features (effective length, GC content, fragment-end mappability) for all bins before the call
- Downstream detection functions accept the output without schema errors and return a data frame of DCI results with bin_1, bin_2, pvalue and pvalue.adj.FDR columns

## Limitations

- The normalization requires an externally generated bias.info file (effective length, GC content, mappability of fragment ends); its generation is only available at http://dna.cs.miami.edu/scHiCNorm
- No version numbers are stated for scHiCDiff or R in the article, and no changelog is available, so reproducibility depends on the developmental source install (scHiCDiff_1.0.tar.gz)
- The function contract is terse ('The function returns the normalized Hi-C matrix.') with no documented tuning parameters for the Gaussian convolution imputation or NB-hurdle bias removal

## Evidence

- [methods] The inputs of the normalization function scHiCDiff.norm are illustrated below: "The inputs of the normalization function scHiCDiff.norm are illustrated below"
- [readme] bias.info.path: pathway of the three local features of all bins: "bias.info.path      The pathway of the three local features (effective length,GC content "
- [readme] dat_HiC is an N*N scHi-C matrix: "dat_HiC             A N*N scHi-C matrix."
- [readme] Install the developmental version from source in R: "install.packages("path/scHiCDiff_1.0.tar.gz", repos = NULL, type ="source")"
- [abstract] scHiCDiff is implemented in R and available on GitHub: "scHiCDiff is implemented in R and freely available at GitHub (https://github.com/wmalab/scHiCDiff)"
