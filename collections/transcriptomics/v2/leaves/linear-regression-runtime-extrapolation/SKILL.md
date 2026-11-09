---
name: linear-regression-runtime-extrapolation
description: Use when comparing the computational cost (runtime, peak memory) of scDNA–scRNA
  integration methods as input size scales, and you need a quantitative extrapolation
  beyond measured cell counts.
when_to_use_negative:
- You need accuracy/assignment-quality comparisons rather than computational cost
  scaling (use cell-to-clone assignment accuracy evaluation instead).
- Input data has no per-patient structure or you cannot preserve it during subsampling,
  since the benchmark subsamples RNA and DNA cells within each patient.
- Runtime is not dominated by a deterministic solver step, so a linear extrapolation
  of solve time to larger sizes is not justified.
tools:
- name: MaCroDNA
  role: Subject of the benchmark; instantiated as MaCroDNA(rna_df, dna_df) and timed
    via macrodna.cell2cell_assignment() at each sampling rate
  repo: github.com/NakhlehLab/MaCroDNA
- name: Gurobi
  role: Mixed-integer optimizer invoked by MaCroDNA's cell2cell_assignment; its MIP
    solve (gap tolerance 1.00e-04) dominates measured runtime
- name: pandas
  role: Holds the scDNA (dna_df) and scRNA (rna_df) count matrices and the mapping
    outputs used in the timing runs
- name: clonealign
  role: Comparison method benchmarked against MaCroDNA for runtime and peak-memory
    scaling on the same dataset
- name: Seurat
  role: Comparison method benchmarked as the representative of commonly used single-cell
    multi-omic approaches
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:077c93cc774810d08d76877e885b5ec1844613b4477d0c83c55dca66b394c2c1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gurobi
  - MaCroDNA
  - Seurat
  - clonealign
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# linear-regression-runtime-extrapolation

## Summary

Benchmarks wall-clock runtime and peak memory of MaCroDNA against clonealign and Seurat by subsampling cells at fixed rates, fitting a linear regression to the measured runtimes, and extrapolating to a larger cell count.

## When to use

Use when comparing the computational cost (runtime, peak memory) of scDNA–scRNA integration methods as input size scales, and you need a quantitative extrapolation beyond measured cell counts.

## When NOT to use

- You need accuracy/assignment-quality comparisons rather than computational cost scaling (use cell-to-clone assignment accuracy evaluation instead).
- Input data has no per-patient structure or you cannot preserve it during subsampling, since the benchmark subsamples RNA and DNA cells within each patient.
- Runtime is not dominated by a deterministic solver step, so a linear extrapolation of solve time to larger sizes is not justified.

## Inputs

- CRC patient scDNA copy-number count matrix (genes x cells, pandas dataframe)
- CRC patient scRNA expression count matrix (genes x cells, pandas dataframe)
- Clone labels for scDNA cells (dna_label dataframe with columns 'clone' and 'cell')
- Sampling rates lambda in [0.5, 0.6, 0.7, 0.8, 0.9, 1.0]
- intNMF-inferred cluster labels for the three methods under comparison

## Outputs

- scaling_curves.png plotting runtime and peak memory versus number of RNA cells per lambda
- runtime_extrapolation.csv with fitted linear-regression coefficients and predicted runtime at 20,000 cells (~10 minutes)
- Per-run records of wall-clock time (time.perf_counter) and peak memory (psutil/tracemalloc)

## How to apply

Load the CRC patient scDNA copy-number and scRNA expression count matrices into pandas dataframes dna_df and rna_df (genes as rows, cells as columns). For each sampling rate lambda in [0.5, 0.6, 0.7, 0.8, 0.9, 1.0], randomly subset RNA and DNA cells within each patient to that fraction, keeping per-patient structure intact. At each rate, instantiate MaCroDNA(rna_df, dna_df), run macrodna.cell2cell_assignment(), and record wall-clock time with time.perf_counter and peak memory with psutil/tracemalloc; note the Gurobi MIP solve (gap tolerance 1.00e-04) dominates runtime, so timing captures the optimizer cost. Repeat for competitor methods (clonealign, Seurat) on clusters inferred by intNMF to obtain comparable points. Fit a linear regression of runtime versus number of RNA cells to the measured points, report the fitted coefficients and R-squared, and extrapolate to the target size (e.g. 20,000 cells).

## Related tools

- **MaCroDNA** (Subject of the benchmark; instantiated as MaCroDNA(rna_df, dna_df) and timed via macrodna.cell2cell_assignment() at each sampling rate) — github.com/NakhlehLab/MaCroDNA
- **Gurobi** (Mixed-integer optimizer invoked by MaCroDNA's cell2cell_assignment; its MIP solve (gap tolerance 1.00e-04) dominates measured runtime)
- **pandas** (Holds the scDNA (dna_df) and scRNA (rna_df) count matrices and the mapping outputs used in the timing runs)
- **clonealign** (Comparison method benchmarked against MaCroDNA for runtime and peak-memory scaling on the same dataset)
- **Seurat** (Comparison method benchmarked as the representative of commonly used single-cell multi-omic approaches)

## Examples

```
macrodna = MaCroDNA(rna_df, dna_df); cell2cell, cell2cell_step = macrodna.cell2cell_assignment()
```

## Evaluation signals

- Linear regression of runtime versus cell count yields R-squared > 0.96 for MaCroDNA's measured points.
- Extrapolated runtime at 20,000 cells is approximately 10 minutes.
- MaCroDNA's peak memory at 835 cells is 0.18 GB, the lowest among MaCroDNA, clonealign, and Seurat.
- Measured runtime and memory points reproduce the monotone scaling trends of the paper's performance figure.
- Each timed run's Gurobi log reports an optimal solution with gap 0.0000% at MIP gap tolerance 1.00e-04.

## Limitations

- The measured runtime is dominated by the Gurobi MIP solve, so timing reflects optimizer cost rather than all pipeline overhead.
- Matching cells is distinctively challenging in homogeneous samples, which can affect benchmarking in low-heterogeneity datasets.
- The linear extrapolation is fitted to measured points from a limited size range; reliability beyond the fitted range (e.g. 20,000 cells) rests on the linearity assumption.
- There is no ground truth cell correspondence for the BE dataset, so computational benchmarking on that dataset cannot be paired with accuracy validation.

## Evidence

- [other] A linear regression fitted to MaCroDNA's measured runtimes (R2 > 0.96) extrapolates to approximately 10 minutes of runtime for 20,000 cells: [evidence span withheld — anchor status: not_found]
- [other] each call invokes the Gurobi mixed-integer optimizer (MIP gap tolerance 1.00e-04) whose solve time dominates the measured runtime: [evidence span withheld — anchor status: not_found]
- [readme] For mapping RNA cells to DNA cells, the input should be two dataframes `dna_df` and `rna_df`. In these two dataframes, the row ids should be the genes and the column ids should be the cells.: "For mapping RNA cells to DNA cells, the input should be two dataframes `dna_df` and `rna_df`."
- [discussion] We assessed MaCroDNA's performance against clonealign21—as the state-of-the-art method for integrating the scRNA-seq and scDNA-seq data and Seurat19—as the representative of the commonly used methods: "and Seurat19—as the representative of the commonly used methods in the single-cell multi-omic studies—through their application to a CRC dataset31"
- [methods] MaCroDNA is under the directory `src/MaCroDNA/macrodna.py`: "MaCroDNA is under the directory `src/MaCroDNA/macrodna.py`"
