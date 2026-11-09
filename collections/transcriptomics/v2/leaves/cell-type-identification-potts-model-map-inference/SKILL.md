---
name: cell-type-identification-potts-model-map-inference
description: Use when you need per-cell (not per-spot) discrete cell type labels for
  ST spots (Visium, Slide-seq), given nucleus segmentation outputs and a labeled scRNA-seq
  reference, with spatial smoothness enforced.
when_to_use_negative:
- Spot-level cell-type proportions suffice and single-cell resolution is not required
  — deconvolution methods such as RCTD, Cell2location, CARD, or spatialDWLS already
  estimate proportions per spot.
- No spatial coordinates for spots/cells are available, since the Potts prior relies
  on spatial smoothness across neighboring cells/slices.
- No labeled scRNA-seq reference is available, or platform effects between ST and
  reference data cannot be corrected.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SpatialScope
  role: Runs the MAP/Potts-model cell type identification with smoothness hyperparameter
    ν = 10 to assign labels to segmented cells
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: StarDist
  role: Nucleus segmentation on H&E images (SpatialScope default) producing cell counts
    and cell_locations per spot that feed the identification step
- name: PASTE
  role: Pairwise alignment of adjacent slices to construct 3D ST data so the Potts
    model can use 3D spatial information for more reliable spatial priors
- name: RCTD
  role: 'Baseline comparator: StarDist+RCTD misidentifies L4 and L6 IT cells in other
    layers due to lack of spatial smoothness constraint'
  repo: https://github.com/dmcable/spacexr
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:e0a664c88992eef41b570ee50eb0b5254aa3b378cd53214640f2ba9a02b3ba80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - PASTE
  - RCTD
  - SpatialScope
  - StarDist
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cell-type-identification-potts-model-map-inference

## Summary

SpatialScope's cell type identification step assigns a discrete cell type to each nucleus/cell segmented from spatial transcriptomics spots via MAP inference under a Potts (spatial smoothness) model with hyperparameter ν = 10, using a labeled scRNA-seq reference and optionally 3D-aligned adjacent slices for more reliable spatial priors.

## When to use

Use when you need per-cell (not per-spot) discrete cell type labels for ST spots (Visium, Slide-seq), given nucleus segmentation outputs and a labeled scRNA-seq reference, with spatial smoothness enforced.

## When NOT to use

- Spot-level cell-type proportions suffice and single-cell resolution is not required — deconvolution methods such as RCTD, Cell2location, CARD, or spatialDWLS already estimate proportions per spot.
- No spatial coordinates for spots/cells are available, since the Potts prior relies on spatial smoothness across neighboring cells/slices.
- No labeled scRNA-seq reference is available, or platform effects between ST and reference data cannot be corrected.

## Inputs

- Seq-based ST dataset with spot-level counts (e.g., two adjacent 10x Visium mouse brain cortex sagittal slices, 812 and 794 cortex spots after filtering)
- Nucleus segmentation outputs from H&E images: cell count and cell_locations per spot (StarDist)
- Labeled scRNA-seq reference (e.g., Smart-seq2 mouse brain cortical reference, 14,249 cells, 23 cell types from the VISp area)
- Optional: 3D alignment of adjacent slices computed with PASTE for spatial priors

## Outputs

- Per-cell discrete cell type labels (CellTypeLabel_nu10.csv)
- Layered cell type map (e.g., L2/3, L4, L5, L6 glutamatergic cortical layers) for downstream visualization and comparison with RCTD

## How to apply

After nucleus segmentation (StarDist for H&E images, per SpatialScope default) yields cell counts and cell_locations per spot, preprocess the scRNA-seq reference (e.g., the Smart-seq2 mouse VISp reference of 14,249 cells across 23 cell types) with total counts normalization and log(1+x) transformation, then select the top 1000 most highly variable genes plus 50 top marker genes per cell type as training genes. Run MAP/Potts-model inference on the segmented cells with the smoothness hyperparameter ν = 10 default; when adjacent slices are available, align them with PASTE so inference incorporates spatial information in 3D space for more reliable spatial priors. Save the discrete per-cell labels as CellTypeLabel_nu10.csv and validate that assignments reproduce known spatial structure (e.g., L4 and L6 IT cells placed in their corresponding cortical layers).

## Related tools

- **SpatialScope** (Runs the MAP/Potts-model cell type identification with smoothness hyperparameter ν = 10 to assign labels to segmented cells) — https://github.com/YangLabHKUST/SpatialScope
- **StarDist** (Nucleus segmentation on H&E images (SpatialScope default) producing cell counts and cell_locations per spot that feed the identification step)
- **PASTE** (Pairwise alignment of adjacent slices to construct 3D ST data so the Potts model can use 3D spatial information for more reliable spatial priors)
- **RCTD** (Baseline comparator: StarDist+RCTD misidentifies L4 and L6 IT cells in other layers due to lack of spatial smoothness constraint) — https://github.com/dmcable/spacexr

## Examples

```
$ git clone https://github.com/YangLabHKUST/SpatialScope.git && cd SpatialScope && conda env create -f environment.yml
```

## Evaluation signals

- CellTypeLabel_nu10.csv exists and contains one discrete cell type label per segmented cell (matching cell_locations counts per spot).
- Layered validation: L4 and L6 IT cells are assigned to their corresponding cortical layers, unlike the StarDist+RCTD comparison that misidentifies these cells.
- The inference used the smoothness hyperparameter ν = 10 (default) and, where applicable, the PASTE-aligned 3D ST data.
- Reference preprocessing followed the recipe: total counts normalization, log(1+x) transformation, top 1000 HVGs plus 50 top marker genes per cell type as training genes.
- Platform effects are appropriately corrected before/alongside identification, since uncorrected platform effects yield unsatisfactory results.

## Limitations

- The Potts model assumption that neighboring cells belong to the same cell type may not always be valid, biasing labels in heterogeneous regions.
- Cell type identification results would not be satisfactory if platform effects between ST and scRNA-seq data are not appropriately corrected.
- No software version numbers or computational environment details are reported in the article.

## Evidence

- [methods] SpatialScope's three-step pipeline is cited from the methods section.: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [methods] Reference gene selection recipe for the identification step.: [evidence span withheld — anchor status: not_found]
- [discussion] Potts prior limitation from the discussion.: "the assumption that neighboring cells belong to the same cell type may not always be valid"
- [discussion] Platform-effect correction requirement from the discussion.: "Cell type identification and gene expression decomposition results would not be satisfactory if the platform effects are not appropriately corrected."
- [readme] Concrete repository installation entry point from the SpatialScope README.: "$ git clone https://github.com/YangLabHKUST/SpatialScope.git"
