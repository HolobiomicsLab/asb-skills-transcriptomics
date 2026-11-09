---
name: semi-simulated-st-data-construction
description: Use when you need paired SC-ST data with known spot-level cell-type ground
  truth to validate spatial mapping/deconvolution methods, starting from single-cell
  data with global x/y coordinates.
when_to_use_negative:
- You need a true biological ST dataset with tissue-level spatial structure, not a
  synthetic benchmark — the article applies STEM to real hSCC and hepatic lobule ST
  data for discovery
- Your single-cell data lack global spatial coordinates, so no grid intersections
  over tissue can be defined
- You are evaluating ST-side deconvolution only and existing spot-level data with
  cell-type priors already suffice
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: STEM / scSTEM
  role: 'Downstream consumer of the semi-simulated data: trained on the pseudo-ST
    spots vs. raw SeqFISH cells to benchmark spatially-aware embedding transfer (dropout
    rate computed as 0.5837 for E1z2; sigma=3, alpha=0.8, n_genes=351)'
  repo: https://github.com/WhirlFirst/STEM
- name: pandas
  role: Loads the SeqFISH counts, cell metadata, and writes the three CSV artifacts
    (simu_st_counts.csv, simu_st_metadata.csv, simu_st_celltype.csv)
- name: scanpy
  role: sc.pp.normalize_total and sc.pp.log1p are applied to the constructed pseudo-ST
    data before model training; sc.pp.calculate_qc_metrics gives median n_genes_by_counts
    for the dropout-rate calculation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:0c796b7d5bb27a98f24b8e959acbd7078eac35ccda7604196d3bdb74622f1076
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - STEM
  - STEM / scSTEM
  - pandas
  - scSTEM
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# semi-simulated-st-data-construction

## Summary

Synthesizes pseudo-Visium-like ST spots from the Spatial Mouse Atlas seqFISH mouse embryo single-cell slides (E1z2/E2z2/E3z2) by summing single-cell expression at spatial grid intersections, producing ground-truth ST data for benchmarking SC-ST mapping methods like STEM.

## When to use

Use when you need paired SC-ST data with known spot-level cell-type ground truth to validate spatial mapping/deconvolution methods, starting from single-cell data with global x/y coordinates.

## When NOT to use

- You need a true biological ST dataset with tissue-level spatial structure, not a synthetic benchmark — the article applies STEM to real hSCC and hepatic lobule ST data for discovery
- Your single-cell data lack global spatial coordinates, so no grid intersections over tissue can be defined
- You are evaluating ST-side deconvolution only and existing spot-level data with cell-type priors already suffice

## Inputs

- SeqFISH mouse embryo single-cell expression matrix (e.g. data/mousedata_2020/E1z2/simu_sc_counts.csv, genes x cells or cells x genes CSV)
- Cell metadata with global x/y coordinates and cell-type annotations (data/mousedata_2020/E1z2/metadata.csv from Figshare DOI 10.6084/m9.figshare.24452812)
- Grid definition parameters: 30x40 spot layout, ~50-70% covered-cell sampling fraction, minimum 3 cells per spot

## Outputs

- simu_st_counts.csv — pseudo-spot gene expression matrix (spots x 351 genes)
- simu_st_metadata.csv — pseudo-spot spatial coordinate table
- simu_st_celltype.csv — spot-level cell-type proportion ground-truth table

## How to apply

Load the raw SeqFISH single-cell expression matrix and cell metadata (x_global/y_global coordinates and cell-type annotations) for the E1z2/E2z2/E3z2 embryo slides from the Figshare deposit (DOI 10.6084/m9.figshare.24452812) with pandas. Define a 30x40 grid of pseudo-spot locations over each slide's spatial coordinates; each spot mimics a 10X Visium capture location. For each grid intersection, identify the covered single cells and compute the spot's gene expression as the summation of the expression of a portion (~50-70%) of the covered cells, mimicking Visium capture efficiency. Discard pseudo-spots covering fewer than three cells. Assemble three artifacts: simu_st_counts.csv (spot-by-gene expression), simu_st_metadata.csv (spot coordinates), and simu_st_celltype.csv (spot-level cell-type proportion ground truth); validation is that these reproduce the deposited semi-simulated files used for Figure 2 of the paper.

## Related tools

- **STEM / scSTEM** (Downstream consumer of the semi-simulated data: trained on the pseudo-ST spots vs. raw SeqFISH cells to benchmark spatially-aware embedding transfer (dropout rate computed as 0.5837 for E1z2; sigma=3, alpha=0.8, n_genes=351)) — https://github.com/WhirlFirst/STEM
- **pandas** (Loads the SeqFISH counts, cell metadata, and writes the three CSV artifacts (simu_st_counts.csv, simu_st_metadata.csv, simu_st_celltype.csv))
- **scanpy** (sc.pp.normalize_total and sc.pp.log1p are applied to the constructed pseudo-ST data before model training; sc.pp.calculate_qc_metrics gives median n_genes_by_counts for the dropout-rate calculation)

## Examples

```
import pandas as pd; scdata = pd.read_csv('./data/mousedata_2020/E1z2/simu_sc_counts.csv', index_col=0).T; scmeta = pd.read_csv('./data/mousedata_2020/E1z2/metadata.csv', index_col=0)
```

## Evaluation signals

- The three outputs (simu_st_counts.csv, simu_st_metadata.csv, simu_st_celltype.csv) reproduce the Figshare-deposited semi-simulated files used for Figure 2 of the paper
- Every retained pseudo-spot covers at least three single cells; spots covering fewer are discarded
- Each spot's expression equals the summed expression of its covered cells (with ~50-70% cell sampling), and the gene panel matches the 351 genes used as opt.n_genes in STEM training
- Spot coordinates form the 30x40 grid layout consistent with 10X Visium spot adjacency; the derived median n_genes_by_counts ratio yields a dropout rate consistent with the README value 0.5836734693877551 for E1z2

## Limitations

- The semi-simulation covers only a portion of single cells per spot (~50-70%) and discards spots with <3 cells, so it is a partial simulation of Visium capture, not an exact generative model
- The synthetic spots derive from the same embryo slides as the SC data, so the benchmark does not test cross-donor transfer; applying STEM to SC and ST data from different donors remains untested future work
- Downstream interpretation relies on the dropout-rate heuristic (1 - median SC n_genes_by_counts / median ST n_genes_by_counts), which is dataset-specific (0.5837 for E1z2)

## Evidence

- [other] Pseudo-ST spots were generated at grid intersections on the tissue slide, covering a portion of the single cells, and each spot's gene expression was computed as the summation of the expression of all covered cells.: [evidence span withheld — anchor status: not_found]
- [other] Discard pseudo-spots that cover fewer than three cells.: [evidence span withheld — anchor status: not_found]
- [readme] First, we load the simulated ST data as ST data and the raw SeqFISH data as SC data. Then we normalized and log-scaled these data.: "First, we load the simulated ST data as ST data and the raw SeqFISH data as SC data. Then we normalized and log-scaled these data."
- [readme] The processed data and trained models used for reproducing the results are deposited in [Figshare](https://doi.org/10.6084/m9.figshare.24452812).: "The processed data and trained models used for reproducing the results are deposited in [Figshare](https://doi.org/10.6084/m9.figshare.24452812)."
- [abstract] Semi-simulation and real data experiments verify that the embeddings preserved spatial information and eliminated technical biases between SC and ST data.: "Semi-simulation and real data experiments verify that the embeddings preserved spatial information and eliminated technical biases between SC and ST data."
