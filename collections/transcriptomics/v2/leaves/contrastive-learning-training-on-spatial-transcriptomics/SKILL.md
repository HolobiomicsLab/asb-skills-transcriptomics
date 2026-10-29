---
name: contrastive-learning-training-on-spatial-transcriptomics
description: Use when you have an ST dataset with truth locations in .obs[['x','y']]
  (AnnData h5ad) and want to reconstruct spatial relationships or neighbor structure
  of query single-cell RNA-seq data.
when_to_use_negative:
- When the tissue has a high level of repeating elements, which the paper notes 'could
  pose a challenge for the spatial reconstruction of SC cells'
- When there is a large gap between query SC data and the ST reference and you require
  the reference's cell distribution — forced SC assignment 'could introduce more bias';
  consider the MDS-based denovo mode instead, accepting that it loses reference pre-knowledge
- When you need genome-wide spatial resolution directly from ST data rather than transferring
  spatial information from a reference (CellContrast reconstructs relationships for
  SC data using an ST reference, not an ST enhancement method)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: CellContrast
  role: 'Core tool: provides train/eval/inference/reconstruct modules implementing
    the encoder-projector infoNCE contrastive training and cosine-similarity-based
    spatial reconstruction'
  repo: https://github.com/HKU-BAL/CellContrast
- name: PyTorch
  role: Deep-learning backend for the contrastive encoder-projector model (SGD optimizer,
    cosine-decayed learning rate)
- name: scran
  role: Log-normalization of the SeqFISH count data before training
- name: Scipy
  role: jensenshannon function used for the JSD evaluation metric downstream of training
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:dbdddf7639c7e90117b7873bc257eb484dcee34b465963a51b48b3f6e0d92d6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/CellContrast
  tools:
  - CeLEry
  - CellContrast
  - CytoSpace
  - NovoSpaRc
  - PyTorch
  - SageNet
  - Scipy
  - Tangram
  - scran
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# contrastive-learning-training-on-spatial-transcriptomics

## Summary

Train CellContrast's encoder-projector model on spatial transcriptomics (ST) data with truth cell locations using infoNCE contrastive learning, so that physically proximate cells show higher cosine similarity in latent space. At inference the projector is discarded and the encoder representation supports SC-SC neighbor identification and SC-ST spatial mapping.

## When to use

Use when you have an ST dataset with truth locations in .obs[['x','y']] (AnnData h5ad) and want to reconstruct spatial relationships or neighbor structure of query single-cell RNA-seq data.

## When NOT to use

- When the tissue has a high level of repeating elements, which the paper notes 'could pose a challenge for the spatial reconstruction of SC cells'
- When there is a large gap between query SC data and the ST reference and you require the reference's cell distribution — forced SC assignment 'could introduce more bias'; consider the MDS-based denovo mode instead, accepting that it loses reference pre-knowledge
- When you need genome-wide spatial resolution directly from ST data rather than transferring spatial information from a reference (CellContrast reconstructs relationships for SC data using an ST reference, not an ST enhancement method)

## Inputs

- ST training AnnData h5ad with truth locations in .obs[['x','y']] (e.g., SeqFISH embryo 1 L1, 10,150 cells, 351 genes; or Stereo-seq MOSTA E9.5_E2S1, 5,266 spots, 348 genes)
- Query single-cell h5ad file with truth coordinates for evaluation
- Parameter JSON file (parameters/parameters_spot.json or parameters/parameters_singleCell.json)

## Outputs

- Trained encoder checkpoint saved to cellContrast_models/epoch_3000.pt
- results.csv with per-sample nearest neighbor hit (k = 20, 40, ..., 200), JSD, and Spearman's rank correlation
- Spatially reconstructed SC h5ad adding .uns[['cosine sim of rep','representation','referenced x','referenced y','de novo x','de novo y']]

## How to apply

