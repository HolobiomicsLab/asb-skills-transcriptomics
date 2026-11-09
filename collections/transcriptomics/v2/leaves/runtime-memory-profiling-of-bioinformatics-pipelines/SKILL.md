---
name: runtime-memory-profiling-of-bioinformatics-pipelines
description: Use when a claim about a bioinformatics tool's speed or memory footprint
  (e.g. '7x faster', '15 min for 300M reads') must be independently reproduced across
  scaling points, with matched thread settings and per-fraction measurements.
when_to_use_negative:
- The goal is functional correctness or count accuracy of the tools, not performance
  characteristics — this protocol measures only time and memory, not output quality
- Only a single small input is available and no scaling series can be constructed;
  a single-point measurement cannot verify the claimed >2x–7x speedups across sample
  sizes
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: fortuna
  role: Pseudoalignment-based splicing-event counter under benchmark; run with --quant
    on kallisto index using 4 threads (-thr 4)
  repo: https://github.com/canzarlab/fortuna
- name: kallisto
  role: Pseudoalignment engine bundled inside fortuna; its index is built by fortuna's
    --index step and drives the --quant step being timed
  repo: https://github.com/pachterlab/kallisto
- name: Whippet
  role: Alignment-free splicing-quantification baseline run under identical thread
    settings for runtime/memory comparison
- name: STAR
  role: Traditional 1-pass spliced aligner; its full align-and-count chain (align
    → samtools sort/index → sigcount) forms the slow baseline
- name: sigcount
  role: Counting step completing the STAR-based align-and-count baseline chain that
    is timed end-to-end
- name: samtools
  role: Sorts and indexes the STAR BAM between alignment and counting as part of the
    baseline chain
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:eb9c0298dfdda24722208681c7b8d3a427685f00e3ded56d95113bd8ded94b01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/fortuna
  tools:
  - STAR
  - Whippet
  - fortuna
  - kallisto
  - samtools
  - sigcount
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# runtime-memory-profiling-of-bioinformatics-pipelines

## Summary

Benchmarking protocol for comparing runtime (wall-clock) and peak memory of RNA-seq splicing/quantification pipelines — here fortuna vs Whippet vs STAR+sigcount — on controlled fractional subsamples of a large FASTQ, demonstrating fortuna's ~7x speed advantage (~15 min for ~300M reads on 4 threads).

## When to use

Use when a claim about a bioinformatics tool's speed or memory footprint (e.g. '7x faster', '15 min for 300M reads') must be independently reproduced across scaling points, with matched thread settings and per-fraction measurements.

## When NOT to use

- The goal is functional correctness or count accuracy of the tools, not performance characteristics — this protocol measures only time and memory, not output quality
- Only a single small input is available and no scaling series can be constructed; a single-point measurement cannot verify the claimed >2x–7x speedups across sample sizes

## Inputs

- Source FASTQ (here the ~291M-read ASD sample 29) to be fractionally subsampled at 10–90%
- Pre-built tool indices (fortuna/kallisto index for the target read length, STAR genome index)
- Reference annotation used to build indices (e.g. GENCODE human release 42, GRCh38.p13, preprocessed with processGTF.sh)

## Outputs

- benchmark_scaling.tsv: per-tool runtime (wall-clock) and peak-memory measurements across the 9 subsample fractions
- Per-subsample fortuna outputs (e.g. cnt.tsv signature counts) confirming each run completed successfully

## How to apply

First generate a scaling series by subsampling the source FASTQ — here 10% through 90% fractions of the ~291M-read ASD sample 29 — so runtime and memory can be evaluated as a function of input size rather than at a single point. Run each tool under identical thread settings (4 threads, fortuna's '-thr 4'), wrapping each invocation with a time-and-memory profiler (e.g. /usr/bin/time -v or equivalent) to capture wall-clock seconds and peak RSS. For the align-and-count baseline, time the full chain: STAR 1-pass alignment, samtools sort/index of the BAM, then sigcount quantification, not just the aligner alone. Record all measurements per tool per fraction into a single table (benchmark_scaling.tsv). Validate against the article's reported values: fortuna ~15 min for ~291M reads on 4 threads, peak memory Whippet ~2 GB, kallisto ~4 GB, fortuna up to ~24 GB, STAR ~29 GB, and fortuna >2x faster than Whippet on all but the smallest fraction.

## Related tools

- **fortuna** (Pseudoalignment-based splicing-event counter under benchmark; run with --quant on kallisto index using 4 threads (-thr 4)) — https://github.com/canzarlab/fortuna
- **kallisto** (Pseudoalignment engine bundled inside fortuna; its index is built by fortuna's --index step and drives the --quant step being timed) — https://github.com/pachterlab/kallisto
- **Whippet** (Alignment-free splicing-quantification baseline run under identical thread settings for runtime/memory comparison)
- **STAR** (Traditional 1-pass spliced aligner; its full align-and-count chain (align → samtools sort/index → sigcount) forms the slow baseline)
- **sigcount** (Counting step completing the STAR-based align-and-count baseline chain that is timed end-to-end)
- **samtools** (Sorts and indexes the STAR BAM between alignment and counting as part of the baseline chain) — https://github.com/samtools/htslib

## Examples

```
/usr/bin/time -v fortuna --quant -thr 4 -i fortuna.idx -r subsample_50.fastq.gz -o out50/
```

## Evaluation signals

- fortuna processes the full ~291M-read sample in approximately 15 min using four threads (-thr 4)
- fortuna is >2x faster than Whippet on all but the smallest subsample fraction, and ~7x faster than the STAR(1-pass)+sigcount align-and-count chain (159 min vs ~15 min)
- Peak memory footprints fall near the reported ranges: Whippet ~2 GB, kallisto ~4 GB, fortuna up to ~24 GB, STAR ~29 GB
- All 9 subsample fractions (10–90%) for all three pipelines complete and are represented in benchmark_scaling.tsv with both runtime and peak-memory columns populated
- Sanity check on the tool being benchmarked: on the README example dataset (~240,000 STAT1 reads from sample 4, patient 11352.p1) fortuna maps >99% of reads

## Limitations

- fortuna's peak memory (up to ~24 GB) is much larger than Whippet's (~2 GB); on memory-constrained nodes the speed advantage may not be usable
- Benchmark timing depends on thread count (4 threads here) and read length (pre-built fortuna indices exist only for 75/100/150 bp human reads); results do not transfer to other thread settings or read lengths without re-indexing
- Speed claims exclude potential secondary costs of the align-and-count baseline such as BAM storage; only the align-sort-index-count chain runtime was timed

## Evidence

- [abstract] fortuna was around 7x faster than traditional align and count approaches: "fortuna was around 7× faster than traditional align and count approaches, and was able to analyze almost 300 million reads in just 15 min when using four threads."
- [readme] kallisto benchmark throughput quote: [evidence span withheld — anchor status: not_found]
- [readme] fortuna index read lengths for human: "you can download here the fortuna index for different read lengths Other species will follow."
- [readme] example dataset size for validating mapping rate: "It contains approximately 240,000 reads taken from sample 4 (patient id 11352.p1) in [4] that map to gene STAT1. fortuna will map over 99% of those reads"
