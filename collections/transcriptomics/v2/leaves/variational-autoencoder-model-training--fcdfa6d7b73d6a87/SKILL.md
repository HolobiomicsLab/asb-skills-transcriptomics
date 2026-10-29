---
name: variational-autoencoder-model-training
description: Use when paired (or unpaired) scRNA-seq and scATAC-seq raw count matrices
  are available and a cross-modality translator must be trained to infer one modality
  from the other, e.g. for annotation or imputation.
when_to_use_negative:
- The target modality or analysis context is spatial transcriptomics or single-cell
  Hi-C — the article names these only as future extensions not yet supported
- Only a precomputed feature/embedding table is available — scButterfly requires raw
  count matrices for load_data and its TF-IDF/CLR preprocessing
- A dedicated scATAC-seq annotator (e.g. EpiAnno, Cellcano) already runs on the native
  ATAC data and translation-based annotation is not the goal
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3168
tools:
- name: scButterfly
  role: Provides the Butterfly API (load_data, data_preprocessing, augmentation, construct_model,
    train_model, test_model) implementing the dual-aligned variational autoencoder
    translator
  repo: https://github.com/BioX-NKU/scButterfly
- name: Python 3.9
  role: Runtime environment for scButterfly, installed in a dedicated conda environment
    (conda create scButterfly python==3.9)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:87afdf33e13db962c4051ce2ab4066240342290b0926d038f9ec50a95e034856
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - Python
  - Python 3.9
  - scButterfly
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# variational-autoencoder-model-training

## Summary

Train scButterfly's dual-aligned variational autoencoder translator to map single-cell profiles between scRNA-seq and scATAC-seq (or scADT-seq), using data augmentation (scButterfly-C via MultiVI) to improve translation from limited paired multi-omics data.

## When to use

Use when paired (or unpaired) scRNA-seq and scATAC-seq raw count matrices are available and a cross-modality translator must be trained to infer one modality from the other, e.g. for annotation or imputation.

## When NOT to use

- The target modality or analysis context is spatial transcriptomics or single-cell Hi-C — the article names these only as future extensions not yet supported
- Only a precomputed feature/embedding table is available — scButterfly requires raw count matrices for load_data and its TF-IDF/CLR preprocessing
- A dedicated scATAC-seq annotator (e.g. EpiAnno, Cellcano) already runs on the native ATAC data and translation-based annotation is not the goal

## Inputs

- Raw count AnnData object of scRNA-seq data (cells x genes)
- Raw count AnnData object of scATAC-seq data (cells x peaks, sorted by chromosome)
- Lists of cell IDs: train_id, test_id, and optional validation_id
- chrom_list: list of peak counts per chromosome
- Optional cell_type labels in adata.obs (scButterfly-T) for augmentation

## Outputs

- Trained dual-aligned VAE translator (model checkpoints in './model' by default)
- Translated profiles via Butterfly.test_model returning A2R_predict and R2A_predict

## How to apply

Instantiate a Butterfly object and load raw count matrices with Butterfly.load_data, passing AnnData objects (n_obs x n_vars for genes and peaks) plus train_id, test_id and optional validation_id lists. Preprocess with Butterfly.data_preprocessing, which internally applies RNA_data_preprocessing, ATAC_data_preprocessing, inverse_TFIDF and CLR_transform; keep the default settings as the README recommends them for best results. Optionally call Butterfly.augmentation with aug_type='MultiVI_augmentation' (scButterfly-C, which trains a MultiVI model to get cluster labels) or 'cell_type_augmentation' (scButterfly-T, which uses 'cell_type' in adata.obs) to expand the training split with synthetic samples. Build the model with Butterfly.construct_model(chrom_list), supplying a list of per-chromosome peak counts with peaks sorted by chromosome, then train with Butterfly.train_model (optionally setting output_path for checkpoints and load_model for pretrained weights). The dual-aligned VAE design with masked per-modality encoders and data augmentation alleviates limited paired cell numbers and preserves cellular heterogeneity during translation.

## Related tools

- **scButterfly** (Provides the Butterfly API (load_data, data_preprocessing, augmentation, construct_model, train_model, test_model) implementing the dual-aligned variational autoencoder translator) — https://github.com/BioX-NKU/scButterfly
- **Python 3.9** (Runtime environment for scButterfly, installed in a dedicated conda environment (conda create scButterfly python==3.9))

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id); butterfly.data_preprocessing(); butterfly.augmentation('MultiVI_augmentation'); butterfly.construct_model(chrom_list); butterfly.train_model()
```

## Evaluation signals

- Butterfly.test_model returns both A2R_predict and R2A_predict; verify translated profiles match the expected dimensions of the target modality
- For annotation use, compute accuracy, Cohen's kappa, F1-macro and F1-weighted with calculate_cluster_index and check consistency with the EpiAnno/Cellcano comparison (Fig. 4d,e) and pbmc_annotation_metrics.csv
- Confirm peaks were sorted by chromosome before construct_model, since chrom_list must give per-chromosome peak counts
- Augmentation with MultiVI (scButterfly-C) should improve prediction over the basic model (scButterfly-B) — README notes augmentation 'will cause more training time used, but promise better result for predicting'
- Training/validation split respected: validation_id defaults to 20% of train_id cells if not provided

## Limitations

- Performance for cross-cell-type and cross-organ translation is flagged as needing improvement via advanced machine learning techniques with adaptive and interpretable model parameters
- Incorporation of cell heterogeneity information from public bulk omics/annotated cell atlas, prior gene regulatory knowledge, spatial transcriptomics, and single-cell Hi-C are future directions not implemented
- Augmentation increases training time; scButterfly-T silently falls back to MultiVI_augmentation if no 'cell_type' column is found in adata.obs
- Peaks must be sorted by chromosome for construct_model, otherwise chrom_list will be incorrect

## Evidence

- [abstract] dual-aligned VAE with augmentation: "we propose scButterfly, a versatile single-cell cross-modality translation method based on dual-aligned variational autoencoders and data augmentation schemes."
- [readme] load_data signature: [evidence span withheld — anchor status: external_doc]
- [readme] augmentation types: [evidence span withheld — anchor status: external_doc]
- [readme] chrom_list requirement: [evidence span withheld — anchor status: external_doc]
- [readme] default preprocessing: [evidence span withheld — anchor status: external_doc]
- [readme] test_model returns: [evidence span withheld — anchor status: external_doc]
- [discussion] future performance caveat: "advanced machine learning techniques with adaptive and interpretable model parameters can be considered to explore novel avenues for boosting the performance of scButterfly"
- [methods] environment setup: "conda create scButterfly python==3.9"
