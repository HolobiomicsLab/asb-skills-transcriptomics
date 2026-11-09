---
name: gaussian-convolution-imputation-for-sparse-contact-maps
description: Use when raw single-cell Hi-C contact matrices (N*N dat_HiC) are too
  sparse for differential testing and a bias-aware normalized count matrix is needed
  before scHiCDiff.KS/CVM/NB/ZINB detection.
when_to_use_negative:
- The count table already contains normalized read counts — the README's detection
  functions (scHiCDiff.KS/CVM/NB/ZINB) expect pre-normalized input, so re-normalizing
  is unnecessary.
- No bias feature file (effective length, GC content, fragment-end mappability) is
  available and it cannot be generated via http://dna.cs.miami.edu/scHiCNorm.
- The data are bulk Hi-C rather than single-cell Hi-C, since the imputation and ZINB-style
  zero handling target single-cell-level sparsity.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: scHiCDiff
  role: R package whose normalization function scHiCDiff.norm performs Gaussian convolution
    imputation, NBH bias removal, and distance normalization of the contact matrix
  repo: https://github.com/wmalab/scHiCDiff
- name: R
  role: implementation environment in which dat_HiC is loaded, scHiCDiff.norm is called,
    and the normalized matrix is saved as .rds
  repo: https://github.com/wmalab/scHiCDiff
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad625
    title: 'scHiCDiff: detecting differential chromatin interactions in single-cell
      Hi-C data'
  source_content_hash: sha256:26704d608b2735c266bfc40c5142f1b8fb06ce41780fbb9755a883a70d1cef4f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wmalab/scHiCDiff
  tools:
  - R
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# gaussian-convolution-imputation-for-sparse-contact-maps

## Summary

Preprocessing stage of the scHiCDiff pipeline in which a sparse N*N single-cell Hi-C contact matrix is imputed with a Gaussian convolution filter and then normalized by scHiCDiff.norm (scHiCNorm Negative Binomial Hurdle) using local bias features and cell-specific genomic-distance normalization.

## When to use

Use when raw single-cell Hi-C contact matrices (N*N dat_HiC) are too sparse for differential testing and a bias-aware normalized count matrix is needed before scHiCDiff.KS/CVM/NB/ZINB detection.

## When NOT to use

- The count table already contains normalized read counts — the README's detection functions (scHiCDiff.KS/CVM/NB/ZINB) expect pre-normalized input, so re-normalizing is unnecessary.
- No bias feature file (effective length, GC content, fragment-end mappability) is available and it cannot be generated via http://dna.cs.miami.edu/scHiCNorm.
- The data are bulk Hi-C rather than single-cell Hi-C, since the imputation and ZINB-style zero handling target single-cell-level sparsity.

## Inputs

- N*N single-cell Hi-C contact matrix (dat_HiC) in R
- bias.info.path file with effective length, GC content, and fragment-end mappability for all bins

## Outputs

- Normalized Hi-C matrix (same N*N dimensions as input), savable as normalized_hic_matrix.rds
- Downstream count.table: non-negative matrix of scHi-C normalized read counts (rows = bin pairs, columns = samples/cells)

## How to apply

Load the raw N*N scHi-C contact matrix (dat_HiC) into R and the bias.info.path file containing the three local features of all bins — effective length, GC content, and mappability of fragment ends. Apply the fixed preprocessing of the normalization stage, which imputes the sparse contact matrix with a Gaussian convolution filter to tackle zero inflation from single-cell sparsity. Then call scHiCDiff.norm(dat_HiC, bias.info.path), which removes systematic biases via the scHiCNorm Negative Binomial Hurdle procedure and applies cell-specific genomic-distance normalization. Save the returned normalized Hi-C matrix (e.g. normalized_hic_matrix.rds) so that it can be formatted as the count.table input (rows = bin pairs, columns = cells) for the downstream detection functions. The bias feature file itself can be generated following http://dna.cs.miami.edu/scHiCNorm.

## Related tools

- **scHiCDiff** (R package whose normalization function scHiCDiff.norm performs Gaussian convolution imputation, NBH bias removal, and distance normalization of the contact matrix) — https://github.com/wmalab/scHiCDiff
- **R** (implementation environment in which dat_HiC is loaded, scHiCDiff.norm is called, and the normalized matrix is saved as .rds) — https://github.com/wmalab/scHiCDiff

## Examples

```
normalized_hic_matrix <- scHiCDiff.norm(dat_HiC, bias.info.path)
```

## Evaluation signals

- Output matrix is square with the same N*N dimensions as the input contact matrix.
- All entries of the returned normalized Hi-C matrix are defined (no missing/NA values).
- The returned object matches the documented contract: 'The function returns the normalized Hi-C matrix.'
- When reshaped as count.table, all values are non-negative normalized read counts, as required by the detection functions.
- Row/column count of count.table matches bin pairs by cell (e.g. the sample data: 100 cell columns = 50 condition-1 + 50 condition-2).

## Limitations

- No version number is stated for scHiCDiff or R in the article or README (only a developmental scHiCDiff_1.0.tar.gz install).
- The bias feature file must be generated externally; the paper points to http://dna.cs.miami.edu/scHiCNorm for its generation.
- Normalization requires the three local features of all bins, so bins lacking effective length, GC content, or mappability annotations cannot be processed.

## Evidence

- [full_text] Gaussian convolution imputation rationale: [evidence span withheld — anchor status: not_found]
- [readme] Bias feature inputs: "bias.info.path      The pathway of the three local features (effective length,GC content 
                    and mappability of fragment ends) of all bins."
- [readme] Input contract: "dat_HiC             A N*N scHi-C matrix."
- [methods] Normalization output contract: "The function returns the normalized Hi-C matrix."
- [methods] Normalization function identity: "The inputs of the normalization function scHiCDiff.norm are illustrated below"
