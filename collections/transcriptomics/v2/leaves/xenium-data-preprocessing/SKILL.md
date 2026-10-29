---
name: xenium-data-preprocessing
description: Use when you have a raw Xenium output bundle (transcripts.csv.gz and
  morphology_mip.ome.tif) and need to generate BIDCell training inputs (nuclei segmentation,
  expression maps, patches, cell-gene matrix) before model training.
when_to_use_negative:
- Nuclei images are split into fields of view (e.g. CosMx data) without first calling
  model.stitch_nuclei() — the Xenium configuration assumes a single image
- You already possess a segmented cell mask (*_connected.tif) and only need to extract
  expression or run downstream replicability comparisons
- You are using a non-Xenium platform configuration without adapting the yaml (CosMx,
  MERSCOPE, Stereo-seq each have their own example config)
edam_operation: http://edamontology.org/operation_2939
tools:
- name: BIDCell
  role: Python framework whose BIDCellModel.preprocess() chains segment_nuclei, generate_expression_maps,
    generate_patches, make_cell_gene_mat(is_cell=False) and preannotate to produce
    segmentation-model training inputs
  repo: https://github.com/SydneyBioX/BIDCell
- name: 10x Genomics Xenium
  role: Source of the raw in situ output bundle (transcripts.csv.gz, morphology_mip.ome.tif)
    that the preprocessing stages consume
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:fe9b6022e502c3e1910182d4feed45c858eefc58e2d5a0510749337c78c10512
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - BIDCell
  edam_operation: http://edamontology.org/operation_2939
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# xenium-data-preprocessing

## Summary

Preprocesses raw 10x Genomics Xenium in situ breast cancer output bundles (transcripts.csv.gz, morphology_mip.ome.tif) into the inputs BIDCell's self-supervised UNet3+ model needs: stitched/segmented nuclei, expression maps, patches, a cell-gene matrix, and preannotations.

## When to use

Use when you have a raw Xenium output bundle (transcripts.csv.gz and morphology_mip.ome.tif) and need to generate BIDCell training inputs (nuclei segmentation, expression maps, patches, cell-gene matrix) before model training.

## When NOT to use

- Nuclei images are split into fields of view (e.g. CosMx data) without first calling model.stitch_nuclei() — the Xenium configuration assumes a single image
- You already possess a segmented cell mask (*_connected.tif) and only need to extract expression or run downstream replicability comparisons
- You are using a non-Xenium platform configuration without adapting the yaml (CosMx, MERSCOPE, Stereo-seq each have their own example config)

## Inputs

- Xenium transcripts.csv.gz (per-transcript coordinates)
- morphology_mip.ome.tif (DAPI image)
- xenium_example_config.yaml (platform parameter file with paths, cpus, model_params)
- sc_breast.csv (single-cell reference: average expression per cell type)
- sc_breast_markers_pos.csv and sc_breast_markers_neg.csv (cell-type marker gene lists)

## Outputs

- Segmented nuclei masks
- Expression maps for spatially resolved genes
- Training patches
- Cell-gene count matrix (make_cell_gene_mat with is_cell=False)
- Preannotated training targets for the segmentation model

## How to apply

Download the Xenium Output Bundle In Situ Replicate from the 10x Genomics preview-dataset page and set file paths (transcripts.csv.gz, morphology_mip.ome.tif) under the `files` section of xenium_example_config.yaml, tuning the `cpus` parameter to your machine. Instantiate BIDCellModel with the Xenium yaml and run model.preprocess(), which executes segment_nuclei, generate_expression_maps, generate_patches, make_cell_gene_mat(is_cell=False), and preannotate in sequence (stitch_nuclei is only needed when nuclei images come as separate FOVs, e.g. CosMx). Use the provided breast cancer single-cell reference and marker files (sc_breast.csv, sc_breast_markers_pos.csv, sc_breast_markers_neg.csv) for the preannotate step. Rationale: these stages convert transcript-level and DAPI-image-level signals into training patches plus preannotations that drive BIDCell's biologically-informed loss functions, so correctness here determines downstream segmentation quality.

## Related tools

- **BIDCell** (Python framework whose BIDCellModel.preprocess() chains segment_nuclei, generate_expression_maps, generate_patches, make_cell_gene_mat(is_cell=False) and preannotate to produce segmentation-model training inputs) — https://github.com/SydneyBioX/BIDCell
- **10x Genomics Xenium** (Source of the raw in situ output bundle (transcripts.csv.gz, morphology_mip.ome.tif) that the preprocessing stages consume)

## Examples

```
from bidcell import BIDCellModel; model = BIDCellModel("xenium_example_config.yaml"); model.preprocess()
```

## Evaluation signals

- model.preprocess() completes without the pickle.UnpicklingError ('pickle data was truncated') that indicates the cpus setting was too high
- All five sub-steps (segment_nuclei, generate_expression_maps, generate_patches, make_cell_gene_mat, preannotate) produce their artifacts in the configured output directory
- The DAPI image morphology_mip.ome.tif dimensions match the segmentation output tif dimensions expected by xeniumranger import-segmentation
- The preannotation step consumed the provided breast cancer reference files (sc_breast.csv plus pos/neg markers) without path errors
- Training that follows (model.train(), ~4,000 steps on a >=12GB GPU) converges, indicating patches and preannotations were generated correctly

## Limitations

- Hardware demands are high: a GPU with at least 12GB VRAM is strongly recommended for the deep learning component, and 32GB RAM for data processing
- Higher cpus settings speed up runs but may freeze the system or cause 'pickle data was truncated' errors
- Preprocessing is 2D: analyses in a two-dimensional perspective may achieve limited representation of cells' three-dimensional structure

## Evidence

- [readme] Xenium bundle paths: "the paths for the transcripts (`transcripts.csv.gz`) and DAPI (`morphology_mip.ome.tif`) files."
- [methods] Xenium dataset source: "The full dataset (Xenium Output Bundle In Situ Replicate 1) may be downloaded from https://www.10xgenomics.com/products/xenium-in-situ/preview-dataset-human-breast"
- [readme] hardware requirements: "A GPU with at least 12GB VRAM is strongly recommended for the deep learning component, and 32GB RAM for data processing."
- [readme] FOV stitching caveat: "model.stitch_nuclei() # for when nuclei images are separated into FOVs (e.g., CosMx)"
- [readme] config copy behavior: "This will copy the .yaml for the respective vendor into your working directory, for example `xenium_example_config.yaml`."
