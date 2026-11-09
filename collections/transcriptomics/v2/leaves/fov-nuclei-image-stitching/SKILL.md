---
name: fov-nuclei-image-stitching
description: Use when input nuclei images are separated into FOVs (e.g., NanoString
  CosMx Lung5-1 data) and a whole-section image is needed before model.segment_nuclei()
  in the BIDCell pipeline.
when_to_use_negative:
- Input nuclei image is already a whole-slide image (e.g., Xenium morphology_mip.ome.tif)
  — stitching is only for FOV-separated exports
- Data come from transcript-only or segmentation-free methods (e.g., Baysor, pciSeq)
  that do not require image mosaicking
tools:
- name: BIDCell
  role: Provides the model.stitch_nuclei() preprocessing function that mosaics FOV-separated
    nuclei images before segmentation
  repo: https://github.com/SydneyBioX/BIDCell
- name: NanoString CosMx
  role: SST platform whose FOV-separated nuclei image exports are the input requiring
    stitching
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:29bd1ec8fb2b82abe0e7787040d2bc27e113979783ab7de5870db429fb03dfbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - BIDCell
  - NanoString CosMx
  - TISCH2
  - Vizgen MERSCOPE
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# fov-nuclei-image-stitching

## Summary

Stitch field-of-view (FOV)-separated nuclei images into a single whole-section image prior to BIDCell nuclei segmentation, required for platforms such as NanoString CosMx where nuclei images are exported per FOV.

## When to use

Use when input nuclei images are separated into FOVs (e.g., NanoString CosMx Lung5-1 data) and a whole-section image is needed before model.segment_nuclei() in the BIDCell pipeline.

## When NOT to use

- Input nuclei image is already a whole-slide image (e.g., Xenium morphology_mip.ome.tif) — stitching is only for FOV-separated exports
- Data come from transcript-only or segmentation-free methods (e.g., Baysor, pciSeq) that do not require image mosaicking

## Inputs

- FOV-separated nuclei images from NanoString CosMx (e.g., Lung5-1 NSCLC dataset)
- Platform-specific BIDCell .yaml config file with file paths and cpus set

## Outputs

- Stitched whole-section nuclei image consumed by model.segment_nuclei()
- Downstream *_connected.tif segmentation file and expr_mat.csv cell-gene matrix (after full pipeline)

## How to apply

After generating the platform-specific configuration with BIDCellModel.get_example_config('cosmx') and editing file paths (transcripts, DAPI/nuclei images) and the cpus parameter, call model.stitch_nuclei() to mosaic the FOV-separated nuclei images into one contiguous image. This must run BEFORE model.segment_nuclei(), which then produces the initial nuclei segmentation that feeds into model.generate_expression_maps() and model.generate_patches(). Vendors such as Xenium provide whole-slide nuclei images, so stitching is unnecessary there. Rationale: BIDCell's downstream patch generation and training assume a spatially coherent whole-section nuclei image; skipping stitching on CosMx data would break spatial alignment between nuclei and transcript coordinates.

## Related tools

- **BIDCell** (Provides the model.stitch_nuclei() preprocessing function that mosaics FOV-separated nuclei images before segmentation) — https://github.com/SydneyBioX/BIDCell
- **NanoString CosMx** (SST platform whose FOV-separated nuclei image exports are the input requiring stitching)

## Examples

```
from bidcell import BIDCellModel; model = BIDCellModel("cosmx_example_config.yaml"); model.stitch_nuclei(); model.segment_nuclei()
```

## Evaluation signals

- Stitched image covers the full tissue area with no gaps or duplicated FOVs across the mosaic
- Pipeline order is respected: stitch_nuclei() output precedes segment_nuclei(); the full run yields a *_connected.tif segmentation file
- Downstream expr_mat.csv cell-gene matrix is produced and per-cell transcript counts align spatially with stitched nuclei positions
- On CosMx-Lung benchmark, resulting segmentation shows low B-cell contamination by T-cell/negative markers as in the Fig. 4 generalisability panels

## Limitations

- CosMx nuclei images are FOV-separated; this workflow step is specific to that export format and not needed for Xenium or MERSCOPE whole/near-whole section images
- Only <1,000 genes are needed for segmentation, and incorrect file paths or an oversized cpus setting in the .yaml can crash preprocessing (e.g., pickle.UnpicklingError: pickle data was truncated)
- A GPU with at least 12GB VRAM and 32GB RAM is strongly recommended for the full pipeline that follows stitching

## Evidence

- [methods] stitch_nuclei called before segmentation: "model.stitch_nuclei() # for when nuclei images are separated into FOVs (e.g., CosMx)"
- [readme] platform example configs: "Examples are provided for 4 major platforms, including Xenium, CosMx, MERSCOPE, and Stereo-seq."
- [abstract] generalisation across platforms: "we demonstrate that BIDCell outperforms other state-of-the-art methods according to many metrics across a variety of tissue types and technology platforms"