Prepare the ST training data as AnnData h5ad with truth locations in .obs[['x','y']], log-normalized with scran. Choose the parameter file by ST modality: parameters/parameters_spot.json for sequencing-based (e.g., Stereo-seq) ST, parameters/parameters_singleCell.json for imaging-based ST — the latter is selected via the --single_cell flag. Training constructs contrastive pairs from nearest spatial neighbors of the ST reference (k=20 for spot-based defaults; k=80 was used for the SeqFISH embryo 1 L1 benchmark), optimizes infoNCE loss (temperature 0.05, batch size 64) with SGD (momentum 0.9, weight_decay 5e-4, initial lr 0.1 with cosine decay) for 3,000 epochs, and saves the checkpoint as cellContrast_models/epoch_3000.pt. At inference, discard the projector and use encoder outputs: higher cosine similarity of learnt representations indicates shorter cell-cell distance, identifying candidate neighbors; SC-ST mapping with a weighted similarity metric then places query cells at reference coordinates, optionally with MDS (--enable_denovo) when the SC-ST gap is large.

## Related tools

- **CellContrast** (Core tool: provides train/eval/inference/reconstruct modules implementing the encoder-projector infoNCE contrastive training and cosine-similarity-based spatial reconstruction) — https://github.com/HKU-BAL/CellContrast
- **PyTorch** (Deep-learning backend for the contrastive encoder-projector model (SGD optimizer, cosine-decayed learning rate))
- **scran** (Log-normalization of the SeqFISH count data before training)
- **Scipy** (jensenshannon function used for the JSD evaluation metric downstream of training)

## Examples

```
python cellContrast.py train --train_data_path train_ST.h5ad --save_folder cellContrast_models/ --single_cell --parameter_file parameters/parameters_singleCell.json
```

## Evaluation signals

- On SeqFISH embryo 1 L1 reference: median average neighbor hit ratio near 47.32% at k = 20..200, vs 36.22% SageNet and 14.27% CeLEry
- On embryo 1 L2 test: average JSD near 0.237 and Spearman's rho near 0.874 between predicted and true pairwise distances
- Cross-platform (Stereo-seq E9.5 -> SeqFISH E8.5): mean average hit number ~55.388 true neighbors at k=200, mean JSD ~0.305, mean Spearman's rho ~0.283
- Checkpoint file cellContrast_models/epoch_3000.pt exists after the 3,000-epoch run; results.csv contains neighbor hit, JSD, and Spearman's rho rows for each test sample
- Eval command must reuse the same parameter file (spot vs singleCell) as training; mismatched parameter files invalidate the benchmark

## Limitations

- SC-ST mapping introduces a 'cell clumping' issue where multiple neighbors are projected into the same reference location, losing predicted SC-SC distances
- Large gaps between ST and SC samples lead to a performance drop in global reconstruction across all methods, and forcing SC assignment to resemble ST may introduce more bias
- MDS-based (denovo) mapping is more robust to SC-ST gaps but loses the pre-knowledge of the reference's cell distribution
- A certain level of mismatch may exist between query SC data and ST reference due to limited ST resources
- Default training parameters are defined for sequencing-based ST; imaging-based ST requires the --single_cell predefined parameters

## Evidence

- [abstract] abstract_introduction: "we introduce CellContrast, a computational method for reconstructing spatial relationships among SC cells from spatial transcriptomics (ST) reference"
- [discussion] training_procedure: "constructing training batches containing contrastive pairs, derived from ST data with truth locations and optimizing them using the infoNCE loss"
- [intro] inference_principle: "During inference, we discarded the projector and used the output of the encoder for spatial relationship reconstruction, based on the principle that higher cosine similarity indicates shorter spatial"
- [discussion] cosine_neighbors: "The resulting cell-cell distances are estimated through cosine similarities of learnt representations and therefore identifying the potential cell neighbors."
- [readme] parameter_file_flag: "Default parameters are defined for sequencing-based ST, adopt the predefined parameters for imaging-based ST data by setting `--single_cell`."
- [readme] annadata_coords: "CellContrast model was trained based on ST data (which should be in [AnnData](https://anndata.readthedocs.io/en/latest/) format, with truth locations in `.obs[['x','y']])`"
