---
name: benchmark-scaling-curve-reconstruction
description: Use when comparing how runtime and peak memory scale with read count
  for an alignment-free pseudoaligner (fortuna) versus spliced-align-and-count baselines
  (Whippet, STAR + sigcount) on subsampled FASTQ data.
when_to_use_negative:
- The goal is accuracy of splicing-event detection or counting (e.g. validating fortuna's
  >99% mapping rate or novel exon-skipping calls) rather than runtime/memory benchmarking
- Only one tool is being evaluated and no align-and-count baseline (Whippet, STAR
  + sigcount) is run, so a scaling comparison cannot be made
- The input sample is small enough that runtime differences are negligible and a full
  10–90% subsampling series would be wasted effort
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: fortuna
  role: Benchmarked alignment-free pseudoaligner; run with --quant step and 4 threads
    on each subsample, timing and memory recorded
  repo: https://github.com/canzarlab/fortuna
- name: kallisto
  role: Provides the pseudoalignment engine underlying fortuna's --quant step; reference
    point for the ~4 GB peak-memory figure
  repo: https://github.com/pachterlab/kallisto
- name: Whippet
  role: Spliced-alignment baseline run under identical thread settings on every subsample;
    ~2 GB peak memory
- name: STAR
  role: 1-pass spliced aligner forming the traditional align-and-count baseline (~29
    GB peak memory); chained with samtools and sigcount
- name: sigcount
  role: Counts splicing signatures from the STAR 1-pass BAM to complete the align-and-count
    chain
- name: samtools
  role: Sorts and indexes the STAR 1-pass BAM before sigcount quantification
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad419
    title: Counting pseudoalignments to novel splicing events
  source_content_hash: sha256:5c0779dbfe90d777036cf4c2626fc9a4f9f5fefd95b505304cee83991fd065c9
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
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# benchmark-scaling-curve-reconstruction

## Summary

Reconstructs a runtime and peak-memory scaling curve across 10–90% read subsamples by timing fortuna's kallisto-based --quant step, Whippet, and a STAR 1-pass + sigcount align-and-count chain on the same FASTQ subsamples, assembling per-tool measurements into a single TSV for head-to-head speed/memory comparison.

## When to use

Use when comparing how runtime and peak memory scale with read count for an alignment-free pseudoaligner (fortuna) versus spliced-align-and-count baselines (Whippet, STAR + sigcount) on subsampled FASTQ data.

## When NOT to use

- The goal is accuracy of splicing-event detection or counting (e.g. validating fortuna's >99% mapping rate or novel exon-skipping calls) rather than runtime/memory benchmarking
- Only one tool is being evaluated and no align-and-count baseline (Whippet, STAR + sigcount) is run, so a scaling comparison cannot be made
- The input sample is small enough that runtime differences are negligible and a full 10–90% subsampling series would be wasted effort

## Inputs

- Large paired/single-end RNA-seq FASTQ (e.g. ~291M-read ASD sample 29)
- Fortuna pre-built index matched to the target read length (75bp/100bp/150bp H. sapiens GENCODE release 42 GRCh38.p13 index)
- Preprocessed (segmented) GTF annotation from bin/processGTF.sh
- Genome FASTA + GTF for STAR 1-pass reference generation

## Outputs

- benchmark_scaling.tsv with per-tool wall-time and peak memory for each of the 9 subsample fractions (10–90%)
- Per-tool count outputs from fortuna --quant, Whippet, and sigcount for each fraction

## How to apply

Generate fractional subsamples (10% through 90%) of a large FASTQ — here the ~291M-read ASD sample 29 — so runtime can be traced as a function of input size. Run fortuna's --quant step (kallisto-based pseudoalignment) on each subsample with a fixed thread count (-thr 4) and target read length, timing each run and recording peak memory; run Whippet under identical thread settings; and run STAR in 1-pass mode followed by samtools sort/index and sigcount quantification, timing the full align-and-count chain. Fix threads and read length across all tools so differences reflect the algorithms, not resource allocation. Assemble the per-tool wall-time and peak-memory measurements across the 9 subsample fractions into benchmark_scaling.tsv, then validate against the paper's reference points: fortuna analyzed almost 300 million reads in about 15 min on four threads, was around 7x faster than align-and-count approaches (and more than 2x faster than Whippet on all but the smallest subsample), with peak memory near 2 GB (Whippet), 4 GB (kallisto), up to ~24 GB (fortuna) and ~29 GB (STAR).

## Related tools

- **fortuna** (Benchmarked alignment-free pseudoaligner; run with --quant step and 4 threads on each subsample, timing and memory recorded) — https://github.com/canzarlab/fortuna
- **kallisto** (Provides the pseudoalignment engine underlying fortuna's --quant step; reference point for the ~4 GB peak-memory figure) — https://github.com/pachterlab/kallisto
- **Whippet** (Spliced-alignment baseline run under identical thread settings on every subsample; ~2 GB peak memory)
- **STAR** (1-pass spliced aligner forming the traditional align-and-count baseline (~29 GB peak memory); chained with samtools and sigcount)
- **sigcount** (Counts splicing signatures from the STAR 1-pass BAM to complete the align-and-count chain)
- **samtools** (Sorts and indexes the STAR 1-pass BAM before sigcount quantification) — https://github.com/samtools/htslib

## Examples

```
./fortuna --quant -thr 4 <fortuna_index> subsample_10pct.fq  # time and record peak memory, repeat for 20–90% fractions
```

## Evaluation signals

- fortuna processes the nearly 300M-read sample in about 15 min using four threads
- fortuna is roughly 7x faster than the align-and-count alternatives overall and at least 2x faster than Whippet on all subsample fractions except the smallest
- Recorded peak-memory footprints fall near reported ranges: ~2 GB (Whippet), ~4 GB (kallisto), up to ~24 GB (fortuna), ~29 GB (STAR)
- benchmark_scaling.tsv contains one runtime/memory row per tool per fraction, covering all 9 subsample levels (10–90%)
- Thread settings (-thr 4) and target read length are identical for fortuna and Whippet runs on each fraction

## Limitations

- fortuna's peak memory scales with input (up to ~24 GB on the full ~291M-read sample), exceeding Whippet (~2 GB) and kallisto (~4 GB), so small-memory machines may need to stay with the align-and-count-free baselines despite the runtime advantage
- The speed advantage shrinks at the smallest subsample, where fortuna was not more than two times faster than Whippet
- fair comparison requires identical thread settings across tools; mixing thread counts invalidates the scaling curve
- fortuna must be built with C++11 and CMake >=2.8.12, and the bundled Docker image was tested on Linux and Mac OS only, not Windows

## Evidence

- [abstract] fortuna was around 7× faster than traditional align and count approaches, and was able to analyze almost 300 million reads in just 15 min when using four threads.: "fortuna was around 7× faster than traditional align and count approaches, and was able to analyze almost 300 million reads in just 15 min when using four threads."
- [results] Peak memory and 2x-vs-Whippet result: [evidence span withheld — anchor status: not_found]
- [readme] Preprocessing requirement for the annotation used at each benchmark step: "A transcript annotation in GTF format needs to be preprocessed by computing minimal disjoint segments bounded by splice sites"
