---
name: dapi-image-nuclei-segmentation
description: Use when preparing subcellular spatial transcriptomics data (Xenium,
  CosMx, MERSCOPE, Stereo-seq) for BIDCell segmentation and a DAPI/nuclear-stain image
  is available from which initial nuclear masks must be generated.
when_to_use_negative:
- The platform's nuclei images are separated into FOVs and have not been stitched
  yet — run model.stitch_nuclei() before segmenting
- No nuclear stain/DAPI channel is available in the input imaging data
- The goal is transcript-only (segmentation-free) cell clustering, e.g., Baysor-style
  methods, rather than image-based segmentation
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3316
tools:
- name: BIDCell
  role: Framework exposing model.segment_nuclei() (and model.stitch_nuclei()) within
    model.preprocess() to perform Cellpose-based nuclei segmentation from the DAPI
    image
  repo: https://github.com/SydneyBioX/BIDCell
- name: Cellpose
  role: Pretrained nuclei segmentation model used by BIDCell for the initial segmentation
    of the nuclear marker image
- name: 10x Genomics Xenium
  role: Source of the Xenium Output Bundle In Situ Replicate 1 providing morphology_mip.ome.tif
    DAPI image and transcripts.csv.gz
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:060e8c05c8ef114fb9506c95354a91c951cf5684dc683a45f4c9a938eaa29d35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - BIDCell
  - Cellpose
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# dapi-image-nuclei-segmentation

## Summary

Segment nuclei from the DAPI morphology image (e.g., morphology_mip.ome.tif from a 10x Genomics Xenium Output Bundle) using Cellpose-based nuclei segmentation within the BIDCell pipeline, producing the initial nuclear masks that seed downstream biologically-informed cell segmentation.

## When to use

Use when preparing subcellular spatial transcriptomics data (Xenium, CosMx, MERSCOPE, Stereo-seq) for BIDCell segmentation and a DAPI/nuclear-stain image is available from which initial nuclear masks must be generated.

## When NOT to use

- The platform's nuclei images are separated into FOVs and have not been stitched yet — run model.stitch_nuclei() before segmenting
- No nuclear stain/DAPI channel is available in the input imaging data
- The goal is transcript-only (segmentation-free) cell clustering, e.g., Baysor-style methods, rather than image-based segmentation

## Inputs

- DAPI/nuclear-stain morphology image (e.g., morphology_mip.ome.tif from the Xenium Output Bundle In Situ Replicate 1)
- BIDCell platform YAML configuration (e.g., xenium_example_config.yaml) with file paths and cpus set
- Nuclei images possibly separated into FOVs (for CosMx, stitched via model.stitch_nuclei())

## Outputs

- Nuclear segmentation masks used by downstream BIDCell training and prediction
- Final whole-cell segmentation mask (*_connected.tif) whose pixel values correspond to cell IDs
- Cell-gene expression matrix (expr_mat.csv)

## How to apply

Inside BIDCell, first ensure the DAPI image (morphology_mip.ome.tif) and quality-filtered transcripts (transcripts.csv.gz) paths are set in the platform YAML config (e.g., xenium_example_config.yaml) and that cpus matches the system. Call model.segment_nuclei() as part of model.preprocess(); for platforms whose nuclei images are split into fields of view (e.g., CosMx), run model.stitch_nuclei() first. Cellpose-based segmentation is used because pretrained models such as Cellpose and Mesmer are the standard starting point for nuclear marker-based segmentation, though models pretrained on other imaging modalities transfer imperfectly to SST images. The resulting nuclei are then expanded during model training/prediction using biologically-informed loss functions and single-cell reference annotations (e.g., sc_breast.csv with positive/negative marker files) to delineate whole-cell boundaries. Validate by inspecting the segmentation mask whose pixel values correspond to cell IDs (the *_connected.tif output) and comparing cell counts against reported values such as ~103,209 cells on Xenium-BreastCancer1.

## Related tools

- **BIDCell** (Framework exposing model.segment_nuclei() (and model.stitch_nuclei()) within model.preprocess() to perform Cellpose-based nuclei segmentation from the DAPI image) — https://github.com/SydneyBioX/BIDCell
- **Cellpose** (Pretrained nuclei segmentation model used by BIDCell for the initial segmentation of the nuclear marker image)
- **10x Genomics Xenium** (Source of the Xenium Output Bundle In Situ Replicate 1 providing morphology_mip.ome.tif DAPI image and transcripts.csv.gz)

## Examples

```
from bidcell import BIDCellModel; model = BIDCellModel('xenium_example_config.yaml'); model.segment_nuclei()
```

## Evaluation signals

- The segmentation output is a .tif file ending in _connected.tif whose pixel values correspond to cell IDs
- Counting unique cell IDs in the *_connected.tif mask on Xenium-BreastCancer1 should approach the reported ~103,209 cells
- Fraction of transcripts assigned to cells should be near the reported ~53.4% on Xenium-BreastCancer1
- Nuclei segmentation completes as part of model.preprocess() without the pickle.UnpicklingError: pickle data was truncated error (reduce cpus if it appears)

## Limitations

- Models pretrained on other imaging modalities do not transfer well to SST images (e.g., Cellpose), so initial nuclei may be imperfect
- In MERSCOPE data with cell membrane markers, 25% of segmented cells lack nuclei, likely elongated melanocytes or fibroblasts in the section without a nucleus
- A GPU with at least 12GB VRAM and 32GB RAM is recommended; higher cpus speeds runs but may freeze the system or cause truncated pickle errors
- Arbitrary over-segmentation of nuclei can artificially produce more homogeneous cell bodies, motivating multiple CellSPA metrics

## Evidence

- [intro] initial nuclei segmentation is accomplished with a nuclear marker, using thresholding or pretrained models such as Cellpose5 and Mesmer6: "initial nuclei segmentation is accomplished with a nuclear marker, using thresholding or pretrained models such as Cellpose5 and Mesmer6"
- [readme] model.segment_nuclei(): "model.segment_nuclei()"
- [discussion] models pretrained on other imaging modalities do not transfer well to SST images: "we have shown (e.g., with Cellpose) that models pretrained on other imaging modalities do not transfer well to SST images"
- [readme] the value corresponds to cell IDs. File name ends in _connected.tif: ".tif file of segmented cells, where the value corresponds to cell IDs. File name ends in `_connected.tif`"
- [readme] stitch nuclei for FOV platforms: "model.stitch_nuclei() # for when nuclei images are separated into FOVs (e.g., CosMx)"
