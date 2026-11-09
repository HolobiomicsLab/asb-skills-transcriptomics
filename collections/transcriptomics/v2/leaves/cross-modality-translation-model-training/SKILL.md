---
name: cross-modality-translation-model-training
description: Use when paired single-cell count matrices (e.g., scRNA-seq genes and
  scATAC-seq peaks as AnnData) are available and a held-out single-modality profile
  must be translated into the other modality while preserving cellular heterogeneity.
when_to_use_negative:
- The two modalities are completely unpaired at the cell level with no paired data
  available for training — the paired dual-alignment workflow here assumes paired
  measurements (unpaired/diagonal training is a distinct variant)
- Input data is already normalized/processed rather than raw count matrices — scButterfly's
  preprocessing expects raw counts and applies its own RNA normalization and TF-IDF/ATAC
  handling
- The goal is translation into a modality beyond scRNA-seq/scATAC-seq pairs such as
  spatial transcriptomics or single-cell Hi-C — the article names these as future
  directions not yet supported
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scButterfly
  role: Core package providing Butterfly.load_data, data_preprocessing, augmentation,
    construct_model, train_model, and test_model for the dual-aligned VAE cross-modality
    translator
  repo: https://github.com/BioX-NKU/scButterfly
- name: MultiVI
  role: scvi-tools model trained by scButterfly-C (MultiVI_augmentation) to derive
    cluster labels used for synthetic-sample data augmentation
  repo: https://docs.scvi-tools.org/en/stable/tutorials/notebooks/MultiVI_tutorial.html
- name: Python 3.9
  role: Runtime environment (conda env with python==3.9) in which scButterfly is installed
    from PyPI or the GitHub wheel
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47418-x
    title: 'scButterfly: a versatile single-cell cross-modality translation method
      via dual-aligned variational autoencoders'
  source_content_hash: sha256:3653b2a89b457858d310d7ca04ad03046f8dd87de55e7ce4cf34227dd1c40490
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioX-NKU/scButterfly
  tools:
  - BABEL
  - JAMIE
  - MultiVI
  - Polarbear
  - PyTorch
  - Python
  - Python 3.9
  - scButterfly
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cross-modality-translation-model-training

## Summary

Train a dual-aligned variational autoencoder (scButterfly) that translates single-cell profiles between modalities (e.g., scRNA-seq and scATAC-seq) from raw paired count matrices, with optional data augmentation via cell-type labels (scButterfly-T) or MultiVI cluster labels (scButterfly-C).

## When to use

Use when paired single-cell count matrices (e.g., scRNA-seq genes and scATAC-seq peaks as AnnData) are available and a held-out single-modality profile must be translated into the other modality while preserving cellular heterogeneity.

## When NOT to use

- The two modalities are completely unpaired at the cell level with no paired data available for training — the paired dual-alignment workflow here assumes paired measurements (unpaired/diagonal training is a distinct variant)
- Input data is already normalized/processed rather than raw count matrices — scButterfly's preprocessing expects raw counts and applies its own RNA normalization and TF-IDF/ATAC handling
- The goal is translation into a modality beyond scRNA-seq/scATAC-seq pairs such as spatial transcriptomics or single-cell Hi-C — the article names these as future directions not yet supported

## Inputs

- Raw scRNA-seq count matrix (AnnData, n_obs x n_vars, cells x genes)
- Raw scATAC-seq count matrix (AnnData, n_obs x n_vars, cells x peaks)
- train_id list of cell IDs for training
- test_id list of cell IDs for testing
- validation_id list of cell IDs (optional; defaults to 20% of train_id)
- chrom_list: list of peak counts per chromosome, peaks sorted by chromosome
- Optional: cell_type column in adata.obs for cell_type_augmentation

## Outputs

- Trained scButterfly model checkpoint (saved under './model' output_path by default)
- A2R_predict: translated ATAC-to-RNA profiles for held-out cells
- R2A_predict: translated RNA-to-ATAC profiles for held-out cells
- Optional processed-data files and process/model/training logging files

## How to apply

