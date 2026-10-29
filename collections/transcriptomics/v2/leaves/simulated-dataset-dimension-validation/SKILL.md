---
name: simulated-dataset-dimension-validation
description: Use when regenerating or receiving simulated scRNA-seq datasets for benchmarking
  (Splatter/PROSSTT output) and you must confirm each file is a valid CSV/h5ad count
  matrix at the canonical 2000×5000 dimension.
when_to_use_negative:
- Input is already a validated empirical dataset (e.g. HSPCs HRA000084 or pancreatic
  cells GSE84133) whose dimensions are inherent to the source study rather than the
  2000×5000 simulation spec
- The simulated matrices are intended for a different benchmark with cell/gene counts
  other than 2000 cells and 5000 genes
- Raw sequencing output that has not yet been reduced to a cell-by-gene count matrix
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Splatter
  role: R package used to generate simulated scenarios data1, data3, data4 and the
    scenario4 zero-inflation variants via dropout.shape=-1 and dropout.mid settings
- name: PROSSTT
  role: Python simulator used to generate data2 and data5 at 2000 cells × 5000 genes
- name: scGAE
  role: Downstream method whose Run_scGAE.ipynb notebook documents where the simulation
    parameter scripts are found
  repo: https://github.com/ZixiangLuo1161/scGAE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:21103a48ebdc4e82ae9aebfc17a18550bf4795174f57dbd2bea4fce2d9da4b53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZixiangLuo1161/scGAE
  tools:
  - PROSSTT
  - Splatter
  - scGAE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# simulated-dataset-dimension-validation

## Summary

Validates that regenerated simulated scRNA-seq benchmark matrices (data1–data5 plus the four scenario4 zero-inflation variants from the scGAE benchmark) are well-formed count matrices with exactly 2000 cells (rows) and 5000 genes (columns) before downstream analysis.

## When to use

Use when regenerating or receiving simulated scRNA-seq datasets for benchmarking (Splatter/PROSSTT output) and you must confirm each file is a valid CSV/h5ad count matrix at the canonical 2000×5000 dimension.

## When NOT to use

- Input is already a validated empirical dataset (e.g. HSPCs HRA000084 or pancreatic cells GSE84133) whose dimensions are inherent to the source study rather than the 2000×5000 simulation spec
- The simulated matrices are intended for a different benchmark with cell/gene counts other than 2000 cells and 5000 genes
- Raw sequencing output that has not yet been reduced to a cell-by-gene count matrix

## Inputs

- Simulated scRNA-seq count matrices data1, data3, data4 (Splatter, 2000 cells × 5000 genes)
- Simulated scRNA-seq count matrices data2, data5 (PROSSTT, 2000 cells × 5000 genes)
- Four scenario4 zero-inflation variant matrices (dropout.shape=-1; dropout.mid -0.5/0/0.5/1)
- Simulation parameter scripts documented in Run_scGAE.ipynb

## Outputs

- Nine validated CSV or h5ad count matrix files (data1–data5 plus four scenario4 variants)
- Per-file dimension confirmation: exactly 2000 rows (cells) × 5000 columns (genes)

## How to apply

Generate data1, data3, and data4 with the Splatter R package and data2 and data5 with PROSSTT, each simulated with 2000 cells and 5000 genes per the Run_scGAE.ipynb simulation scripts. Produce the four scenario4 zero-inflation variants by applying dropout with shape=-1 and dropout.mid in {-0.5, 0, 0.5, 1}, yielding 12%, 17%, 23%, and 30% dropout respectively. Write each of the nine count matrices to disk as CSV or h5ad. Then reload every file and programmatically confirm it parses as a valid count matrix whose dimensions are exactly 2000 rows (cells) by 5000 columns (genes); any file failing to load or deviating from 2000×5000 must be regenerated.

## Related tools

- **Splatter** (R package used to generate simulated scenarios data1, data3, data4 and the scenario4 zero-inflation variants via dropout.shape=-1 and dropout.mid settings)
- **PROSSTT** (Python simulator used to generate data2 and data5 at 2000 cells × 5000 genes)
- **scGAE** (Downstream method whose Run_scGAE.ipynb notebook documents where the simulation parameter scripts are found) — https://github.com/ZixiangLuo1161/scGAE

## Examples

```
import anndata as ad; a = ad.read_h5ad('data1.h5ad'); assert a.shape == (2000, 5000), 'dimensions must be 2000 cells x 5000 genes'
```

## Evaluation signals

- Every one of the nine output files loads without error as a valid CSV or h5ad count matrix
- Each file has exactly 2000 rows (cells) and 5000 columns (genes)
- The four scenario4 variants exhibit dropout rates of approximately 12%, 17%, 23%, and 30% corresponding to dropout.mid = -0.5, 0, 0.5, 1
- All nine files required by the benchmark are present on disk (data1–data5 plus the four scenario4 variants)
- data1/data3/data4 trace back to Splatter scripts while data2/data5 trace back to PROSSTT, matching the documented simulation assignment

## Limitations

- The 12–30% dropout rates for scenario4 variants are approximate targets from the specified dropout.mid grid, not exact guarantees
- No random seed or stochastic-variation control is reported in the source work, so regenerated simulations may vary run-to-run
- No software/package versions are specified for the simulation tools in the paper; README versions (e.g. scanpy 1.8.0, anndata 0.7.4) pertain to the scGAE model environment

## Evidence

- [other] Generate data1, data3, and data4 with the Splatter R package, each simulated with 2000 cells and 5000 genes.: [evidence span withheld — anchor status: not_found]
- [other] scenario4 zero-inflation dropout parameters: [evidence span withheld — anchor status: not_found]
- [readme] notebook pointer: [evidence span withheld — anchor status: external_doc]
