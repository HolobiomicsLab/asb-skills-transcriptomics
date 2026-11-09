---
name: wall-clock-speedup-analysis
description: Use when a FrameD fault-injection pipeline configuration (e.g. RS(9,47)
  code, 25x read depth, 10% fault rate, 368 simulations) must be benchmarked from
  1 core to many MPI ranks to demonstrate scalability.
when_to_use_negative:
- The pipeline has not yet been validated functionally (correct encode/decode under
  fault injection), since a fast-but-wrong simulation measures nothing useful
- Only commodity hardware without an MPI implementation is available; FrameD's HPC
  workflow assumes LSF-managed batch parallelism and the small examples only exercise
  12 cores
- The question is simulation output quality (e.g. whether multiple sequence alignment
  helps at a given error rate) rather than execution-time scalability
tools:
- name: FrameD
  role: Simulation infrastructure whose transparent MPI parallelism is benchmarked;
    runs the fault-injection pipeline serially and under rank scaling
  repo: https://github.com/dna-storage/framed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad572
    title: 'FrameD: framework for DNA-based data storage design, verification, and
      validation'
  source_content_hash: sha256:ab7398f5e812a6c585c9ce8e00508cd4c4dfc9c2a3bb3c251befe28688626fcc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dna-storage/framed
  tools:
  - FrameD
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# wall-clock-speedup-analysis

## Summary

Measures how transparent MPI parallelism in FrameD reduces wall-clock time for fault-injection simulation of DNA storage pipelines, computing speedup factors as T_serial / T_parallel across rank counts.

## When to use

Use when a FrameD fault-injection pipeline configuration (e.g. RS(9,47) code, 25x read depth, 10% fault rate, 368 simulations) must be benchmarked from 1 core to many MPI ranks to demonstrate scalability.

## When NOT to use

- The pipeline has not yet been validated functionally (correct encode/decode under fault injection), since a fast-but-wrong simulation measures nothing useful
- Only commodity hardware without an MPI implementation is available; FrameD's HPC workflow assumes LSF-managed batch parallelism and the small examples only exercise 12 cores
- The question is simulation output quality (e.g. whether multiple sequence alignment helps at a given error rate) rather than execution-time scalability

## Inputs

- FrameD pipeline configuration (e.g. JSON under examples/) specifying the storage code (RS(9,47)), sequencing read depth (25x), injected fault rate (10%), and number of fault-injection simulations (368)
- HPC node with MPI implementation (Intel MPI 2017.1 or Open MPI 4.1.5 tested), serial baseline run on 1 core
- Wall-clock times from serial and each MPI rank count (1, 32, 92)

## Outputs

- benchmark_results.csv with measured wall-clock times per rank count and computed speedup factors (T_serial / T_parallel)
- Speedup values for 32 ranks (target 25.3x) and 92 ranks (target 65.99x) and the serial baseline time (~20325 s)

## How to apply

Configure the FrameD pipeline with the target DNA storage code (e.g. RS(9,47) with 25x sequencing read depth, 10% injected fault rate, and 368 fault-injection simulation iterations), then run the identical configuration serially on 1 core of the Intel Xeon Gold6226R node and record the wall-clock time (~20325 s in the paper). Re-run the exact same configuration under FrameD's transparent MPI parallelism at intermediate rank counts (32 ranks measured 802 s, 25.3x) and at high rank counts (92 ranks measured 65.99x), recording each wall-clock time. Compute speedup as T_serial / T_parallel for every rank count and collect the measured times and speedup factors into benchmark_results.csv. The rationale is that FrameD provides transparent parallelism to handle the large number of sequencing reads and the many fault-injection iterations, so scalability of the full pipeline—not a kernel microbenchmark—is what is being validated.

## Related tools

- **FrameD** (Simulation infrastructure whose transparent MPI parallelism is benchmarked; runs the fault-injection pipeline serially and under rank scaling) — https://github.com/dna-storage/framed

## Examples

```
tcsh examples/small/run_small.csh
```

## Evaluation signals

- Serial baseline wall-clock time matches the reported value (~20325 s on 1 core of the Intel Xeon Gold6226R node)
- 32-rank run reproduces the reported speedup of 25.3x (wall-clock ~802 s)
- 92-rank run reproduces the reported speedup of 65.99x
- Each speedup is computed consistently as T_serial / T_parallel and the pipeline configuration is bit-identical across runs (same code, 25x read depth, 10% fault rate, 368 simulations)
- All measured times and speedup factors are present in benchmark_results.csv and match the values reported in the paper

## Limitations

- The full HPC workflow assumes an IBM LSF managed platform; other schedulers (Slurm, Cobalt) require extending the TcshJob class in tools/lsf/submit.py
- Non-HPC users can only emulate parallelism with subshell background jobs (examples/small uses 12 cores), which limits the reachable speedup regime
- Speedup saturation: 92 ranks yield 65.99x, not linear scaling, so speedup should be reported per rank count rather than assumed to scale with rank count
- Encoding or decoding large files may perform poorly on standard computers, so the serial baseline must be run on the reference HPC node (Intel Xeon Gold6226R) to be comparable

## Evidence

- [abstract] transparent MPI parallelism benchmark: "FrameD provides the user with transparent parallelism to deal with a large number of reads from sequencing and the need for many fault injection iterations"
- [readme] HPC and MPI requirement: [evidence span withheld — anchor status: external_doc]
- [readme] small non-HPC parallel example: [evidence span withheld — anchor status: external_doc]
