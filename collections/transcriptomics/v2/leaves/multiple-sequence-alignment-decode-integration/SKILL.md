---
name: multiple-sequence-alignment-decode-integration
description: Use when decoding DNA storage sequencing reads in FrameD and multiple
  noisy copies per strand must be consolidated before inner-code reversal, especially
  under fault injection.
when_to_use_negative:
- Error profile has low error rate or error types for which MSA is not beneficial
  — the paper states MSA inclusion depends on error rate and error type and is not
  always beneficial
- Each strand exists as only a single read, so there are no redundant copies to cluster
  or align
- You need to consolidate strands after the inner code instead — FrameD provides an
  alternative decode mode for post-inner-code consolidation
edam_operation: http://edamontology.org/operation_0492
edam_topics: []
tools:
- name: FrameD
  role: Provides the decode pipeline skeleton where clustering and MSA steps are inserted
    before inner-code reversal, plus core encoding, decoding, and file manipulation
    support
  repo: https://github.com/dna-storage/framed
- name: DNArSim
  role: Nanopore channel simulator used to inject realistic synthesis/sequencing/basecalling
    errors into the pipeline, generating the strand copies the clustering+MSA decode
    path must handle
  repo: https://github.com/BHam-1/DNArSim
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad572
    title: 'FrameD: framework for DNA-based data storage design, verification, and
      validation'
  source_content_hash: sha256:72eb31e5eb222eac9fe0af04956f5bf1922ffb7e7339f4f1c329e2fb097bc8ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dna-storage/framed
  tools:
  - DNArSim
  - FrameD
  edam_operation: http://edamontology.org/operation_0492
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# multiple-sequence-alignment-decode-integration

## Summary

Integrates strand clustering (MinHash/edit-distance) and multiple sequence alignment (e.g. Muscle) into the FrameD decode pipeline before reversing the inner code, so multiple sequenced copies of each encoded strand are aggregated toward a consensus base-sequence.

## When to use

Use when decoding DNA storage sequencing reads in FrameD and multiple noisy copies per strand must be consolidated before inner-code reversal, especially under fault injection.

## When NOT to use

- Error profile has low error rate or error types for which MSA is not beneficial — the paper states MSA inclusion depends on error rate and error type and is not always beneficial
- Each strand exists as only a single read, so there are no redundant copies to cluster or align
- You need to consolidate strands after the inner code instead — FrameD provides an alternative decode mode for post-inner-code consolidation

## Inputs

- Sequencing reads of DNA strands (multiple noisy copies per encoded strand) from FrameD fault-injection simulation
- FrameD decode pipeline configuration (error model, e.g. i.i.d. or DNArSim nanopore channel)
- Optional MSA tool setting (e.g. Muscle) and clustering method (MinHash/edit-distance)

## Outputs

- Consensus base-sequences per strand after inner-code reversal
- Reconstructed decoded file (packets reassembled via outer code/indexing)
- Fault-injection statistics (e.g. fi.stats, JSON results) for MSA vs. no-MSA decode configurations

## How to apply

Build the FrameD decode path so that sequencing reads are first clustered by strand identity using MinHash/edit-distance, then optionally run a multiple sequence alignment such as Muscle across cluster members, performing both steps before the inner encoding is reversed so information from all copies contributes to a consensus. FrameD structures this as strand -> clustered/aligned copies -> inner decode -> indexed base-sequences -> packets -> file. The MSA step is a decision point, not a default: the paper found the choice to include multiple sequence alignment depends on the error rate and the type of errors injected, and is not always beneficial, so evaluate decode success with and without MSA under your error model (e.g. i.i.d. or DNArSim nanopore fault injection). Validate by running a file -> encode -> decode round trip and confirming byte-for-byte reconstruction of the original input.

## Related tools

- **FrameD** (Provides the decode pipeline skeleton where clustering and MSA steps are inserted before inner-code reversal, plus core encoding, decoding, and file manipulation support) — https://github.com/dna-storage/framed
- **DNArSim** (Nanopore channel simulator used to inject realistic synthesis/sequencing/basecalling errors into the pipeline, generating the strand copies the clustering+MSA decode path must handle) — https://github.com/BHam-1/DNArSim

## Examples

```
cd $DNASTORAGE_HOME && tcsh examples/small/run_small.csh
```

## Evaluation signals

- File -> encode -> decode round trip reconstructs the original input byte-for-byte
- Decode succeeds with clustered+MSA consensus where unconsolidated copies fail under the same fault-injection error model
- Comparing MSA vs. no-MSA configurations via db_gen.py-generated fault-injection dataframes shows the expected error-rate-dependent benefit
- Consensus base-sequences respect the inner code so inner-code reversal yields valid indexed base-sequences/packets

## Limitations

- MSA is not always beneficial; its value depends on the error rate and the type of errors injected
- Full-scale encode/decode and analysis perform best on HPC with MPI and IBM LSF-style batch parallelism; other schedulers require extending the TcshJob/LSFJob classes in submit.py
- Encoding or decoding large files may perform poorly on standard computers with limited RAM/compute

## Evidence

- [abstract] MSA benefit is error-dependent: "We found that the choice to include multiple sequence alignment in the pipeline is dependent on the error rate and the type of errors being injected and is not always beneficial."
- [abstract] parallelism for sequencing reads: "FrameD provides the user with transparent parallelism to deal with a large number of reads from sequencing and the need for many fault injection iterations"
- [intro] core decode/file support: "Core encoding, decoding, and file manipulation support for modeling DNA-based information storage systems as published at"
- [readme] non-HPC pipeline example: [evidence span withheld — anchor status: external_doc]
