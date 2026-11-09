---
name: original-spliseq-baseline-filtering-reimplementation
description: 'Use when benchmarking a SPLiT-seq preprocessing tool against the original
  identification method: paired-end FASTQ reads must be filtered with the original
  rules to define the baseline read set for overlap/consistency computation.'
when_to_use_negative:
- Input is not SPLiT-seq combinatorial-barcoded data — the last-6-base barcode match
  and three 8 nt barcode rules are SPLiT-seq specific and the fault-tolerant/indel
  alignment was proposed only for SPLiT-seq data
- The goal is SCSit performance (runtime or tolerance) evaluation rather than baseline
  consistency comparison — other skills cover the -t 4 runtime assessment and >1 mismatch/indel
  tolerance testing
- Reads have already been processed through SCSit only and no original-method baseline
  read set is required
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0622
tools:
- name: SCSit
  role: Produces the SCSit-identified labeled reads whose overlap with the original-method
    read set is quantified
  repo: https://github.com/shang-qian/SCSit
- name: Python 3.6
  role: Scripting language in which the original SPLiT-seq baseline filters (barcode
    match, UMI quality, mismatch threshold) are reimplemented and read-set intersections
    computed
- name: BLAST
  role: Alignment tool underlying the original-method barcode determination baseline
    that SCSit's consistency is measured against
- name: BWA
  role: Alternative alignment tool of the original alignment-method baseline cited
    for comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.021
    title: 'SCSit: A high-efficiency preprocessing tool for single-cell sequencing
      data from SPLiT-seq'
  source_content_hash: sha256:363097eb93cebd8552461ca4f0d094410d7782f86f8a8d667a122576e4d2ea26
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
  - Python
  - Python 3.6
  - SCSit
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# original-spliseq-baseline-filtering-reimplementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reimplements the original SPLiT-seq read-identification baseline (barcode last-6-base match, UMI quality, and 8 nt barcode mismatch filters) so SCSit's identified reads can be compared against the original method per sample, yielding 96-97% consistency measurements across five SPLiT-seq datasets.

## When to use

Use when benchmarking a SPLiT-seq preprocessing tool against the original identification method: paired-end FASTQ reads must be filtered with the original rules to define the baseline read set for overlap/consistency computation.

## When NOT to use

- Input is not SPLiT-seq combinatorial-barcoded data — the last-6-base barcode match and three 8 nt barcode rules are SPLiT-seq specific and the fault-tolerant/indel alignment was proposed only for SPLiT-seq data
- The goal is SCSit performance (runtime or tolerance) evaluation rather than baseline consistency comparison — other skills cover the -t 4 runtime assessment and >1 mismatch/indel tolerance testing
- Reads have already been processed through SCSit only and no original-method baseline read set is required

## Inputs

- Paired-end FASTQ files (Illumina 150 nt kits) for five SPLiT-seq samples (SAMN08567259, SAMN08567260, SAMN08567261, SAMN08567263, SAMN08567264)
- SCSit-identified labeled reads with combinatorial barcodes and UMI (output FASTQ pair from SCSit)
- Barcode list (96 well plate oligonucleotides per round) and primer list (TXT)

## Outputs

- Per-sample set of original-method identified reads (read ID, three 8 nt cell barcodes, UMI)
- Per-sample intersection statistics: identified-read counts, overlap counts, and consistency percentages
- Summary table of consistency between SCSit and original-method identified reads per sample (expected 96-97%)

## How to apply

For each of the five paired-end FASTQ samples, implement the original SPLiT-seq identification in Python: first discard reads whose last 6 bases do not match the barcode sequence, then filter UMIs containing more than 1 low-quality base (phred <= 10), and reject reads whose three 8 nt cell barcodes contain more than one mismatch. Run this restricted to the same filtered clean data as SCSit (itself run as a C program with e.g. -t 4 threads) so the two methods operate on comparable inputs. Intersect the sets of identified reads — keyed by read ID, barcode combination, and UMI — per sample, and compute the fraction of original-method reads that SCSit uniquely identifies. Record per-sample identified-read counts, overlaps, and consistency percentages in a summary table. The rationale is that SCSit's extra tolerance (one mismatch and one indel) should add reads that largely agree with the original baseline, with consistency reaching the reported 96-97% range.

## Related tools

- **SCSit** (Produces the SCSit-identified labeled reads whose overlap with the original-method read set is quantified) — https://github.com/shang-qian/SCSit
- **Python 3.6** (Scripting language in which the original SPLiT-seq baseline filters (barcode match, UMI quality, mismatch threshold) are reimplemented and read-set intersections computed)
- **BLAST** (Alignment tool underlying the original-method barcode determination baseline that SCSit's consistency is measured against)
- **BWA** (Alternative alignment tool of the original alignment-method baseline cited for comparison)

## Evaluation signals

- Per-sample consistency of identified reads between SCSit and the original method falls in the reported 96-97% range for all five samples
- Per-sample summary table reports identified-read counts, overlap counts, and consistency percentages for each of SAMN08567259/60/61/63/64
- Baseline filters reproduce the original rules exactly: last-6-base barcode match, UMI with at most 1 low-quality base (phred <= 10), and at most one mismatch across the three 8 nt cell barcodes
- Both methods are restricted to the same filtered clean data before set intersection, so identified-read comparisons are on comparable inputs
- Unidentified-in-original reads should fall in the 90-97% range of uniquely identified reads in SCSit, consistent with Table S3

## Limitations

- Quantitative per-sample comparison results are deferred to supplementary Table S3, cited without being provided or linked in the text
- SCSit's default tolerance is restricted to one mismatch and one indel; the baseline comparison therefore does not test higher mismatch/indel settings (which add only 4-5% identified reads at 20-40% more runtime, Table S4)
- The original method discards barcode sequences with more than one mismatch, so consistency against it cannot measure recovery of reads with multiple barcoding errors or indels

## Evidence

- [methods] UMI quality score filtering: [evidence span withheld — anchor status: not_found]
