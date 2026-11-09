---
name: cell-subset-resampling-across-patients
description: Use when you need to characterize computational scaling (wall-clock time,
  peak memory) of MaCroDNA, clonealign, or Seurat on patient-stratified scDNA/scRNA
  data by re-running at controlled sampling rates such as 0.5–1.0.
when_to_use_negative:
- The goal is assignment accuracy rather than computational cost — use the CRC ground-truth
  cell-to-clone and clonal-prevalence evaluation instead
- The dataset lacks paired patient-stratified scDNA/scRNA samples, so within-patient
  stratified subsampling is not meaningful
- Only probabilistic assignment confidence is of interest — MaCroDNA (and hence this
  benchmark target) provides no probabilistic confidence measures unlike clonealign
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_2229
tools:
- name: MaCroDNA
  role: Method under benchmark; instantiated per sampling rate and run via cell2cell_assignment()
    to measure scaling
  repo: github.com/NakhlehLab/MaCroDNA
- name: Gurobi
  role: Mixed-integer optimizer (MIP gap tolerance 1.00e-04) invoked inside cell2cell_assignment();
    its solve time dominates measured runtime
- name: pandas
  role: Holds dna_df and rna_df count matrices and per-rate random cell subsets
- name: Seurat
  role: Baseline method benchmarked for runtime and peak-memory scaling against MaCroDNA
    and clonealign
- name: clonealign
  role: Baseline method benchmarked for runtime and peak-memory scaling; showed the
    poorest performance
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44014-3
    title: Accurate integration of single-cell DNA and RNA for analyzing intratumor
      heterogeneity using MaCroDNA
  source_content_hash: sha256:a364b3d9778af3443aa661beee2585a988b8fbcdf68a6ef66f1736f1f92a8543
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gurobi
  - MaCroDNA
  - Seurat
  - clonealign
  - pandas
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cell-subset-resampling-across-patients

## Summary

Benchmark how a cross-modality single-cell method's runtime and peak memory scale with input size by repeatedly subsampling paired scRNA-seq/scDNA-seq cells within each patient at fixed fractions and re-running the association step.

## When to use

Use when you need to characterize computational scaling (wall-clock time, peak memory) of MaCroDNA, clonealign, or Seurat on patient-stratified scDNA/scRNA data by re-running at controlled sampling rates such as 0.5–1.0.

## When NOT to use

- The goal is assignment accuracy rather than computational cost — use the CRC ground-truth cell-to-clone and clonal-prevalence evaluation instead
- The dataset lacks paired patient-stratified scDNA/scRNA samples, so within-patient stratified subsampling is not meaningful
- Only probabilistic assignment confidence is of interest — MaCroDNA (and hence this benchmark target) provides no probabilistic confidence measures unlike clonealign

## Inputs

- CRC patient scDNA-seq copy-number matrix (pandas dataframe dna_df, genes as rows, cells as columns)
- CRC patient scRNA-seq gene expression count matrix (pandas dataframe rna_df, genes as rows, cells as columns)
- Sampling rates lambda in [0.5, 0.6, 0.7, 0.8, 0.9, 1.0]

## Outputs

- scaling_curves.png plotting runtime and peak memory versus number of RNA cells for each lambda
- runtime_extrapolation.csv with fitted linear-regression coefficients and the predicted runtime (~10 minutes) at 20,000 cells
- Per-λ wall-clock time and peak-memory measurements per method (MaCroDNA, clonealign, Seurat)

## How to apply

Load the CRC patient scDNA copy-number and scRNA expression count matrices into pandas dataframes dna_df and rna_df with genes as rows and cells as columns. For each sampling rate lambda in [0.5, 0.6, 0.7, 0.8, 0.9, 1.0], randomly subset RNA and DNA cells within each patient to that fraction (stratified by patient, mirroring the paper's resampling design). At each rate instantiate MaCroDNA(rna_df, dna_df) and call cell2cell_assignment(), timing with time.perf_counter and tracking peak memory with psutil/tracemalloc; the Gurobi mixed-integer solve (MIP gap tolerance 1.00e-04) dominates runtime, so it must be included in the measurement. Plot runtime and peak memory against RNA cell count per lambda to reproduce the scaling curves, then fit a linear regression of runtime versus cell count and extrapolate to 20,000 cells. Correctness is judged by a monotone scaling trend and an extrapolated runtime near 10 minutes for 20,000 cells (MaCroDNA R2 > 0.96, 0.18 GB for 835 cells).

## Related tools

- **MaCroDNA** (Method under benchmark; instantiated per sampling rate and run via cell2cell_assignment() to measure scaling) — github.com/NakhlehLab/MaCroDNA
- **Gurobi** (Mixed-integer optimizer (MIP gap tolerance 1.00e-04) invoked inside cell2cell_assignment(); its solve time dominates measured runtime)
- **pandas** (Holds dna_df and rna_df count matrices and per-rate random cell subsets)
- **Seurat** (Baseline method benchmarked for runtime and peak-memory scaling against MaCroDNA and clonealign)
- **clonealign** (Baseline method benchmarked for runtime and peak-memory scaling; showed the poorest performance)

## Examples

```
macrodna = MaCroDNA(rna_df, dna_df); t0 = time.perf_counter(); cell2cell, _ = macrodna.cell2cell_assignment(); print(time.perf_counter() - t0, psutil.Process().memory_info().rss)
```

## Evaluation signals

- Linear regression of runtime versus cell count yields R2 > 0.96 for MaCroDNA
- Extrapolated runtime at 20,000 cells is approximately 10 minutes
- MaCroDNA peak memory is the lowest of the three methods (0.18 GB at 835 cells)
- Runtime and memory points increase monotonically with the number of RNA cells at every lambda
- Each MaCroDNA run reports Gurobi reaching an optimal solution with gap 0.0000% (tolerance 1.00e-04)

## Limitations

- Solve time is dominated by the Gurobi MIP step, so measured runtime reflects the optimizer rather than pure data handling
- Results were obtained on the CRC dataset with clusters inferred by intNMF; scaling on other datasets or clustering schemes is not established
- In homogeneous samples, matching cells is distinctively challenging — a biologically driven issue that can complicate runs at any subsample size

## Evidence

- [other] At each sampling rate, instantiate MaCroDNA(rna_df, dna_df) and run macrodna.cell2cell_assignment(), recording wall-clock time: [evidence span withheld — anchor status: not_found]
- [other] Fit a linear regression of runtime versus cell count to the measured points and extrapolate to 20,000 cells: [evidence span withheld — anchor status: not_found]
- [abstract] we demonstrate the advantage of MaCroDNA over existing methods in accuracy and speed: "we demonstrate the advantage of MaCroDNA over existing methods in accuracy and speed"
- [readme] For mapping RNA cells to DNA cells, the input should be two dataframes `dna_df` and `rna_df`: "For mapping RNA cells to DNA cells, the input should be two dataframes `dna_df` and `rna_df`"
- [methods] cell2cell, cell2cell_step = macrodna.cell2cell_assignment(): "cell2cell, cell2cell_step = macrodna.cell2cell_assignment()"
