---
name: parallel-execution-runtime-benchmarking
description: Use when comparing runtime of a parallelized pipeline against slower
  baselines on the same input (e.g., A. thaliana microarray matrix), recording wall-clock
  time and memory alongside AUPRC to show a speed-quality trade-off.
when_to_use_negative:
- When only network quality matters and runtime is irrelevant — a single quality evaluation
  via au_pr_roc suffices without timing instrumentation
- When the input expression data is small enough that all candidate methods finish
  quickly — the runtime comparison against multi-hour baselines (CLR 18.5 h, Arboreto
  44.1 h) is only meaningful at genome scale (tens of thousands of genes)
- When the tool was built with MPI/OpenMP/SIMD disabled — the benchmark would not
  measure the parallel performance being claimed
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0605
tools:
- name: MCPNet
  role: Parallelized GRN reconstruction pipeline whose `mcpnet` executable runs MI
    computation, MCP score calculation, and AUPR/AUROC evaluation in a single timed
    run
  repo: https://github.com/AluruLab/MCPNet
- name: C++
  role: Implementation language enabling multi-core (OpenMP) and MPI multi-node parallelization
    that the runtime benchmark exercises
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad373
    title: 'MCPNet: a parallel maximum capacity-based genome-scale gene network construction
      framework'
  source_content_hash: sha256:56ff2f8e7ccce9f8502e75478912269d4568826f293979a413af8d1eaae53fff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AluruLab/MCPNet
  tools:
  - C++
  - MCPNet
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# parallel-execution-runtime-benchmarking

## Summary

Benchmark wall-clock runtime and memory of a parallel network reconstruction tool (MCPNet) against baseline methods (CLR, Arboreto) on a real dataset, while simultaneously validating output quality via AUPRC. This isolates the performance benefit of multi-core/MPI parallelization from network-quality differences.

## When to use

Use when comparing runtime of a parallelized pipeline against slower baselines on the same input (e.g., A. thaliana microarray matrix), recording wall-clock time and memory alongside AUPRC to show a speed-quality trade-off.

## When NOT to use

- When only network quality matters and runtime is irrelevant — a single quality evaluation via au_pr_roc suffices without timing instrumentation
- When the input expression data is small enough that all candidate methods finish quickly — the runtime comparison against multi-hour baselines (CLR 18.5 h, Arboreto 44.1 h) is only meaningful at genome scale (tens of thousands of genes)
- When the tool was built with MPI/OpenMP/SIMD disabled — the benchmark would not measure the parallel performance being claimed

## Inputs

- Gene expression matrix in HDF5 format (.h5), e.g., A. thaliana Development-category microarray data
- Groundtruth interaction matrix in CSV format (passed via -x)
- Compiled MCPNet binaries in {MCPNet_build_dir}/bin

## Outputs

- Per-method-variant network outputs (output prefix given via -o)
- AUPR and AUROC metrics printed by the pipeline when groundtruth is supplied
- auprc_results.csv containing AUPRC for each method variant (e.g., R4 and M4) and elapsed wall-clock runtime

## How to apply

Build the tool from source with default CMake options (MPI, OpenMP, HDF5, SIMD left ON) so the parallel paths are exercised. Run the full pipeline executable (`mcpnet`) on the gene expression matrix in HDF5 format with method flags `-m 1 2 3`, MI estimator `--mi-method 1`, and `-t 4` threads per compute node, supplying the groundtruth interaction matrix via `-x` so AUPR/AUROC are reported in the same run. Record the wall-clock runtime and peak memory of the run, then compare against baselines run on their own resources (CLR at 18.5 h on 1 CPU core; Arboreto at 44.1 h across eight nodes with 437 GB). Write per-method AUPRC values plus elapsed runtime to a results CSV, and judge success by (a) AUPRC values lying in [0,1] and reproducing published values, and (b) runtime being orders of magnitude below the slowest baseline.

## Related tools

- **MCPNet** (Parallelized GRN reconstruction pipeline whose `mcpnet` executable runs MI computation, MCP score calculation, and AUPR/AUROC evaluation in a single timed run) — https://github.com/AluruLab/MCPNet
- **C++** (Implementation language enabling multi-core (OpenMP) and MPI multi-node parallelization that the runtime benchmark exercises)

## Examples

```
{MCPNet_build_dir}/bin/mcpnet -i {expression}.h5 -o {out_prefix} -m 1 2 3 -t 4 --mi-method 1 -x {groundtruth}.csv
```

## Evaluation signals

- Each reported AUPRC value lies in [0,1]
- R4 and M4 AUPRC values reproduce the Table 4 results, with R4 and M4 ranking best
- Recorded MCPNet runtime is far below the 18.5 h CLR baseline (article reports <10 min on one node)
- Memory footprint stays well below the 437 GB (eight-node) requirement reported for Arboreto
- Pipeline emits AUPR and AUROC for each `-m` method variant when `-x` groundtruth is supplied

## Limitations

- Runtime comparison is only meaningful if the parallel build options (MPI, OpenMP, HDF5, SIMD) are left ON as recommended; disabling them changes the performance profile
- Baseline runtimes (CLR 18.5 h on 1 core; Arboreto 44.1 h on eight nodes with 437 GB) were measured on potentially different hardware, so cross-method timing comparisons are hardware-dependent
- The `-t` thread count is per compute node; scaling results depend on the number of cores/nodes available

## Evidence

- [results] A. thaliana runtime advantage: [evidence span withheld — anchor status: not_found]
- [readme] build rationale: "CMake allows customizing the compile options, including turning on or off support for OpenMP, MPI, HDF5, and SIMD instructions.  We recommend that these be left as default, ON."
- [readme] pipeline capabilities: "this is the primary executable that runs the MCPNet pipeline, including MI computation, MCP score calculation, and AUPR and AUROC metrics if groundtruth is supplied"
- [abstract] scaling claim: "and also scales well to tens of thousands of genes and hundreds of CPU cores"
