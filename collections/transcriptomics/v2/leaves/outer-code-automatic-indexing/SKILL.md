---
name: outer-code-automatic-indexing
description: Use when reconstructing or designing a FrameD-style DNA storage encode
  path (file -> packets -> base-sequences -> DNA strands) and the outer code layer
  needs automatic index insertion across packets, including special indexes like Luby
  Seed.
when_to_use_negative:
- The pipeline stage under design is the inner code (mapping indexed base-sequences
  to DNA strands) rather than packet-level indexing and ordering
- The task is simulating physical channel errors (synthesis/nanopore/basecalling,
  e.g. with DNArSim) rather than encoding packet indices
- Only a post-sequencing consensus step (clustering plus MSA) is needed; indexing
  will not resolve per-strand errors after the inner code
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: FrameD
  role: provides the core encoding, decoding, and file-manipulation infrastructure
    on which the outer code layer with automatic indexing is built
  repo: https://github.com/dna-storage/framed
- name: DNArSim
  role: nanopore channel simulator whose injected strand errors exercise the automatic
    indexing on decode (missing/duplicated/reordered packets)
  repo: https://github.com/BHam-1/DNArSim
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad572
    title: 'FrameD: framework for DNA-based data storage design, verification, and
      validation'
  source_content_hash: sha256:fba289b1cb49026a3d1cddc3ba5c4d0709ed9b0d02b343b216564facdc77a5c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dna-storage/framed
  tools:
  - DNArSim
  - FrameD
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# outer-code-automatic-indexing

## Summary

Implements the outer code layer of a DNA storage pipeline, automatically assigning indexes to encoded base-sequences so packets can be reassembled and errors localized after sequencing, while exposing an interface that accommodates special index schemes such as Luby Seed.

## When to use

Use when reconstructing or designing a FrameD-style DNA storage encode path (file -> packets -> base-sequences -> DNA strands) and the outer code layer needs automatic index insertion across packets, including special indexes like Luby Seed.

## When NOT to use

- The pipeline stage under design is the inner code (mapping indexed base-sequences to DNA strands) rather than packet-level indexing and ordering
- The task is simulating physical channel errors (synthesis/nanopore/basecalling, e.g. with DNArSim) rather than encoding packet indices
- Only a post-sequencing consensus step (clustering plus MSA) is needed; indexing will not resolve per-strand errors after the inner code

## Inputs

- Input file to be stored, decomposed into packets and base-sequences by the FrameD file-manipulation layer
- Index scheme selection (default automatic indexing or a special index such as Luby Seed)

## Outputs

- Indexed base-sequences (packets with outer-code indices) ready for the inner encoder to map onto DNA strands
- On decode: reassembled, index-ordered packets from sequenced strand copies, ultimately a reconstructed file

## How to apply

Structure the encode pipeline as file -> packets -> base-sequences, then build the outer code layer on top of FrameD's core encoding and file-manipulation support so that every base-sequence packet receives an index automatically rather than by manual bookkeeping. Expose the outer code through an interface that lets users substitute special index types — the workflow explicitly names Luby Seed as a supported special index — instead of hard-coding a single index scheme. Automatic indexing serves the decode side: after sequencing, indexed packets can be ordered and de-duplicated even when fault injection or a nanopore channel model (e.g. DNArSim with channel memory k=6) produces missing, duplicated, or corrupted copies. Keep this layer separate from the inner encoder that maps indexed base-sequences to DNA strands, since the decode path may insert clustering/MSA (e.g. Muscle) before reversing the inner code. Validate by running a file -> encode -> decode round trip and confirming byte-for-byte identity with the original input.

## Related tools

- **FrameD** (provides the core encoding, decoding, and file-manipulation infrastructure on which the outer code layer with automatic indexing is built) — https://github.com/dna-storage/framed
- **DNArSim** (nanopore channel simulator whose injected strand errors exercise the automatic indexing on decode (missing/duplicated/reordered packets)) — https://github.com/BHam-1/DNArSim

## Evaluation signals

- A file -> encode -> decode round trip reproduces the original input byte-for-byte
- Every base-sequence packet in the encode output carries an automatically assigned index (no manual index assignment required)
- A special index scheme such as Luby Seed can be selected through the interface without changing the outer code implementation
- Decode succeeds in reassembling packet order from sequenced strand copies even after fault injection (i.i.d or DNArSim nanopore model)
- Outer-code layer remains cleanly separated from the inner encoder, allowing clustering/MSA to be inserted between them during decode

## Limitations

- Decoding or encoding large files may perform poorly on commodity hardware; the pipeline performs best on HPC systems with MPI, and the job infrastructure assumes an IBM LSF-managed platform (other schedulers require extending the TcshJob/LsfJob classes)
- The overall pipeline design is sensitive to error model: including MSA/consensus steps is dependent on error rate and error type and is not always beneficial, so index effectiveness should be evaluated under the intended fault-injection configuration
- Handled at scale via MPI parallelism plus batch-level parallelism across 240 paper simulations, which is only tractable on HPC systems with a supported scheduler

## Evidence

- [other] outer code with automatic indexing and special index interface: [evidence span withheld — anchor status: not_found]
- [intro] FrameD core capabilities: "Core encoding, decoding, and file manipulation support for modeling DNA-based information storage systems as published at"
- [abstract] MSA benefit depends on error model: "We found that the choice to include multiple sequence alignment in the pipeline is dependent on the error rate and the type of errors being injected and is not always beneficial."
- [readme] scheduler assumption for batch-level parallelism: [evidence span withheld — anchor status: external_doc]
- [other] round-trip validation criterion: [evidence span withheld — anchor status: not_found]
