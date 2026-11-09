---
name: comparative-method-performance-evaluation
description: Use when a paper claims one method is 'fast' or memory-efficient versus
  rivals and the runtime/RAM ordering must be reproduced by running each tool on identical
  ST and scRNA-seq inputs and profiling execution.
when_to_use_negative:
- The goal is deconvolution accuracy evaluation (e.g. MAE/MSE per cell type) rather
  than runtime and RAM benchmarking — use model evaluation metrics like calculateEvalMetrics
  instead.
- Only one method is being run, with no competing methods to compare against — there
  is no comparative ordering to establish.
- The compared methods are run on different inputs or hardware, making runtime/RAM
  numbers non-comparable.
tools:
- name: SpatialDDLS
  role: 'Method under benchmark: neural-network-based deconvolution whose wall-clock
    time and peak RAM are profiled through createSpatialDDLSobject, genMixedCellProp,
    simMixedProfiles (threads = 3), trainDeconvModel (60 epochs) and deconvSpatialDDLS'
  repo: github.com/diegommcc/SpatialDDLS
- name: SpatialDDLSdata
  role: Supplies the benchmark inputs (MouseDLN.ST and MouseDLN.SCE) installed via
    devtools::install_github so all three methods run on identical data
  repo: github.com/diegommcc/SpatialDDLSdata
- name: RCTD
  role: Competing deconvolution method run on the same ST dataset and scRNA-seq reference
    to record wall-clock running time and peak RAM for the comparison table
- name: cell2location
  role: Competing deconvolution method run on the same inputs; reported as the slowest
    but with the lowest peak RAM of the three methods
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae072
    title: 'SpatialDDLS: an R package to deconvolute spatial transcriptomics data
      using neural networks'
  source_content_hash: sha256:341b5606755d0f6e36c1dd3aafd66247fe2054d4054fdd80af9b29790926d3b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RCTD
  - SingleCellExperiment
  - SpatialDDLS
  - SpatialDDLSdata
  - SpatialExperiment
  - cell2location
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# comparative-method-performance-evaluation

## Summary

Benchmarks the computational cost (wall-clock running time and peak RAM consumption) of competing cell type deconvolution methods — here SpatialDDLS, RCTD, and cell2location — on the same spatial transcriptomics inputs, to substantiate claims such as SpatialDDLS being an accurate and fast alternative to state-of-the-art tools.

## When to use

Use when a paper claims one method is 'fast' or memory-efficient versus rivals and the runtime/RAM ordering must be reproduced by running each tool on identical ST and scRNA-seq inputs and profiling execution.

## When NOT to use

- The goal is deconvolution accuracy evaluation (e.g. MAE/MSE per cell type) rather than runtime and RAM benchmarking — use model evaluation metrics like calculateEvalMetrics instead.
- Only one method is being run, with no competing methods to compare against — there is no comparative ordering to establish.
- The compared methods are run on different inputs or hardware, making runtime/RAM numbers non-comparable.

## Inputs

- 10x Visium spatial transcriptomics dataset (MouseDLN.ST, SpatialExperiment, from GEO accession GSE173778)
- Paired 10x Chromium scRNA-seq reference (MouseDLN.SCE, SingleCellExperiment)
- Executable installations of each compared method (SpatialDDLS R package with TensorFlow/Keras Python env, RCTD, cell2location)

## Outputs

- Per-method wall-clock running time measurements across the benchmark datasets
- Per-method peak RAM consumption measurements
- Aggregated runtime/RAM comparison table (performance_benchmark_table.csv)

## How to apply

Run every competing method on the same dataset pair so timing and memory comparisons are fair: load the murine lymph node 10x Visium ST data (MouseDLN.ST, GSE173778) and the paired 10x Chromium scRNA-seq reference (MouseDLN.SCE) via the SpatialDDLSdata package. For SpatialDDLS, build the object with createSpatialDDLSobject (sc.filt.genes.cluster = TRUE, sc.n.genes.per.cluster = 150, sc.min.mean.counts = 2), simulate 10,000 mixed transcriptional profiles of 50 cells (genMixedCellProp with min.zero.prop = 5; simMixedProfiles with threads = 3), train the fully connected network for 60 epochs (trainDeconvModel), and deconvolve (deconvSpatialDDLS, k.spots = 6, fast.pca = TRUE), recording wall-clock time and peak RAM at each resource-relevant step. Then run RCTD and cell2location on the same ST slides and scRNA-seq reference under identical measurement conditions, recording the same two metrics. Aggregate per-method measurements into a single runtime/RAM comparison table (performance_benchmark_table.csv) and validate that the compiled ordering matches the reported one.

## Related tools

- **SpatialDDLS** (Method under benchmark: neural-network-based deconvolution whose wall-clock time and peak RAM are profiled through createSpatialDDLSobject, genMixedCellProp, simMixedProfiles (threads = 3), trainDeconvModel (60 epochs) and deconvSpatialDDLS) — github.com/diegommcc/SpatialDDLS
- **SpatialDDLSdata** (Supplies the benchmark inputs (MouseDLN.ST and MouseDLN.SCE) installed via devtools::install_github so all three methods run on identical data) — github.com/diegommcc/SpatialDDLSdata
- **RCTD** (Competing deconvolution method run on the same ST dataset and scRNA-seq reference to record wall-clock running time and peak RAM for the comparison table)
- **cell2location** (Competing deconvolution method run on the same inputs; reported as the slowest but with the lowest peak RAM of the three methods)

## Examples

```
mouseDLN.SDDLS <- simMixedProfiles(mouseDLN.SDDLS, threads = 3)
```

## Evaluation signals

- The compiled performance table reproduces the reported ordering: SpatialDDLS runtime comparable to RCTD, cell2location the slowest of the three methods.
- cell2location shows the lowest peak RAM of the three methods, consistent with the reported RAM superiority over SpatialDDLS and RCTD.
- All three methods were run on the same inputs (MouseDLN.ST and MouseDLN.SCE from GSE173778), so per-method measurements are directly comparable.
- Both metrics (wall-clock running time and peak RAM) are recorded for every method, not just one, before aggregation into performance_benchmark_table.csv.
- SpatialDDLS-specific parameters used in timing (10,000 simulated profiles of 50 cells, min.zero.prop = 5, 60 training epochs, threads = 3, k.spots = 6, fast.pca = TRUE) match the paper's benchmark configuration.

## Limitations

- Runtime and RAM measurements are hardware- and configuration-dependent (e.g. simMixedProfiles threads = 3, fast.pca = TRUE), so absolute values may differ on other machines even if the ordering holds.
- The SpatialDDLS R package depends on tensorflow and keras R packages and needs a working Python interpreter with the TensorFlow library (installable via installTFpython), which is a prerequisite for running the benchmark.
- Benchmark conclusions are grounded in the murine lymph node 10x Visium dataset (GSE173778); ordering may not generalize to other tissues, platforms, or dataset sizes.

## Evidence

- [abstract] we demonstrate that SpatialDDLS is an accurate and fast alternative to the available state-of-the art tools: "we demonstrate that SpatialDDLS is an accurate and fast alternative to the available state-of-the art tools"
- [readme] Overall, SpatialDDLS is a powerful tool for cell type deconvolution in spatial transcriptomics data, providing a reliable, fast and flexible solution for researchers in the field.: "Overall, SpatialDDLS is a powerful tool for cell type deconvolution in spatial transcriptomics data, providing a reliable, fast and flexible solution for researchers in the field."
- [methods] The raw data is publicly available on GEO ([GSE173778](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE173778)): "The raw data is publicly available on GEO ([GSE173778](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE173778))"
