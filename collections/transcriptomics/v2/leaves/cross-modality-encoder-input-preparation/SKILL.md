---
name: cross-modality-encoder-input-preparation
description: Use when a raw scATAC-seq peak-by-cell count matrix (AnnData, rows=cells,
  columns=peaks) must be converted into a VAE-compatible, [0,1]-scaled matrix before
  feeding a cross-modality translation model's ATAC encoder.
when_to_use_negative:
- Input is already TF-IDF-transformed, binarized, or a normalized feature table rather
  than raw peak counts
- Preparing scRNA-seq data for the RNA encoder — use RNA_data_preprocessing instead,
  since ATAC-specific binarization/TF-IDF is modality-specific
- Unpaired or multi-modal inputs requiring data augmentation or joint MultiVI modeling
  — this skill covers only per-modality preprocessing prior to augmentation
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scButterfly
  role: Provides butterfly.load_data and butterfly.data_preprocessing APIs wrapping
    data_processing.ATAC_data_preprocessing, data_processing.inverse_TFIDF, binarization,
    0.5% peak-frequency filtering, TF-IDF, and [0,1] scaling of the peak-by-cell matrix
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python 3.9
  role: Runtime environment for installing and running scButterfly (conda create scButterfly
    python==3.9)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:f09da55779676a9b55bef28fbe5c312be81990ab3fdcf022154c750d33a33b8d
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-modality-encoder-input-preparation

## Summary

Transforms raw single-cell ATAC-seq peak-by-cell count matrices (cells × peaks in AnnData format) into the normalized, value-bounded input required by the scButterfly ATAC variational autoencoder, via binarization, rare-peak filtering, TF-IDF transformation, and [0,1] scaling. This mirrors the modality-specific customary preprocessing applied before scButterfly's encoder/decoder pretraining.

## When to use

Use when a raw scATAC-seq peak-by-cell count matrix (AnnData, rows=cells, columns=peaks) must be converted into a VAE-compatible, [0,1]-scaled matrix before feeding a cross-modality translation model's ATAC encoder.

## When NOT to use

- Input is already TF-IDF-transformed, binarized, or a normalized feature table rather than raw peak counts
- Preparing scRNA-seq data for the RNA encoder — use RNA_data_preprocessing instead, since ATAC-specific binarization/TF-IDF is modality-specific
- Unpaired or multi-modal inputs requiring data augmentation or joint MultiVI modeling — this skill covers only per-modality preprocessing prior to augmentation

## Inputs

- Raw scATAC-seq peak-by-cell count matrix as an AnnData object (n_obs × n_vars; rows = cells, columns = peaks)
- Paired scRNA-seq AnnData count matrix and train/test/validation cell ID lists (for butterfly.load_data)

## Outputs

- Binarized, peak-filtered (≥0.5% cell frequency), TF-IDF-transformed, [0,1]-scaled peak-by-cell matrix ready for ATAC encoder input
- Optionally saved processed matrix (e.g., atac_preprocessed.npz) via save_data/file_path parameters

## How to apply

Load the raw paired scRNA/scATAC peak-by-cell count matrix with butterfly.load_data(RNA_data, ATAC_data, train_id, test_id, validation_id), then apply the ATAC-specific preprocessing implemented as data_processing.ATAC_data_preprocessing (invoked via butterfly.data_preprocessing()). First binarize the peak count matrix so each peak-cell entry encodes accessibility presence/absence. Next filter the binarized matrix to retain only peaks active in at least 0.5% of cells, dropping rare/noisy peaks. Apply TF-IDF transformation to normalize for cell sequencing depth and peak prevalence (the module also provides data_processing.inverse_TFIDF for the reverse mapping, e.g., to reconstruct count-like outputs from ATAC decoder predictions). Finally min-max scale values to the [0,1] range so the matrix is compatible with variational autoencoder input; save the result (e.g., as atac_preprocessed.npz). Use the recommended default parameters of butterfly.data_preprocessing, since the README strongly recommends defaults for best model results.

## Related tools

- **scButterfly** (Provides butterfly.load_data and butterfly.data_preprocessing APIs wrapping data_processing.ATAC_data_preprocessing, data_processing.inverse_TFIDF, binarization, 0.5% peak-frequency filtering, TF-IDF, and [0,1] scaling of the peak-by-cell matrix) — https://github.com/BioX-NKU/scButterfly
- **Python 3.9** (Runtime environment for installing and running scButterfly (conda create scButterfly python==3.9))

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id, validation_id); butterfly.data_preprocessing()
```

## Evaluation signals

- All values in the preprocessed peak-by-cell matrix fall within [0,1] after min-max scaling
- Shape of the filtered matrix equals (retained peaks, cells), with retained peaks consistent with the 0.5% peak-frequency threshold
- Matrix is strictly binarized before TF-IDF: each raw peak-cell entry is 0 or 1 prior to transformation
- Saved output file (e.g., atac_preprocessed.npz) exists and loads with the expected peak-by-cell dimensions
- Default parameters of butterfly.data_preprocessing were used (README strongly recommends defaults for best model results)

## Limitations

- Rare peaks active in fewer than 0.5% of cells are permanently dropped, which may remove weakly accessible but informative peaks
- Peaks must be sorted by chromosome and a chrom_list of per-chromosome peak counts supplied for construct_model; preprocessing alone does not produce this
- If using data augmentation, scButterfly-T will fall back to MultiVI_augmentation when no cell_type label exists in adata.obs, changing downstream training behavior beyond this preprocessing skill

## Evidence

- [methods] ATAC-specific preprocessing with binarization, 0.5% peak filter, TF-IDF, [0,1] scaling: "data_processing.ATAC_data_preprocessing"
- [methods] Reverse TF-IDF mapping available for ATAC decoder outputs: "data_processing.inverse_TFIDF"
- [readme] README: raw count matrices must be loaded before preprocessing via butterfly.load_data: [evidence span withheld — anchor status: external_doc]
- [readme] README: ATAC AnnData shape and defaults recommendation: [evidence span withheld — anchor status: external_doc]
- [readme] README: default preprocessing settings are recommended for best results: [evidence span withheld — anchor status: external_doc]
