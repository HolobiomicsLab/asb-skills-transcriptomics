---
name: hpc-scaling-benchmarking
description: 'Use when a simulation pipeline (e.g., fault-injection over many sequencing
  reads) must be shown to scale on HPC: serial baseline timing is available and speedup
  must be measured at increasing MPI rank counts.'
when_to_use_negative:
- The pipeline or hardware lacks an MPI implementation, or the workflow needs an unsupported
  scheduler (e.g., SLURM/Cobalt) without extending the TcshJob class in tools/lsf/submit.py
  — the README states the infrastructure assumes an IBM LSF managed platform
- 'The configuration is a small non-HPC test: use the background sub-shell examples
  in examples/small instead of full rank-scaling runs'
- The analysis goal is fault-injection accuracy (error rate / MSA choice) rather than
  performance — scaling benchmarks answer nothing about whether multiple sequence
  alignment is beneficial
tools:
- name: FrameD
  role: Simulation infrastructure whose transparent MPI parallelism is benchmarked;
    runs encoding/decoding and fault-injection pipeline across rank counts
  repo: github.com/dna-storage/framed
- name: DNArSim
  role: Optional external nanopore channel simulator invoked by FrameD configurations
    for simulating nanopore sequencing within the fault-injection runs being scaled
  repo: github.com/BHam-1/DNArSim
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad572
    title: 'FrameD: framework for DNA-based data storage design, verification, and
      validation'
  source_content_hash: sha256:9e7643edaa1ad561896020f19d1ae90c6e3a494823883dd8c08f5580f41174c6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNArSim
  - FrameD
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hpc-scaling-benchmarking

## Summary

Benchmark wall-clock scaling of a fault-injection simulation pipeline under transparent MPI parallelism, computing speedup factors (T_serial / T_parallel) as MPI rank count increases. Applied here to FrameD's RS(9,47) DNA storage pipeline to demonstrate scalability from 1 core to 92 ranks.

## When to use

Use when a simulation pipeline (e.g., fault-injection over many sequencing reads) must be shown to scale on HPC: serial baseline timing is available and speedup must be measured at increasing MPI rank counts.

## When NOT to use

- The pipeline or hardware lacks an MPI implementation, or the workflow needs an unsupported scheduler (e.g., SLURM/Cobalt) without extending the TcshJob class in tools/lsf/submit.py — the README states the infrastructure assumes an IBM LSF managed platform
- The configuration is a small non-HPC test: use the background sub-shell examples in examples/small instead of full rank-scaling runs
- The analysis goal is fault-injection accuracy (error rate / MSA choice) rather than performance — scaling benchmarks answer nothing about whether multiple sequence alignment is beneficial

## Inputs

- FrameD pipeline configuration (e.g., .json config with RS(9,47) code, 25x read depth, 10% fault rate, 368 fault-injection iterations)
- HPC node with MPI implementation installed (tested with Intel MPI 2017 Update 1 and OpenMPI 4.1.5) and a batch scheduler (IBM LSF assumed)
- Serial (1-core) wall-clock baseline measurement

## Outputs

- Wall-clock times at each rank count (1, 32, 92 MPI ranks)
- Computed speedup factors (T_serial / T_parallel), e.g. 25.3x at 32 ranks and 65.99x at 92 ranks
- benchmark_results.csv aggregating measured times and speedup factors

## How to apply

Configure the pipeline once (here: RS(9,47) code, 25x sequencing read depth, 10% injected fault rate, 368 fault-injection simulation iterations) and run it serially on 1 core of the Intel Xeon Gold6226R node, recording wall-clock time (~20325 s). Re-run the identical configuration under FrameD's transparent MPI parallelism at intermediate rank count (32 ranks; target 802 s, 25.3x), then again at higher rank count (92 ranks; target 65.99x). Compute each speedup as T_serial / T_parallel and collect measured times and factors into benchmark_results.csv. Rationale: FrameD provides transparent parallelism to deal with large numbers of reads from sequencing and the need for many fault injection iterations, so scaling behavior is the direct validation that the parallel infrastructure is effective; batch-level parallelism via an LSF-managed platform is assumed per the README.

## Related tools

- **FrameD** (Simulation infrastructure whose transparent MPI parallelism is benchmarked; runs encoding/decoding and fault-injection pipeline across rank counts) — github.com/dna-storage/framed
- **DNArSim** (Optional external nanopore channel simulator invoked by FrameD configurations for simulating nanopore sequencing within the fault-injection runs being scaled) — github.com/BHam-1/DNArSim

## Examples

```
docker pull kvolkel/framed:revision; docker run -it kvolkel/framed:revision
```

## Evaluation signals

- Serial 1-core wall-clock time matches the reported ~20325 s for the RS(9,47) configuration
- 32-rank run yields ~802 s and a speedup of ~25.3x (T_serial / T_parallel)
- 92-rank run yields a speedup of ~65.99x
- All measured times and speedup factors are recorded in benchmark_results.csv and agree with values reported in the paper
- The parallel and serial runs used an identical configuration (same code, 25x read depth, 10% fault rate, 368 simulations) so speedup is attributable to parallelism, not configuration drift

## Limitations

- The full 240-simulation paper workflow requires an HPC system and, at the moment, an LSF scheduler; other schedulers (SLURM, Cobalt) need explicit support added via the TcshJob/LSFJob class in tools/lsf/submit.py
- MPI runtimes matter: the infrastructure was tested with Intel MPI 2017 Update 1 and OpenMPI 4.1.5; other implementations may behave differently
- Speedup is sub-linear (65.99x at 92 ranks), so results should not be extrapolated linearly to higher rank counts
- Single-node vs multi-node runs differ: the 65.99x result came from a multi-node run with 92 compute ranks on Intel Xeon Gold6226R nodes

## Evidence

- [abstract] FrameD provides transparent parallelism to handle large numbers of sequencing reads and many fault injection iterations: "FrameD provides the user with transparent parallelism to deal with a large number of reads from sequencing and the need for many fault injection iterations"
- [readme] we leverage MPI parallelism: [evidence span withheld — anchor status: external_doc]
- [readme] MPI implementations tested: [evidence span withheld — anchor status: not_found]
- [readme] This will only be tractable on HPC systems: [evidence span withheld — anchor status: external_doc]
