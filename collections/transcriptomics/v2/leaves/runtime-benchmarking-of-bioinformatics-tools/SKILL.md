---
name: runtime-benchmarking-of-bioinformatics-tools
description: Use when a paper claims a new tool is substantially faster than an existing
  method and you must reproduce or verify the runtime comparison on the same datasets
  and hardware (e.g., 4 CPU cores).
when_to_use_negative:
- When the comparison target is output quality (mapped-read counts, read consistency)
  rather than speed — SCSit quality claims require STAR mapping to mm10-hg19 or mm10,
  not a timing benchmark.
- When datasets or hardware differ from those used in the original claim (e.g., different
  samples or thread counts), since ratio thresholds like 'less than 10%' are not transferable
  across configurations.
- When the tool under test is not SPLiT-seq preprocessing — the original-method filters
  (last 6 base barcode match, UMI phred <= 10 filtering, ≤1 barcode mismatch) are
  SPLiT-seq specific.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: SCSit
  role: The tool being benchmarked; multithreaded C program that identifies three
    rounds of combinatorial barcodes and UMI from SPLiT-seq FASTQ reads
  repo: https://github.com/shang-qian/SCSit
- name: BLAST
  role: 'Baseline runtime competitor: the original SPLiT-seq method uses blastn-short
    serially on a single CPU core to determine the three barcodes'
- name: BWA
  role: Alternative alignment baseline mentioned in the paper; too slow/ineffective
    for the 8 bp barcode lengths, included in runtime context
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:14d6e5b121f1da6e799d2d9e5eb8b609558ceb11977e549dfa479dd3c5a44ce9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/shang-qian/SCSit
  repo_url: https://github.com/shang-qian/SCSit
  tools:
  - BLAST
  - BWA
  - SCSit
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# runtime-benchmarking-of-bioinformatics-tools

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the wall-clock runtime of a new preprocessing tool (e.g., SCSit for SPLiT-seq data) against the original method (blastn-short-based SPLiT-seq preprocessing) on identical datasets under identical CPU resources, to substantiate performance claims such as 'SCSit runtime is less than 10% of the original'.

## When to use

Use when a paper claims a new tool is substantially faster than an existing method and you must reproduce or verify the runtime comparison on the same datasets and hardware (e.g., 4 CPU cores).

## When NOT to use

- When the comparison target is output quality (mapped-read counts, read consistency) rather than speed — SCSit quality claims require STAR mapping to mm10-hg19 or mm10, not a timing benchmark.
- When datasets or hardware differ from those used in the original claim (e.g., different samples or thread counts), since ratio thresholds like 'less than 10%' are not transferable across configurations.
- When the tool under test is not SPLiT-seq preprocessing — the original-method filters (last 6 base barcode match, UMI phred <= 10 filtering, ≤1 barcode mismatch) are SPLiT-seq specific.

## Inputs

- Paired-end FASTQ files for five SPLiT-seq samples (SAMN08567263, SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567264) from SRA
- SCSit inputs: primer list (primer.list, TXT) and barcode list (barcode.list, TXT of the 96-well plate oligonucleotides)
- Configuration specifying identical CPU core count (4) for both tools

## Outputs

- Wall-clock runtime measurements per sample for SCSit and for the original blastn-short-based SPLiT-seq preprocessing
- Comparison table (e.g., runtime_comparison.csv) with columns: sample accession, SCSit runtime, original runtime, runtime ratio
- Validation result confirming all ratios are below the 10% threshold

## How to apply

Download the paired-end FASTQ files for the same five SPLiT-seq samples used in the paper (SAMN08567263, SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567264) from SRA. Run the new tool (SCSit) on each sample with multithreading enabled under the paper's conditions (4 CPU cores, e.g., '-t 4') and record wall-clock time per sample. Run the original method (SPLiT-seq preprocessing with blastn-short, which runs serially on a single CPU core) on the same samples under identical conditions and record times. For each sample compute the ratio of new-tool runtime to original runtime and assemble a comparison table (sample accession, SCSit runtime, original runtime, ratio). Acceptance criterion from the paper: every ratio must satisfy SCSit_runtime < 0.10 * original_runtime, matching the finding that 'the runtime of SCSit is less than 10% of the original'. Fix thread counts and dataset sizes explicitly, since runtime claims are hardware- and input-size dependent.

## Related tools

- **SCSit** (The tool being benchmarked; multithreaded C program that identifies three rounds of combinatorial barcodes and UMI from SPLiT-seq FASTQ reads) — https://github.com/shang-qian/SCSit
- **BLAST** (Baseline runtime competitor: the original SPLiT-seq method uses blastn-short serially on a single CPU core to determine the three barcodes)
- **BWA** (Alternative alignment baseline mentioned in the paper; too slow/ineffective for the 8 bp barcode lengths, included in runtime context)

## Examples

```
scsit -r1 input_r1.fastq -r2 input_r2.fastq -p primer.list -b barcode.list -t 4 -o output
```

## Evaluation signals

- Every per-sample runtime ratio (SCSit / original) is below 0.10, matching Table 1 (e.g., 0.62 h vs 17.68 h for SAMN08567263).
- Both tools were run on the same five sample accessions under the same 4 CPU core limit, so the comparison is like-for-like.
- Wall-clock times are recorded for each sample individually, not only as an average.
- The output table (runtime_comparison.csv) has the expected schema: sample accession, SCSit runtime, original runtime, ratio.
- Thread settings match the paper's setup: SCSit run multithreaded (e.g., '-t 4'); the original blastn-short method runs serially on a single core as documented.

## Limitations

- Runtime ratios are hardware- and dataset-size dependent; the paper assessed runtime on five specific SPLiT-seq datasets under 4 CPU cores.
- The original blastn-short method 'can only run serially on a single CPU core', which partly explains the speed gap; a fair re-implementation with parallelism could change ratios.
- BWA is 'faster, but the length of the barcodes (8 bp) is too short for BWA to take advantage for sequence alignment', so aligner choice for baselines is constrained.
- Higher mismatch/indel tolerance in SCSit 'takes 20 ~ 40% more runtime' (Table S4), so the <10% claim applies only to the default one-mismatch/one-indel configuration.

## Evidence
