---
name: encoder-decoder-pipeline-configuration
description: Use when a DNA storage pipeline design must be evaluated under injected
  errors (i.i.d. or nanopore DNArSim channel faults), including parameter sweeps over
  encoder/decoder settings, to compare pipeline stage choices across error rates.
when_to_use_negative:
- When the goal is to simulate the physical sequencing channel itself rather than
  inject faults into an already-designed pipeline — use DNArSim directly for nanopore
  channel simulation
- When input is real sequencing read data to be decoded rather than an in-silico fault-injection
  experiment over encoded oligos
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0196
tools:
- name: FrameD
  role: Core tool providing encoding, decoding, file manipulation, JSON-configured
    fault injection, and transparent MPI/batch parallelism
  repo: https://github.com/dna-storage/framed
- name: DNArSim
  role: External nanopore channel simulator invoked by FrameD's DNArSim configuration
    examples to model memory-dependent sequencing errors
  repo: https://github.com/BHam-1/DNArSim
- name: db_gen.py
  role: Compiles per-parameter fault-injection leaf statistics into a single Pandas
    dataframe for cross-configuration comparison
  repo: https://github.com/dna-storage/framed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad572
    title: 'FrameD: framework for DNA-based data storage design, verification, and
      validation'
  source_content_hash: sha256:e20e62af7dade971df87e953c80cd67770ebffa2652d673ec96e447133f25adf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dna-storage/framed
  tools:
  - DNArSim
  - FrameD
  - db_gen.py
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# encoder-decoder-pipeline-configuration

## Summary

Configure and run FrameD's JSON-driven fault-injection pipeline: encode an input data file into DNA oligos, sample errors from a configured fault distribution, and decode through the user-selected pipeline stages, recording per-parameter results and aggregated statistics.

## When to use

Use when a DNA storage pipeline design must be evaluated under injected errors (i.i.d. or nanopore DNArSim channel faults), including parameter sweeps over encoder/decoder settings, to compare pipeline stage choices across error rates.

## When NOT to use

- When the goal is to simulate the physical sequencing channel itself rather than inject faults into an already-designed pipeline — use DNArSim directly for nanopore channel simulation
- When input is real sequencing read data to be decoded rather than an in-silico fault-injection experiment over encoded oligos

## Inputs

- JSON configuration file specifying encoder parameters, decoder parameters, fault distribution parameters, copy distribution parameters, and optional parameter sweep
- Input data file to be encoded into DNA oligos
- Optional channel simulator FASTQ-style input (e.g., DNArSim reference sequence, number of reads, channel memory k=6) for nanopore error models

## Outputs

- Unique output directory per parameter combination containing parameter record files with experiment parameters
- 'fi.stats' statistics file per parameter combination with aggregated counters (decode failures, byte-error locations, base-error locations)
- Compiled Pandas dataframe (e.g., fault_injection_dataframe) from db_gen.py for analysis

## How to apply

Parse the JSON configuration to extract encoder parameters, decoder parameters, fault distribution parameters, copy distribution parameters, and any optional parameter sweep. Encode the input data file into DNA oligos with FrameD's core encoding module, then for each parameter combination generate the configured number of copies per oligo and sample errors from the fault distribution, exploiting FrameD's transparent MPI/batch-level parallelism (IBM LSF by default, extendable via the TcshJob class in tools/lsf/submit.py, or subshell background jobs for small runs). Reconstruct the storage pipeline and decode the error-carrying oligos back to the original data, recording decode failures and byte-error and base-error locations in per-parameter parameter record files, and aggregate counters into a human-readable 'fi.stats' statistics file in each leaf output directory. For cross-configuration analysis, compile all leaf results into a Pandas dataframe with db_gen.py and analyze it in notebooks/framed.ipynb.

## Related tools

- **FrameD** (Core tool providing encoding, decoding, file manipulation, JSON-configured fault injection, and transparent MPI/batch parallelism) — https://github.com/dna-storage/framed
- **DNArSim** (External nanopore channel simulator invoked by FrameD's DNArSim configuration examples to model memory-dependent sequencing errors) — https://github.com/BHam-1/DNArSim
- **db_gen.py** (Compiles per-parameter fault-injection leaf statistics into a single Pandas dataframe for cross-configuration comparison) — https://github.com/dna-storage/framed

## Examples

```
cd $DNASTORAGE_HOME && tcsh examples/small/run_small.csh
```

## Evaluation signals

- A statistics file (fi.stats) exists for every parameter combination in the sweep
- Each statistics file contains aggregated counters including decode failures and byte-error and base-error locations
- Each leaf output directory records the experiment's parameters, enabling exact reproduction of every run
- The number of executed simulations matches the expanded parameter-sweep count (e.g., 240 simulations for the paper's 6 JSON configurations)
- The compiled dataframe loads and reproduces the paper's figure-generating analysis in notebooks/framed.ipynb

## Limitations

- The built-in HPC job generation assumes an IBM LSF scheduler; other schedulers (Slurm, Cobalt) require extending the TcshJob/LSFJob class in tools/lsf/submit.py
- Full parameter sweeps (240 simulations) are only tractable on HPC systems; commodity-hardware runs are limited to the small subshell examples
- Encoding or decoding large files may perform poorly on standard computers, requiring a more capable system or HPC
- Whether including multiple sequence alignment in the pipeline is beneficial depends on error rate and injected error type and is not always beneficial

## Evidence

- [abstract] FrameD provides transparent parallelism to deal with a large number of reads from sequencing and the need for many fault injection iterations: "FrameD provides the user with transparent parallelism to deal with a large number of reads from sequencing and the need for many fault injection iterations"
- [readme] There are 6 total .json files used for the paper: [evidence span withheld — anchor status: external_doc]
- [readme] a human readable "fi.stats" file containing statistics gathered during fault injection: [evidence span withheld — anchor status: external_doc]
- [readme] The final command generates a Pandas dataframe, and stores it at the top directory path of the collection of results: [evidence span withheld — anchor status: external_doc]
- [results] We found that the choice to include multiple sequence alignment in the pipeline is dependent on the error rate and the type of errors being injected and is not always beneficial.: "We found that the choice to include multiple sequence alignment in the pipeline is dependent on the error rate and the type of errors being injected and is not always beneficial."
