---
name: contrastive-learning-model-training
description: Use when you have an ST reference AnnData (e.g., 10X Visium or Stereo-seq
  spots) with truth coordinates in .obs[['x','y']] and need spatially preserved representations
  for query single-cell data lacking coordinates.
when_to_use_negative:
- The ST reference lacks truth per-spot/per-cell coordinates in .obs[['x','y']], since
  contrastive pairs cannot be derived without known locations
- Your data is query single-cell data only with no ST reference to train on (CellContrast
  requires an ST reference with truth locations)
- You only need SC-to-ST cell-type mapping with assignment probabilities rather than
  local spatial-relationship reconstruction
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellContrast
  role: Provides the train module (python cellContrast.py train) that builds contrastive
    batches, runs the encoder-projector model, and saves epoch_3000.pt
  repo: https://github.com/HKU-BAL/CellContrast
- name: PyTorch
  role: Implements the two-layer MLP encoder (1,024/512) and projection head (256/128)
    and the SGD optimizer with infoNCE loss (tau=0.05)
  repo: https://pytorch.org/get-started/locally/
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:58b7521ec845bc742857fc80836d36768c5933c144c639a293ca57ba54ee90d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/CellContrast
  tools:
  - CellContrast
  - PyTorch
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# contrastive-learning-model-training

## Summary

Trains CellContrast's encoder-projector contrastive model on spatial transcriptomics reference data with truth coordinates, using infoNCE loss on positive pairs of spatially proximate cells, to learn a latent space where cosine similarity reflects physical proximity for downstream single-cell spatial reconstruction.

## When to use

Use when you have an ST reference AnnData (e.g., 10X Visium or Stereo-seq spots) with truth coordinates in .obs[['x','y']] and need spatially preserved representations for query single-cell data lacking coordinates.

## When NOT to use

- The ST reference lacks truth per-spot/per-cell coordinates in .obs[['x','y']], since contrastive pairs cannot be derived without known locations
- Your data is query single-cell data only with no ST reference to train on (CellContrast requires an ST reference with truth locations)
- You only need SC-to-ST cell-type mapping with assignment probabilities rather than local spatial-relationship reconstruction

## Inputs

- ST reference AnnData (.h5ad) with truth cell/spot coordinates in .obs[['x','y']] (e.g., 10X Visium human breast sample S06, 3,037 spots)
- Log-normalized (scran) gene expression matrix, all genes retained (e.g., 36,503 genes)
- Parameter file (parameters/parameters_spot. for sequencing-based ST, parameters/parameters_singleCell. for imaging-based ST via --single_cell)

## Outputs

- Model checkpoint cellContrast_models/epoch_3000.pt
- Per-epoch infoNCE loss records (training loss curve)
- Encoder representations of query SC cells used downstream for neighbor identification and spatial inference (spatial_reconstructed_sc.h5ad)

## How to apply

Load the ST training AnnData and log-normalize gene expressions (scran), keeping all genes (e.g., 36,503 for the 10X Visium human breast samples) as input. Construct batches of n=64: pick n/2 anchor cells and pair each with one of its k nearest spatial neighbors as the positive (k=20 for spot-based Visium/Stereo-seq, k=80 for single-cell-resolution SeqFISH/MERSCOPE), treating remaining batch cells as negatives. Build in PyTorch a two-layer MLP encoder (1,024, 512 nodes; BatchNorm + leaky ReLU) and a two-layer projection head (256, 128; ReLU on first layer), then train 3,000 epochs minimizing the infoNCE loss with temperature tau=0.05 via SGD (momentum=0.9, weight_decay=5e-4, initial lr 0.1 with cosine decay), saving to cellContrast_models/epoch_3000.pt. At inference, discard the projection head and use encoder outputs, where higher cosine similarity indicates shorter spatial distance; >1,000 training epochs are beneficial for the contrastive learning.

## Related tools

- **CellContrast** (Provides the train module (python cellContrast.py train) that builds contrastive batches, runs the encoder-projector model, and saves epoch_3000.pt) — https://github.com/HKU-BAL/CellContrast
- **PyTorch** (Implements the two-layer MLP encoder (1,024/512) and projection head (256/128) and the SGD optimizer with infoNCE loss (tau=0.05)) — https://pytorch.org/get-started/locally/

## Examples

```
python cellContrast.py train --train_data_path train_ST.h5ad --save_folder cellContrast_models/ --parameter_file parameters/parameters_spot.
```

## Evaluation signals

- cellContrast_models/epoch_3000.pt exists after training completes
- Recorded per-epoch infoNCE loss curve shows a decreasing trend over the 3,000 epochs
- Held-out benchmark: on 10X Visium human breast S06 (train) -> S08 (test), neighbor hit at k=200 reaches ~58.556 true neighbors and mean neighborhood JSD ~0.306 (beating SageNet 0.369 and NovoSpaRc 0.393)
- The same parameter file (spot vs. single-cell, k=20 vs. k=80) used in training is reused for eval and inference
- Post-inference, higher cosine similarity of learnt representations corresponds to shorter spatial distance between cells

## Limitations

- Tissues with a high level of repeating elements could pose a challenge for spatial reconstruction of SC cells
- A large gap between ST reference and query SC data leads to a performance drop in global reconstruction across all methods, and forcing SC assignment to resemble ST could introduce more bias
- Sufficient training epochs (>1,000) are needed; contrastive learning under fewer epochs underperforms

## Evidence

- [intro] In this study, we introduced CellContrast, a deep-learning method that employs a contrastive framework for spatial relationship reconstruction of SC data.: [evidence span withheld — anchor status: not_found]
- [discussion] constructing training batches containing contrastive pairs, derived from ST data with truth locations and optimizing them using the infoNCE loss: "constructing training batches containing contrastive pairs, derived from ST data with truth locations and optimizing them using the infoNCE loss"
- [intro] During inference, we discarded the projector and used the output of the encoder for spatial relationship reconstruction: "During inference, we discarded the projector and used the output of the encoder for spatial relationship reconstruction, based on the principle that higher cosine similarity indicates shorter spatial"
- [readme] CellContrast model was trained based on ST data (which should be in AnnData format, with truth locations in .obs[['x','y']]): "CellContrast model was trained based on ST data (which should be in [AnnData](https://anndata.readthedocs.io/en/latest/) format, with truth locations in `.obs[['x','y']])`."
- [readme] Default parameters are defined for sequencing-based ST, adopt the predefined parameters for imaging-based ST data by setting --single_cell.: ":bangbang: Default parameters are defined for sequencing-based ST, adopt the predefined parameters for imaging-based ST data by setting `--single_cell`."
