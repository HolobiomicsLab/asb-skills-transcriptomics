---
name: splatter-parameterized-simulation
description: Use when regenerating the simulated scRNA-seq scenarios needed to benchmark
  scGAE, or when controlled-dropout synthetic single-cell count matrices of exactly
  2000 cells x 5000 genes are required.
when_to_use_negative:
- The goal is to run scGAE on empirical data (e.g. HSPCs HRA000084 or pancreatic cells
  GSE84133) rather than to regenerate the simulated benchmark scenarios
- A different cell/gene dimensionality than 2000 x 5000 is required, since the validated
  benchmark outputs are fixed at exactly 2000 cells x 5000 genes
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Splatter
  role: R package used to generate data1, data3, and data4, each simulated with 2000
    cells and 5000 genes, and to apply dropout (dropout.shape=-1, varied dropout.mid)
    for the scenario4 zero-inflation variants
- name: PROSSTT
  role: Python simulator used to generate data2 and data5, likewise with 2000 cells
    and 5000 genes
- name: scGAE
  role: Downstream method the nine simulated count matrices are generated to benchmark;
    simulation parameters are documented in its Run_scGAE.ipynb notebook
  repo: github.com/ZixiangLuo1161/scGAE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:c24ca721446617a2919dd096b235bdd61cb10f5380aca511317106ffb2abad92
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PROSSTT
  - R
  - Splatter
  - scGAE
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- splatter-parameterized-simulation-design
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# splatter-parameterized-simulation

## Summary

Reproduce the five simulated scRNA-seq benchmark datasets (data1-data5) used to evaluate scGAE by generating 2000-cell x 5000-gene count matrices with the Splatter R package (data1, data3, data4) and PROSSTT Python (data2, data5), plus four zero-inflated scenario4 variants with controlled dropout rates.

## When to use

Use when regenerating the simulated scRNA-seq scenarios needed to benchmark scGAE, or when controlled-dropout synthetic single-cell count matrices of exactly 2000 cells x 5000 genes are required.

## When NOT to use

- The goal is to run scGAE on empirical data (e.g. HSPCs HRA000084 or pancreatic cells GSE84133) rather than to regenerate the simulated benchmark scenarios
- A different cell/gene dimensionality than 2000 x 5000 is required, since the validated benchmark outputs are fixed at exactly 2000 cells x 5000 genes

## Inputs

- Splatter simulation parameter scripts (documented in Run_scGAE.ipynb)
- Simulation configuration for PROSSTT (Python) for data2 and data5
- Dropout parameters for scenario4 variants: dropout.shape=-1, dropout.mid in (-0.5, 0, 0.5, 1)

## Outputs

- Nine count matrices as CSV or h5ad files: data1, data2, data3, data4, data5, plus four scenario4 zero-inflation variants
- Each matrix validated as 2000 rows (cells) x 5000 columns (genes) with expected dropout rates of 12%, 17%, 23%, and 30% for the scenario4 variants

## How to apply

Load the simulation parameter scripts referenced in the repository notebook Run_scGAE.ipynb, which documents where the simulation code lives. Generate data1, data3, and data4 with the Splatter R package, each simulated with 2000 cells and 5000 genes; generate data2 and data5 with PROSSTT in Python with the same dimensions. For the four scenario4 zero-inflation variants, apply dropout with dropout.shape=-1 and dropout.mid set to -0.5, 0, 0.5, and 1, which yields dropout rates of approximately 12%, 17%, 23%, and 30% respectively. Write each of the nine count matrices (data1-data5 plus the four scenario4 variants) to disk as CSV or h5ad, then validate each file by reloading it and confirming it is a valid count matrix with exactly 2000 rows (cells) and 5000 columns (genes).

## Related tools

- **Splatter** (R package used to generate data1, data3, and data4, each simulated with 2000 cells and 5000 genes, and to apply dropout (dropout.shape=-1, varied dropout.mid) for the scenario4 zero-inflation variants)
- **PROSSTT** (Python simulator used to generate data2 and data5, likewise with 2000 cells and 5000 genes)
- **scGAE** (Downstream method the nine simulated count matrices are generated to benchmark; simulation parameters are documented in its Run_scGAE.ipynb notebook) — github.com/ZixiangLuo1161/scGAE

## Examples

```
library(splatter); sim <- splatSimulate(newSplatParams(nGenes=5000, batchCells=2000, dropout.shape=-1, dropout.mid=0.5), method="groups")
```

## Evaluation signals

- Each output file loads as a valid CSV or h5ad count matrix with exactly 2000 rows (cells) and 5000 columns (genes)
- The four scenario4 variants show dropout rates of approximately 12%, 17%, 23%, and 30% corresponding to dropout.mid values of -0.5, 0, 0.5, and 1 with dropout.shape=-1
- Nine distinct matrices are produced in total (data1-data5 plus four scenario4 variants), covering both Splatter- and PROSSTT-generated scenarios
- data1, data3, and data4 are traceable to Splatter while data2 and data5 are traceable to PROSSTT, matching the documented tool-to-scenario assignment

## Limitations

- No software/package versions are specified for the simulation tools in the article or README, so exact reproducibility of the simulated matrices may require the Run_scGAE.ipynb scripts
- The optimal K for KNN graph construction varies among datasets in the downstream scGAE analysis, so simulated scenarios may need per-scenario tuning when used for benchmarking

## Evidence

- [other] Generate data1, data3, and data4 with the Splatter R package, each simulated with 2000 cells and 5000 genes.: [evidence span withheld — anchor status: not_found]
- [other] scenario4 zero-inflation variants: [evidence span withheld — anchor status: not_found]
- [readme] Run_scGAE.ipynb reference: [evidence span withheld — anchor status: external_doc]
