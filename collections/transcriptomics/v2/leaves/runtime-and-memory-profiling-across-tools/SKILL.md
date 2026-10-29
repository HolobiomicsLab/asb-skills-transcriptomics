---
name: runtime-and-memory-profiling-across-tools
description: Use when benchmarking computational pipelines on large-scale scRNA-seq
  data (e.g. >0.6 million cells) under a fixed RAM limit, and you need per-tool peak
  memory, runtime, and crash/completion outcomes as comparable metrics.
when_to_use_negative:
- The dataset is small enough that all candidate tools fit comfortably in available
  RAM — profiling then adds overhead without discriminating between tools
- The goal is statistical accuracy benchmarking only, with no resource constraint
  of interest and no need for memory/time comparability across tools
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: scaLR
  role: Platform under test; pipeline.py accepts -m/--memoryprofiler to log memory
    usage across data processing, feature extraction, training, and evaluation stages
  repo: https://github.com/infocusp/scaLR
- name: Python
  role: Execution environment (tested with Python 3.10.20) in which all compared pipelines
    are run and profiled
- name: scBalance
  role: Comparison annotation pipeline executed under the 128 GB memory limit via
    custom scripts from the scaLR comparison repository
- name: scvi
  role: Comparison annotation pipeline executed under the 128 GB memory limit via
    custom scripts from the scaLR comparison repository
- name: devCellPy
  role: Comparison annotation pipeline executed under the 128 GB memory limit via
    custom scripts from the scaLR comparison repository
- name: cellPLM
  role: Comparison annotation pipeline executed under the 128 GB memory limit; only
    tool besides scaLR able to complete on PBMCs-normal within 128 GB
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:fc27c29ae402a67a0e10d690770afdb6adb6adae7216192218eae03f2306a7e7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/infocusp/scaLR
  repo_url: https://github.com/infocusp/scaLR
  tools:
  - Python
  - cellPLM
  - devCellPy
  - scBalance
  - scaLR
  - scvi
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# runtime-and-memory-profiling-across-tools

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill for systematically recording run completion/failure, peak memory usage, and total runtime for each tool-dataset pair under a fixed resource ceiling (128 GB RAM), used to compare annotation pipelines like scaLR, scBalance, scvi, devCellPy, and CellPLM on large scRNA-seq datasets.

## When to use

Use when benchmarking computational pipelines on large-scale scRNA-seq data (e.g. >0.6 million cells) under a fixed RAM limit, and you need per-tool peak memory, runtime, and crash/completion outcomes as comparable metrics.

## When NOT to use

- The dataset is small enough that all candidate tools fit comfortably in available RAM — profiling then adds overhead without discriminating between tools
- The goal is statistical accuracy benchmarking only, with no resource constraint of interest and no need for memory/time comparability across tools

## Inputs

- Large-scale scRNA-seq benchmark datasets in anndata (.h5ad) format, e.g. PBMCs-normal (>0.6 million cells, ~0.4 million training cells) and PBMCs-SLE from Cellxgene or the Broad Single Cell Portal
- Tool executables/scripts for each annotation pipeline under test (scaLR pipeline.py with config.yaml, plus custom comparison scripts for scBalance, scvi, devCellPy, cellPLM)
- A fixed resource limit definition (e.g. 128 GB RAM)

## Outputs

- Per tool-dataset pair record of run completion or crash
- Peak memory usage per run
- Total runtime per run
- Aggregated benchmark comparison table (e.g. benchmark_comparison.csv)
- For completed runs: precision, recall, f1-score, and accuracy on the test set

## How to apply

Run each tool-dataset pair as a separate job under the same memory ceiling (e.g. 128 GB), using batch partitioning or backed-mode loading where the tool supports it (scaLR reads data in backed mode and trains in batches from disk). For each run, record whether the pipeline completes or crashes, together with peak memory usage and total runtime; scaLR's pipeline.py supports this directly via the -m/--memoryprofiler flag. For runs that complete, additionally evaluate cell-type annotation accuracy on the test set using precision, recall, f1-score, and accuracy. Assemble the completion/failure pattern plus memory and time measurements into a comparison table (e.g. benchmark_comparison.csv). The rationale is that memory footprint, not just accuracy, is a decisive criterion: in the scaLR study only CellPLM could complete on the PBMCs-normal dataset (~0.4M training cells) within 128 GB while all other tested Python tools exceeded the limit.

## Related tools

- **scaLR** (Platform under test; pipeline.py accepts -m/--memoryprofiler to log memory usage across data processing, feature extraction, training, and evaluation stages) — https://github.com/infocusp/scaLR
- **Python** (Execution environment (tested with Python 3.10.20) in which all compared pipelines are run and profiled)
- **scBalance** (Comparison annotation pipeline executed under the 128 GB memory limit via custom scripts from the scaLR comparison repository)
- **scvi** (Comparison annotation pipeline executed under the 128 GB memory limit via custom scripts from the scaLR comparison repository)
- **devCellPy** (Comparison annotation pipeline executed under the 128 GB memory limit via custom scripts from the scaLR comparison repository)
- **cellPLM** (Comparison annotation pipeline executed under the 128 GB memory limit; only tool besides scaLR able to complete on PBMCs-normal within 128 GB)

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m
```

## Evaluation signals

- The completion/failure pattern across tool-dataset pairs reproduces the paper's reported outcomes: baseline tools crash on PBMCs-SLE, and only CellPLM completes on PBMCs-normal within 128 GB
- Each tool-dataset pair has a recorded peak memory value and runtime, including for failed runs where 'more than 128GB' is recorded
- Completed runs' test-set precision, recall, f1-score, and accuracy are comparable against the reference benchmark table (Table 4)
- Runs of the same tool on the same data with -m/--memoryprofiler enabled produce consistent memory traces across repeated executions
- scaLR matches baseline annotation accuracy while requiring less training time and memory

## Limitations

- scaLR's memory usage is inversely proportional to execution time: reducing memory consumption may increase processing time, so memory-runtime trade-offs must be reported together
- Memory profiling results depend on input data format and preprocessing; scaLR's performance may vary based on input data format, requiring users to ensure proper data preprocessing for optimal results
- Comparison tool versions are not stated, and specific hyperparameters (epochs, DNN layers) for final runs are only referenced via sensitivity analysis without reported values, limiting exact reproducibility of timings
- Feature importance cannot be extracted from several comparison tools (scBalance, scvi, devCellPy, CellPLM), so memory/runtime comparisons are the main comparable dimension for those tools

## Evidence

- [other] On the PBMCs-normal dataset (>0.6 million cells, ~0.4 million training cells), only CellPLM could run, while all other tested tools required more than 128GB of memory: [evidence span withheld — anchor status: not_found]
- [other] Record for each tool-dataset pair whether the run completes or crashes, together with peak memory usage and total runtime.: [evidence span withheld — anchor status: not_found]
- [readme] python pipeline.py --config /path/to/config.yaml -l -m to run the scaLR.: [evidence span withheld — anchor status: external_doc]
- [readme] optional flags to log all parts of the pipelines (-l | --log) and to analyze memory usage (-m | --memoryprofiler): [evidence span withheld — anchor status: external_doc]
