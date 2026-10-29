---
name: mpi-parallel-benchmark-execution
description: Use when a FrameD fault-injection configuration must be benchmarked for
  parallel scalability, i.e. when evaluating whether FrameD delivers measurable speedups
  from 1 core to 32 and 92 MPI ranks on the identical pipeline settings.
when_to_use_negative:
- Running on commodity hardware without an HPC scheduler/MPI — FrameD's batch-level
  parallelism assumes an IBM LSF managed platform, and the full 368-simulation configuration
  is only tractable on HPC systems
- The analysis goal is fault-injection accuracy (e.g. whether multiple sequence alignment
  helps at a given error rate), not runtime scalability — benchmark timings alone
  cannot answer error-rate questions
- The pipeline configuration differs between the serial and parallel runs (different
  code, read depth, fault rate, or number of simulations) — speedup ratios are then
  not comparable
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: FrameD
  role: Provides transparent MPI parallelism over the fault-injection simulation pipeline
    and the 1-core baseline whose wall-clock times yield the speedup measurements
  repo: github.com/dna-storage/framed
- name: DNArSim
  role: Nanopore channel simulator that can supply the error model for FrameD pipeline
    configurations being benchmarked (as in examples/DNArSim)
  repo: BHam-1/DNArSim
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad572
    title: 'FrameD: framework for DNA-based data storage design, verification, and
      validation'
  source_content_hash: sha256:1412cba4d2ef82ad20282344a9ec46414acaee496ef2a2b7dab92b44e2e9c67b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNArSim
  - FrameD
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# mpi-parallel-benchmark-execution

## Summary

Measures wall-clock scalability of FrameD's fault-injection simulation pipeline under its transparent MPI parallelism, comparing serial runtime against multi-rank runs to compute speedup factors for a fixed DNA storage pipeline configuration (e.g. RS(9,47) code, 25x read depth, 10% fault rate, 368 simulations).

## When to use

Use when a FrameD fault-injection configuration must be benchmarked for parallel scalability, i.e. when evaluating whether FrameD delivers measurable speedups from 1 core to 32 and 92 MPI ranks on the identical pipeline settings.

## When NOT to use

- Running on commodity hardware without an HPC scheduler/MPI — FrameD's batch-level parallelism assumes an IBM LSF managed platform, and the full 368-simulation configuration is only tractable on HPC systems
- The analysis goal is fault-injection accuracy (e.g. whether multiple sequence alignment helps at a given error rate), not runtime scalability — benchmark timings alone cannot answer error-rate questions
- The pipeline configuration differs between the serial and parallel runs (different code, read depth, fault rate, or number of simulations) — speedup ratios are then not comparable

## Inputs

- FrameD pipeline configuration JSON (e.g. under examples/iid for the i.i.d error model) specifying RS(9,47) encoding, 25x read depth, 10% fault rate, 368 fault-injection simulations
- HPC node (Intel Xeon Gold6226R) with an installed MPI implementation (Intel MPI 2017 Update 1 or OpenMPI 4.1.5)
- Identical configuration for each parallelism level: 1 core, 32 MPI ranks, 92 MPI ranks

## Outputs

- Wall-clock runtimes (s) for serial (1 core), 32-rank, and 92-rank executions
- Computed speedup factors (T_serial / T_parallel), e.g. 25.3x at 32 ranks and 65.99x at 92 ranks
- benchmark_results.csv with all measured times and speedup factors
- Per-run fault-injection statistics (fi.stats files in each leaf result directory)

## How to apply

Fix a single pipeline configuration — for the RS(9,47) benchmark this is the RS(9,47) code with 25x sequencing read depth, a 10% injected fault rate, and 368 fault-injection simulation iterations — and hold it identical across all runs. First run the configuration serially on 1 core of the Intel Xeon Gold6226R node and record the wall-clock time (target ~20325 s). Then re-run the identical configuration under FrameD's transparent MPI parallelism, first with 32 ranks (target ~802 s, 25.3x) and again with 92 ranks (target 65.99x). Compute each speedup as T_serial / T_parallel and collect all measured times and speedup factors into benchmark_results.csv. Validate that the recorded speedups and serial time match the published values before claiming scalability.

## Related tools

- **FrameD** (Provides transparent MPI parallelism over the fault-injection simulation pipeline and the 1-core baseline whose wall-clock times yield the speedup measurements) — github.com/dna-storage/framed
- **DNArSim** (Nanopore channel simulator that can supply the error model for FrameD pipeline configurations being benchmarked (as in examples/DNArSim)) — BHam-1/DNArSim

## Examples

```
cd $DNASTORAGE_HOME; tcsh examples/small/run_small.csh
```

## Evaluation signals

- Serial 1-core wall-clock time on the Intel Xeon Gold6226R node matches the published target (~20325 s)
- 32-rank run reproduces ~802 s and a speedup of 25.3x for the RS(9,47), 25x depth, 10% fault rate, 368-simulation configuration
- 92-rank run reproduces a 65.99x speedup
- Each speedup equals T_serial / T_parallel computed from the recorded benchmark_results.csv times
- Identical configuration files (RS(9,47) code, 25x read depth, 10% fault rate, 368 simulations) are verifiably reused across all three runs

## Limitations

- Speedup is sub-linear at high rank counts (65.99x on 92 ranks vs 25.3x on 32 ranks), so scaling beyond tens of ranks yields diminishing returns
- Batch-level parallelism assumes an IBM LSF managed platform; other schedulers (Cobalt, Slurm) require extending the TcshJob/LSFJob class in tools/lsf/submit.py
- Large simulation sets (240+ simulations in the paper examples) are only tractable on HPC systems and will not complete on commodity hardware
- Results depend on the MPI implementation; the code was tested with Intel MPI Library 2017 Update 1 and OpenMPI 4.1.5

## Evidence

- [abstract] FrameD provides the user with transparent parallelism to deal with a large number of reads from sequencing and the need for many fault injection iterations: "FrameD provides the user with transparent parallelism to deal with a large number of reads from sequencing and the need for many fault injection iterations"
- [abstract] available in a public Github repository at https://github.com/dna-storage/framed: "available in a public Github repository at https://github.com/dna-storage/framed"
- [readme] we not only leverage MPI parallelism, but we also rely on batch level parallelism. Our infrastructure currently assumes an IBM LSF managed platform: [evidence span withheld — anchor status: external_doc]
- [readme] The infrastructure to encode, decode, and analyze sequencing data will perform best if using an HPC system, or system with considerble number of cores at its disposal, with an MPI implementation installed.: [evidence span withheld — anchor status: external_doc]
