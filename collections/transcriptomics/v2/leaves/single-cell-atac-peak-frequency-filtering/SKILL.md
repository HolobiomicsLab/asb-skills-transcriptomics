---
name: single-cell-atac-peak-frequency-filtering
description: Use when a raw scATAC-seq peak-by-cell count matrix (cells × peaks, AnnData
  with peaks as n_vars) must be normalized and denoised before feeding a variational
  autoencoder or cross-modality translation model.
when_to_use_negative:
- Input is already TF-IDF-transformed or binarized — reapplying preprocessing would
  double-transform and distort accessibility values
- 'The target modality is scRNA-seq or scADT-seq: use data_processing.RNA_data_preprocessing
  or data_processing.CLR_transform instead'
- 'You need an inverse (predicted-matrix → count) transformation: use data_processing.inverse_TFIDF
  rather than the forward preprocessing'
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0769
tools:
- name: scButterfly
  role: Provides ATAC_data_preprocessing (binarization, 0.5% peak-frequency filter,
    TF-IDF, [0,1] scaling) and inverse_TFIDF within its data_processing module, invoked
    via Butterfly.load_data and Butterfly.data_preprocessing
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python 3.9
  role: Runtime environment for the scButterfly package (conda create scButterfly
    python==3.9)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:affca9b474140158a77c241c557c3b38ddf4249a829d23744cd96d79b275d14b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - Python
  - Python 3.9
  - scButterfly
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# single-cell-atac-peak-frequency-filtering

## Summary

ATAC-specific preprocessing from scButterfly's data_processing.ATAC_data_preprocessing: binarize the peak-by-cell count matrix, filter peaks active in fewer than 0.5% of cells, apply TF-IDF transformation, and min-max scale values to [0,1] to prepare the matrix for the ATAC encoder of the dual-aligned VAE.

## When to use

Use when a raw scATAC-seq peak-by-cell count matrix (cells × peaks, AnnData with peaks as n_vars) must be normalized and denoised before feeding a variational autoencoder or cross-modality translation model.

## When NOT to use

- Input is already TF-IDF-transformed or binarized — reapplying preprocessing would double-transform and distort accessibility values
- The target modality is scRNA-seq or scADT-seq: use data_processing.RNA_data_preprocessing or data_processing.CLR_transform instead
- You need an inverse (predicted-matrix → count) transformation: use data_processing.inverse_TFIDF rather than the forward preprocessing

## Inputs

- Raw scATAC-seq peak-by-cell count matrix as an AnnData object (n_obs = cells, n_vars = peaks), e.g. from a paired RNA/ATAC tutorial dataset
- Cell ID lists: train_id, test_id, and optional validation_id

## Outputs

- Transformed peak-by-cell matrix with values in [0,1], shape (retained peaks, cells), ready for the scButterfly ATAC encoder input
- Optional saved processed data file (via save_data=True and file_path, e.g. atac_preprocessed.npz) and process logging

## How to apply

Load the raw peak-by-cell count matrix via Butterfly.load_data, then apply scButterfly.data_processing.ATAC_data_preprocessing: (1) binarize the matrix so each entry reflects accessibility presence/absence; (2) drop peaks that are accessible in fewer than 0.5% of cells to remove rare/noisy peaks; (3) apply TF-IDF to normalize for cell sequencing depth and peak prevalence (data_processing.inverse_TFIDF is available for the reverse mapping, e.g. to convert predicted profiles back to count space); (4) min-max scale TF-IDF values into [0,1] for compatibility with VAE input. This step runs before the modality-specific ATAC encoder/decoder pretraining and augmentation, and is invoked as part of butterfly.data_preprocessing(); the authors recommend keeping the default settings for best results.

## Related tools

- **scButterfly** (Provides ATAC_data_preprocessing (binarization, 0.5% peak-frequency filter, TF-IDF, [0,1] scaling) and inverse_TFIDF within its data_processing module, invoked via Butterfly.load_data and Butterfly.data_preprocessing) — https://github.com/BioX-NKU/scButterfly
- **Python 3.9** (Runtime environment for the scButterfly package (conda create scButterfly python==3.9))

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id); butterfly.data_preprocessing(save_data=True, file_path='.')
```

## Evaluation signals

- All values of the transformed peak-by-cell matrix fall within [0,1]
- Matrix shape equals (retained peaks, cells) consistent with the 0.5% peak-frequency filter — no peak present in fewer than 0.5% of cells remains
- Each peak-cell entry is binary before TF-IDF (binarization invariant holds)
- Process logging emitted (logging_path set) and processed data saved to file_path when save_data=True
- Downstream ATAC encoder/decoder pretraining accepts the matrix without dimension or range errors

## Limitations

- The README recommends against refining the preprocessing parameters: 'we strongly recommend the default settings to keep the best result for model'
- Drops rare peaks (<0.5% of cells), which removes genuinely rare but biologically real accessibility signals
- ATAC preprocessing output is on a transformed [0,1] scale, not original counts; inverse_TFIDF must be applied to interpret predicted profiles in count space

## Evidence

- [methods] ATAC_data_preprocessing: binarize, 0.5% filter, TF-IDF, [0,1] scaling: "data_processing.ATAC_data_preprocessing"
- [methods] The peak-by-cell matrix is binarized, peaks active in fewer than 0.5% of cells are filtered out, the data undergo TF-IDF transformation, and values are scaled to [0,1]: [evidence span withheld — anchor status: not_found]
- [readme] README: preprocessing is applied to the raw count matrix before model training: [evidence span withheld — anchor status: external_doc]
- [readme] README: default settings recommended for preprocessing: [evidence span withheld — anchor status: external_doc]
- [methods] README: inverse TF-IDF exists for reverse mapping: "data_processing.inverse_TFIDF"
