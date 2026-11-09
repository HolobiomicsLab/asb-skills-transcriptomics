---
name: sparse-matrix-minmax-scaling
description: Use when a raw scATAC-seq peak-by-cell count matrix must be turned into
  bounded, normalized encoder input for scButterfly's dual-aligned VAE — binarize,
  filter rare peaks, TF-IDF, then scale to [0,1].
when_to_use_negative:
- Preprocessing the scRNA-seq modality — use data_processing.RNA_data_preprocessing
  instead; ADT data uses data_processing.CLR_transform
- The matrix has already been binarized/TF-IDF-transformed by a prior scButterfly
  run — reapplying would double-transform the features
- Converting predicted ATAC profiles back to approximate raw counts — use data_processing.inverse_TFIDF,
  not re-scaling
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scButterfly
  role: Provides data_processing.ATAC_data_preprocessing, which binarizes the peak
    matrix, filters peaks active in <0.5% of cells, applies TF-IDF, and min-max scales
    values to [0,1] for ATAC encoder input; inverse_TFIDF supplies the reverse mapping.
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python 3.9
  role: Runtime environment for the scButterfly package (conda create scButterfly
    python==3.9); preprocessing is invoked through the Butterfly API in Python.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:a17be4a0c7e3998ffa7362ca9a09acee9a2dab3526a57fffc8f89d3a694034ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - Python
  - Python 3.9
  - scButterfly
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# sparse-matrix-minmax-scaling

## Summary

Final scaling step of scButterfly's ATAC-specific preprocessing (data_processing.ATAC_data_preprocessing): after binarization, 0.5% peak-frequency filtering, and TF-IDF transformation of a raw scATAC-seq peak-by-cell count matrix, values are min-max scaled to [0,1] so the matrix is compatible with the variational autoencoder's ATAC encoder.

## When to use

Use when a raw scATAC-seq peak-by-cell count matrix must be turned into bounded, normalized encoder input for scButterfly's dual-aligned VAE — binarize, filter rare peaks, TF-IDF, then scale to [0,1].

## When NOT to use

- Preprocessing the scRNA-seq modality — use data_processing.RNA_data_preprocessing instead; ADT data uses data_processing.CLR_transform
- The matrix has already been binarized/TF-IDF-transformed by a prior scButterfly run — reapplying would double-transform the features
- Converting predicted ATAC profiles back to approximate raw counts — use data_processing.inverse_TFIDF, not re-scaling

## Inputs

- Raw scATAC-seq peak-by-cell count matrix (AnnData, n_obs × n_vars, rows = cells, columns = peaks) loaded via butterfly.load_data
- Train/test/validation cell ID lists (train_id, test_id, optional validation_id)

## Outputs

- Transformed peak-by-cell matrix with values in [0,1] (optionally saved, e.g., atac_preprocessed.npz), ready for ATAC encoder input
- Preprocessing log via logging_path; retained-peak shape consistent with the 0.5% cell-frequency filter

## How to apply

Load the paired scATAC-seq peak-by-cell raw count AnnData via butterfly.load_data, then run butterfly.data_preprocessing(), which internally calls scButterfly.data_processing.ATAC_data_preprocessing. First binarize the peak matrix so each entry encodes accessibility presence/absence, then drop peaks active in fewer than 0.5% of cells to remove rare/noisy peaks. Next apply TF-IDF (the reverse mapping is available as data_processing.inverse_TFIDF) to normalize for cell sequencing depth and peak prevalence. Finally min-max scale the TF-IDF-transformed values to the [0,1] range so the matrix is compatible with variational autoencoder input; save the result (e.g., atac_preprocessed.npz) with save_data=True and a file_path. The README strongly recommends keeping the default preprocessing settings to preserve the best model result.

## Related tools

- **scButterfly** (Provides data_processing.ATAC_data_preprocessing, which binarizes the peak matrix, filters peaks active in <0.5% of cells, applies TF-IDF, and min-max scales values to [0,1] for ATAC encoder input; inverse_TFIDF supplies the reverse mapping.) — https://github.com/BioX-NKU/scButterfly
- **Python 3.9** (Runtime environment for the scButterfly package (conda create scButterfly python==3.9); preprocessing is invoked through the Butterfly API in Python.)

## Examples

```
from scButterfly.butterfly import Butterfly; b = Butterfly(); b.load_data(RNA_data, ATAC_data, train_id, test_id); b.data_preprocessing(save_data=True, file_path='./')
```

## Evaluation signals

- Shape check: output matrix has shape (retained peaks, cells) consistent with retaining only peaks active in at least 0.5% of cells
- Range check: every entry of the scaled TF-IDF matrix falls within [0,1] (min-max scaling invariant: min = 0, max = 1)
- Binarization check: before filtering/TF-IDF, the raw count matrix contains only 0/1 accessibility values
- Invertibility check: applying data_processing.inverse_TFIDF to the transformed matrix recovers the TF-IDF-input scale without error
- Save/round-trip check: with save_data=True and file_path set, atac_preprocessed.npz exists and reloads with the expected dims and value range

## Limitations

- The README strongly recommends the default preprocessing settings; deviating parameters risk degrading model results
- scATAC-seq dropout events (false zeros from loss of DNA material during library preparation) motivate binarization/TF-IDF, but very sparse or small datasets may lose too many peaks under the 0.5% cell-frequency filter
- Peaks must be sorted by chromosome before construct_model (chrom_list of per-chromosome peak counts), so downstream steps assume chromosome-sorted peak order

## Evidence

- [methods] ATAC preprocessing pipeline definition: "data_processing.ATAC_data_preprocessing"
- [readme] README: preprocessing entry point: [evidence span withheld — anchor status: external_doc]
- [readme] README: default settings recommendation: [evidence span withheld — anchor status: external_doc]
- [intro] Article: dropout rationale for ATAC denoising: "dropout events due to the loss of DNA material during library preparation require tailored denoising approaches for the large number of false zeros"
- [readme] README: ATAC AnnData orientation: [evidence span withheld — anchor status: external_doc]
