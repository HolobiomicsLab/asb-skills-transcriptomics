---
name: paired-multiomics-data-preprocessing
description: Use when you have raw paired scRNA-seq/scATAC-seq count matrices (AnnData
  objects) that must be normalized and feature-processed before training or evaluating
  a scButterfly cross-modality translation model.
when_to_use_negative:
- Input is already normalized or feature-processed — scButterfly expects the raw count
  matrix as load_data input
- The task is training the model or running augmentation (scButterfly-T/scButterfly-C),
  not the preceding preprocessing stage
- Data is unpaired single-modality data intended for diagonal training scenarios,
  where different setup is needed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scButterfly
  role: Provides Butterfly.load_data and butterfly.data_preprocessing as well as data_processing.RNA_data_preprocessing
    and data_processing.ATAC_data_preprocessing to load raw paired count matrices
    and produce model-ready inputs
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python 3.9
  role: Runtime environment for scButterfly installation and execution of the preprocessing
    pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:ac621081dedec959b29f1558eecc0159bf662b6397a32cfb730eef178a7a7239
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - BABEL
  - JAMIE
  - Polarbear
  - Python
  - Python 3.9
  - scButterfly
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# paired-multiomics-data-preprocessing

## Summary

Load and preprocess paired single-modality scRNA-seq and scATAC-seq count matrices for scButterfly cross-modality translation, using the package's RNA_data_preprocessing and ATAC_data_preprocessing routines on raw AnnData count matrices.

## When to use

Use when you have raw paired scRNA-seq/scATAC-seq count matrices (AnnData objects) that must be normalized and feature-processed before training or evaluating a scButterfly cross-modality translation model.

## When NOT to use

- Input is already normalized or feature-processed — scButterfly expects the raw count matrix as load_data input
- The task is training the model or running augmentation (scButterfly-T/scButterfly-C), not the preceding preprocessing stage
- Data is unpaired single-modality data intended for diagonal training scenarios, where different setup is needed

## Inputs

- Raw scRNA-seq count matrix as AnnData object (n_obs x n_vars, rows = cells, columns = genes)
- Raw scATAC-seq count matrix as AnnData object (n_obs x n_vars, rows = cells, columns = peaks, sorted by chromosome)
- List of cell IDs for training (train_id)
- List of cell IDs for testing (test_id)
- Optional list of cell IDs for validation (validation_id; defaults to 20% of train_id)

## Outputs

- Preprocessed RNA expression matrix ready for scButterfly model construction
- Preprocessed ATAC peak matrix ready for scButterfly model construction
- chrom_list: a list of peak counts per chromosome for construct_model
- Optionally saved processed data files and preprocessing log files (save_data/file_path/logging_path)

## How to apply

Instantiate a Butterfly object and load the raw count matrices via butterfly.load_data(RNA_data, ATAC_data, train_id, test_id, validation_id), passing AnnData objects (genes as columns for RNA, peaks as columns for ATAC) plus explicit train/test/validation cell-ID lists (default validation is 20% of train_id if None). Apply data_processing.RNA_data_preprocessing and data_processing.ATAC_data_preprocessing to the respective matrices — or the integrated butterfly.data_preprocessing() wrapper — strongly preferring default settings, as the README states defaults keep the best result. For ATAC data, remember to sort peaks by chromosome and record the per-chromosome peak counts as chrom_list, which is required later by butterfly.construct_model(chrom_list). Related transform helpers include data_processing.inverse_TFIDF and data_processing.CLR_transform for ADT/other modalities. Downstream translation quality depends on this step, so keep parameters fixed across all methods and folds in a comparison.

## Related tools

- **scButterfly** (Provides Butterfly.load_data and butterfly.data_preprocessing as well as data_processing.RNA_data_preprocessing and data_processing.ATAC_data_preprocessing to load raw paired count matrices and produce model-ready inputs) — https://github.com/BioX-NKU/scButterfly
- **Python 3.9** (Runtime environment for scButterfly installation and execution of the preprocessing pipeline)

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id, validation_id); butterfly.data_preprocessing()
```

## Evaluation signals

- The loaded inputs are raw count matrices (AnnData with cells as rows, genes/peaks as columns), not normalized values
- train/test/validation ID lists are disjoint and validation_id defaults to 20% of train_id when not supplied
- ATAC peaks are sorted by chromosome and chrom_list (peak counts per chromosome) sums to the total number of ATAC features
- Default preprocessing parameters were retained (README strongly recommends defaults for best translation results)
- Preprocessed matrices feed directly into butterfly.construct_model(chrom_list) and butterfly.train_model() without shape errors

## Limitations

- Non-default preprocessing settings are possible but discouraged: 'we strongly recommend the default settings to keep the best result for model'
- Peaks must be sorted by chromosome before construct_model; unsorted peak order will break the chrom_list requirement
- Supporting additional modalities such as spatial transcriptomics and single-cell Hi-C is named as a future direction not yet supported

## Evidence

- [readme] Before data preprocessing, you should load the raw count matrix of scRNA-seq and scATAC-seq data via butterfly.load_data: [evidence span withheld — anchor status: external_doc]
- [readme] Anndata object with rows as cells and columns as genes/peaks: [evidence span withheld — anchor status: external_doc]
- [readme] Default preprocessing settings recommended for best results: [evidence span withheld — anchor status: external_doc]
- [readme] chrom_list must list peak counts per chromosome with peaks sorted: [evidence span withheld — anchor status: external_doc]
- [methods] Article methods name the RNA and ATAC preprocessing functions: "data_processing.RNA_data_preprocessing"
- [readme] Validation ID defaults to 20% of training cells: [evidence span withheld — anchor status: external_doc]
