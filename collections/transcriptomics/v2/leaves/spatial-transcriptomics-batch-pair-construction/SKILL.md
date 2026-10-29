---
name: spatial-transcriptomics-batch-pair-construction
description: Use when you have an ST AnnData with truth coordinates in .obs[['x','y']]
  and need to train a contrastive model whose encoder representations encode cell-cell
  spatial proximity for downstream SC spatial reconstruction.
when_to_use_negative:
- No ST data with truth locations is available — the contrastive pairs must be derived
  from ST data with truth locations, so purely query SC data cannot train the model
- The goal is SC-to-ST mapping/deconvolution or spot enhancement rather than learning
  spatially preserved representations of cell neighborhoods
- Imaging-based and spot-based parameters are confused — default parameters are defined
  for sequencing-based ST; the --single_cell flag must not be omitted for imaging-based
  ST
tools:
- name: CellContrast
  role: Provides the contrastive learning framework (encoder-projector architecture,
    batch pair construction, infoNCE optimization) implemented via the cellContrast.py
    train module
  repo: https://github.com/HKU-BAL/CellContrast
- name: PyTorch
  role: Deep-learning backend used to build the MLP encoder (1,024/512, BatchNorm,
    leaky ReLU) and projection head (256/128, ReLU) and to run SGD with infoNCE loss
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:e2e230afbfae9b409908ff0f1ee8543bd4823a9f95de54cecca035e1e1fde626
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/CellContrast
  tools:
  - CellContrast
  - PyTorch
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# spatial-transcriptomics-batch-pair-construction

## Summary

Trains CellContrast's contrastive encoder-projector model on spatial transcriptomics (ST) data by constructing contrastive batches (anchor–positive–negative cell pairs from true coordinates) and minimizing the infoNCE loss, so that the encoder learns spatially preserved representations for reconstructing single-cell (SC) spatial relationships.

## When to use

Use when you have an ST AnnData with truth coordinates in .obs[['x','y']] and need to train a contrastive model whose encoder representations encode cell-cell spatial proximity for downstream SC spatial reconstruction.

## When NOT to use

- No ST data with truth locations is available — the contrastive pairs must be derived from ST data with truth locations, so purely query SC data cannot train the model
- The goal is SC-to-ST mapping/deconvolution or spot enhancement rather than learning spatially preserved representations of cell neighborhoods
- Imaging-based and spot-based parameters are confused — default parameters are defined for sequencing-based ST; the --single_cell flag must not be omitted for imaging-based ST

## Inputs

- ST training AnnData (.h5ad) with truth cell coordinates in .obs[['x','y']]
- Parameter JSON file (parameters/parameters_spot. for spot-based ST or parameters/parameters_singleCell. for imaging-based ST)

## Outputs

- Trained model checkpoint cellContrast_models/epoch_3000.pt
- Per-epoch infoNCE training loss curve image

## How to apply

Load the ST training AnnData with truth coordinates in .obs[['x','y']] and log-normalize gene expressions with scran. Construct training batches of size n=64: randomly select n/2 anchor cells, pair each anchor with one of its k nearest spatial neighbors as the positive (k=80 for single-cell-resolution SeqFISH/MERSCOPE data, k=20 for spot-based Stereo-seq/Visium), and treat the remaining n-2 batch cells as negatives. Build in PyTorch a two-layer MLP encoder (1,024 and 512 nodes) with BatchNorm and leaky ReLU, plus a two-layer projection head (256, 128) with ReLU on the first layer; train for 3,000 epochs minimizing the infoNCE loss with temperature tau=0.05. Optimize with SGD (momentum=0.9, weight_decay=5e-4) at initial learning rate 0.1 with cosine decay, then save the checkpoint as epoch_3000.pt under cellContrast_models/ and record per-epoch losses for a training loss curve.

## Related tools

- **CellContrast** (Provides the contrastive learning framework (encoder-projector architecture, batch pair construction, infoNCE optimization) implemented via the cellContrast.py train module) — https://github.com/HKU-BAL/CellContrast
- **PyTorch** (Deep-learning backend used to build the MLP encoder (1,024/512, BatchNorm, leaky ReLU) and projection head (256/128, ReLU) and to run SGD with infoNCE loss)

## Examples

```
python cellContrast.py train --train_data_path train_ST.h5ad --save_folder cellContrast_models/ --single_cell --parameter_file parameters/parameters_singleCell.
```

## Evaluation signals

- cellContrast_models/epoch_3000.pt checkpoint exists after training
- Recorded training loss curve shows a monotonically decreasing infoNCE loss, consistent with the paper's finding that sufficient training epochs (>1,000) benefit contrastive learning
- Batch construction is correct: batch size n=64 with n/2 anchors, one kNN positive per anchor (k=80 single-cell / k=20 spot-based), and n-2 negatives
- Training uses the specified hyperparameters: tau=0.05, SGD momentum=0.9, weight_decay=5e-4, initial lr=0.1 with cosine decay over 3,000 epochs
- Encoder and projector dimensions match (encoder 1,024→512, projector 256→128) and the projector is discarded at inference

## Limitations

- Sufficient training epochs are required — the paper notes that more than 1,000 epochs are beneficial for contrastive learning
- A large gap between ST reference and SC query data can lead to a performance drop in global reconstruction, and forcing SC assignment to resemble a mismatched ST can introduce bias
- Parameter defaults are defined for sequencing-based ST; using them on imaging-based ST (or vice versa) without the --single_cell flag / singleCell parameter file will misconfigure the kNN neighborhood (k=20 vs k=80)

## Evidence

- [readme] CellContrast model was trained based on ST data (which should be in [AnnData](https://anndata.readthedocs.io/en/latest/) format, with truth locations in `.obs[['x','y']]`): "CellContrast model was trained based on ST data (which should be in [AnnData](https://anndata.readthedocs.io/en/latest/) format, with truth locations in `.obs[['x','y']]`)"
- [discussion] constructing training batches containing contrastive pairs, derived from ST data with truth locations and optimizing them using the infoNCE loss: "constructing training batches containing contrastive pairs, derived from ST data with truth locations and optimizing them using the infoNCE loss"
- [intro] we introduced CellContrast, a deep-learning method that employs a contrastive learning framework for spatial relationship reconstruction of SC data: "we introduced CellContrast, a deep-learning method that employs a contrastive learning framework for spatial relationship reconstruction of SC data"
- [intro] During inference, we discarded the projector and used the output of the encoder for spatial relationship reconstruction: "During inference, we discarded the projector and used the output of the encoder for spatial relationship reconstruction"
