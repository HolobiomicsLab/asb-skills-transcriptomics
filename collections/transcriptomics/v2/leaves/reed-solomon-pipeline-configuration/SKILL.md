---
name: reed-solomon-pipeline-configuration
description: Use when evaluating whether a Reed-Solomon (e.g. RS(9,47)) DNA storage
  encoding pipeline recovers data under injected faults, or when benchmarking FrameD's
  transparent MPI parallelism across fault-injection iterations.
when_to_use_negative:
- For nanopore-channel simulations with the DNArSim error model — use the examples/DNArSim
  configuration path instead of the i.i.d. RS(9,47) fault-injection setup
- On a single commodity workstation when reproducing the 240 full-scale paper simulations
  — README states 'This will only be tractable on HPC systems, and at the moment these
  commands will only work on HPC systems with an LSF scheduler'
- For quick pipeline/error-model testing where only a small encode/decode exercise
  is needed — use examples/small (12 cores) rather than the full RS fault-injection
  campaign
edam_operation: http://edamontology.org/operation_2938
tools:
- name: FrameD
  role: Fault-injection simulation infrastructure providing core encoding/decoding
    support and transparent MPI parallelism for the RS(9,47) pipeline runs
  repo: github.com/dna-storage/framed
- name: DNArSim
  role: Alternative nanopore-channel simulator referenced alongside FrameD's i.i.d.
    fault-injection configurations; not used for the RS(9,47) i.i.d. scaling benchmark
    itself
  repo: github.com/BHam-1/DNArSim
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad572
    title: 'FrameD: framework for DNA-based data storage design, verification, and
      validation'
  source_content_hash: sha256:e879a7518e175f338c0e8f6a77c812098cd302b82fdf4d30a37dfe203dec4233
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DNArSim
  - FrameD
  edam_operation: http://edamontology.org/operation_2938
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# reed-solomon-pipeline-configuration

## Summary

Configure FrameD's fault-injection test bed for an RS(9,47) DNA storage pipeline with 25x sequencing read depth, a 10% injected fault rate, and 368 simulation iterations, then measure scaling from serial execution to 32 and 92 MPI ranks.

## When to use

Use when evaluating whether a Reed-Solomon (e.g. RS(9,47)) DNA storage encoding pipeline recovers data under injected faults, or when benchmarking FrameD's transparent MPI parallelism across fault-injection iterations.

## When NOT to use

- For nanopore-channel simulations with the DNArSim error model — use the examples/DNArSim configuration path instead of the i.i.d. RS(9,47) fault-injection setup
- On a single commodity workstation when reproducing the 240 full-scale paper simulations — README states 'This will only be tractable on HPC systems, and at the moment these commands will only work on HPC systems with an LSF scheduler'
- For quick pipeline/error-model testing where only a small encode/decode exercise is needed — use examples/small (12 cores) rather than the full RS fault-injection campaign

## Inputs

- FrameD .json pipeline configuration file specifying RS(9,47) encoding, 25x read depth, 10% fault rate, 368 fault-injection iterations
- C-shell job-generation script (tcsh) targeting the HPC scheduler
- HPC environment with MPI implementation (Intel MPI 2017 or OpenMPI 4.1.5) and LSF scheduler (or adapted submit.py)

## Outputs

- Per-run wall-clock times and computed speedup factors (T_serial / T_parallel) for 1, 32, and 92 ranks, recorded in benchmark_results.csv
- 'fi.stats' human-readable statistics files in each fault-injection leaf directory
- Aggregated Pandas dataframe (fault_injection_dataframe) from db_gen.py for figure generation via notebooks/framed.ipynb

## How to apply

Set up the FrameD pipeline with the RS(9,47) code, 25x sequencing read depth, 10% fault injection rate, and 368 fault-injection simulation iterations, using the .json configuration files under examples/ and the accompanying C-shell job-generation scripts. Run the identical configuration serially on one core of an Intel Xeon Gold6226R node (reference wall-clock ~20325 s), then re-run under FrameD's transparent MPI parallelism with 32 ranks (~802 s, 25.3x speedup) and 92 ranks (65.99x speedup), on an HPC platform with an MPI implementation installed. FrameD assumes an IBM LSF managed platform for batch-level parallelism; extend the TcshJob class in tools/lsf/submit.py for Slurm/Cobalt, or use the subshell background-job mode in examples/small for commodity hardware. After runs complete, compile per-leaf 'fi.stats' files with db_gen.py into a Pandas dataframe and compute speedup as T_serial / T_parallel.

## Related tools

- **FrameD** (Fault-injection simulation infrastructure providing core encoding/decoding support and transparent MPI parallelism for the RS(9,47) pipeline runs) — github.com/dna-storage/framed
- **DNArSim** (Alternative nanopore-channel simulator referenced alongside FrameD's i.i.d. fault-injection configurations; not used for the RS(9,47) i.i.d. scaling benchmark itself) — github.com/BHam-1/DNArSim

## Examples

```
cd $DNASTORAGE_HOME; tcsh examples/small/run_small.csh
```

## Evaluation signals

- Serial (1-core) wall-clock time matches ~20325 s on an Intel Xeon Gold6226R node
- Speedup at 32 MPI ranks matches ~25.3x and at 92 ranks matches ~65.99x
- All 368 fault-injection iterations per configuration complete without failure, and all 240 paper simulations are launched by the job-generation script
- Each leaf result directory contains a readable 'fi.stats' file, and db_gen.py successfully builds fault_injection_dataframe at the top results path
- Computed speedup equals T_serial / T_parallel and reproduces the paper figures when run through notebooks/framed.ipynb

## Limitations

- Full paper-scale job generation only works on HPC systems with an IBM LSF scheduler; Slurm/Cobalt require extending the LSFJob/TcshJob class in tools/lsf/lsf_utils/submit.py
- Multiple sequence alignment in the pipeline is dependent on error rate and injected error type and is not always beneficial, so pipeline stage choices must be justified per configuration
- Encoding or decoding large files may perform poorly on standard computers; the infrastructure performs best on HPC systems with many cores and MPI installed

## Evidence

- [abstract] FrameD provides the user with transparent parallelism to deal with a large number of reads from sequencing and the need for many fault injection iterations: "FrameD provides the user with transparent parallelism to deal with a large number of reads from sequencing and the need for many fault injection iterations"
- [abstract] fault-injection-based test bed: "it provides a fault-injection-based test bed in which DNA storage systems can be evaluated"
- [readme] job generation via c-shell and jsons: [evidence span withheld — anchor status: external_doc]
- [readme] results aggregation with db_gen.py: [evidence span withheld — anchor status: external_doc]
