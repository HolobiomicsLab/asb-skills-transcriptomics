---
name: subcellular-spatial-transcriptomics-cell-segmentation
description: Use when you have SST data (Xenium, CosMx, MERSCOPE, Stereo-seq) with
  a transcripts file and DAPI/nuclei image and need cell-level segmentation and a
  per-cell gene expression matrix.
when_to_use_negative:
- Data are imaging-based but lack a transcripts file or nuclei image, or are not from
  an SST platform supported by a vendor config
- The goal is segmentation-free, transcript-based cell clustering (e.g. Baysor-style
  methods) rather than image-based cell body delineation
- Manual pixel-level annotation is required for ground truth; the article notes manual
  annotation of individual pixels is unattainable for SST data
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: BIDCell
  role: 'Core segmentation framework: preprocessing (nuclei segmentation, expression
    maps, patches, preannotation), UNet3+ training, prediction, and cell-gene matrix
    extraction'
  repo: https://github.com/SydneyBioX/BIDCell
- name: Cellpose
  role: Pretrained model used for the initial nuclei segmentation step from the DAPI
    image
- name: scClassify
  role: Pretrained hierarchical cell-type classification models used for annotating
    spatial cells (e.g. hippocampal subregions in Xenium-MouseBrain)
  repo: https://github.com/SydneyBioX/scClassify
- name: 10x Genomics Xenium
  role: Primary benchmark platform supplying Xenium Output Bundle (transcripts.csv.gz,
    morphology_mip.ome.tif) for Xenium-BreastCancer1/2 and Xenium-MouseBrain
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_008
  - task_004
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:a7dda1762f980d1fd5d9be6ebb7462aca996c1b0e78f60bb332031e07d7aea8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - BIDCell
  - Cellpose
  - Human Cell Atlas
  - NanoString CosMx
  - TISCH2
  - Vizgen MERSCOPE
  - scClassify
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# subcellular-spatial-transcriptomics-cell-segmentation

## Summary

Segment individual cells in subcellular spatial transcriptomics (SST) data using BIDCell, a self-supervised deep learning framework with biologically-informed loss functions that incorporate single-cell reference expression, cell type, and morphology priors.

## When to use

Use when you have SST data (Xenium, CosMx, MERSCOPE, Stereo-seq) with a transcripts file and DAPI/nuclei image and need cell-level segmentation and a per-cell gene expression matrix.

## When NOT to use

- Data are imaging-based but lack a transcripts file or nuclei image, or are not from an SST platform supported by a vendor config
- The goal is segmentation-free, transcript-based cell clustering (e.g. Baysor-style methods) rather than image-based cell body delineation
- Manual pixel-level annotation is required for ground truth; the article notes manual annotation of individual pixels is unattainable for SST data

## Inputs

- transcripts file (e.g. transcripts.csv.gz) from a Xenium/CosMx/MERSCOPE/Stereo-seq output bundle
- DAPI/nuclei image (e.g. morphology_mip.ome.tif)
- BIDCell .yaml configuration file for the vendor platform
- Single-cell reference expression table (e.g. sc_breast.csv)
- Positive/negative marker files (e.g. sc_breast_markers_pos.csv, sc_breast_markers_neg.csv)

## Outputs

- Segmentation mask .tif file ending in _connected.tif with pixel values corresponding to cell IDs
- Cell-gene expression matrix expr_mat.csv
- Optional CellSPA metric summaries (cell counts, % transcripts assigned, purity F1, morphology metrics)

## How to apply

Create a vendor-specific .yaml config via BIDCellModel.get_example_config('xenium'/'cosmx'/'merscope'/'stereoseq') and set paths to transcripts.csv.gz and morphology_mip.ome.tif plus single-cell reference and positive/negative marker files. Run preprocessing (optionally stitch_nuclei for FOV-separated images like CosMx, then segment_nuclei, generate_expression_maps, generate_patches, make_cell_gene_mat, preannotate), then train the default UNet3+ model for ~4,000 steps (~10 min on a >=12GB GPU) and predict to obtain a *_connected.tif mask whose pixel values are cell IDs. Extract per-cell expression with make_cell_gene_mat(is_cell=False) to yield expr_mat.csv; validate by counting unique cell IDs (e.g. ~103,209 cells on Xenium-BreastCancer1 with ~53.4% of transcripts assigned). Assess segmentation with CellSPA-style metrics (purity F1, captured transcripts, Chromium correlation, morphology, cross-replicate agreement). Positive/negative markers come from the highest and lowest 10 percentile of expression per cell type, removing positive markers common to at least a third of cell types.

