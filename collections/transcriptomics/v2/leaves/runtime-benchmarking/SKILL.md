---
name: runtime-benchmarking
description: Use when comparing processing-time (wall-clock runtime) of scKB-COPILOT
  against Cell Ranger on the same scRNA-seq FASTQ datasets, or when reproducing the
  paper's runtime benchmark table across toy col0, full col0, and PBMC datasets.
when_to_use_negative:
- Input is already processed count matrices rather than raw scRNA-seq FASTQ files,
  since the benchmark measures end-to-end alignment and count matrix generation time
- The goal is accuracy or downstream-quality comparison rather than processing-time
  comparison
- The pipeline being benchmarked is run in iterative quality-filtering mode or with
  do.seurat=TRUE downstream analysis, which adds non-comparable COPILOT runtime
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scKB
  role: Wrapper of kallisto and bustools whose accelerated alignment and count matrix
    generation runtime is measured in non-iterative mode
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: COPILOT
  role: Quality filtering step timed with do.seurat=FALSE and do.annotation=FALSE,
    supplying the required mt.pattern
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: kallisto
  role: Pseudoalignment engine inside scKB/COPILOT responsible for the ~30-fold speedup
    being benchmarked
- name: bustools
  role: Generates transcript count matrices inside scKB; its runtime is included in
    the scKB measurement
- name: Cell Ranger
  role: Baseline tool run with 'cellranger count' and the --nosecondary flag whose
    runtime is recorded for comparison
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:f0cf43d37e0ba45fb098e62d3e85ac2fae80683cc07d88be4a2c4b0be79f72d3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ohlerlab/COPILOT
  repo_url: https://github.com/Hsu-Che-Wei/COPILOT
  tools:
  - COPILOT
  - Cell Ranger
  - bustools
  - kallisto
  - scKB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# runtime-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark wall-clock runtime of the scKB-COPILOT pipeline (kallisto/bustools pseudoalignment plus COPILOT non-arbitrary quality filtering) against 10x Genomics Cell Ranger 6.1.2 across matched scRNA-seq FASTQ datasets, verifying the reported ~30-fold processing-time advantage over CellRanger 3.1.

## When to use

Use when comparing processing-time (wall-clock runtime) of scKB-COPILOT against Cell Ranger on the same scRNA-seq FASTQ datasets, or when reproducing the paper's runtime benchmark table across toy col0, full col0, and PBMC datasets.

## When NOT to use

- Input is already processed count matrices rather than raw scRNA-seq FASTQ files, since the benchmark measures end-to-end alignment and count matrix generation time
- The goal is accuracy or downstream-quality comparison rather than processing-time comparison
- The pipeline being benchmarked is run in iterative quality-filtering mode or with do.seurat=TRUE downstream analysis, which adds non-comparable COPILOT runtime

## Inputs

- scRNA-seq FASTQ files for each benchmarked dataset (toy col0, full col0 10x v3, PBMC)
- pre-built Cell Ranger transcriptome reference
- kallisto transcriptome index and genome for scKB
- COPILOT parameters: mt.pattern (e.g. 'ATMG'), filtering-only settings

## Outputs

- runtime_benchmark.csv with CellRanger and scKB-COPILOT wall-clock runtimes per dataset
- per-tool runtimes (e.g. CellRanger 640 min vs scKB 22 min + COPILOT 3 min on full col0)
- computed speedup ratio between CellRanger and scKB-COPILOT

## How to apply

Run Cell Ranger 'cellranger count' with the --nosecondary flag on each benchmarked FASTQ dataset (toy col0, full col0 with 386 million reads at 10x Genomics v3 chemistry, PBMC) and record wall-clock runtime. In parallel, run the scKB pipeline (kallisto pseudoalignment plus bustools transcript count matrix generation) in non-iterative mode on the same FASTQs, then run COPILOT on each scKB output with filtering only (do.seurat=FALSE, do.annotation=FALSE), supplying mt.pattern (e.g. 'ATMG' for Arabidopsis) and default filtering parameters (mt.threshold=5, top.percent=1, filtering.ratio=1, min.UMI.low.quality=100, min.UMI.high.quality=300). Tabulate CellRanger versus scKB and COPILOT runtimes side by side per dataset in runtime_benchmark.csv. The rationale is that kallisto pseudoalignment makes scKB/COPILOT about 30 times faster than CellRanger 3.1: on the full col0 dataset Cell Ranger took 640 min while scKB-COPILOT took 25 min (scKB 22 min, COPILOT 3 min).

## Related tools

- **scKB** (Wrapper of kallisto and bustools whose accelerated alignment and count matrix generation runtime is measured in non-iterative mode) — https://github.com/Hsu-Che-Wei/COPILOT
- **COPILOT** (Quality filtering step timed with do.seurat=FALSE and do.annotation=FALSE, supplying the required mt.pattern) — https://github.com/Hsu-Che-Wei/COPILOT
- **kallisto** (Pseudoalignment engine inside scKB/COPILOT responsible for the ~30-fold speedup being benchmarked)
- **bustools** (Generates transcript count matrices inside scKB; its runtime is included in the scKB measurement)
- **Cell Ranger** (Baseline tool run with 'cellranger count' and the --nosecondary flag whose runtime is recorded for comparison)

## Examples

```
cellranger count --id=col0_benchmark --transcriptome=/path/to/ref --fastqs=/path/to/fastqs --nosecondary
```

## Evaluation signals

- Tabulated speedup ratio between CellRanger and scKB-COPILOT is approximately 30-fold, consistent with the reported processing-time advantage
- Cell Ranger is run with 'cellranger count' using the --nosecondary flag so its runtime excludes secondary analysis and is comparable
- scKB is run in non-iterative mode and COPILOT in filtering-only mode (do.seurat=FALSE, do.annotation=FALSE) with default parameters (mt.threshold=5, top.percent=1, filtering.ratio=1, min.UMI.low.quality=100, min.UMI.high.quality=300)
- Each runtime measurement is a recorded wall-clock time for the same FASTQ datasets across tools
- On the full col0 dataset (386 million reads, 10x v3), runtimes reproduce the order CellRanger 640 min >> scKB 22 min > COPILOT 3 min

## Limitations

- The ~30-fold speedup claim is relative to CellRanger 3.1; the benchmark protocol uses Cell Ranger 6.1.2, so absolute runtime ratios may differ across CellRanger versions
- Runtime depends on dataset size and chemistry (e.g. 386 million reads at 10x Genomics v3 for full col0); timings do not transfer to other datasets
- COPILOT runtime is only representative of filtering-only mode; enabling do.seurat=TRUE downstream analysis (normalization, PCA, UMAP, clustering via Seurat) increases runtime, as seen in the ~2-minute toy-data run with full downstream steps

## Evidence
