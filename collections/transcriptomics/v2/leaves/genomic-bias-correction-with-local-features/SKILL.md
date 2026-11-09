---
name: genomic-bias-correction-with-local-features
description: Use when raw N*N single-cell Hi-C contact matrices are sparse and biased
  by local genomic features (effective length, GC content, fragment-end mappability)
  and must be normalized before DCI testing with scHiCDiff.KS/CVM/NB/ZINB.
when_to_use_negative:
- The input count table is already a matrix of normalized scHi-C read counts (as in
  the sample data sim 1.rds) ready for detection functions — re-normalizing would
  double-correct the bias.
- The data is bulk Hi-C interaction counts rather than single-cell contact matrices,
  since the imputation and cell-specific distance normalization target single-cell
  sparsity.
- You lack the bias information file (effective length, GC content, fragment-end mappability)
  for the bins and cannot generate it, as scHiCDiff.norm requires it as a mandatory
  input.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_0622
tools:
- name: scHiCDiff
  role: Provides the scHiCDiff.norm normalization function (Gaussian convolution imputation,
    scHiCNorm NB-hurdle bias removal, distance normalization) that implements this
    skill
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: Environment in which scHiCDiff is implemented and called (install.packages
    of the source tarball, then scHiCDiff.norm)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:ecb0e709526d6fec70134abc6e000c5cce4d9a1d18a38c814d775648c8f06f02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# genomic-bias-correction-with-local-features

## Summary

Normalizes sparse single-cell Hi-C contact matrices with scHiCDiff by first imputing contacts via a Gaussian convolution filter, then removing systematic biases (effective length, GC content, fragment-end mappability) using the scHiCNorm Negative Binomial Hurdle procedure plus cell-specific genomic-distance normalization, so detection functions receive bias-corrected counts.

## When to use

Use when raw N*N single-cell Hi-C contact matrices are sparse and biased by local genomic features (effective length, GC content, fragment-end mappability) and must be normalized before DCI testing with scHiCDiff.KS/CVM/NB/ZINB.

## When NOT to use

- The input count table is already a matrix of normalized scHi-C read counts (as in the sample data sim 1.rds) ready for detection functions — re-normalizing would double-correct the bias.
- The data is bulk Hi-C interaction counts rather than single-cell contact matrices, since the imputation and cell-specific distance normalization target single-cell sparsity.
- You lack the bias information file (effective length, GC content, fragment-end mappability) for the bins and cannot generate it, as scHiCDiff.norm requires it as a mandatory input.

## Inputs

- dat_HiC: an N*N single-cell Hi-C contact matrix (raw, sparse)
- bias.info.path: file path of the three local features (effective length, GC content, mappability of fragment ends) of all bins

## Outputs

- normalized_hic_matrix.rds: the normalized Hi-C matrix, square N*N with the same dimensions as the input and no missing values
- A non-negative matrix of scHi-C normalized read counts (bin pairs as rows, samples/cells as columns) suitable for the scHiCDiff detection functions

## How to apply

Load the raw N*N scHi-C contact matrix (dat_HiC) and a bias file (bias.info.path) holding the three local features—effective length, GC content, and fragment-end mappability—for all bins; these features can be generated from http://dna.cs.miami.edu/scHiCDiff. Apply the fixed preprocessing of the normalization stage, which imputes the sparse contact matrix with a Gaussian convolution filter to tackle sparsity. Then call scHiCDiff.norm(dat_HiC, bias.info.path), which applies the scHiCNorm Negative Binomial Hurdle bias-removal procedure (Liu and Wang 2018) and cell-specific genomic-distance normalization, returning the normalized Hi-C matrix. Rationale: scHiCDiff detection functions expect a non-negative matrix of scHi-C normalized read counts with bin-pair rows and cell columns, so bias removal must precede differential testing; the zero-inflated structure of single-cell data is handled downstream (e.g., by the ZINB/NBH options) only after normalization.

## Related tools

- **scHiCDiff** (Provides the scHiCDiff.norm normalization function (Gaussian convolution imputation, scHiCNorm NB-hurdle bias removal, distance normalization) that implements this skill) — https://github.com/wmalab/scHiCDiff
- **R** (Environment in which scHiCDiff is implemented and called (install.packages of the source tarball, then scHiCDiff.norm))

## Examples

```
bias.info.path <- "path/bias.info"; normalized <- scHiCDiff.norm(dat_HiC, bias.info.path); saveRDS(normalized, "normalized_hic_matrix.rds")
```

## Evaluation signals

- The returned object is a square N*N matrix with the same dimensions as the input contact matrix.
- All entries of the normalized matrix are defined (no missing values), consistent with the function contract that it 'returns the normalized Hi-C matrix'.
- normalized_hic_matrix.rds exists on disk and can be loaded back with readRDS as a matrix.
- Rows/columns of downstream detection input correspond to bin pairs and cells, and counts are non-negative, matching the documented detection-function input contract.
- The bias.info.path file actually contains the three local features (effective length, GC content, fragment-end mappability) for all bins before the call.

## Limitations

- No version numbers for scHiCDiff or R are stated in the article or README; only the developmental version scHiCDiff_1.0.tar.gz install path is documented.
- The bias feature generation is hosted externally at http://dna.cs.miami.edu/scHiCDiff, so availability of those precomputed feature items is a dependency.
- The README lists the normalization function as scHiCDiff.sim in one place and scHiCDiff.norm in the input documentation — the exact function naming is ambiguous in the source text.
- Only the MIT license is stated in the README, while the resolved repository license is NOASSERTION, so reuse terms are not fully clear.

## Evidence

- [abstract] scHiCDiff is implemented in R and freely available at GitHub: "scHiCDiff is implemented in R and freely available at GitHub (https://github.com/wmalab/scHiCDiff)"
- [intro] The first type is the normalization function (scHiCDiff.sim): "The first type is the normalization function (scHiCDiff.sim) and the other type is the detection function (scHiCDiff.KS, scHiCDiff.CVM, scHiCDiff.NB and scHiCDiff.ZINB)"
- [methods] The inputs of the normalization function scHiCDiff.norm are illustrated below: "The inputs of the normalization function scHiCDiff.norm are illustrated below"
- [readme] bias.info.path ... three local features (effective length,GC content and mappability of fragment ends) of all bins: "The pathway of the three local features (effective length,GC content 
                    and mappability of fragment ends) of all bins."
- [methods] The function returns the normalized Hi-C matrix.: "The function returns the normalized Hi-C matrix."
- [readme] count.table ... A non-negative matrix of scHi-C normalized read counts.: "count.table      A non-negative  matrix of scHi-C normalized read counts.The rows of the 
                 matrix are bin pair and columns are samples/cells."
