---
name: count-matrix-log2-normalization
description: Use when raw scRNA-seq or proteomics count matrices (TSV/CSV) must be
  transformed to the 0–1 range expected by FAVA's sigmoid-scaled VAE output layer,
  before running favapy network inference.
when_to_use_negative:
- The matrix is already normalized to [0,1] or already passed through FAVA's pre-processing
  layer — re-applying row-max scaling would distort the values.
- The input is not a count matrix (e.g. latent-space embeddings or already-computed
  correlation networks) — this skill targets raw high-dimensional omics counts feeding
  the VAE.
- You plan to feed the data into a model whose output activation is not 0–1 sigmoid-scaled
  — the row-max scaling is specifically matched to FAVA's sigmoid output layer.
tools:
- name: FAVA (favapy)
  role: Provides the pre-processing layer (log2 normalization + [0,1] row-max scaling)
    whose output preprocessed_matrix.tsv is consumed as the favapy command-line data
    file
  repo: https://github.com/mikelkou/fava
- name: Python
  role: Language used to implement the pre-processing (loading TSV/CSV count matrices
    with pandas, log2 transformation, row-max scaling)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae010
    title: 'FAVA: high-quality functional association networks inferred from scRNA-seq
      and proteomics data'
  source_content_hash: sha256:82d2a6a4c941991075acfd19c51f0ab69dd01ee5ca5687c6eeaf2f8bbdb30514
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mikelkou/fava
  tools:
  - FAVA (favapy)
  - Python
  - favapy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# count-matrix-log2-normalization

## Summary

Reconstructs FAVA's pre-processing stage: log2 normalization of raw count matrices followed by [0,1] row-max scaling, so transformed values match the 0–1 scaled sigmoid output layer of the FAVA VAE.

## When to use

Use when raw scRNA-seq or proteomics count matrices (TSV/CSV) must be transformed to the 0–1 range expected by FAVA's sigmoid-scaled VAE output layer, before running favapy network inference.

## When NOT to use

- The matrix is already normalized to [0,1] or already passed through FAVA's pre-processing layer — re-applying row-max scaling would distort the values.
- The input is not a count matrix (e.g. latent-space embeddings or already-computed correlation networks) — this skill targets raw high-dimensional omics counts feeding the VAE.
- You plan to feed the data into a model whose output activation is not 0–1 sigmoid-scaled — the row-max scaling is specifically matched to FAVA's sigmoid output layer.

## Inputs

- Raw count matrix from scRNA-seq or proteomics data (TSV format by default, CSV accepted), loaded with pandas

## Outputs

- preprocessed_matrix.tsv — the log2-normalized, [0,1] row-max-scaled count matrix that FAVA consumes as its command-line data file

## How to apply

Load the raw count matrix with pandas, accepting TSV (FAVA's default) or CSV format. Apply log2 normalization to the count values of the matrix. Then scale every value into the [0,1] interval by dividing each row's values by that row's maximum — this mirrors the 0–1 scaled sigmoid output activation of the FAVA VAE, so VAE output and preprocessed input share the same range. Write the result to preprocessed_matrix.tsv, the file format FAVA consumes as its command-line data file, and pass it to favapy for VAE compression into the low-dimensional latent space used for correlation-based network inference.

## Related tools

- **FAVA (favapy)** (Provides the pre-processing layer (log2 normalization + [0,1] row-max scaling) whose output preprocessed_matrix.tsv is consumed as the favapy command-line data file) — https://github.com/mikelkou/fava
- **Python** (Language used to implement the pre-processing (loading TSV/CSV count matrices with pandas, log2 transformation, row-max scaling))

## Examples

```
favapy preprocessed_matrix.tsv ./output/
```

## Evaluation signals

- All values in the preprocessed matrix fall within the [0,1] interval.
- The maximum value of every row equals exactly 1 (row-max scaling invariant).
- Values reflect log2 of the original counts, i.e. reconstructing 2**value reproduces the raw count up to the row-max factor.
- The output file is named preprocessed_matrix.tsv and is accepted directly by the favapy CLI as <path-to-data-file>.
- Downstream VAE output (sigmoid-activated) is on the same 0–1 scale as the preprocessed input, so correlations computed in latent space are well-scaled.

## Limitations

- The [0,1] row-max scaling is matched to FAVA's sigmoid output layer; it is not a generic normalization and may be inappropriate for other downstream models.
- Rows whose maximum is 0 (all-zero counts) have no defined row-max scaling, and log2 is undefined for zero counts — the article/README do not specify a pseudocount policy.
- FAVA itself handles both scRNA-seq and proteomics high-dimensional data; applying this pre-processing outside that scope (e.g. to non-count modalities) is unsupported by the documented workflow.

## Evidence

- [other] values are normalized to the [0,1] range by dividing each value by the maximum of the row it belongs to, matching the 0-1 scaled sigmoid output layer of the VAE: [evidence span withheld — anchor status: not_found]
- [other] Write the preprocessed matrix to preprocessed_matrix.tsv, the format FAVA consumes as its command-line data file.: [evidence span withheld — anchor status: not_found]
- [methods] favapy <path-to-data-file> <path-to-save-output>: "Run FAVA from the command line as follows:

.. code-block:: bash

   favapy <path-to-data-file> <path-to-save-output>"
- [abstract] FAVA functions within the scverse ecosystem, employing AnnData as its input source: "FAVA functions within the scverse ecosystem, employing AnnData as its input source"
