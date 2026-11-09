---
name: stochastic-copy-and-error-model-sampling
description: 'Use when evaluating a DNA storage pipeline design under simulated synthesis/sequencing
  noise: you need per-oligo copy counts and error draws (substitutions, deletions,
  decode failures) parameterized by a JSON fault configuration before decoding.'
when_to_use_negative:
- You already have real sequencing reads from a wet-lab experiment — this skill injects
  simulated faults, not measured errors
- You only need to encode/decode a file without evaluating error resilience — use
  FrameD's core encoding module directly without the fault-injection test bed
- Your scheduler is not IBM LSF and you cannot extend TcshJob/LSFJob in submit.py,
  and you cannot run via the provided Docker image or subshell background-job mode
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: FrameD
  role: 'Runs the fault-injection test bed: encodes oligos, generates stochastic copies,
    samples errors per the configured fault distribution, and aggregates statistics
    with transparent parallelism'
  repo: https://github.com/dna-storage/framed
- name: DNArSim
  role: Channel model with memory used as an alternative fault distribution to simulate
    nanopore sequencing errors (synthesis to basecalling) for input oligos
  repo: https://github.com/BHam-1/DNArSim
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad572
    title: 'FrameD: framework for DNA-based data storage design, verification, and
      validation'
  source_content_hash: sha256:7df1ee6126d3cca3dfff2798f20c672f202e066c383117379a39062a52938bfa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dna-storage/framed
  tools:
  - DNArSim
  - FrameD
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Stochastic copy and error model sampling

## Summary

Simulates DNA storage noise by generating the configured number of stochastic copies per encoded oligo and sampling errors from a fault distribution (i.i.d. or a channel model with memory such as DNArSim), so fault-injection experiments can be evaluated across parameter sweeps.

## When to use

Use when evaluating a DNA storage pipeline design under simulated synthesis/sequencing noise: you need per-oligo copy counts and error draws (substitutions, deletions, decode failures) parameterized by a JSON fault configuration before decoding.

## When NOT to use

- You already have real sequencing reads from a wet-lab experiment — this skill injects simulated faults, not measured errors
- You only need to encode/decode a file without evaluating error resilience — use FrameD's core encoding module directly without the fault-injection test bed
- Your scheduler is not IBM LSF and you cannot extend TcshJob/LSFJob in submit.py, and you cannot run via the provided Docker image or subshell background-job mode

## Inputs

- JSON configuration file specifying encoder parameters, decoder parameters, fault distribution parameters, copy distribution parameters, and optional parameter sweep
- Input data file to be encoded into DNA oligos
- Number of copies per oligo and number of fault injection iterations
- Optional DNArSim nanopore channel settings (input FASTA reference, number of reads, channel memory length k)

## Outputs

- Per-parameter-combination output directories containing parameter record files (parameters, decode failures, byte-error and base-error locations)
- A human-readable 'fi.stats' statistics file per parameter combination with aggregated simulation-event counters
- Error-carrying DNA oligo copies (e.g., FASTQ reads when simulating via DNArSim)

## How to apply

Parse the JSON configuration to extract encoder parameters, decoder parameters, fault distribution parameters, copy distribution parameters, and any optional parameter sweep. First encode the input data file into DNA oligos using FrameD's core encoding module. Then, for each parameter combination (expanding the sweep), generate the specified number of copies per oligo and sample errors according to the configured fault distribution — either the i.i.d. error model or the DNArSim nanopore channel model with memory (channel memory length k, default k=6) — leveraging FrameD's transparent parallelism over many fault injection iterations. Reconstruct the pipeline and decode the error-carrying oligos, recording decode failures and byte-/base-error locations in per-parameter parameter record files, then aggregate counters into a single statistics file per parameter-combination output directory.

## Related tools

- **FrameD** (Runs the fault-injection test bed: encodes oligos, generates stochastic copies, samples errors per the configured fault distribution, and aggregates statistics with transparent parallelism) — https://github.com/dna-storage/framed
- **DNArSim** (Channel model with memory used as an alternative fault distribution to simulate nanopore sequencing errors (synthesis to basecalling) for input oligos) — https://github.com/BHam-1/DNArSim

## Examples

```
cd $DNASTORAGE_HOME && tcsh examples/small/run_small.csh
```

## Evaluation signals

- A statistics file ('fi.stats') exists for every parameter combination in the sweep
- Each statistics file contains aggregated counters including decode failures and byte-error and base-error locations
- The number of error-sampled copies matches the copy distribution parameters for each parameter combination
- Fault distribution choice (i.i.d. vs DNArSim nanopore) matches the JSON configuration directory used (examples/iid vs examples/DNArSim)
- Decoded output after error injection can be compared across error rates and injected error types, reproducing pipeline-stage comparisons (e.g., multiple sequence alignment benefit varying with error rate)

## Limitations

- Full paper-scale runs (240 simulations) are only tractable on HPC systems; the provided launch scripts assume an IBM LSF scheduler and must be extended for Slurm/Cobalt via the TcshJob/LSFJob class in tools/lsf/submit.py
- Encoding or decoding large files may perform poorly on standard computers, and MPI must be installed (tested with Intel MPI 2017 and Open MPI 4.1.5)
- The DNArSim error profile is tied to a specific synthesis/sequencing/basecalling setup (Thermofisher GeneArt synthesis, MinION R9.4_180mv_450bps, Guppy v5.0.7 super-accurate mode), so sampled errors may not transfer to other platforms

## Evidence

- [abstract] FrameD provides transparent parallelism for fault injection: "FrameD provides the user with transparent parallelism to deal with a large number of reads from sequencing and the need for many fault injection iterations"
- [abstract] Fault-injection test bed: "it provides a fault-injection-based test bed in which DNA storage systems can be evaluated"
- [readme] MPI plus batch-level parallelism for error evaluation: [evidence span withheld — anchor status: external_doc]
- [readme] fi.stats files in leaf directories: [evidence span withheld — anchor status: external_doc]
- [readme] DNArSim channel invocation with memory length k: [evidence span withheld — anchor status: external_doc]
- [abstract] MSA choice depends on error rate and error type: "We found that the choice to include multiple sequence alignment in the pipeline is dependent on the error rate and the type of errors being injected and is not always beneficial."
