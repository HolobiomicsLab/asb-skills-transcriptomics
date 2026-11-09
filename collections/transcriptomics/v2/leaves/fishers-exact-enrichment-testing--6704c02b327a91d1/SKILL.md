---
name: fishers-exact-enrichment-testing
description: Use when comparing a set of cell-type co-localization pairs detected
  on reconstructed or predicted spatial coordinates against a reference set of supported
  co-localizations, to quantify concordance with a p-value.
when_to_use_negative:
- When ground-truth locations exist (ST data with truth cell locations) — use Moran's
  R directly on the reference instead of testing reconstructed coordinates against
  it
- When query SC data has a large gap or mismatch from the ST reference, where forced
  SC-to-ST assignment can introduce bias and reconstructed co-localizations may be
  unreliable
- When comparing continuous spatial expression gradients rather than discrete binary
  cell-type occupancy per location
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CellContrast
  role: Produces reconstructed SC coordinates (spatial_reconstructed_sc.h5ad) via
    encoder representations and weighted SC-SC/SC-ST similarity projection; one of
    the three coordinate sources whose detected co-localization counts are Fisher-tested
  repo: https://github.com/HKU-BAL/CellContrast
- name: CeLEry
  role: Generates a comparative reconstruction (default parameters) of S27 whose co-localization
    detections are Fisher-tested against the reference
- name: SageNet
  role: Produces an MDS projection of predicted cell-cell distances into pseudo space;
    its 38 detected pairs are Fisher-tested (kernel l=1e-4)
- name: SpatialDM
  role: Runs Moran's R with binary cell-type occupancy (n_neighbors=20, radial basis
    kernel l=0.1 or 1e-4) to call significant co-localizations at p<=0.05 that feed
    the Fisher contingency tables
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:348cb716b18aee57dfb771ab8a63746fac83f9e26bcd68e00bbdd46758dae792
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/CellContrast
  tools:
  - CeLEry
  - CellContrast
  - SageNet
  - SpatialDM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Fisher's exact enrichment testing

## Summary

Uses 2x2 Fisher's exact tests to assess whether cell-type co-localization pairs detected on spatially reconstructed single-cell data (e.g. CellContrast/CeLEry/SageNet coordinates of mouse SC atlas sample S27, E8.5) are enriched among reference co-localizations from SeqFISH ground-truth ST datasets.

## When to use

Use when comparing a set of cell-type co-localization pairs detected on reconstructed or predicted spatial coordinates against a reference set of supported co-localizations, to quantify concordance with a p-value.

## When NOT to use

- When ground-truth locations exist (ST data with truth cell locations) — use Moran's R directly on the reference instead of testing reconstructed coordinates against it
- When query SC data has a large gap or mismatch from the ST reference, where forced SC-to-ST assignment can introduce bias and reconstructed co-localizations may be unreliable
- When comparing continuous spatial expression gradients rather than discrete binary cell-type occupancy per location

## Inputs

- spatial_reconstructed_sc.h5ad — spatially reconstructed SC coordinates (CellContrast/CeLEry/SageNet) for mouse SC atlas sample S27 (6,460 cells)
- SeqFISH embryo 1 L1 ST reference (10,150 cells) with 350 overlapping genes and six SeqFISH reference datasets for building reference co-localization patterns
- Cell-type occupancy binary matrix per reconstructed location

## Outputs

- colocalization_results.csv — detected co-localization pair counts and Fisher's exact test p-values for CellContrast, CeLEry, and SageNet
- Per-method lists of significant co-localized cell-type pairs (SpatialDM Moran's R p<=0.05)

## How to apply

First detect co-localizations by encoding a binary cell-type occupancy variable (0 or 1) per location and running SpatialDM Moran's R with n_neighbors=20 and a radial basis kernel l=0.1 for CellContrast and CeLEry coordinates (l=1e-4 for the SageNet MDS pseudo-space), calling significant pairs at p<=0.05. Filter detected cell-type pairs to those with reference support >1 from the six SeqFISH reference datasets so both sides of the contingency table are comparable. Build a 2x2 table of detected/not-detected versus reference-supported (support >1)/not-supported co-localizations and run Fisher's exact test per method. For sample S27 the expected outputs are 42 pairs (CellContrast, p=2.793e-9), 42 pairs (CeLEry, p=1.121e-6), and 38 pairs (SageNet, p=7.409e-6), with 27 co-localizations consistently detected across all three methods; save counts and p-values to colocalization_results.csv.

## Related tools

- **CellContrast** (Produces reconstructed SC coordinates (spatial_reconstructed_sc.h5ad) via encoder representations and weighted SC-SC/SC-ST similarity projection; one of the three coordinate sources whose detected co-localization counts are Fisher-tested) — https://github.com/HKU-BAL/CellContrast
- **CeLEry** (Generates a comparative reconstruction (default parameters) of S27 whose co-localization detections are Fisher-tested against the reference)
- **SageNet** (Produces an MDS projection of predicted cell-cell distances into pseudo space; its 38 detected pairs are Fisher-tested (kernel l=1e-4))
- **SpatialDM** (Runs Moran's R with binary cell-type occupancy (n_neighbors=20, radial basis kernel l=0.1 or 1e-4) to call significant co-localizations at p<=0.05 that feed the Fisher contingency tables)

## Examples

```
from scipy.stats import fisher_exact; odds, p = fisher_exact([[42, 398], [ref_pairs, rest]])  # 2x2 detected-vs-reference co-localization table
```

## Evaluation signals

- Reproduced co-localization counts equal 42 pairs (CellContrast), 42 pairs (CeLEry), and 38 pairs (SageNet)
- Fisher's exact p-values match 2.793e-9 (CellContrast), 1.121e-6 (CeLEry), and 7.409e-6 (SageNet) within reasonable numerical tolerance
- 27 co-localizations are consistently detected across all three methods
- Detected pairs are filtered to reference support >1 from the six SeqFISH reference datasets before testing
- colocalization_results.csv contains counts and p-values for all three methods

## Limitations

- SC-ST mapping introduces a 'cell clumping' issue where multiple neighbors are projected into the same reference location, distorting reconstructed spatial relationships
- Large gaps between ST and SC samples lead to a performance drop in global reconstruction, and forcing SC assignment to resemble ST could introduce more bias
- MDS-based mapping (used for SageNet pseudo-space) loses the pre-knowledge of the reference's cell distribution, and a certain level of mismatch may exist between query SC data and ST reference due to limited ST resources

## Evidence

- [other] Filter detected cell-type pairs to those with reference support >1 from the six SeqFISH reference datasets and compare counts (42 pairs for CellContrast and CeLEry, 38 for SageNet).: [evidence span withheld — anchor status: not_found]
- [other] Encode a binary cell-type occupancy variable (0 or 1) per location and run SpatialDM Moran's R with n_neighbors=20 and radial basis kernel l=0.1: [evidence span withheld — anchor status: not_found]
- [discussion] we reconstructed spatial locations for cells from a mouse embryo on E8.5 and detected cell-type co-localization patterns with Moran’s R: "we reconstructed spatial locations for cells from a mouse embryo on E8.5 and detected cell-type co-localization patterns with Moran’s R"
- [other] calling significant co-localizations at p<=0.05: [evidence span withheld — anchor status: not_found]
- [readme] Output file: spatially reconstructed h5ad file of annData: "Output file: spatially reconstructed h5ad file of annData"
