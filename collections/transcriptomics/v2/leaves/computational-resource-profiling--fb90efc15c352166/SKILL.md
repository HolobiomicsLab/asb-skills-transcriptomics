---
name: computational-resource-profiling
description: Use when benchmarking index-construction cost of AMQ structures (PAC,
  HowDeSBT, SeqOthello, COBS) on large RNA-seq or genome collections, to record CPU
  time, peak RAM, temp disk, and final index size per tool.
when_to_use_negative:
- The analysis goal is query-side evaluation (query throughput or which datasets contain
  a query sequence), not index-construction resource cost
- Inputs are already-built indexes and the question is comparison of index size or
  query output, so no construction run needs to be profiled
- Only a single small dataset is indexed, where construction-time ratios between tools
  carry no benchmarking meaning
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PAC
  role: Primary tool whose construction CPU time, peak RAM, temp-disk footprint, and
    final index size are profiled; streams input so no temp disk beyond the index
  repo: https://github.com/Malfoy/PAC
- name: HowDeSBT
  role: Comparator SBT-based AMQ whose construction cost is measured at matching k=21
    and false positive rate
- name: SeqOthello
  role: Comparator compressed AMQ whose construction time and index size are recorded
    to compute PAC's ratio (~3x faster)
- name: Cobs
  role: Comparator Bloom-filter-based AMQ built on the same inputs with comparable
    FPR to record construction CPU time, temp disk, RAM, and index size
- name: kmtricks
  role: Computes k=21 solid kmers used to build the HowDeSBT comparator index at a
    matching false positive rate
- name: g++ 8 or higher
  role: C++17-ready compiler required to build PAC from source before profiling its
    resource usage
- name: zlib
  role: Dependency (zlib1g-dev on Debian/Ubuntu) needed to compile PAC for reading
    gzipped FASTA/FASTQ inputs during profiled builds
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad225
    title: Scalable sequence database search using partitioned aggregated Bloom comb
      trees
  source_content_hash: sha256:2edd1b917e2bb8e6be05bff944fdc01ae0521e58de93a764c642adcf8c5381d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Malfoy/PAC
  tools:
  - Cobs
  - HowDeSBT
  - PAC
  - SeqOthello
  - g++ 8 or higher
  - kmtricks
  - zlib
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# computational-resource-profiling

## Summary

Profiles and compares the construction cost of approximate membership query (AMQ) indexes such as PAC, SeqOthello, HowDeSBT and COBS by recording construction CPU time, peak RAM, temporary disk usage, and final index size on identical inputs.

## When to use

Use when benchmarking index-construction cost of AMQ structures (PAC, HowDeSBT, SeqOthello, COBS) on large RNA-seq or genome collections, to record CPU time, peak RAM, temp disk, and final index size per tool.

## When NOT to use

- The analysis goal is query-side evaluation (query throughput or which datasets contain a query sequence), not index-construction resource cost
- Inputs are already-built indexes and the question is comparison of index size or query output, so no construction run needs to be profiled
- Only a single small dataset is indexed, where construction-time ratios between tools carry no benchmarking meaning

## Inputs

- File of files (fof.txt) listing one dataset file per line, e.g. gzipped bcalm2 unitig FASTAs for the 2585 human RNA-seq SRR benchmark
- Raw SRA RNA-seq sample files (FASTA, MultiLine FASTA, or FASTQ, gzipped or not) for incremental scalability batches up to 32,768 samples
- A previously built PAC index directory, when profiling update-mode construction (-l)
- Index-construction parameters: k-mer size (-k, e.g. 21 or 31), Bloom filter bit size (-b, e.g. 1000000000), thread count (-c), partition count (-p), unique-kmer filter (-u)

## Outputs

- construction_benchmark_table.tsv with per-tool (PAC, SeqOthello, HowDeSBT, COBS) construction CPU time, peak RAM, temporary disk usage, and final index size
- PAC construction-time ratios relative to other compressed methods (reported ~3x over SeqOthello, ~6x over HowDeSBT)
- Per-batch scalability log of CPU time, peak RAM, temp disk, and final index size (reported ~5 days / ~60 CPU days, ~22 GB RAM, ~1.1 TB index at 32,768 samples)

## How to apply

