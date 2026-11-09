---
name: scatac-scrna-adt-data-preprocessing
description: Use when raw count matrices of scRNA-seq, scATAC-seq, and/or CITE-seq
  ADT data must be normalized and preprocessed before training or applying scButterfly
  cross-modality translators.
when_to_use_negative:
- Input matrices are not raw counts (already normalized or batch-corrected), since
  load_data expects raw count matrices
- Modalities outside the supported scRNA-seq/scATAC-seq/ADT set, such as spatial transcriptomics
  or single-cell Hi-C, which the paper names as future directions not yet supported
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scButterfly
  role: Provides Butterfly.load_data and data_preprocessing APIs used to load raw
    counts and normalize scRNA-seq, scATAC-seq, and ADT inputs
  repo: https://github.com/BioX-NKU/scButterfly
- name: scButterfly data_processing
  role: Houses RNA_data_preprocessing, ATAC_data_preprocessing, inverse_TFIDF, and
    CLR_transform routines applied in this preprocessing skill
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python 3.9
  role: Runtime environment recommended for installing and running scButterfly preprocessing
    (conda create scButterfly python==3.9)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:4521cd6bb390b969ec657f2e0accc2f0124234828c434e73f4c4870a1299d93e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - BABEL
  - Polarbear
  - Python
  - Python 3.9
  - scButterfly
  - scButterfly data_processing
  - sciPENN
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# scatac-scrna-adt-data-preprocessing

## Summary

Preprocess raw scRNA-seq, scATAC-seq, and ADT count matrices with scButterfly's RNA_data_preprocessing, ATAC_data_preprocessing, and CLR_transform so they are suitably normalized and denoised for downstream cross-modality translation with dual-aligned VAEs.

## When to use

Use when raw count matrices of scRNA-seq, scATAC-seq, and/or CITE-seq ADT data must be normalized and preprocessed before training or applying scButterfly cross-modality translators.

## When NOT to use

- Input matrices are not raw counts (already normalized or batch-corrected), since load_data expects raw count matrices
- Modalities outside the supported scRNA-seq/scATAC-seq/ADT set, such as spatial transcriptomics or single-cell Hi-C, which the paper names as future directions not yet supported

## Inputs

- AnnData objects with raw scRNA-seq count matrices (cells x genes)
- AnnData objects with raw scATAC-seq count matrices (cells x peaks)
- CITE-seq ADT protein count matrices (e.g., CITE_BMMC paired scRNA-seq/ADT data)
- Train/test/validation cell ID lists for butterfly.load_data

## Outputs

- Preprocessed RNA expression matrices (RNA_data_preprocessing output)
- Preprocessed ATAC peak matrices (ATAC_data_preprocessing output)
- CLR-transformed ADT matrices (CLR_transform output)
- Optional saved processed data files and processing logs (save_data/file_path/logging_path)

## How to apply

Load raw count AnnData objects via butterfly.Butterfly.load_data (raw count matrices are required, not already-normalized data). Apply scButterfly.data_processing.RNA_data_preprocessing for scRNA-seq inputs, data_processing.ATAC_data_preprocessing for scATAC-seq peak-by-cell matrices (sorting peaks by chromosome, supplying a chrom_list to construct_model), and data_processing.CLR_transform for ADT profiles from CITE-seq data such as CITE_BMMC. The README recommends keeping the default preprocessing settings (via butterfly.data_preprocessing with save_data/file_path/logging_path options) to preserve the best model results. After preprocessing, optionally apply butterfly.augmentation with cell_type_augmentation or MultiVI_augmentation before construct_model and train_model. Correct application is verified by the preprocessed matrices feeding successfully into Butterfly training and yielding competitive AMI/ARI/HOM/NMI clustering scores on held-out cells.

## Related tools

- **scButterfly** (Provides Butterfly.load_data and data_preprocessing APIs used to load raw counts and normalize scRNA-seq, scATAC-seq, and ADT inputs) — https://github.com/BioX-NKU/scButterfly
- **scButterfly data_processing** (Houses RNA_data_preprocessing, ATAC_data_preprocessing, inverse_TFIDF, and CLR_transform routines applied in this preprocessing skill) — https://github.com/BioX-NKU/scButterfly
- **Python 3.9** (Runtime environment recommended for installing and running scButterfly preprocessing (conda create scButterfly python==3.9))

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id, validation_id); butterfly.data_preprocessing()
```

## Evaluation signals

- Preprocessed outputs come from the default settings of butterfly.data_preprocessing, as the README 'strongly recommend[s] the default settings to keep the best result for model'
- ATAC peaks are sorted by chromosome and a chrom_list of per-chromosome peak counts is supplied to construct_model without errors
- ADT profiles are CLR-transformed and the chained translator (epigenome -> transcriptome -> proteome) completes inference on BMMC and CITE_BMMC
- Downstream translation achieves AMI, ARI, HOM, and NMI scores via calculate_cluster_index that are competitive against BABEL+sciPENN and Polarbear+sciPENN baselines

## Limitations

- The README states scButterfly 'strongly recommend[s] the default settings' for preprocessing, so non-default parameters risk degrading translation performance
- Limited multi-omics cell numbers and dropout-driven false zeros in single-cell data remain challenging and motivated the data augmentation scheme rather than being fully solved by preprocessing
- Incorporating cell heterogeneity information from public bulk omics or prior gene-regulatory knowledge to improve preprocessing/translation is proposed as future work, not implemented

## Evidence

- [readme] Before data preprocessing, you should load the raw count matrix of scRNA-seq and scATAC-seq data via `butterfly.load_data`: [evidence span withheld — anchor status: external_doc]
- [readme] we strongly recommend the default settings to keep the best result for model: [evidence span withheld — anchor status: external_doc]
- [methods] data_processing.RNA_data_preprocessing: "data_processing.RNA_data_preprocessing"
- [readme] scButterfly need a list of peaks count for each chromosome, remember to sort peaks with chromosomes: [evidence span withheld — anchor status: external_doc]
- [intro] dropout events due to the loss of DNA material during library preparation require tailored denoising approaches for the large number of false zeros: "dropout events due to the loss of DNA material during library preparation require tailored denoising approaches for the large number of false zeros"
