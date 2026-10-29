---
name: peak-memory-benchmarking
description: Use when comparing computational cost (runtime, peak memory) of scDNA–scRNA
  integration methods as cell counts scale, or when you need to extrapolate MaCroDNA
  runtime to datasets larger than the measured ones.
when_to_use_negative:
- You need assignment accuracy or clonal-prevalence evaluation rather than computational-cost
  scaling — use the cell-to-clone accuracy benchmark instead
- The dataset has no per-patient structure or ground truth clusters (e.g. the BE dataset),
  so intNMF-based sampling of comparable cell counts is not possible
- Your method is not invoked via a single dominating solve step (MaCroDNA's Gurobi
  MIP dominates timing), so wall-clock timing around the whole call will not attribute
  cost correctly
tools:
- name: MaCroDNA
  role: Method under benchmark; instantiated as MaCroDNA(rna_df, dna_df) and timed
    via macrodna.cell2cell_assignment() at each sampling rate
  repo: https://github.com/NakhlehLab/MaCroDNA
- name: Gurobi
  role: Mixed-integer optimizer invoked by MaCroDNA's cell2cell_assignment; its solve
    time (MIP gap tolerance 1.00e-04) dominates the measured runtime
- name: pandas
  role: Holds and subsets the dna_df and rna_df count matrices at each sampling rate
    lambda
- name: clonealign
  role: Baseline method benchmarked against MaCroDNA for runtime and peak memory
- name: Seurat
  role: Baseline method benchmarked against MaCroDNA for runtime and peak memory
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:681e1e5f09a9ae7be417d1265cec547e94778ca83696ee8f682baca4071347d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NakhlehLab/MaCroDNA
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
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# peak-memory-benchmarking

## Summary

Benchmark wall-clock runtime and peak memory of MaCroDNA against clonealign and Seurat as input size scales, by sampling the CRC scDNA/scRNA data at fractions lambda = 0.5–1.0, timing each Gurobi-driven cell2cell_assignment() call, and extrapolating runtime with a linear regression.

## When to use

Use when comparing computational cost (runtime, peak memory) of scDNA–scRNA integration methods as cell counts scale, or when you need to extrapolate MaCroDNA runtime to datasets larger than the measured ones.

## When NOT to use

- You need assignment accuracy or clonal-prevalence evaluation rather than computational-cost scaling — use the cell-to-clone accuracy benchmark instead
- The dataset has no per-patient structure or ground truth clusters (e.g. the BE dataset), so intNMF-based sampling of comparable cell counts is not possible
- Your method is not invoked via a single dominating solve step (MaCroDNA's Gurobi MIP dominates timing), so wall-clock timing around the whole call will not attribute cost correctly

## Inputs

- CRC patient scDNA-seq copy-number matrix (genes as rows, cells as columns) as a pandas dataframe dna_df
- CRC patient scRNA-seq expression count matrix (genes as rows, cells as columns) as a pandas dataframe rna_df
- Sampling rates lambda in [0.5, 0.6, 0.7, 0.8, 0.9, 1.0] applied per patient

## Outputs

- Per-lambda runtime and peak-memory measurements paired with RNA cell counts (scaling curves, e.g. scaling_curves.png)
- Linear-regression coefficients of runtime versus cell count and extrapolated runtime at 20,000 cells (runtime_extrapolation.csv, ~10 minutes)

## How to apply

Load the CRC patient scDNA copy-number and scRNA expression count matrices into pandas dataframes dna_df and rna_df with genes as rows and cells as columns. For each sampling rate lambda in [0.5, 0.6, 0.7, 0.8, 0.9, 1.0], randomly subset RNA and DNA cells within each patient to that fraction, then instantiate MaCroDNA(rna_df, dna_df) and run macrodna.cell2cell_assignment(), recording wall-clock time with time.perf_counter and peak memory with psutil/tracemalloc. Each call invokes the Gurobi mixed-integer optimizer (MIP gap tolerance 1.00e-04), whose solve time dominates the measured runtime, so timing should bracket the full call. Plot runtime and peak memory against the number of RNA cells for each lambda to reproduce the scaling curves, then fit a linear regression of runtime versus cell count and extrapolate to 20,000 cells. Report the fitted coefficients and the predicted runtime (~10 minutes at 20,000 cells) alongside MaCroDNA's measured memory use (0.18 GB for 835 cells) relative to Seurat and clonealign.

## Related tools

- **MaCroDNA** (Method under benchmark; instantiated as MaCroDNA(rna_df, dna_df) and timed via macrodna.cell2cell_assignment() at each sampling rate) — https://github.com/NakhlehLab/MaCroDNA
- **Gurobi** (Mixed-integer optimizer invoked by MaCroDNA's cell2cell_assignment; its solve time (MIP gap tolerance 1.00e-04) dominates the measured runtime)
- **pandas** (Holds and subsets the dna_df and rna_df count matrices at each sampling rate lambda)
- **clonealign** (Baseline method benchmarked against MaCroDNA for runtime and peak memory)
- **Seurat** (Baseline method benchmarked against MaCroDNA for runtime and peak memory)

## Examples

```
from MaCroDNA import MaCroDNA; macrodna = MaCroDNA(rna_df, dna_df); cell2cell, cell2cell_step = macrodna.cell2cell_assignment()
```

## Evaluation signals

- Linear regression of runtime versus cell count fits with R2 > 0.96
- Extrapolated runtime at 20,000 cells is approximately 10 minutes
- Measured runtime and memory points reproduce monotone scaling trends across lambda = 0.5–1.0
- MaCroDNA shows the lowest peak memory of the three methods (0.18 GB for 835 cells) relative to Seurat and clonealign
- Each timed call reports Gurobi reaching an optimal solution within the 1.00e-04 MIP gap tolerance, confirming the timed run completed rather than timing out

## Limitations

- The measured runtime is dominated by Gurobi MIP solve time, so results are tied to the optimizer, hardware, and license configuration (Gurobi requires a separate install; academic license is free)
- Runtime extrapolation assumes linearity beyond the measured range; the regression is only validated up to the sampled sizes
- MaCroDNA does not provide probabilistic confidence measures for the assignments, unlike statistical methods such as clonealign

## Evidence

- [readme] Optimal solution found (tolerance 1.00e-04): "Optimal solution found (tolerance 1.00e-04)
Best objective 2.830007718086e+00, best bound 2.830007718086e+00, gap 0.0000%"
- [readme] For mapping RNA cells to DNA cells, the input should be two dataframes dna_df and rna_df: "For mapping RNA cells to DNA cells, the input should be two dataframes `dna_df` and 
rna_df`."
- [methods] MaCroDNA is under the directory src/MaCroDNA/macrodna.py: "MaCroDNA is under the directory `src/MaCroDNA/macrodna.py`"
- [readme] MaCroDNA is a correlation-based method to perform mapping between scRNA-seq gene expression and scDNA-seq copy number values: "MaCroDNA (**Ma**tching **Cro**ss-**D**omain **N**ucleic **A**cids) is a correlation-based 
method to perform mapping between scRNA-seq gene expression and scDNA-seq copy number values."
- [abstract] We demonstrate the advantage of MaCroDNA over existing methods in accuracy and speed: "Using ground truth information from colorectal cancer data, we demonstrate the advantage of MaCroDNA over existing methods in accuracy and speed."