Assemble the dataset list as a file of files (one indexed file per line, e.g. via 'readlink -f' over the dataset folder of 2585 gzipped bcalm2 SRR unitig FASTAs), then build each tool's index with matching parameters — for PAC use './PAC -f fof.txt -d index_pac -k 21 -b 1000000000' so Bloom filters are sized to the first power of two above 1,000,000,000 bits for a comparable false positive rate, and build HowDeSBT with kmtricks computing k=21 solid kmers at the same FPR, plus SeqOthello and COBS on the same inputs. For every build record four metrics: construction CPU time, peak RAM, temporary disk usage, and final index size; note that PAC streams so its construction leaves no disk footprint beyond the index itself, which is a validation signal, not a measurement error. Aggregate the four metrics per tool into a construction_benchmark_table.tsv and compute PAC's construction-time ratio relative to the other compressed methods (expected: ~3x over SeqOthello, ~6x over HowDeSBT). For scalability profiling, run incremental batches with update mode ('./PAC -f <fof_batch> -l <previous_index> -d <index_updated> -k 31 -u'), optionally raising 'ulimit -n' and tuning -p partitions and -c threads, logging CPU time, RAM, temp disk, and index size after each batch up to 32,768 SRA samples.

## Related tools

- **PAC** (Primary tool whose construction CPU time, peak RAM, temp-disk footprint, and final index size are profiled; streams input so no temp disk beyond the index) — https://github.com/Malfoy/PAC
- **HowDeSBT** (Comparator SBT-based AMQ whose construction cost is measured at matching k=21 and false positive rate)
- **SeqOthello** (Comparator compressed AMQ whose construction time and index size are recorded to compute PAC's ratio (~3x faster))
- **Cobs** (Comparator Bloom-filter-based AMQ built on the same inputs with comparable FPR to record construction CPU time, temp disk, RAM, and index size)
- **kmtricks** (Computes k=21 solid kmers used to build the HowDeSBT comparator index at a matching false positive rate)
- **g++ 8 or higher** (C++17-ready compiler required to build PAC from source before profiling its resource usage)
- **zlib** (Dependency (zlib1g-dev on Debian/Ubuntu) needed to compile PAC for reading gzipped FASTA/FASTQ inputs during profiled builds)

## Examples

```
./PAC -f fof.txt -d index_pac -k 21 -b 1000000000
```

## Evaluation signals

- construction_benchmark_table.tsv reproduces PAC's 3-6 fold construction-time improvement over other compressed methods for comparable index size
- PAC's profiled construction shows no temporary disk footprint besides the index itself (streaming behavior)
- In the scalability run, PAC alone completes up to 32,768 samples in ~5 days (~60 CPU days) at ~22 GB RAM with a ~1.1 TB index, while SeqOthello crashes (2^12 batch) and HowDeSBT times out (2^11 batch)
- All four metrics (CPU time, peak RAM, temp disk, final index size) are recorded per tool with matched k-mer size (k=21) and Bloom filters sized to the first power of two above 1,000,000,000 bits for comparable FPR
- Incremental update-mode builds succeed for each batch, with per-batch resource logs consistent with the final reported scale

## Limitations

- PAC memory usage can increase linearly with the number of threads (-c), so thread count must be fixed when comparing resource metrics across runs
- More partitions (-p) usually speed up construction and reduce memory but create more index files and may require raising the open-file limit (ulimit -n)
- Larger Bloom filters decrease false positive rate but increase index size, so FPR must be matched across tools or time ratios are not comparable
- At the largest scale, SeqOthello (crashed at 2^12 batch) and HowDeSBT (timed out at 2^11 batch) do not complete, so ratios can only be measured on the 2585-sample benchmark, not at 32,768 samples

## Evidence

- [abstract] 3-6 fold construction improvement: "It shows a 3-6 fold improvement in construction time compared to other compressed methods for comparable index size."
- [abstract] streaming construction footprint: "PAC index construction works in a streaming fashion without any disk footprint besides the index itself."
- [abstract] scalability result: "They include 32 000 human RNA-seq samples in 5 days"
- [intro] low resource construction: "A PAC index can be built from thousands datasets quickly and with low ressource memory and disk footprint"
- [readme] thread-count memory tradeoff: "PAC is able to efficiently use a high amount of threads to divide the construction time but the memory usage can increase linearly with the amount of threads used."
- [readme] partition tradeoff: "More partitions usually speed up and reduce the memory usage during construction and query operations but create more files."
