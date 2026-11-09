---
name: packet-base-sequence-file-segmentation
description: Use when simulating or implementing a DNA-based data storage encode path
  that must convert an arbitrary binary file into addressable packets and base-sequences
  prior to inner-code DNA strand synthesis, e.g. when building or validating a FrameD-style
  pipeline.
when_to_use_negative:
- The input is already a set of sequenced DNA reads or FASTQ strands — segmentation
  applies on the encode side; for those inputs the decode path (clustering, MSA, inner-code
  reversal) is the relevant skill.
- The goal is to model channel errors (e.g. nanopore synthesis/sequencing/basecalling
  errors) rather than to segment and encode a file — a simulator such as DNArSim is
  the appropriate tool.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: FrameD
  role: Provides the core encoding, decoding, and file manipulation support used to
    build the file -> packets -> base-sequences pipeline skeleton and the outer/inner
    encode layers
  repo: https://github.com/dna-storage/framed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad572
    title: 'FrameD: framework for DNA-based data storage design, verification, and
      validation'
  source_content_hash: sha256:6449bff09d4d8fc0c062d9434ce1d9eef3aa4d5d1c1c1be86a802a695c913d2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dna-storage/framed
  tools:
  - FrameD
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Packet-base-sequence file segmentation

## Summary

Segmentation stage of the FrameD DNA storage pipeline in which an input file is decomposed into packets and then into base-sequences before indexing and inner encoding to DNA strands. It establishes the modular file -> packets -> base-sequences structure that FrameD's core encoding and file-manipulation support is built around.

## When to use

Use when simulating or implementing a DNA-based data storage encode path that must convert an arbitrary binary file into addressable packets and base-sequences prior to inner-code DNA strand synthesis, e.g. when building or validating a FrameD-style pipeline.

## When NOT to use

- The input is already a set of sequenced DNA reads or FASTQ strands — segmentation applies on the encode side; for those inputs the decode path (clustering, MSA, inner-code reversal) is the relevant skill.
- The goal is to model channel errors (e.g. nanopore synthesis/sequencing/basecalling errors) rather than to segment and encode a file — a simulator such as DNArSim is the appropriate tool.

## Inputs

- Arbitrary input file (binary data) to be encoded for DNA storage
- FrameD pipeline configuration (e.g. JSON configuration files such as those under examples/iid and examples/DNArSim)

## Outputs

- Packets of file data with automatically assigned outer-code indexes (optionally special indexes such as Luby Seed)
- Indexed base-sequences ready for the inner encoder
- Encoded DNA strands (base sequences suitable for synthesis and sequencing simulation)

## How to apply

Structure the encode path as file -> packets -> base-sequences using FrameD's core encoding, decoding, and file-manipulation support. Apply the outer code layer over packets with automatic indexing, exposing an interface that also supports special indexes such as Luby Seed so packets can be reconstructed from subsets of reads. Then map each indexed base-sequence through the inner encoder to yield individual DNA strands for synthesis. Decision point: keep segmentation fixed and instead vary downstream clustering/MSA and consolidation choices in the decode path, since FrameD's design treats the pipeline as modular components. Validate by running a file -> encode -> decode round trip and confirming the reconstructed file matches the original input byte-for-byte.

## Related tools

- **FrameD** (Provides the core encoding, decoding, and file manipulation support used to build the file -> packets -> base-sequences pipeline skeleton and the outer/inner encode layers) — https://github.com/dna-storage/framed

## Examples

```
cd $DNASTORAGE_HOME; tcsh examples/small/run_small.csh
```

## Evaluation signals

- A full file -> encode -> decode round trip reconstructs the original file byte-for-byte.
- Every base-sequence entering the inner encoder carries an outer-code index (automatic or special, e.g. Luby Seed).
- The packet -> base-sequence mapping is deterministic and reversible through the inner code so indexed base-sequences can be mapped back to indexed packets during decode.
- Pipeline runs with the shipped configurations (e.g. examples/small/run_small.csh) complete without leaving unindexed or orphaned packets.

## Limitations

- Encoding or decoding large files may perform poorly on a standard computer; FrameD's infrastructure performs best on an HPC system or a machine with many cores and an MPI implementation.
- Full-scale batch-level simulation assumes an IBM LSF-managed platform; other schedulers (Cobalt, Slurm) require extending the TcshJob/LsfJob classes in tools/lsf/submit.py.
- Only macOS and Linux are supported (tested on CentOS 7, Python 3.6-3.10, MPI such as Intel MPI 2017 or Open MPI 4.1.5, Julia 1.6.2), though Docker (kvolkel/framed:revision) replicates the environment.

## Evidence

- [abstract] FrameD core file-manipulation and modular pipeline skeleton: "we introduce FrameD, a simulation infrastructure for DNA storage systems that leverages the underlying modularity of DNA storage system designs"
- [intro] Core encode/decode/file-manipulation support: "Core encoding, decoding, and file manipulation support for modeling DNA-based information storage systems"
- [readme] Repository overview and environment: "Core encoding, decoding, and file manipulation support for modeling DNA-based information storage systems"
- [readme] Non-HPC small example invocation: [evidence span withheld — anchor status: external_doc]
