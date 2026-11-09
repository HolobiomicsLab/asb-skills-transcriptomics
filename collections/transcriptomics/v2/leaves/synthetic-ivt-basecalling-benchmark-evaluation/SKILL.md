---
name: synthetic-ivt-basecalling-benchmark-evaluation
description: Use when evaluating whether a basecalling model ab initio distinguishes
  modified nucleotides (I vs m6A vs canonical) on synthetic constructs where ground-truth
  modification sites are known.
when_to_use_negative:
- Data are real cellular RNA without ground-truth modification sites — per-class precision/recall
  against known positions cannot be computed
- Goal is detecting modifications outside the model vocabulary (e.g. m1A or chemically
  similar modifications, for which streamlined NanoSpeech models are still under development)
- Input is already basecalled reads without raw signal (pod5/fast5), since the modification-aware
  basecaller operates ab initio on raw ionic currents
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: NanoSpeech
  role: Modification-aware transformer basecaller with expanded vocabulary (A, C,
    G, U, I, m6A) for ab initio calling of modified bases in a single step
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: Oxford Nanopore Technology
  role: Direct RNA sequencing platform producing the raw dRNA squiggle signals (SQK-RNA002/SQK-RNA004)
    basecalled and benchmarked here
- name: Dorado
  role: Canonical/alternative ONT modification-aware basecaller used as a state-of-the-art
    accuracy comparison baseline
  repo: https://github.com/nanoporetech/dorado
- name: m6ABasecaller
  role: Existing modification-aware basecaller (A,C,G,U,m6A, Guppy-compatible with
    modPhred) representing the prior approach being benchmarked against
  repo: novoalab/m6ABasecaller
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:537eda1ba2250d30efbf8702f4e68426363d47009cfacdc3e1d32b8423f5fbb6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/F0nz0/NanoSpeech_basecaller
  repo_url: https://github.com/F0nz0/NanoSpeech_basecaller
  tools:
  - Dorado
  - NanoSpeech
  - Oxford Nanopore Technology
  - m6ABasecaller
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# synthetic-IVT-basecalling-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks a modification-aware basecaller (NanoSpeech) on synthetic IVT dRNA constructs with known inosine and m6A positions, computing per-class precision/recall and per-read identity against a canonical basecaller (Guppy).

## When to use

Use when evaluating whether a basecalling model ab initio distinguishes modified nucleotides (I vs m6A vs canonical) on synthetic constructs where ground-truth modification sites are known.

## When NOT to use

- Data are real cellular RNA without ground-truth modification sites — per-class precision/recall against known positions cannot be computed
- Goal is detecting modifications outside the model vocabulary (e.g. m1A or chemically similar modifications, for which streamlined NanoSpeech models are still under development)
- Input is already basecalled reads without raw signal (pod5/fast5), since the modification-aware basecaller operates ab initio on raw ionic currents

## Inputs

- Raw ONT dRNA pod5/fast5 folder from synthetic IVT constructs with known inosine and m6A positions
- NanoSpeech model weights (.h5 with coupled .h5.cfg)
- Reference transcriptome (or genome) FASTA

## Outputs

- Basecalled fasta/fastq with modified-base indexes in read headers
- Sorted, indexed BAM of aligned reads
- {bam}.per_read.bed and {bam}.genome_space.tsv of modification coordinates
- Per-class precision/recall table and per-read identity comparison vs Guppy

## How to apply

Launch the modification-aware basecaller (e.g. NanoSpeech_multi_mod.py with the I+m6A .h5 weights and auto-loaded .h5.cfg) on raw pod5/fast5 dRNA reads, allotting ~3 GB GPU memory per parallel worker (e.g. ~15 threads on an A100-40GB). Map basecalled fasta/q reads with minimap2 -ax map-ont -k 14 --secondary=no --MD (or -ax splice -uf -k 14 --secondary=no --MD for a genome), filter/convert with samtools view -b -F 2304 | samtools sort, and index. Run modification_detector.py -b {BAM} -f {fasta/q} -q {min base quality} to produce {bam}.per_read.bed and {bam}.genome_space.tsv. Compute per-class precision/recall of I and m6A calls against the known modified positions of the synthetic constructs, and compare per-read identity/consensus accuracy against a standard basecaller (Guppy) on the modified sequences.

## Related tools

- **NanoSpeech** (Modification-aware transformer basecaller with expanded vocabulary (A, C, G, U, I, m6A) for ab initio calling of modified bases in a single step) — https://github.com/F0nz0/NanoSpeech_basecaller
- **Oxford Nanopore Technology** (Direct RNA sequencing platform producing the raw dRNA squiggle signals (SQK-RNA002/SQK-RNA004) basecalled and benchmarked here)
- **Dorado** (Canonical/alternative ONT modification-aware basecaller used as a state-of-the-art accuracy comparison baseline) — https://github.com/nanoporetech/dorado
- **m6ABasecaller** (Existing modification-aware basecaller (A,C,G,U,m6A, Guppy-compatible with modPhred) representing the prior approach being benchmarked against) — novoalab/m6ABasecaller

## Examples

```
python3 NanoSpeech_multi_mod.py -d {pod5/fast5_folder} -o {out.fasta} -m {I+m6A model weights .h5} -t {n_threads}
```

## Evaluation signals

- Per-class precision ≥0.98 for canonical, inosine, and m6A classes on the unseen synthetic reads
- Per-class recall ≈0.99 with a low false positive rate
- Zero I↔m6A cross-misclassification events
- Per-read identity/consensus accuracy higher than the Guppy baseline on modified sequences
- {bam}.per_read.bed and {bam}.genome_space.tsv are produced with mapping coordinates for every called modified nucleotide

## Limitations

- Primarily tested on synthetic constructs as a proof of concept; real-world performance depends on number/quality of reads, contaminations, and unknown IVT conditions
- Lack of reliable benchmarks and unavailable training datasets of existing tools limit unbiased performance comparisons
- Inference artefacts or 'hallucinations' can arise from highly similar chemical structures or intrinsic errors in modified samples; detection of highly similar structures is still under development

## Evidence