Load the raw count matrices with Butterfly.load_data, passing train_id, test_id, and validation_id cell lists (validation defaults to 20% of train_id if unset). Apply butterfly.data_preprocessing with default settings, which includes RNA normalization and ATAC peak handling with intra-chromosomal connection pruning (peaks must be sorted by chromosome for construct_model's chrom_list). Optionally call butterfly.augmentation with 'cell_type_augmentation' (scButterfly-T, requires cell_type in adata.obs) or 'MultiVI_augmentation' (scButterfly-C, trains MultiVI from scvi-tools); use aug_type=None for scButterfly-B. Construct the fixed seven-module network via butterfly.construct_model(chrom_list) — RNA and ATAC variational encoders with 50%/30% input masking, a Gaussian-sampling translator, two sigmoid discriminators, and a Sigmoid decoder output for ATAC — then run butterfly.train_model with the pretraining-then-integrative loss-weight schedule. Predict both directions with butterfly.test_model, which returns A2R_predict and R2A_predict and can integrate evaluation.

## Related tools

- **scButterfly** (Core package providing Butterfly.load_data, data_preprocessing, augmentation, construct_model, train_model, and test_model for the dual-aligned VAE cross-modality translator) — https://github.com/BioX-NKU/scButterfly
- **MultiVI** (scvi-tools model trained by scButterfly-C (MultiVI_augmentation) to derive cluster labels used for synthetic-sample data augmentation) — https://docs.scvi-tools.org/en/stable/tutorials/notebooks/MultiVI_tutorial.html
- **Python 3.9** (Runtime environment (conda env with python==3.9) in which scButterfly is installed from PyPI or the GitHub wheel)

## Examples

```
from scButterfly.butterfly import Butterfly; butterfly = Butterfly(); butterfly.load_data(RNA_data, ATAC_data, train_id, test_id); butterfly.data_preprocessing(); butterfly.augmentation('MultiVI_augmentation'); butterfly.construct_model(chrom_list); butterfly.train_model(); A2R_predict, R2A_predict = butterfly.test_model()
```

## Evaluation signals

- Confirmed the trained model checkpoint exists (e.g., under the train_model output_path './model') and butterfly.test_model runs end-to-end producing both A2R_predict and R2A_predict
- Cluster translated profiles with calculate_cluster.calculate_cluster_index and check AMI, ARI, HOM, and NMI against ground-truth cell labels improve over baselines (BABEL, Polarbear, JAMIE) in five-fold cross-validation
- One-sided paired Wilcoxon signed-rank tests over per-fold metrics show significant advantage of scButterfly-B over each baseline (n=140 across 7 datasets x 5 folds x 2 directions)
- At limited training-cell fractions (20%-100%), scButterfly-T and scButterfly-C clustering metrics exceed scButterfly-B, with the three-fold augmentation ratio confirmed as optimal
- Preprocessing used default settings (strongly recommended by the README) and peaks were sorted by chromosome before passing chrom_list to construct_model

## Limitations

- Encoder masking (50% RNA, 30% ATAC) and inter-chromosomal connection pruning address dropout noise but performance for cross-cell-type and cross-organ translation is flagged as needing improvement via advanced machine learning with adaptive and interpretable model parameters
- Cell-type-augmented training (scButterfly-T) silently falls back to MultiVI_augmentation when cell_type is absent from adata.obs
- Extensions to spatial transcriptomics and single-cell Hi-C are future directions; incorporation of bulk-omics/atlas heterogeneity and gene-regulatory prior knowledge is proposed but not implemented
- Data augmentation and MultiVI training increase training time, though they promise better prediction results

## Evidence

- [abstract] Here, we propose scButterfly, a versatile single-cell cross-modality translation method based on dual-aligned variational autoencoders and data augmentation schemes.: "Here, we propose scButterfly, a versatile single-cell cross-modality translation method based on dual-aligned variational autoencoders and data augmentation schemes."
- [methods] scButterfly introduces a masking strategy for the encoders: [evidence span withheld — anchor status: not_found]
- [methods] butterfly.load_data: [evidence span withheld — anchor status: not_found]
- [readme] a list of peaks count for each chromosome: [evidence span withheld — anchor status: external_doc]
- [readme] aug_type from cell_type_augmentation or MultiVI_augmentation: [evidence span withheld — anchor status: external_doc]
