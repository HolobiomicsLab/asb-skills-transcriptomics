---
name: dna-storage-encode-decode-pipeline-implementation
description: 'Use when implementing or verifying a DNA-based storage system pipeline
  in simulation: you have a digital file to encode into DNA strands, noisy simulated
  sequencing reads to decode, and need a fault-injection test bed with round-trip
  validation.'
when_to_use_negative:
- You only need to simulate the synthesis-to-basecalling channel (e.g. nanopore read
  generation) without an encode/decode pipeline — use the DNArSim channel simulator
  directly instead.
- You need a production/archival DNA storage encoder, not a simulation and verification
  infrastructure; FrameD is a fault-injection test bed for evaluating designs.
- The input is already decoded strand consensus and the task is downstream statistical
  analysis only — use the db_gen.py dataframe compilation and notebooks rather than
  rebuilding the pipeline.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3316
tools:
- name: FrameD
  role: Core framework providing encoding, decoding, and file manipulation modules
    for building and validating the DNA storage pipeline, with fault-injection simulation
    and transparent MPI parallelism
  repo: https://github.com/dna-storage/framed
- name: DNArSim
  role: External nanopore channel simulator used as an error model for the decode-side
    fault injection in FrameD configurations
  repo: https://github.com/BHam-1/DNArSim
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad572
    title: 'FrameD: framework for DNA-based data storage design, verification, and
      validation'
  source_content_hash: sha256:5df4ecbae5f5d8538898520f93a08316ef2dfb3c36c3b1a89fa7e000aa7e977e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dna-storage/framed
  tools:
  - DNArSim
  - FrameD
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# dna-storage-encode-decode-pipeline-implementation

## Summary

Assemble a complete DNA data storage encode/decode pipeline using FrameD's core encoding, decoding, and file-manipulation modules, structuring the encode path as file -> packets -> base-sequences -> DNA strands and the decode path with optional strand clustering + multiple sequence alignment (MSA) or post-inner-code consolidation to resolve errors across the many copies of each strand produced by synthesis and sequencing.

## When to use

Use when implementing or verifying a DNA-based storage system pipeline in simulation: you have a digital file to encode into DNA strands, noisy simulated sequencing reads to decode, and need a fault-injection test bed with round-trip validation.

## When NOT to use

- You only need to simulate the synthesis-to-basecalling channel (e.g. nanopore read generation) without an encode/decode pipeline — use the DNArSim channel simulator directly instead.
- You need a production/archival DNA storage encoder, not a simulation and verification infrastructure; FrameD is a fault-injection test bed for evaluating designs.
- The input is already decoded strand consensus and the task is downstream statistical analysis only — use the db_gen.py dataframe compilation and notebooks rather than rebuilding the pipeline.

## Inputs

- Digital file (byte stream) to be encoded into DNA strands
- FrameD JSON configuration files specifying pipeline and error model (e.g. examples/iid for i.i.d errors, examples/DNArSim for nanopore channel simulation)
- Simulated or real sequencing reads (FASTQ-style strand copies) containing synthesis/sequencing errors

## Outputs

- Indexed DNA base-sequences / DNA strands from the encode path
- Reconstructed file byte-for-byte identical to the original input after decode
- Fault-injection statistics and JSON results per leaf directory, compilable into a Pandas dataframe via db_gen.py

## How to apply

Build the fixed FrameD pipeline skeleton using its core file-manipulation support, structuring the encode path as file -> packets -> base-sequences, then implement the outer code layer with automatic indexing and an interface that supports special indexes such as Luby Seed. Implement the inner encoder mapping indexed base-sequences to DNA strands. On decode, choose between two consolidation points: clustering of strand copies (MinHash/edit-distance) plus optional MSA (e.g. Muscle) performed BEFORE reversing the inner code, or consolidation AFTER the inner code; whether to include MSA depends on the error rate and the type of injected errors and is not always beneficial. Leverage FrameD's transparent parallelism (MPI plus batch-level job submission) for large read counts and many fault-injection iterations. Validate by running a file -> encode -> decode round trip and confirming the reconstructed file matches the original byte-for-byte.

## Related tools

- **FrameD** (Core framework providing encoding, decoding, and file manipulation modules for building and validating the DNA storage pipeline, with fault-injection simulation and transparent MPI parallelism) — https://github.com/dna-storage/framed
- **DNArSim** (External nanopore channel simulator used as an error model for the decode-side fault injection in FrameD configurations) — https://github.com/BHam-1/DNArSim

## Examples

```
cd $DNASTORAGE_HOME && tcsh examples/small/run_small.csh
```

## Evaluation signals

- Round-trip invariant: file -> encode -> decode reconstructs a file byte-for-byte identical to the original input
- Decode with clustering+MSA yields higher correct-strand recovery than plain inner-code reversal at the tested error rates; if MSA degrades results, it matches the paper's finding that MSA is not always beneficial for low error rates
- Special index modes (e.g. Luby Seed) are accepted through the outer-code indexing interface without pipeline restructuring
- Each fault-injection leaf directory contains a human-readable fi.stats file and JSON statistics after a run
- db_gen.py produces a Pandas dataframe at the top results path whose per-configuration rows match the fault-injection statistics in fi.stats files

## Limitations

- HPC-scale configurations (240 simulations in the paper) assume an IBM LSF managed platform; other schedulers (Slurm, Cobalt) require extending the TcshJob class in tools/lsf/submit.py
- Encoding or decoding large files may perform poorly on standard computers, necessitating a more capable system or HPC with MPI
- Whether to include MSA in the pipeline depends on the error rate and the type of errors injected and is not always beneficial
- Python 3.6–3.10 and Julia ~1.6.2, plus an MPI implementation, are required; environment activation must be repeated every new session

## Evidence

- [intro] Core encoding, decoding, and file manipulation support: "Core encoding, decoding, and file manipulation support for modeling DNA-based information storage systems as published at"
- [abstract] MSA benefit depends on error rate: "We found that the choice to include multiple sequence alignment in the pipeline is dependent on the error rate and the type of errors being injected and is not always beneficial."
- [abstract] Transparent parallelism: "FrameD provides the user with transparent parallelism to deal with a large number of reads from sequencing and the need for many fault injection iterations"
- [readme] Small non-HPC example: [evidence span withheld — anchor status: external_doc]
