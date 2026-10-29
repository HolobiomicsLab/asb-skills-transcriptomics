---
name: cell-morphology-metric-correlation
description: Use when comparing BIDCell vs vendor 10x segmentation masks on Xenium
  data, or testing cross-replicate concordance of cell morphology, as part of the
  CellSPA category-A baseline and category agreement metrics.
when_to_use_negative:
- Input is an already-summarized feature table rather than a pixel-labeled segmentation
  mask and transcript file — morphology metrics must be derived from *_connected.tif-style
  cell-ID masks.
- Nuclei images are split into fields of view (e.g. CosMx) and have not yet been stitched/segmented
  — run BIDCell's stitch_nuclei and preprocess stages first.
- Evaluating segmentation quality by a single morphology metric alone — the article
  notes homogenous cell bodies can result from arbitrary over-segmentation of nuclei,
  motivating multiple metrics.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BIDCell
  role: Provides the *_connected.tif cell-ID segmentation mask and the Xenium pipeline
    (preprocess, train, predict) whose outputs the morphology metrics are computed
    on
  repo: https://github.com/SydneyBioX/BIDCell
- name: 10x Genomics Xenium
  role: Source platform for transcripts.csv.gz, morphology_mip.ome.tif, and the vendor
    10x segmentation used as the comparison baseline
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44560-w
    title: 'BIDCell: Biologically-informed self-supervised learning for segmentation
      of subcellular spatial transcriptomics data'
  source_content_hash: sha256:340da1a245641e6bcdd83a49db75d84f354c07afba7e6da2f6a088ddf4027747
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/BIDCell
  tools:
  - 10x Genomics Xenium
  - BIDCell
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cell-morphology-metric-correlation

## Summary

Computes per-cell morphology metrics (area, elongation, eccentricity, solidity) from cell segmentation masks of subcellular spatial transcriptomics data and correlates them between segmentation methods or between replicate sections, following the CellSPA evaluation framework.

## When to use

Use when comparing BIDCell vs vendor 10x segmentation masks on Xenium data, or testing cross-replicate concordance of cell morphology, as part of the CellSPA category-A baseline and category agreement metrics.

## When NOT to use

- Input is an already-summarized feature table rather than a pixel-labeled segmentation mask and transcript file — morphology metrics must be derived from *_connected.tif-style cell-ID masks.
- Nuclei images are split into fields of view (e.g. CosMx) and have not yet been stitched/segmented — run BIDCell's stitch_nuclei and preprocess stages first.
- Evaluating segmentation quality by a single morphology metric alone — the article notes homogenous cell bodies can result from arbitrary over-segmentation of nuclei, motivating multiple metrics.

## Inputs

- BIDCell segmentation mask (*_connected.tif, pixel values = cell IDs)
- Vendor 10x segmentation from the Xenium Output Bundle In Situ Replicate 1
- transcripts.csv.gz from the Xenium output bundle
- morphology_mip.ome.tif (DAPI image defining shared mask dimensions)
- Cell-gene expression matrix expr_mat.csv per replicate (for cross-replicate agreement)

## Outputs

- cellspa_baseline_metrics.csv — side-by-side per-method baseline counts and morphology metrics
- Per-cell morphology metrics (area, elongation, eccentricity, solidity) for each segmentation method
- Cross-replicate agreement metrics (morphology correlations, cell-type proportion comparisons, UMAP/tSNE replicate-label mixing)

## How to apply

Load the BIDCell segmentation mask (the *_connected.tif file whose pixel values correspond to cell IDs) and the vendor 10x segmentation shipped with the Xenium Output Bundle In Situ Replicate 1. Load transcripts.csv.gz from the Xenium output bundle and assign transcripts to cells under each mask, computing number of cells, percentage of transcripts assigned (e.g. BIDCell reached ~100,000 cells with 53.4% transcripts assigned on Xenium-BreastCancer1), and transcripts/genes per cell. Ensure both masks are compared at the same DAPI image dimensions (morphology_mip.ome.tif), resizing the segmentation tif if needed for xeniumranger import-segmentation, then compute region-properties morphology metrics (elongation, eccentricity, solidity, area) per cell for each method. Assemble the per-method summaries side by side (reproducing Fig. 2b-e baseline/morphology comparisons), and for replicability (Fig. 3i-j, Supplementary Fig. 16), correlate baseline cell characteristics, cell type proportions, and morphology metrics between Xenium-BreastCancer1 and Xenium-BreastCancer2 alongside UMAP/ tSNE mixing of replicate labels. Multiple metrics are used because apparently favorable morphology outcomes (e.g. homogenous cell bodies) can also arise from arbitrary over-segmentation of nuclei.

## Related tools

- **BIDCell** (Provides the *_connected.tif cell-ID segmentation mask and the Xenium pipeline (preprocess, train, predict) whose outputs the morphology metrics are computed on) — https://github.com/SydneyBioX/BIDCell
- **10x Genomics Xenium** (Source platform for transcripts.csv.gz, morphology_mip.ome.tif, and the vendor 10x segmentation used as the comparison baseline)

## Examples

```
from skimage.measure import regionprops; props = regionprops(mask)  # elongation, eccentricity, solidity per cell ID in *_connected.tif
```

## Evaluation signals

- The metrics table contains rows for both BIDCell and the 10x vendor segmentation with all category-A counts and morphology metrics present, enabling side-by-side comparison as in Fig. 2b-e.
- Segmentation masks are confirmed to be at the same dimensions as morphology_mip.ome.tif before region-properties computation.
- Baseline expectations on Xenium-BreastCancer1: BIDCell produces approximately 100,000 cells with ~53.4% of transcripts assigned.
- For replicability, agreement metrics reproduce the replicate-concordance patterns reported in Fig. 3i-j and Supplementary Fig. 16 (well-mixed replicate labels in UMAP/tSNE, high cross-replicate correlations).
- The *_connected.tif mask and expr_mat.csv exist for each replicate before computing cross-replicate correlations.

## Limitations

- Achieving more homogenous cell bodies can result from arbitrary over-segmentation of nuclei, so morphology metrics alone can be misleading.
- Two-dimensional analyses may achieve limited representation of cells' three-dimensional structure, affecting area/shape metrics.
- Varied and elongated cell morphologies (melanocytes, fibroblasts, dendrites, axons) make cell boundaries hard to identify without nearby nuclei; in MERSCOPE data ~25% of segmented cells lack nuclei.

## Evidence

- [methods] resize segmentation to DAPI image dimensions for xeniumranger import-segmentation: "The .tif file needs to be resized to be the same dimensions as the DAPI image (`morphology_mip.ome.tif`)"
- [intro] CellSPA five-category evaluation framework: "we created a comprehensive evaluation framework for cell segmentation, CellSPA, that assesses five complementary categories of criteria"
- [readme] BIDCell expected segmentation output format: ".tif file of segmented cells, where the value corresponds to cell IDs. File name ends in `_connected.tif`"
- [discussion] over-segmentation caveat motivating multiple morphology metrics: [evidence span withheld — anchor status: not_found]
