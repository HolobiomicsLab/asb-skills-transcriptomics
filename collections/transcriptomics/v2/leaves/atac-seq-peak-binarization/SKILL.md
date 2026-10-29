---
name: atac-seq-peak-binarization
description: Use when a raw scATAC-seq peak-by-cell count matrix (AnnData, cells ×
  peaks) must be prepared for a variational autoencoder input — binarization, 0.5%
  peak-frequency filtering, TF-IDF, and [0,1] scaling are needed.
when_to_use_negative:
- Input is already normalized or TF-IDF-transformed scATAC-seq data — this skill operates
  on raw count matrices and double-transforming would invalidate the [0,1] scaling.
- Data is scRNA-seq or scADT-seq — use data_processing.RNA_data_preprocessing or data_processing.CLR_transform
  instead of the ATAC path.
- The analysis needs all peaks preserved (e.g., peak-level regression on rare peaks),
  since the 0.5% cell-frequency filter drops peaks active in fewer than 0.5% of cells.
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: scButterfly
  role: Provides butterfly.load_data and butterfly.data_preprocessing, which invoke
    data_processing.ATAC_data_preprocessing for binarization, 0.5% peak filtering,
    TF-IDF, and [0,1] scaling; data_processing.inverse_TFIDF reverses the TF-IDF mapping.
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python 3.9
  role: Runtime environment for installing and running scButterfly (conda create scButterfly
    python==3.9; pip install scButterfly).
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:fd2c08cc9d3e7aae33fe1d430b15a845dd095c99f452a526417677dc3d6a8895
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - Python
  - Python 3.9
  - scButterfly
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# atac-seq-peak-binarization

## Summary

Binarizes raw scATAC-seq peak-by-cell count matrices and applies scButterfly's ATAC-specific preprocessing (0.5% cell-frequency peak filtering, TF-IDF transformation, [0,1] scaling) so the matrix is suitable for the modality-specific ATAC variational encoder.

## When to use

Use when a raw scATAC-seq peak-by-cell count matrix (AnnData, cells × peaks) must be prepared for a variational autoencoder input — binarization, 0.5% peak-frequency filtering, TF-IDF, and [0,1] scaling are needed.

## When NOT to use

- Input is already normalized or TF-IDF-transformed scATAC-seq data — this skill operates on raw count matrices and double-transforming would invalidate the [0,1] scaling.
- Data is scRNA-seq or scADT-seq — use data_processing.RNA_data_preprocessing or data_processing.CLR_transform instead of the ATAC path.
- The analysis needs all peaks preserved (e.g., peak-level regression on rare peaks), since the 0.5% cell-frequency filter drops peaks active in fewer than 0.5% of cells.

## Inputs

- Raw scATAC-seq peak-by-cell count matrix as an AnnData object of shape n_obs × n_vars (rows = cells, columns = peaks), passed to butterfly.load_data
- Optional cell ID lists for train_id, test_id, and validation_id (default validation split is 20% of train_id)

## Outputs

- Binarized, peak-filtered (≥0.5% cell frequency), TF-IDF-transformed, [0,1]-scaled ATAC matrix ready for the ATAC encoder (e.g., saved as atac_preprocessed.npz with shape (retained peaks, cells))
- Optional processed-data files and preprocessing logs when save_data/logging_path parameters are set

## How to apply

Load the raw scATAC-seq count matrix via `butterfly.load_data` as an AnnData object of shape n_obs × n_vars (rows = cells, columns = peaks), then run `butterfly.data_preprocessing()`, which internally applies `data_processing.ATAC_data_preprocessing`. First binarize the peak count matrix so each peak-cell entry encodes accessibility presence/absence; this addresses dropout-driven false zeros from DNA material loss during library preparation. Filter to retain only peaks active in at least 0.5% of cells, dropping rare/noisy peaks. Apply TF-IDF transformation to normalize for cell sequencing depth and peak prevalence (the reverse mapping is available as `data_processing.inverse_TFIDF`), then min-max scale values to [0,1] so the transformed peak-by-cell matrix is compatible with the variational autoencoder input. The README strongly recommends the default settings of `data_preprocessing` to keep the best model result.

## Related tools

- **scButterfly** (Provides butterfly.load_data and butterfly.data_preprocessing, which invoke data_processing.ATAC_data_preprocessing for binarization, 0.5% peak filtering, TF-IDF, and [0,1] scaling; data_processing.inverse_TFIDF reverses the TF-IDF mapping.) — https://github.com/BioX-NKU/scButterfly
- **Python 3.9** (Runtime environment for installing and running scButterfly (conda create scButterfly python==3.9; pip install scButterfly).)

## Examples

```
butterfly.load_data(RNA_data, ATAC_data, train_id, test_id, validation_id); butterfly.data_preprocessing()
```

## Evaluation signals

- Output matrix values all fall within [0,1] after min-max scaling of the TF-IDF transform.
- Output shape equals (retained peaks, cells), with retained peaks consistent with the ≥0.5% cell-frequency filter applied to the input peak set.
- Binarization check: before TF-IDF, every peak-cell entry is strictly 0 or 1 (presence/absence), never a raw count > 1.
- Round-trip consistency: applying data_processing.inverse_TFIDF to the transformed matrix recovers the filtered binarized matrix.
- Preprocessing completes with default data_preprocessing settings (save_data=False, default validation split of 20% of train_id) without errors or warnings from load_data.

## Limitations

- The 0.5% cell-frequency filter drops peaks active in fewer than 0.5% of cells, which can remove rare cell-type-specific accessibility signal.
- scButterfly supports refining data_preprocessing with other parameters, but the README strongly recommends default settings to keep the best result, limiting safe customization.
- The transformed matrix is tuned for variational autoencoder input ([0,1] scaled); it is not a raw count or plain TF-IDF matrix and should not be reused where raw counts are required.

## Evidence

- [methods] the peak-by-cell matrix is binarized, peaks active in fewer than 0.5% of cells are filtered out, the data undergo TF-IDF transformation, and values are scaled to [0,1]: [evidence span withheld — anchor status: not_found]
- [intro] dropout events due to the loss of DNA material during library preparation require tailored denoising approaches for the large number of false zeros: "dropout events due to the loss of DNA material during library preparation require tailored denoising approaches for the large number of false zeros"
- [readme] ATAC_data | AnnData object of shape `n_obs` × `n_vars`. Rows correspond to cells and columns to peaks.: [evidence span withheld — anchor status: external_doc]
- [readme] we strongly recommend the default settings to keep the best result for model: [evidence span withheld — anchor status: external_doc]
