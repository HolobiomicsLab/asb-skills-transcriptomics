---
name: singlet-doublet-classification-slideseq-spots
description: Use when processing Slide-seq (or other sub-cellular-spot, image-free)
  ST data where no nucleus segmentation is possible and each spot's cell count must
  be estimated before cell-type mapping or gene decomposition.
when_to_use_negative:
- Image-based ST data (Visium, MERFISH) with a paired histology or DAPI image — use
  true nucleus segmentation instead, since Step 1 nucleus segmentation serves as the
  building block when images are available.
- Spots larger than ~two nuclei (e.g., Visium's 55 μm spots often contain multiple
  cells), where the at-most-two-cells assumption is invalid and full deconvolution
  methods like Cell2location or CARD are more appropriate.
- Unlabeled or unpaired scRNA-seq references, since RCTD requires cell-type-labeled
  reference profiles with platform-effect normalization.
tools:
- name: SpatialScope
  role: Pipeline that consumes the singlet/doublet labels as the replacement for Step
    1 nucleus segmentation before cell type identification and gene expression decomposition
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: RCTD (spacexr)
  role: Run in doublet mode to classify each Slide-seq spot as singlet or doublet
    and estimate the cell types / proportions per spot
  repo: https://github.com/dmcable/spacexr
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:eb80a1154506be73e57e01c7ffbf7d623df275d82ee7ccdb989c53516e47a828
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - RCTD
  - RCTD (spacexr)
  - SpatialScope
  - Tangram
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# singlet-doublet-classification-slideseq-spots

## Summary

Replaces image-based nucleus segmentation for Spot-based ST platforms lacking histology (Slide-seq V2 10 μm spots) with an RCTD-inspired singlet/doublet classification, deciding whether one or two cells co-exist per spot so downstream cell type identification and gene expression decomposition can proceed at single-cell resolution.

## When to use

Use when processing Slide-seq (or other sub-cellular-spot, image-free) ST data where no nucleus segmentation is possible and each spot's cell count must be estimated before cell-type mapping or gene decomposition.

## When NOT to use

- Image-based ST data (Visium, MERFISH) with a paired histology or DAPI image — use true nucleus segmentation instead, since Step 1 nucleus segmentation serves as the building block when images are available.
- Spots larger than ~two nuclei (e.g., Visium's 55 μm spots often contain multiple cells), where the at-most-two-cells assumption is invalid and full deconvolution methods like Cell2location or CARD are more appropriate.
- Unlabeled or unpaired scRNA-seq references, since RCTD requires cell-type-labeled reference profiles with platform-effect normalization.

## Inputs

- Slide-seq V2 spatial dataset filtered to genes with non-zero counts and spots with total UMI counts >= 100 (20,141 genes, 8,952 spots in the cerebellum case)
- Paired labeled scRNA-seq reference (e.g., mouse cerebellum snRNA-seq: 24,387 genes, 15,609 cells, 19 cell types)

## Outputs

- Per-spot singlet/doublet classification with RCTD-inferred cell type labels
- Per-spot cell counts (e.g., 10,975 cells across 8,952 spots: 6,929 singlet and 2,023 doublet spots) consumed by SpatialScope Step 2 (cell type identification)

## How to apply

Run RCTD (from the spacexr package) in doublet mode on the filtered Slide-seq dataset, treating each 10 μm spot as a pixel that may source RNA from at most two cells, after platform-effect normalization against the paired scRNA-seq reference (e.g., the mouse cerebellum snRNA-seq atlas of 24,387 genes, 15,609 cells, 19 cell types). The classification outputs per-spot singlet/doublet labels plus assigned cell types, which SpatialScope then converts into a per-spot cell count feeding Step 1 of its three-step pipeline (nucleus segmentation → cell type identification → gene expression decomposition). In the cerebellum experiment this yielded 10,975 cells from 8,952 spots (6,929 singlet spots and 2,023 doublet spots). Rationale: Slide-seq spots are smaller than Visium spots (~55 μm) and typically hold 1–2 nuclei, so the at-most-two-cells assumption bounds the decomposition while RCTD's platform-effect normalization corrects capture-rate differences between the ST and scRNA-seq technologies.

## Related tools

- **SpatialScope** (Pipeline that consumes the singlet/doublet labels as the replacement for Step 1 nucleus segmentation before cell type identification and gene expression decomposition) — https://github.com/YangLabHKUST/SpatialScope
- **RCTD (spacexr)** (Run in doublet mode to classify each Slide-seq spot as singlet or doublet and estimate the cell types / proportions per spot) — https://github.com/dmcable/spacexr

## Examples

```
devtools::install_github("dmcable/spacexr", build_vignettes = FALSE)  # then run RCTD doublet mode per https://github.com/dmcable/spacexr guidelines
```

## Evaluation signals

- Total inferred cells should equal singlet spots + 2 × doublet spots and match the expected tissue cellularity (article: 10,975 cells across 8,952 spots, with 6,929 singlet and 2,023 doublet spots).
- Every spot must receive a classification label (singlet or doublet) — zero unclassified spots after running on the filtered dataset.
- Downstream validation: cell type identification with the Potts-model spatial smoothness prior (ν=10) and subsequent dropout-correction (MAE/PCC vs. Tangram) should only be trusted if Step 1 cell counts are consistent.
- RCTD's platform-effect normalization should have been applied — check that scRNA-seq cell type profiles were normalized to the Slide-seq platform effects before fitting.

## Limitations

- The at-most-two-cells-per-spot assumption is a hard ceiling: a Slide-seq spot with three or more cells will be misclassified.
- Accuracy depends on correcting platform effects; without appropriate correction, downstream cell type identification and gene expression decomposition results are unsatisfactory.
- Classification results feed directly into gene expression decomposition, so errors here propagate to posterior-mean expression estimates.

## Evidence

- [methods] Singlet/Doublet classification replaces nucleus segmentation: "we replace the first step nucleus segmentation by singlet/doublet classification, which assumes that at most two cell types co-exist within a Slide-seq spot"
- [discussion] Nucleus segmentation as Step 1 building block: "Noted that Step 1, Nucleus segmentation serves as a building block to quantify the cell count at each spot."
- [methods] Three-step SpatialScope pipeline context: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [discussion] Platform effects must be corrected: "Cell type identification and gene expression decomposition results would not be satisfactory if the platform effects are not appropriately corrected."
- [readme] RCTD platform-effect normalization (README): [evidence span withheld — anchor status: not_found]
