---
name: self-supervised-segmentation-model-training
description: Use when segmenting individual cells from subcellular spatial transcriptomics
  data (e.g. Xenium transcripts.csv.gz + morphology_mip.ome.tif) without ground-truth
  labels, and when evaluating loss-component contributions via ablation.
when_to_use_negative:
- Nuclei images are separated into fields of view (e.g. CosMx) — use model.stitch_nuclei()
  before preprocessing instead of the standard segmentation path
- You already have a curated cell-by-gene expression matrix and only need evaluation,
  not segmentation model training
- The task is transcript-only, segmentation-free clustering (e.g. Baysor-style) rather
  than image-and-transcript segmentation with morphology
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: BIDCell
  role: Self-supervised deep learning framework providing BIDCellModel with preprocess/train/predict
    stages and the six biologically-informed loss functions
  repo: https://github.com/SydneyBioX/BIDCell
- name: 10x Genomics Xenium
  role: Source platform for the Xenium-BreastCancer1 training data (transcripts.csv.gz
    and morphology_mip.ome.tif)
- name: SydneyBioX/scClassify
  role: Source of example single-cell reference data for cell-type-informed losses
  repo: https://github.com/SydneyBioX/scClassify
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:a43bdff514bf6908eaa3db610957cbc11b7f6e2fa78cb6b7f0674ac64513417d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - BIDCell
  - SydneyBioX/scClassify
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# self-supervised-segmentation-model-training

## Summary

Train BIDCell's self-supervised UNet3+-based segmentation model on Xenium (or CosMx/MERSCOPE/Stereo-seq) patches using six biologically-informed loss functions that combine spatial gene expression with cell morphology, replacing the need for ground-truth manual annotations.

## When to use

Use when segmenting individual cells from subcellular spatial transcriptomics data (e.g. Xenium transcripts.csv.gz + morphology_mip.ome.tif) without ground-truth labels, and when evaluating loss-component contributions via ablation.

## When NOT to use

- Nuclei images are separated into fields of view (e.g. CosMx) — use model.stitch_nuclei() before preprocessing instead of the standard segmentation path
- You already have a curated cell-by-gene expression matrix and only need evaluation, not segmentation model training
- The task is transcript-only, segmentation-free clustering (e.g. Baysor-style) rather than image-and-transcript segmentation with morphology

## Inputs

- transcripts.csv.gz (10x Genomics Xenium Output Bundle In Situ Replicate 1)
- morphology_mip.ome.tif (DAPI morphology image)
- xenium_example_config.yaml with corrected file paths and loss weight lambdas
- Breast cancer single-cell reference and markers: sc_breast.csv, sc_breast_markers_pos.csv, sc_breast_markers_neg.csv

## Outputs

- Segmented-cell mask .tif where pixel values correspond to cell IDs (file name ends in _connected.tif)
- Cell-gene expression matrix expr_mat.csv
- Trained self-supervised segmentation model weights per ablation configuration

## How to apply

Obtain the platform-specific .yaml configuration (e.g. xenium_example_config.yaml via BIDCellModel.get_example_config("xenium")) and correct the file paths for transcripts.csv.gz and morphology_mip.ome.tif. Run model.preprocess() once (segment_nuclei, generate_expression_maps, generate_patches, make_cell_gene_mat, preannotate) to produce shared inputs, supplying the single-cell reference and marker files (sc_breast.csv, sc_breast_markers_pos.csv, sc_breast_markers_neg.csv) whose markers come from the highest and lowest 10th expression percentile per cell type. Train with model.train() targeting ~4,000 training steps as in the reference configuration; the total loss (Eq. 13) is a weighted sum of six biologically-informed components (nuclei encapsulation, cell-calling with eccentricity-driven elongated expansion, over-segmentation, overlap, positive-marker, negative-marker losses). For ablation studies, zero one loss weight lambda in the .yaml (and disable shape-specific expansions) per run, then call model.predict(). The rationale is that the losses work synergistically—capturing expression patterns local to nuclei using cell-type-informed morphologies while keeping cell instances cohesive—so removing components degrades segmentation.

## Related tools

- **BIDCell** (Self-supervised deep learning framework providing BIDCellModel with preprocess/train/predict stages and the six biologically-informed loss functions) — https://github.com/SydneyBioX/BIDCell
- **10x Genomics Xenium** (Source platform for the Xenium-BreastCancer1 training data (transcripts.csv.gz and morphology_mip.ome.tif))
- **SydneyBioX/scClassify** (Source of example single-cell reference data for cell-type-informed losses) — https://github.com/SydneyBioX/scClassify

## Examples

```
from bidcell import BIDCellModel
model = BIDCellModel("xenium_example_config.yaml")
model.preprocess(); model.train(); model.predict()
```

## Evaluation signals

- All per-component and total biologically-informed loss values fall within the normalized valid range [0, ~1]; out-of-range values indicate incorrect normalization or implementation
- Purity F1 and captured transcripts are computed per configuration; removing an individual biologically-informed loss should degrade performance (marked purity F1 increase relative to captured transcripts when losses are combined)
- Chromium correlation of the cell-gene matrix is evaluated alongside purity F1 using CellSPA-style metric categories (Supplementary Figs. 4-5)
- Relative ordering of ablation configurations in ablation_metrics.csv reproduces the degradation patterns in Supplementary Figs. 4-5
- Output .tif exists with the expected _connected.tif suffix and pixel values corresponding to cell IDs

## Limitations

- A GPU with at least 12GB VRAM is strongly recommended for the deep learning component, and 32GB RAM for data processing; the reference run used a 12GB NVIDIA GTX Titan V and 64GB RAM
- Models pretrained on other imaging modalities do not transfer well to SST images (as shown with Cellpose), motivating fully self-supervised training
- Achieving more homogeneous cell bodies can result from arbitrary over-segmentation of nuclei, so multiple metrics must be used; loss weight tuning is required per ablation (zeroing a lambda and disabling shape-specific expansions)
- Errors such as pickle.UnpicklingError (truncated data) or GUI queue freezes may occur — reduce cpus or run headless

## Evidence

- [abstract] we present BIDCell, a self-supervised deep learning-based framework with biologically-informed loss functions that learn relationships between spatially resolved gene expression and cell morphology: "we present BIDCell, a self-supervised deep learning-based framework with biologically-informed loss functions that learn relationships between spatially"
- [readme] preprocess API breakdown: "model.segment_nuclei()
model.generate_expression_maps()
model.generate_patches()
model.make_cell_gene_mat(is_cell=False)
model.preannotate()"
- [readme] default architecture: "The default is UNet3+ https://arxiv.org/abs/2004.08790, and we have found it to perform well across different technologies and tissue types."
- [results] loss synergy finding: [evidence span withheld — anchor status: not_found]
- [methods] marker percentile rule: "The positive and negative markers were those with expressions in the highest and lowest 10 percentile for each cell type of a tissue sample."
