---
name: simulated-scrnaseq-benchmark-data-handling
description: Use when comparing runtime and memory efficiency of scRNA-Seq TE quantification
  tools (e.g., SoloTE vs scTE) on the same simulated BAM inputs across genome versions
  such as hg38 and mm10.
when_to_use_negative:
- The goal is to compare TE expression detection accuracy, UMAP dimensional reduction,
  or marker TE detection rather than computational resource usage
- Input is raw FASTQ or unaligned reads rather than aligned BAM files from CellRanger
  or STAR, which the pipelines under test require
- Evaluating locus-specific TE expression quality or 3' end 10X data performance,
  which the article treats as separate, unassessed benchmarks
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SoloTE
  role: Pipeline under benchmark; run via SoloTE_pipeline.py with --threads 8 on simulated
    BAMs while timing memory and wall-clock
  repo: https://github.com/bvaldebenitom/SoloTE
- name: scTE
  role: Comparison baseline; index-build and per-BAM quantification phases timed independently
- name: BEDTools
  role: Dependency invoked by SoloTE during benchmarked runs (overlap of reads with
    TE annotation); requires v2.29.2 or higher
  repo: https://github.com/arq5x/bedtools2
- name: Samtools
  role: Dependency of the SoloTE pipeline invoked during the benchmarked runs (v1.16
    or higher)
- name: Pysam
  role: Python module required in the SoloTE environment executing the benchmarked
    pipeline
- name: Pandas
  role: Used to compile per-run measurements into the runtime_memory_benchmark.csv
    results table (v1.5.0 or higher)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:cd733d4a5c3099cbfe93914be9040c7110770c65b2b75033a32234820132753a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - BEDTools
  - Pandas
  - Pysam
  - Samtools
  - SoloTE
  - scTE
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# simulated-scrnaseq-benchmark-data-handling

## Summary

A benchmarking procedure for comparing computational resource usage (wall-clock time and peak memory) of scRNA-Seq transposable element quantification pipelines, as used to show SoloTE runs in <25 min using at most 15 Gb per run versus scTE at 20-60 min and 40-60 Gb.

## When to use

Use when comparing runtime and memory efficiency of scRNA-Seq TE quantification tools (e.g., SoloTE vs scTE) on the same simulated BAM inputs across genome versions such as hg38 and mm10.

## When NOT to use

- The goal is to compare TE expression detection accuracy, UMAP dimensional reduction, or marker TE detection rather than computational resource usage
- Input is raw FASTQ or unaligned reads rather than aligned BAM files from CellRanger or STAR, which the pipelines under test require
- Evaluating locus-specific TE expression quality or 3' end 10X data performance, which the article treats as separate, unassessed benchmarks

## Inputs

- Simulated scRNA-Seq BAM files per genome version (hg38, mm10) and TE-group condition
- Transposable element annotation in BED format (e.g., RepeatMasker-derived TE BED via SoloTE_RepeatMasker_to_BED.py)
- scTE TE indexes (index-build phase timed separately from quantification)

## Outputs

- Per-run wall-clock time (minutes) and peak resident memory (Gb) measurements
- runtime_memory_benchmark.csv with columns: tool, genome, TE-group, phase (index-build vs quantification), wall-clock minutes, peak memory Gb
- SoloTE gene+TE expression matrices per condition (byproducts of the benchmarked runs)

## How to apply

Run the SoloTE pipeline on each simulated BAM file per genome (hg38, mm10) and TE-group condition with 8 threads, wrapping the invocation in /usr/bin/time to capture wall-clock time and maximum resident memory. Time scTE's separate TE-index-build step independently from its per-BAM quantification run so resource costs of the two phases are not conflated. Record wall-clock minutes and peak memory in Gb per run, keyed by tool, genome version, TE-group, and phase, then compile into a single results table (e.g., runtime_memory_benchmark.csv). Validate against reported ranges: SoloTE <25 min with <=15 Gb peak memory, scTE 20-60 min and 40-60 Gb, and scTE index-build 5-42 min; flag any measurement outside these ranges.

## Related tools

- **SoloTE** (Pipeline under benchmark; run via SoloTE_pipeline.py with --threads 8 on simulated BAMs while timing memory and wall-clock) — https://github.com/bvaldebenitom/SoloTE
- **scTE** (Comparison baseline; index-build and per-BAM quantification phases timed independently)
- **BEDTools** (Dependency invoked by SoloTE during benchmarked runs (overlap of reads with TE annotation); requires v2.29.2 or higher) — https://github.com/arq5x/bedtools2
- **Samtools** (Dependency of the SoloTE pipeline invoked during the benchmarked runs (v1.16 or higher))
- **Pysam** (Python module required in the SoloTE environment executing the benchmarked pipeline)
- **Pandas** (Used to compile per-run measurements into the runtime_memory_benchmark.csv results table (v1.5.0 or higher))

## Examples

```
/usr/bin/time -v python SoloTE_pipeline.py --threads 8 --bam <BAMfile> --teannotation <TE_BED> --outputprefix <Prefix> --outputdir <OutputDirectory>
```

## Evaluation signals

- SoloTE runs complete in <25 minutes with <=15 Gb peak resident memory per run
- scTE runs fall within 20-60 minutes and 40-60 Gb peak memory per run
- scTE TE-index-build phase independently timed at 5-42 minutes
- Every tool/genome/TE-group condition has both wall-clock and peak-memory entries in runtime_memory_benchmark.csv
- Measurements outside the reported ranges are flagged as validation deviations

## Limitations

- scTE's resource usage must be split into index-build and quantification phases; pooling them overstates or understates per-BAM cost
- Benchmarked with 8 threads on simulated data only; absolute times and memory depend on hardware and input BAM size
- Reported ranges (e.g., 5-42 min for scTE index-build) are wide, so single measurements may not discriminate tools without replicated runs

## Evidence

- [discussion] SoloTE is faster and memory-efficient: "We found that our tool is faster and uses less computational resources than scTE."
- [discussion] Simulated benchmark genomes: "we simulated data from the human genome hg38 version, and from the mouse genome mm10 version"
- [readme] SoloTE invocation used in the benchmark: "python SoloTE_pipeline.py --threads NumberOfThreads --bam BAMfile --teannotation BEDfile --outputprefix Prefix --outputdir OutputDirectory"
- [readme] BEDTools version requirement for the SoloTE environment: "BEDTools v2.29.2 or higher (https://github.com/arq5x/bedtools2/releases)"
- [discussion] First SoloTE step executed in the timed runs: "The first step of SoloTE is the selection of reads not assigned to genes."
