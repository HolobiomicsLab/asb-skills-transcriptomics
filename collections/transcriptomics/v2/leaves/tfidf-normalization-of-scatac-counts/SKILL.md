---
name: tfidf-normalization-of-scatac-counts
description: Use when a raw scATAC-seq peak-by-cell count matrix (AnnData with cells
  as n_obs and peaks as n_vars) must be prepared as encoder input for a cross-modality
  translator such as scButterfly before modality-specific pretraining.
when_to_use_negative:
- The input is already TF-IDF transformed or a preprocessed feature table rather than
  a raw scATAC-seq count matrix
- The input is scRNA-seq data — use data_processing.RNA_data_preprocessing instead;
  ADT-seq uses CLR_transform, not TF-IDF
- You need to convert predicted ATAC profiles back to counts — use data_processing.inverse_TFIDF,
  not the forward preprocessing
tools:
- name: scButterfly
  role: Provides data_processing.ATAC_data_preprocessing (binarization, 0.5% peak
    filter, TF-IDF, [0,1] scaling), data_processing.inverse_TFIDF for the reverse
    mapping, and the butterfly.load_data / butterfly.data_preprocessing API that performs
    the preprocessing
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python 3.9
  role: Runtime environment for scButterfly (conda create scButterfly python==3.9)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:357b64d6c306b3b2650ac4d192aeb4a4dc86d4b6e679c8da0193f5c50c87c7a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - Python
  - Python 3.9
  - scButterfly
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# tfidf-normalization-of-scatac-counts

## Summary

ATAC-specific preprocessing for single-cell cross-modality translation: binarize the raw peak-by-cell scATAC-seq count matrix, filter peaks rare across cells, apply TF-IDF transformation, and min-max scale to [0,1] so the matrix is compatible with the ATAC variational encoder in scButterfly.

## When to use

Use when a raw scATAC-seq peak-by-cell count matrix (AnnData with cells as n_obs and peaks as n_vars) must be prepared as encoder input for a cross-modality translator such as scButterfly before modality-specific pretraining.

## When NOT to use

- The input is already TF-IDF transformed or a preprocessed feature table rather than a raw scATAC-seq count matrix
- The input is scRNA-seq data — use data_processing.RNA_data_preprocessing instead; ADT-seq uses CLR_transform, not TF-IDF
- You need to convert predicted ATAC profiles back to counts — use data_processing.inverse_TFIDF, not the forward preprocessing

## Inputs

- Raw scATAC-seq peak-by-cell count matrix as an AnnData object (n_obs = cells, n_vars = peaks)
- Lists of cell IDs: train_id, test_id, and optional validation_id (default: 20% of train_id)
- For model construction: chrom_list, a list of peaks count per chromosome, with peaks sorted by chromosome

## Outputs

- Transformed peak-by-cell matrix (binarized, 0.5% peak-frequency filtered, TF-IDF transformed, scaled to [0,1]) ready for the ATAC encoder
- Optionally, saved processed data and a process log when save_data=True and file_path/logging_path are set

## How to apply

Load the paired raw scATAC-seq count matrix via butterfly.load_data, passing AnnData objects plus train_id, test_id, and optional validation_id lists. Apply scButterfly's ATAC-specific preprocessing (data_processing.ATAC_data_preprocessing): (1) binarize each peak-cell entry to accessibility presence/absence, (2) drop peaks active in fewer than 0.5% of cells to remove rare/noisy peaks, (3) apply the TF-IDF transformation to normalize for cell sequencing depth and peak prevalence, and (4) min-max scale TF-IDF values to the [0,1] range for variational autoencoder compatibility. The module also provides data_processing.inverse_TFIDF to map translated ATAC outputs back to the original count space. Use the default settings of butterfly.data_preprocessing — the README strongly recommends defaults to keep the best model result.

## Related tools

- **scButterfly** (Provides data_processing.ATAC_data_preprocessing (binarization, 0.5% peak filter, TF-IDF, [0,1] scaling), data_processing.inverse_TFIDF for the reverse mapping, and the butterfly.load_data / butterfly.data_preprocessing API that performs the preprocessing) — https://github.com/BioX-NKU/scButterfly
- **Python 3.9** (Runtime environment for scButterfly (conda create scButterfly python==3.9))

## Examples

```
butterfly.load_data(RNA_data, ATAC_data, train_id, test_id, validation_id); butterfly.data_preprocessing()
```

## Evaluation signals

- The transformed peak-by-cell matrix contains no negative values and every entry falls within [0,1]
- All retained peaks are active in at least 0.5% of cells; rare/noisy peaks from the raw matrix are absent
- The matrix shape is consistent with (retained peaks, cells) after the 0.5% peak-frequency filter, with cells preserved from the input AnnData
- Original peak/cell ordering matches the input AnnData (peaks sorted by chromosome for chrom_list consistency)
- Because ATAC entries were binarized before TF-IDF, translation outputs are invertible via data_processing.inverse_TFIDF back to count-like values

## Limitations

- The README strongly recommends default preprocessing settings to keep the best result, so tuning thresholds may degrade translation performance
- Peaks must be sorted by chromosome and a chrom_list of per-chromosome peak counts supplied, or model construction will be inconsistent
- The article flags that cross-modality modeling of single-cell data is challenged by high dimensionality, technical variation, and dropout events, so the 0.5% filter trades peak coverage for denoising

## Evidence

- [methods] ATAC_data_preprocessing implements ATAC-specific preprocessing: "data_processing.ATAC_data_preprocessing"
- [methods] inverse_TFIDF supports the reverse mapping of transformed ATAC values: "data_processing.inverse_TFIDF"
- [readme] ATAC input format is an AnnData object with cells as rows and peaks as columns: [evidence span withheld — anchor status: external_doc]
- [readme] The butterfly.data_preprocessing API performs the preprocessing step before model training: [evidence span withheld — anchor status: external_doc]
- [readme] Default preprocessing settings are recommended over custom refinement: [evidence span withheld — anchor status: external_doc]
