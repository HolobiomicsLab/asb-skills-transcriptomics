---
name: wall-clock-time-and-memory-profiling
description: Use when comparing the computational cost (runtime and peak memory) of
  two or more pipelines on the same input files, e.g. benchmarking SoloTE vs scTE
  on simulated scRNA-Seq BAMs.
when_to_use_negative:
- The goal is comparing output quality (TE detection sensitivity, UMAP clustering,
  marker detection) rather than computational cost — use functional benchmarking instead
- Inputs differ across the tools being compared (different BAMs, genomes, or thread
  counts), which invalidates a fair resource-usage comparison
- The tool being profiled has no distinct phases and its index-build time should not
  be conflated with per-input quantification time
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0769
tools:
- name: SoloTE
  role: Pipeline being benchmarked; runs per-BAM TE quantification on simulated BAM
    files with 8 threads and its resource usage is captured by /usr/bin/time
  repo: https://github.com/bvaldebenitom/SoloTE
- name: scTE
  role: Comparison tool; its separate TE-index-build step and per-BAM quantification
    run are timed independently
- name: BEDTools
  role: Dependency of the SoloTE pipeline being benchmarked; BEDTools v2.29.2 or higher
    required in PATH
  repo: https://github.com/arq5x/bedtools2/releases
- name: Samtools
  role: Dependency of the SoloTE pipeline; Samtools v1.16 or higher required in PATH
- name: Pysam
  role: Python module dependency of the SoloTE pipeline being profiled
- name: Pandas
  role: Python module (v1.5.0 or higher) usable to compile per-run measurements into
    the runtime_memory_benchmark.csv table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:c93c0c8189043e0377e82da0f7bb679fa949e77aa6d43761abe95b676fdf05ca
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# wall-clock-time-and-memory-profiling

## Summary

Benchmarks computational resource usage of scRNA-Seq TE-quantification tools by wrapping each tool invocation in /usr/bin/time with fixed thread counts, recording wall-clock minutes and peak memory (Gb) per run. Used in the SoloTE paper to show SoloTE (<25 min, ≤15 Gb) outperforms scTE (20–60 min, 40–60 Gb) on simulated hg38/mm10 BAM files.

## When to use

Use when comparing the computational cost (runtime and peak memory) of two or more pipelines on the same input files, e.g. benchmarking SoloTE vs scTE on simulated scRNA-Seq BAMs.

## When NOT to use

- The goal is comparing output quality (TE detection sensitivity, UMAP clustering, marker detection) rather than computational cost — use functional benchmarking instead
- Inputs differ across the tools being compared (different BAMs, genomes, or thread counts), which invalidates a fair resource-usage comparison
- The tool being profiled has no distinct phases and its index-build time should not be conflated with per-input quantification time

## Inputs

- Simulated scRNA-Seq BAM files per genome (hg38, mm10) and TE-group condition
- TE annotation in BED format (converted from RepeatMasker .rm.out via SoloTE_RepeatMasker_to_BED.py)
- Tool invocations executed with a fixed thread count (8 threads)

## Outputs

- Per-run wall-clock time and peak memory (max RSS) measurements from /usr/bin/time
- runtime_memory_benchmark.csv with columns: tool, genome, TE-group, phase (index-build vs quantification), wall-clock minutes, peak memory in Gb
- Flags for measurements outside reported reference ranges

## How to apply

Run each pipeline on an identical set of inputs under identical threading (e.g. 8 threads): invoke 'python SoloTE_pipeline.py --threads 8 --bam <BAMfile> --teannotation <TE_BED> --outputprefix <Prefix> --outputdir <OutputDirectory>' wrapped in /usr/bin/time to capture wall-clock time and maximum resident set size. For tools with separate phases (scTE), time the TE-index-build step independently from the per-BAM quantification run. Key each measurement by tool, genome version (hg38 or mm10), TE-group condition, and phase. Compile all per-run measurements into a single results table (runtime_memory_benchmark.csv) with columns tool, genome, TE-group, phase, wall-clock minutes, and peak memory in Gb. Validate by flagging any run falling outside the paper's reported ranges (SoloTE <25 min and ≤15 Gb; scTE 20–60 min and 40–60 Gb; scTE index-build 5–42 min).

## Related tools

- **SoloTE** (Pipeline being benchmarked; runs per-BAM TE quantification on simulated BAM files with 8 threads and its resource usage is captured by /usr/bin/time) — https://github.com/bvaldebenitom/SoloTE
- **scTE** (Comparison tool; its separate TE-index-build step and per-BAM quantification run are timed independently)
- **BEDTools** (Dependency of the SoloTE pipeline being benchmarked; BEDTools v2.29.2 or higher required in PATH) — https://github.com/arq5x/bedtools2/releases
- **Samtools** (Dependency of the SoloTE pipeline; Samtools v1.16 or higher required in PATH)
- **Pysam** (Python module dependency of the SoloTE pipeline being profiled)
- **Pandas** (Python module (v1.5.0 or higher) usable to compile per-run measurements into the runtime_memory_benchmark.csv table)

## Examples

```
/usr/bin/time -v python SoloTE_pipeline.py --threads 8 --bam <BAMfile> --teannotation <TE_BED> --outputprefix <Prefix> --outputdir <OutputDirectory>
```

## Evaluation signals

- SoloTE runs complete in <25 minutes with ≤15 Gb peak memory per run
- scTE quantification runs fall in the 20–60 minute and 40–60 Gb ranges
- scTE TE-index-build step takes 5–42 minutes
- All runs used identical inputs and thread count (8 threads) across tools, ensuring a fair comparison
- runtime_memory_benchmark.csv contains one row per tool/genome/TE-group/phase combination with non-empty wall-clock and peak-memory values; any measurement outside reported ranges is flagged

## Limitations

- Resource measurements are specific to the simulated BAM files and genomes used (hg38 and mm10); runs on other data sizes or hardware may fall outside the reported ranges and must be re-validated
- scTE requires phase separation (index-build vs per-BAM quantification); conflating the two phases inflates apparent per-run cost
- SoloTE must start from BAM files produced by upstream aligners (CellRanger or STAR); differences in upstream alignment would confound the comparison

## Evidence

- [results] SoloTE is faster and more memory-efficient than scTE: [evidence span withheld — anchor status: not_found]
- [abstract] SoloTE outperforms previous approach in resources: "SoloTE, a pipeline that outperforms the previous approach in terms of computational resources and by allowing the inclusion of locus-specific TE activity in scRNA-Seq expression matrixes"
- [discussion] Simulated genomes used for benchmarking: "we simulated data from the human genome hg38 version, and from the mouse genome mm10 version"
- [readme] SoloTE invocation from README: "python SoloTE_pipeline.py --threads NumberOfThreads --bam BAMfile --teannotation BEDfile --outputprefix Prefix --outputdir OutputDirectory"
- [readme] SoloTE dependencies: "Samtools v1.16 or higher (http://www.htslib.org/download/) - BEDTools v2.29.2 or higher (https://github.com/arq5x/bedtools2/releases)"
