---
name: gpu-deep-learning-training-and-inference
description: Use when a segmentation mask assigning transcripts to individual cells
  is needed from SST imaging data (e.g., Xenium transcripts.csv.gz + morphology_mip.ome.tif)
  and a ≥12GB VRAM GPU is available to train the UNet3+ model.
when_to_use_negative:
- You intend to reuse a model pretrained on other imaging modalities (e.g., Cellpose)
  directly for SST segmentation without retraining — the article shows such pretrained
  models do not transfer well to SST images.
- No GPU with ≥12GB VRAM is available and the full-size dataset is being processed
  — the deep learning component is strongly GPU-dependent and higher-resolution segmentation
  needs larger patch sizes and more GPU memory.
- Nuclei images are separated into fields of view (e.g., CosMx) — model.stitch_nuclei()
  must be run before training, so plain train/predict alone is insufficient.
tools:
- name: BIDCell
  role: Provides the biologically-informed self-supervised UNet3+ training and inference
    pipeline (BIDCellModel.train() and .predict())
  repo: https://github.com/SydneyBioX/BIDCell
- name: Cellpose
  role: Pretrained model used during preprocessing for initial nuclei segmentation
    from the DAPI image, feeding patches for training
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:b7e9e605af012d08753a32311144eff87ce7aa32bc04078698ea0b7ae7af5b8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - BIDCell
  - Cellpose
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gpu-deep-learning-training-and-inference

## Summary

Train and run inference with BIDCell's biologically-informed UNet3+ segmentation network on GPU to produce cell segmentation masks and cell-gene matrices for subcellular spatial transcriptomics (SST) data such as 10x Genomics Xenium. BIDCell uses self-supervised training with biologically-informed loss functions because manual pixel annotation is unattainable for SST data.

## When to use

Use when a segmentation mask assigning transcripts to individual cells is needed from SST imaging data (e.g., Xenium transcripts.csv.gz + morphology_mip.ome.tif) and a ≥12GB VRAM GPU is available to train the UNet3+ model.

## When NOT to use

- You intend to reuse a model pretrained on other imaging modalities (e.g., Cellpose) directly for SST segmentation without retraining — the article shows such pretrained models do not transfer well to SST images.
- No GPU with ≥12GB VRAM is available and the full-size dataset is being processed — the deep learning component is strongly GPU-dependent and higher-resolution segmentation needs larger patch sizes and more GPU memory.
- Nuclei images are separated into fields of view (e.g., CosMx) — model.stitch_nuclei() must be run before training, so plain train/predict alone is insufficient.

## Inputs

- xenium_example_config.yaml (or cosmx/merscope/stereoseq example config) with correct paths set under 'files'
- Quality-filtered transcripts file (transcripts.csv.gz) from the Xenium Output Bundle In Situ Replicate 1
- DAPI image (morphology_mip.ome.tif)
- Breast cancer single-cell reference (sc_breast.csv) with sc_breast_markers_pos.csv and sc_breast_markers_neg.csv
- Nuclei segmentation produced by model.segment_nuclei() during preprocessing

## Outputs

- Segmentation mask *_connected.tif where pixel values correspond to cell IDs (~103,209 cells on Xenium-BreastCancer1)
- Cell-gene expression matrix expr_mat.csv
- Trained BIDCell UNet3+ model checkpoint

## How to apply

Create a Python>=3.9,<3.13 conda environment and pip install bidcell; a GPU with at least 12GB VRAM is strongly recommended and 32GB RAM for data processing. After running model.preprocess() (Cellpose-based nuclei segmentation from the DAPI image, expression maps, patches, cell-gene matrices, and pre-annotation with a single-cell reference like sc_breast.csv), call model.train() — about 10 minutes for 4,000 steps on a 12GB GPU — then model.predict() (~50 minutes) plus postprocessing (~30 minutes) to obtain the segmentation mask. Keep the default UNet3+ architecture unless you deliberately set model_params.name to another encoder from segmentation-models-pytorch. On multi-GPU machines, restrict training to one device via CUDA_VISIBLE_DEVICES (e.g., GPU ID 3), and tune the yaml 'cpus' parameter to your system: higher cpus speed preprocessing but can freeze the machine, and if you see pickle.UnpicklingError: pickle data was truncated, reduce cpus. Training is self-supervised, so no manual ground-truth annotations are required — biological priors come from the single-cell reference and positive/negative marker files instead.

## Related tools

- **BIDCell** (Provides the biologically-informed self-supervised UNet3+ training and inference pipeline (BIDCellModel.train() and .predict())) — https://github.com/SydneyBioX/BIDCell
- **Cellpose** (Pretrained model used during preprocessing for initial nuclei segmentation from the DAPI image, feeding patches for training)

## Examples

```
from bidcell import BIDCellModel; model = BIDCellModel('xenium_example_config.yaml'); model.train(); model.predict()
```

## Evaluation signals

- Count unique cell IDs in the *_connected.tif mask — expect ~103,209 cells when run on Xenium-BreastCancer1
- Compute the fraction of transcripts assigned to cells — expect ~53.4% on Xenium-BreastCancer1
- Training converges in roughly 4,000 steps (~10 minutes on a 12GB GPU); predict takes ~50 minutes plus ~30 minutes postprocessing
- The expr_mat.csv cell-gene matrix is produced and its cell IDs correspond to the mask values
- No pickle.UnpicklingError (pickle data was truncated) occurs; if it does, cpus is too high for the system

## Limitations

- A GPU with at least 12GB VRAM is strongly recommended and 32GB RAM for data processing; higher 'cpus' values allow faster runtimes but may freeze the system.
- Performing segmentation at a higher resolution requires a larger patch size and therefore more GPU memory.
- Models pretrained on other imaging modalities (e.g., Cellpose) do not transfer well to SST images, so training on the target data is necessary rather than transfer-only inference.
- Analysis is two-dimensional, so it may give limited representation of cells' three-dimensional structure, and varied/elongated morphologies (melanocytes, fibroblasts, dendrites, axons) remain challenging without nearby nuclei.

## Evidence

- [readme] GPU and memory requirements for training/inference: "A GPU with at least 12GB VRAM is strongly recommended for the deep learning component, and 32GB RAM for data processing."
- [readme] Default segmentation architecture: "The default is UNet3+ https://arxiv.org/abs/2004.08790, and we have found it to perform well across different technologies and tissue types."
- [readme] Multi-GPU device selection: "If your machine/server has multiple GPUs, you may select the GPU to use, e.g. for GPU ID 3:"
- [abstract] Self-supervised rationale for training without labels: "we present BIDCell, a self-supervised deep learning-based framework with biologically-informed loss functions that learn relationships between spatially resolved gene expression and cell morphology"
- [readme] Pipeline stage API for train and predict: "model.preprocess()
model.train()
model.predict()"
- [readme] Resolution/GPU memory trade-off: "Performing segmentation at a higher resolution requires a larger patch size, thus more GPU memory."
