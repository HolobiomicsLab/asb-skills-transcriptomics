---
name: rna-modification-stoichiometry-aggregation
description: Use when you have modification-aware basecalled dRNA reads (pod5/fast5
  basecalled to fasta/fastq with mod indexes in headers, aligned to sorted BAM) and
  need per-read plus aggregated modification stoichiometry.
when_to_use_negative:
- Reads were basecalled with a canonical modification-unaware basecaller without any
  modification model or ModKit post-processing — there are no mod tags/headers to
  aggregate
- The goal is de novo detection of highly similar chemical structures (e.g. near-identical
  modified ribonucleosides), which the article states streamlined NanoSpeech models
  cannot yet reliably resolve and may 'hallucinate'
- Input is already a stoichiometry/aggregated modification table rather than raw per-read
  alignments and basecalled reads
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3295
tools:
- name: NanoSpeech
  role: Modification-aware transformer basecaller whose per-read mod indexes in fasta/fastq
    headers are the input to modification_detector.py aggregation; produces per-read
    and transcriptome-space modification predictions
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: Dorado
  role: ONT's modification-aware basecaller used to generate comparator modification
    calls via ModKit post-processing for stoichiometry benchmarking
  repo: https://github.com/nanoporetech/dorado
- name: ModiKit
  role: Post-processing tool applied to Dorado output to extract modification calls/probabilities,
    against which NanoSpeech's single-step aggregation is compared
- name: samtools
  role: Filters (-F 2304), sorts, and indexes the BAM alignments consumed by the aggregation
    step
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:b1c7b81e0433b83c1a8bb76d8fa313a4ff2a23ca3f1e9f81072bbcebd3c149ae
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
  - ModiKit
  - NanoSpeech
  - samtools
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# rna-modification-stoichiometry-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates per-read modification calls from ONT direct RNA BAM alignments into per-read and transcriptome-space modification stoichiometry (e.g. inosine/m6A/m5C/Psi frequencies), enabling quantitative comparison between modification-aware basecallers such as NanoSpeech and Dorado+ModKit pipelines.

## When to use

Use when you have modification-aware basecalled dRNA reads (pod5/fast5 basecalled to fasta/fastq with mod indexes in headers, aligned to sorted BAM) and need per-read plus aggregated modification stoichiometry.

## When NOT to use

- Reads were basecalled with a canonical modification-unaware basecaller without any modification model or ModKit post-processing — there are no mod tags/headers to aggregate
- The goal is de novo detection of highly similar chemical structures (e.g. near-identical modified ribonucleosides), which the article states streamlined NanoSpeech models cannot yet reliably resolve and may 'hallucinate'
- Input is already a stoichiometry/aggregated modification table rather than raw per-read alignments and basecalled reads

## Inputs

- Sorted and indexed BAM file of modification-aware basecalled direct RNA reads (filtered with samtools view -b -F 2304)
- NanoSpeech basecalled fasta/fastq file with modification (e.g. inosine) indexes in the read headers
- Minimum inosine/modification quality threshold (q parameter)
- Reference transcriptome used for minimap2 mapping
- Dorado (+ModKit) basecall/modification outputs for the same reads when benchmarking

## Outputs

- Per-read modification call table (read-level inosine/modification sites with quality scores)
- Genome-space (transcriptome-space) aggregated modification stoichiometry/frequency table
- Summary metrics table comparing NanoSpeech vs Dorado+ModKit per-read identity, consensus accuracy, insertion rate, and modification stoichiometry

## How to apply

After modification-aware basecalling (e.g. NanoSpeech multi-mod on pod5/fast5 raw reads producing fasta/fastq with inosine/mod-base indexes embedded in the read headers) and minimap2 mapping (map-ont or splice presets, -k 14 --secondary=no --MD) followed by samtools filtering (samtools view -b -F 2304), sort and index the BAM. Then run modification_detector.py with the BAM (-b), the basecalled fasta/fastq (-f), and a minimum inosine/modification quality threshold (-q) to retrieve both (A) per-read modification calls and (B) genome-space (transcriptome-space) aggregated predictions. These outputs feed stoichiometry comparisons against alternative tools: Dorado basecalling followed by ModKit post-processing yields the reference mod calls to compare against, reproducing metrics such as modification stoichiometry/inosine frequencies like the Fig. 3A–K panels. Validation requires the computed stoichiometry to reproduce reported relationships (comparable on short/partially modified oligos, NanoSpeech outperforming Dorado on fully modified m6A/m5C/Psi constructs).

## Related tools

- **NanoSpeech** (Modification-aware transformer basecaller whose per-read mod indexes in fasta/fastq headers are the input to modification_detector.py aggregation; produces per-read and transcriptome-space modification predictions) — https://github.com/F0nz0/NanoSpeech_basecaller
- **Dorado** (ONT's modification-aware basecaller used to generate comparator modification calls via ModKit post-processing for stoichiometry benchmarking) — https://github.com/nanoporetech/dorado
- **ModiKit** (Post-processing tool applied to Dorado output to extract modification calls/probabilities, against which NanoSpeech's single-step aggregation is compared)
- **samtools** (Filters (-F 2304), sorts, and indexes the BAM alignments consumed by the aggregation step)

## Examples

```
python3 modification_detector.py -b {aligned_reads.bam} -f {basecalled_reads.fastq} -q {minimum inosine quality}
```

## Evaluation signals

- Computed per-read and aggregated stoichiometry metrics reproduce the NanoSpeech-vs-Dorado relationships reported in Fig. 3A–K (comparable on short/partially modified oligos; NanoSpeech superior on fully modified m6A/m5C/Psi constructs)
- BAM is properly sorted, indexed, and free of secondary/supplementary alignments (filtered via samtools view -b -F 2304) before aggregation
- Every read-level modification call in the aggregate table is traceable to an inosine/mod index in the corresponding fasta/fastq read header
- Aggregated modification frequencies fall within expected stoichiometry ranges for the synthetic oligo and fully-modified IVT control datasets
- All identity, consensus accuracy, insertion-rate, and stoichiometry metrics are saved to a summary table for reproducibility

## Limitations

- NanoSpeech was primarily tested on synthetic constructs as a proof of concept; real-world data quality, read counts, contaminations, and unknown IVT conditions hamper robust training and validation
- Inference artefacts or 'hallucinations' can arise from highly similar chemical structures or intrinsic errors in modified samples
- The single-modification (inosine-only) NanoSpeech version is deprecated; use the multi-mod version which also handles pod5 format
- The often questionable quality of public RNA modification data and the unavailability of training datasets of current tools limit unbiased, reproducible performance comparisons

## Evidence
