---
name: grid-pseudo-spot-coordinate-generation
description: Use when you have single-cell data with global x/y spatial coordinates
  (e.g. seqFISH slides) and need simulated ST spot data with known cell-type ground
  truth to benchmark or validate single-cell-to-spatial mapping methods.
when_to_use_negative:
- You already have real ST spot data and do not need simulated ground truth — deconvolve
  or map directly instead of synthesizing pseudo-spots.
- The single-cell data lacks global spatial coordinates, since grid placement over
  the slide requires per-cell x/y positions.
- You need single-cell-resolution ST benchmarks rather than spot-level mixtures; the
  pseudo-spots discard within-spot single-cell identity except as aggregated cell-type
  proportions.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: STEM
  role: Downstream model that consumes the synthesized pseudo-ST spots and raw SeqFISH
    single-cell data for semi-simulation benchmarking; the simulation itself is assembled
    with pandas before STEM training
  repo: https://github.com/WhirlFirst/STEM
- name: scSTEM
  role: PYPI-installable package of STEM used in the same semi-simulation workflow
  repo: https://github.com/WhirlFirst/STEM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:b785c4c9282243354cea8b454dd82d72f10bfc821195271b92411a81f410257f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - STEM
  - scSTEM
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# grid-pseudo-spot-coordinate-generation

## Summary

Synthesizes pseudo-Visium-like spatial transcriptomics spots from single-cell spatial data by placing a grid of spot locations over tissue slide coordinates and summing the gene expression of the single cells covered by each spot. This creates paired SC/ST ground-truth data for benchmarking spatial mapping methods like STEM in semi-simulation experiments.

## When to use

Use when you have single-cell data with global x/y spatial coordinates (e.g. seqFISH slides) and need simulated ST spot data with known cell-type ground truth to benchmark or validate single-cell-to-spatial mapping methods.

## When NOT to use

- You already have real ST spot data and do not need simulated ground truth — deconvolve or map directly instead of synthesizing pseudo-spots.
- The single-cell data lacks global spatial coordinates, since grid placement over the slide requires per-cell x/y positions.
- You need single-cell-resolution ST benchmarks rather than spot-level mixtures; the pseudo-spots discard within-spot single-cell identity except as aggregated cell-type proportions.

## Inputs

- Single-cell expression count matrix (e.g. raw SeqFISH counts CSV from the Spatial Mouse Atlas, mousedata_2020)
- Cell metadata table with global x/y coordinates and cell-type annotations (metadata.csv)
- Grid parameters: spot grid dimensions (e.g. 30x40) and the fraction of covered cells summed per spot (~50-70%)

## Outputs

- simu_st_counts.csv — pseudo-spot gene expression matrix
- simu_st_metadata.csv — pseudo-spot spatial coordinate metadata table
- simu_st_celltype.csv — spot-level cell-type proportion/ground-truth table

## How to apply

Load the raw single-cell expression matrix and cell metadata (global x/y coordinates, cell-type annotations) from CSV — for the Spatial Mouse Atlas this is the Figshare deposit DOI 10.6084/m9.figshare.24452812 covering the E1z2/E2z2/E3z2 embryo slides. Define a grid of pseudo-spot locations over each slide's spatial coordinates (a 30x40 spatial grid in the paper's semi-simulation), such that each spot covers a portion of the single cells. For each grid intersection, sum the gene expression of approximately 50–70% of the covered cells to form that spot's expression profile, mimicking Visium capture efficiency. Discard pseudo-spots covering fewer than three cells, then assemble and save three artifacts: simu_st_counts.csv (spot-by-gene expression), simu_st_metadata.csv (spot spatial coordinates), and simu_st_celltype.csv (spot-level cell-type proportions serving as ground truth).

## Related tools

- **STEM** (Downstream model that consumes the synthesized pseudo-ST spots and raw SeqFISH single-cell data for semi-simulation benchmarking; the simulation itself is assembled with pandas before STEM training) — https://github.com/WhirlFirst/STEM
- **scSTEM** (PYPI-installable package of STEM used in the same semi-simulation workflow) — https://github.com/WhirlFirst/STEM

## Examples

```
stdata = pd.read_csv('data/mousedata_2020/E1z2/simu_st_counts.csv',index_col=0); spcoor = pd.read_csv('./data/mousedata_2020/E1z2/simu_st_metadata.csv',index_col=0)
```

## Evaluation signals

- The three output artifacts (simu_st_counts.csv, simu_st_metadata.csv, simu_st_celltype.csv) reproduce the deposited semi-simulated ST files used to generate Figure 2 of the paper.
- Every retained pseudo-spot covers at least three single cells; spots covering fewer than three cells are discarded.
- Each spot's expression equals the summed expression of ~50-70% of its covered cells (Visium-like dropout).
- Spot-level cell-type proportions in simu_st_celltype.csv are consistent with the cell-type annotations of the cells covered by each grid spot.
- Number of rows in simu_st_counts.csv matches simu_st_metadata.csv and simu_st_celltype.csv (one row per retained pseudo-spot).

## Limitations

- Pseudo-spots are synthetic Visium-like mixtures; results depend on grid geometry and the chosen 50-70% coverage fraction, so spot characteristics may not capture all real-ST artifacts.
- The semi-simulation is validated only against the Spatial Mouse Atlas seqFISH embryo slides (E1z2/E2z2/E3z2); transfer to other tissues requires re-tuning grid and coverage parameters.
- The article notes applying STEM to SC and ST data from different donors in the same disease state remains untested future work, so simulations assume matched tissue context.

## Evidence

- [methods] Pseudo-ST spots were generated at grid intersections on the tissue slide, covering a portion of the single cells, and each spot's gene expression was computed as the summation of the expression of all covered cells.: [evidence span withheld — anchor status: not_found]
- [readme] The processed data and trained models used for reproducing the results are deposited in [Figshare]: "The processed data and trained models used for reproducing the results are deposited in [Figshare](https://doi.org/10.6084/m9.figshare.24452812)."
- [readme] First, we load the simulated ST data as ST data and the raw SeqFISH data as SC data.: "First, we load the simulated ST data as ST data and the raw SeqFISH data as SC data."
- [abstract] Semi-simulation and real data experiments verify that the embeddings preserved spatial information and eliminated technical biases between SC and ST data.: "Semi-simulation and real data experiments verify that the embeddings preserved spatial information and eliminated technical biases between SC and ST data."
