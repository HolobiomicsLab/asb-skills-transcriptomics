---
name: single-cell-reference-based-segmentation-annotation
description: Use when reproducing or extending the cross-platform generalisability
  benchmark of BIDCell on CosMx or MERSCOPE data, or when single-cell reference data
  are needed to guide biologically-informed segmentation and assess B-cell/T-cell
  marker contamination.
when_to_use_negative:
- Input is Xenium breast cancer data — the dedicated Xenium workflow and xenium_example_config.yaml
  path should be used instead of the CosMx/MERSCOPE path
- Data come from a platform with no matching single-cell reference (reference csv
  must contain average expressions for genes in the spatial dataset)
- The goal is nuclei-only segmentation without cell-body delineation or biological
  loss terms — thresholding or pretrained models such as Cellpose or Mesmer may suffice
edam_operation: http://edamontology.org/operation_3629
edam_topics:
- http://edamontology.org/topic_3521
tools:
- name: BIDCell
  role: Biologically-informed self-supervised segmentation model; provides get_example_config,
    stitch_nuclei, segment_nuclei, train, predict and make_cell_gene_mat steps for
    CosMx/MERSCOPE data
  repo: https://github.com/SydneyBioX/BIDCell
- name: NanoString CosMx
  role: SST platform supplying the NSCLC Lung5-1 data used to test cross-platform
    generalisability via B-cell negative-marker contamination
- name: Vizgen MERSCOPE
  role: SST platform supplying the melanoma patient-2 data used to compute cell-type
    entropy versus coefficient of variation
- name: TISCH2
  role: Public repository from which the lung and melanoma single-cell references
    and marker files are downloaded
- name: scClassify
  role: Source of example single-cell reference data for segmentation inputs
  repo: https://github.com/SydneyBioX/scClassify
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:93a42968007ff7c1093cbb5c115898433bb58d24986ed0c59d5524ea58679ea6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - BIDCell
  - Human Cell Atlas
  - NanoString CosMx
  - TISCH2
  - Vizgen MERSCOPE
  - scClassify
  edam_operation: http://edamontology.org/operation_3629
  edam_topics:
  - http://edamontology.org/topic_3521
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-reference-based-segmentation-annotation

## Summary

Run the BIDCell pipeline with a public single-cell reference and positive/negative marker files to segment cells on non-Xenium SST platforms (NanoString CosMx lung, Vizgen MERSCOPE melanoma) and validate the segmentation via cell-type contamination and entropy-versus-CV metrics.

## When to use

Use when reproducing or extending the cross-platform generalisability benchmark of BIDCell on CosMx or MERSCOPE data, or when single-cell reference data are needed to guide biologically-informed segmentation and assess B-cell/T-cell marker contamination.

## When NOT to use

- Input is Xenium breast cancer data — the dedicated Xenium workflow and xenium_example_config.yaml path should be used instead of the CosMx/MERSCOPE path
- Data come from a platform with no matching single-cell reference (reference csv must contain average expressions for genes in the spatial dataset)
- The goal is nuclei-only segmentation without cell-body delineation or biological loss terms — thresholding or pretrained models such as Cellpose or Mesmer may suffice

## Inputs

- CosMx NSCLC Lung5-1 transcript and FOV-separated nuclei image files
- MERSCOPE melanoma patient-2 transcripts and nuclei images
- Single-cell reference csv (average expression per cell type, e.g. HLCA, TISCH-NSCLC, TISCH-SKCM)
- Positive marker csv (top 10 percentile expression per cell type)
- Negative marker csv (bottom 10 percentile expression per cell type)
- Vendor .yaml config file (cosmx or merscope example)

## Outputs

- *_connected.tif segmentation mask with cell IDs per pixel
- expr_mat.csv cell-by-gene expression matrix per platform
- Cell-type contamination and entropy-versus-CV benchmark panels (Fig. 4a-f)

## How to apply