## Related tools

- **BIDCell** (Core segmentation framework: preprocessing (nuclei segmentation, expression maps, patches, preannotation), UNet3+ training, prediction, and cell-gene matrix extraction) — https://github.com/SydneyBioX/BIDCell
- **Cellpose** (Pretrained model used for the initial nuclei segmentation step from the DAPI image)
- **scClassify** (Pretrained hierarchical cell-type classification models used for annotating spatial cells (e.g. hippocampal subregions in Xenium-MouseBrain)) — https://github.com/SydneyBioX/scClassify
- **10x Genomics Xenium** (Primary benchmark platform supplying Xenium Output Bundle (transcripts.csv.gz, morphology_mip.ome.tif) for Xenium-BreastCancer1/2 and Xenium-MouseBrain)

## Examples

```
from bidcell import BIDCellModel; model = BIDCellModel('xenium_example_config.yaml'); model.preprocess(); model.train(); model.predict()
```

## Evaluation signals

- Count unique cell IDs in the *_connected.tif mask; expect ~103,209 cells on Xenium-BreastCancer1 (approximately 100,000 cells)
- Compute the fraction of transcripts assigned to cells; target ~53.4% on Xenium-BreastCancer1
- expr_mat.csv and the *_connected.tif segmentation file are both produced at the end of the pipeline
- Purity F1, captured transcripts, and Chromium correlation metrics are computed and compared following CellSPA categories; removing individual biologically-informed losses should degrade performance (increased purity F1 relative to captured transcripts)
- Cross-replicate analysis: BIDCell results for Xenium-BreastCancer1 vs 2 show well-mixed replicate labels in UMAP/tSNE and correlated cell type proportions and morphology metrics

## Limitations

- A GPU with at least 12GB VRAM is strongly recommended, plus 32GB RAM for data processing; higher cpus settings may freeze the system
- Cells are three-dimensional but analysis is 2D, so slicing may limit representation of cell structure and affects measured expression
- Varied and elongated morphologies (melanocytes, fibroblasts, dendrites, axons) are hard to segment without nearby nuclei; in MERSCOPE data, ~25% of cells lack nuclei in their segmentation
- More homogeneous cell bodies can also result from arbitrary over-segmentation of nuclei, motivating the use of multiple CellSPA metrics
- Models pretrained on other imaging modalities (e.g. Cellpose) do not transfer well to SST images

## Evidence

- [abstract] we present BIDCell, a self-supervised deep learning-based framework with biologically-informed loss functions that learn relationships between spatially resolved gene expression and cell morphology: "we present BIDCell, a self-supervised deep learning-based framework with biologically-informed loss functions that learn relationships between spatially resolved gene expression and cell morphology"
- [readme] The positive and negative markers were those with expressions in the highest and lowest 10 percentile for each cell type of a tissue sample.: "The positive and negative markers were those with expressions in the highest and lowest 10 percentile for each cell type of a tissue sample. We found that removing positive markers that were common"
- [readme] model.stitch_nuclei() # for when nuclei images are separated into FOVs (e.g., CosMx): "model.stitch_nuclei() # for when nuclei images are separated into FOVs (e.g., CosMx)"
- [readme] .tif file of segmented cells, where the value corresponds to cell IDs. File name ends in `_connected.tif`: ".tif file of segmented cells, where the value corresponds to cell IDs. File name ends in `_connected.tif`"
- [discussion] In MERSCOPE data that display cell membrane markers, there is a percentage (25%) of cells that lack nuclei in their segmentation: "In MERSCOPE data that display cell membrane markers, there is a percentage (25%) of cells that lack nuclei in their segmentation"
