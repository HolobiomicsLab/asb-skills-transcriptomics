---
name: prosstt-trajectory-simulation
description: Use when a benchmark requires synthetic scRNA-seq datasets with known
  developmental trajectory or cluster structure (e.g. ~2000 cells x 5000 genes) to
  evaluate dimensionality-reduction or clustering methods.
when_to_use_negative:
- The goal is to run scGAE itself on data — this skill only produces the synthetic
  benchmark inputs, not the embedding
- The benchmark needs empirical (not simulated) scRNA-seq data — for scGAE use the
  HSPCs (HRA000084) or pancreatic cells (GSE84133) datasets instead
- Cell or gene counts other than the article's 2000 cells x 5000 genes design are
  required without revalidating dimensions
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PROSSTT
  role: Python simulator used to generate the trajectory-type simulated datasets data2
    and data5 (2000 cells, 5000 genes each)
- name: Splatter
  role: R simulator used to generate data1, data3, data4 and the scenario4 zero-inflation
    variants via dropout.shape=-1 and dropout.mid in (-0.5, 0, 0.5, 1)
- name: scGAE
  role: Downstream method being benchmarked on the simulated datasets; its repository
    notebook Run_scGAE.ipynb documents where the simulation code is found
  repo: github.com/ZixiangLuo1161/scGAE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:761b3cd61617dd3dd4ad222536bba428a06e244e569cdbe3f4948e660b0b2a7e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PROSSTT
  - Splatter
  - scGAE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# prosstt-trajectory-simulation

## Summary

Generates simulated single-cell RNA-seq count matrices mimicking trajectory and discrete-cluster scenarios using the PROSSTT Python package, alongside Splatter for zero-inflated variants, to benchmark methods such as scGAE on controlled synthetic data with defined dimensions.

## When to use

Use when a benchmark requires synthetic scRNA-seq datasets with known developmental trajectory or cluster structure (e.g. ~2000 cells x 5000 genes) to evaluate dimensionality-reduction or clustering methods.

## When NOT to use

- The goal is to run scGAE itself on data — this skill only produces the synthetic benchmark inputs, not the embedding
- The benchmark needs empirical (not simulated) scRNA-seq data — for scGAE use the HSPCs (HRA000084) or pancreatic cells (GSE84133) datasets instead
- Cell or gene counts other than the article's 2000 cells x 5000 genes design are required without revalidating dimensions

## Inputs

- Splatter R parameter scripts for Splatter-simulated scenarios (data1, data3, data4, and the scenario4 dropout variants)
- PROSSTT Python parameter scripts for trajectory-simulated scenarios (data2, data5)
- Simulation configuration from the repository notebook Run_scGAE.ipynb

## Outputs

- Nine simulated count matrices (data1-data5 plus four scenario4 zero-inflation variants), each with 2000 cells x 5000 genes
- Files on disk in CSV or h5ad format
- Zero-inflation variants spanning approximately 12%, 17%, 23%, and 30% dropout

## How to apply

Follow the regeneration procedure documented in the scGAE repository notebook Run_scGAE.ipynb, which points to the simulation code. Use PROSSTT (Python) to generate the trajectory-type scenarios (data2 and data5), each simulated with 2000 cells and 5000 genes; use the Splatter R package for data1, data3, and data4. For the zero-inflation stress test (scenario4), apply Splatter dropout with shape=-1 and dropout.mid set to -0.5, 0, 0.5, and 1, yielding approximately 12%, 17%, 23%, and 30% zero inflation respectively. Write each of the resulting nine count matrices (data1-data5 plus the four scenario4 variants) to disk as CSV or h5ad, then validate each file by reloading it and confirming exactly 2000 rows (cells) and 5000 columns (genes).

## Related tools

- **PROSSTT** (Python simulator used to generate the trajectory-type simulated datasets data2 and data5 (2000 cells, 5000 genes each))
- **Splatter** (R simulator used to generate data1, data3, data4 and the scenario4 zero-inflation variants via dropout.shape=-1 and dropout.mid in (-0.5, 0, 0.5, 1))
- **scGAE** (Downstream method being benchmarked on the simulated datasets; its repository notebook Run_scGAE.ipynb documents where the simulation code is found) — github.com/ZixiangLuo1161/scGAE

## Evaluation signals

- Each output file loads successfully as a valid CSV or h5ad count matrix
- Every simulated matrix has exactly 2000 rows (cells) and 5000 columns (genes)
- The four scenario4 variants exhibit approximately 12%, 17%, 23%, and 30% zero inflation, matching dropout.mid = -0.5, 0, 0.5, 1 with dropout.shape=-1
- All nine expected files (data1-data5 plus four scenario4 variants) are present on disk before downstream benchmarking

## Limitations

- No random seed or stochastic-variation control is reported for the simulations in the article context, so exact reproduction of the simulated matrices is not guaranteed
- No software/package versions are specified for the simulation tools; only scGAE's own runtime dependencies (e.g. tensorflow 2.3.0, spektral 0.6.1, scanpy 1.8.0) are listed in the README
- The scenario4 zero-inflation grid covers only the ~12-30% dropout range tested in the scGAE benchmark

## Evidence

- [other] PROSSTT simulation parameters: [evidence span withheld — anchor status: not_found]
- [other] scenario4 dropout grid: [evidence span withheld — anchor status: not_found]
- [readme] Location of simulation code: [evidence span withheld — anchor status: external_doc]