Download the public CosMx NSCLC Lung5-1 and MERSCOPE melanoma patient-2 datasets plus single-cell references (HLCA/TISCH-NSCLC for lung, TISCH-SKCM for melanoma) from TISCH2 or the Human Cell Atlas, formatted as sc_breast.csv-style reference and markers_pos/markers_neg files. Generate vendor configs with BIDCellModel.get_example_config('cosmx') or ('merscope'), edit the transcripts, DAPI/nuclei image paths and cpus parameter, then for CosMx FOV-separated nuclei images call model.stitch_nuclei() before model.segment_nuclei(), followed by generate_expression_maps(), generate_patches(), train() and predict() (default UNet3+, ~4,000 steps, GPU with >=12GB VRAM). Extract per-cell expression with model.make_cell_gene_mat() to produce expr_mat.csv. Positive/negative markers are those in the highest and lowest 10 percentile of expression per cell type, and positive markers common to at least a third of cell types should be removed. Judge success by a *_connected.tif segmentation and expr_mat.csv output per platform, low B-cell expression of negative (contamination-indicating) markers on CosMx-Lung, and competitive cell-type entropy versus coefficient of variation profiles on MERSCOPE-Melanoma.

## Related tools

- **BIDCell** (Biologically-informed self-supervised segmentation model; provides get_example_config, stitch_nuclei, segment_nuclei, train, predict and make_cell_gene_mat steps for CosMx/MERSCOPE data) — https://github.com/SydneyBioX/BIDCell
- **NanoString CosMx** (SST platform supplying the NSCLC Lung5-1 data used to test cross-platform generalisability via B-cell negative-marker contamination)
- **Vizgen MERSCOPE** (SST platform supplying the melanoma patient-2 data used to compute cell-type entropy versus coefficient of variation)
- **TISCH2** (Public repository from which the lung and melanoma single-cell references and marker files are downloaded)
- **scClassify** (Source of example single-cell reference data for segmentation inputs) — https://github.com/SydneyBioX/scClassify

## Examples

```
from bidcell import BIDCellModel; BIDCellModel.get_example_config('cosmx'); model = BIDCellModel('cosmx_example_config.yaml'); model.stitch_nuclei(); model.segment_nuclei(); model.train(); model.predict(); model.make_cell_gene_mat()
```

## Evaluation signals

- Each run produces a *_connected.tif segmentation file whose pixel values correspond to cell IDs
- Each platform run yields an expr_mat.csv cell-gene matrix from make_cell_gene_mat()
- On CosMx-Lung, the percentage of B cells expressing negative (contamination-indicating) markers is lower than in comparator segmentation methods
- On MERSCOPE-Melanoma, the entropy-versus-CV profile of BIDCell is competitive with other segmentation methods
- Config yaml points to existing transcripts and DAPI/nuclei files, and cpus is tuned to avoid pickle.UnpicklingError truncation

## Limitations

- A GPU with at least 12GB VRAM and ~32GB RAM is required; higher cpus settings speed runs but may freeze the system or raise 'pickle data was truncated'
- In MERSCOPE data ~25% of cells lack nuclei in their segmentation, likely elongated melanocytes or fibroblasts in a section without a nucleus
- Manual annotation of individual pixels is unattainable for SST data, so no ground-truth pixel labels exist; two-dimensional analyses may achieve limited representation of cells' three-dimensional structure

## Evidence

- [readme] model.stitch_nuclei() # for when nuclei images are separated into FOVs (e.g., CosMx): "model.stitch_nuclei() # for when nuclei images are separated into FOVs (e.g., CosMx)"
- [readme] GPU requirement: "A GPU with at least 12GB VRAM is strongly recommended for the deep learning component, and 32GB RAM for data processing."
- [methods] reference download sources: "These can be downloaded from public repositories such as TISCH2, Allen Brain Map, and the Human Cell Atlas."
- [readme] marker construction rule: "The positive and negative markers were those with expressions in the highest and lowest 10 percentile for each cell type of a tissue sample."
- [abstract] generalisability finding: "we demonstrate that BIDCell outperforms other state-of-the-art methods according to many metrics across a variety of tissue types and technology platforms"
