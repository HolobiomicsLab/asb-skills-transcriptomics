---
name: flow-cytometry-quality-gating
description: Use when raw FCS flow cytometry measurements (e.g. spheroid-derived HCT116
  cells stained for DNA, RNA, Ki-67, p27 and forward scatter) must be cleaned of doublets,
  dead cells and outliers before computing pseudotime ordering.
when_to_use_negative:
- The data are already gated and only pseudotime values are missing (proceed directly
  to trajectory inference)
- The input is single-cell imaging or time-lapse microscopy data rather than cytometric
  FCS measurements
- The downstream analysis does not require a pseudotemporal ordering (gating quality
  specifically supports trajectory inference for MAPiT-style transformations)
edam_operation: http://edamontology.org/operation_3695
edam_topics: []
tools:
- name: MATLAB
  role: Environment in which gated FCS data are loaded, root cells are selected, and
    the 10000-cell subsample is prepared for trajectory inference
- name: cyt3 / Sight
  role: Matlab toolbox used to load FCS-formatted cytometry data; any loaded piece
    of data is treated as a gate
  repo: https://github.com/dpeerlab/cyt3
- name: Wanderlust
  role: Downstream trajectory algorithm applied to the gated, subsampled cells (from
    the Cyt3 toolbox)
  repo: https://github.com/dpeerlab/cyt3
- name: Diffusion Maps / DPT
  role: Alternative trajectory algorithm producing pseudotime values from the gated
    cells
  repo: https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-60400-z
    title: Reconstructing temporal and spatial dynamics from single-cell pseudotime
      using prior knowledge of real scale cell densities
  source_content_hash: sha256:19c55429924cba8d4d772d36625fc60757ce11eb69e4744c72a93d4532c54af8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dpeerlab/cyt3
  tools:
  - Diffusion Maps / DPT
  - MATLAB
  - Wanderlust
  - cyt3 / Sight
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# flow-cytometry-quality-gating

## Summary

Gates preprocessed single-cell flow cytometry measurements (e.g. DNA/RNA/Ki-67/p27 panels) in MATLAB to remove doublets, dead cells and outlier events before trajectory inference, then subsamples ~10000 gated cells for pseudotime algorithms such as Wanderlust or Diffusion Maps.

## When to use

Use when raw FCS flow cytometry measurements (e.g. spheroid-derived HCT116 cells stained for DNA, RNA, Ki-67, p27 and forward scatter) must be cleaned of doublets, dead cells and outliers before computing pseudotime ordering.

## When NOT to use

- The data are already gated and only pseudotime values are missing (proceed directly to trajectory inference)
- The input is single-cell imaging or time-lapse microscopy data rather than cytometric FCS measurements
- The downstream analysis does not require a pseudotemporal ordering (gating quality specifically supports trajectory inference for MAPiT-style transformations)

## Inputs

- Preprocessed single-cell flow cytometry measurements in FCS format (e.g. HCT116 spheroid cells stained for DNA, RNA, Ki-67 and p27 plus forward scatter)
- Marker channel definitions used for gating (forward scatter, DNA content, Ki-67, RNA, p27)
- Choice of anchor/root cell population (G1 DNA peak cells, or high Ki-67 / high RNA cells)

## Outputs

- Gated single-cell population (doublets, dead cells and outliers removed)
- Random subsample of exactly 10000 gated cells
- pseudotime_ordering.csv — an nx1 single-column vector of pseudotime values (one per sampled cell, n = 10000)

## How to apply

Load the preprocessed single-cell flow cytometry measurements (FCS format, importable via the cyt3/Sight interface) into MATLAB and apply gates that remove doublets, dead cells and outlier events from the event set. Decide which channels define quality: forward scatter for doublet/event size, DNA content for cell-cycle integrity, and marker stains (e.g. Ki-67, RNA, p27) for signal sanity. Select anchor root cells from the gated population — cells in the DNA-based G1 peak for cell-cycle data, or cells with high Ki-67 / high RNA signal for spheroid data — since trajectory algorithms require biologically interpretable root cells. Randomly subsample 10000 cells from the gated population to reduce trajectory-algorithm computation time. Export per-cell pseudotime values from the gated subsample as a single-column (nx1) CSV, e.g. pseudotime_ordering.csv, matching the vector input format expected by MAPiT's jointDensityPseudotimeY.m.

## Related tools

- **MATLAB** (Environment in which gated FCS data are loaded, root cells are selected, and the 10000-cell subsample is prepared for trajectory inference)
- **cyt3 / Sight** (Matlab toolbox used to load FCS-formatted cytometry data; any loaded piece of data is treated as a gate) — https://github.com/dpeerlab/cyt3
- **Wanderlust** (Downstream trajectory algorithm applied to the gated, subsampled cells (from the Cyt3 toolbox)) — https://github.com/dpeerlab/cyt3
- **Diffusion Maps / DPT** (Alternative trajectory algorithm producing pseudotime values from the gated cells) — https://www.helmholtz-muenchen.de/icb/research/groups/machine-learning/projects/dpt/index.html

## Evaluation signals

- Gated population contains no doublets, dead cells or outlier events by visual inspection of forward scatter and DNA channels
- Subsample contains exactly 10000 cells drawn from the gated population
- Anchor root cells fall in the DNA-based G1 peak (cell cycle example) or exhibit high Ki-67 / high RNA signal (spheroid example)
- pseudotime_ordering.csv exists, is importable into MATLAB, has exactly one column and 10000 rows (nx1 vector compatible with jointDensityPseudotimeY.m)

## Limitations

- MAPiT and the trajectory step are reliant on the correctness of the provided cell order, so bad gating directly corrupts downstream pseudotime-to-real-scale transformation
- Random subsampling to 10000 cells discards events and may reduce representation of rare subpopulations
- Gating is performed in MATLAB with third-party toolboxes (cyt3), so workflows must be adapted per application case rather than run as an installable package

## Evidence

- [full_text] Load the preprocessed single-cell flow cytometry measurements into MATLAB and gate the data to remove doublets, dead cells and outlier events.: [evidence span withheld — anchor status: not_found]
- [results] Stain and measure cells by flow cytometry for DNA, RNA, Ki-67 and p27 (plus forward scatter).: [evidence span withheld — anchor status: not_found]
- [full_text] Randomly subsample 10000 cells from the gated population to reduce computation time of the trajectory algorithm.: [evidence span withheld — anchor status: not_found]
- [readme] Sight, accepts new data in FCS format. any piece of data is considered a gate.: [evidence span withheld — anchor status: external_doc]
- [intro] The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order.: "The method makes use of pseudotemporal ordering obtained from trajectory inference algorithms and hence is reliant on the correctness of the provided cell order."
