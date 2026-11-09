---
name: fusion-caller-benchmark-evaluation
description: Use when evaluating or comparing RNA-seq fusion callers on data with
  known ground-truth fusions, especially simulated single-cell datasets containing
  technical chimeric reads, to quantify precision/recall/F-score differences.
when_to_use_negative:
- Data are bulk RNA-seq without single-cell structure — scFusion's joint analysis
  across cells and the single-cell simulation design do not apply
- Input is 10X scRNA-seq rather than Smart-Seq data — scFusion is designed for Smart-Seq
  FASTQ input only
- No ground-truth fusion list exists, so precision/recall/F-score against truth cannot
  be computed
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: scFusion
  role: Single-cell fusion detection method under evaluation, run via scFusion.py
    with a statistical model and deep-learning model
  repo: https://github.com/ZijieJin/scFusion
- name: STAR
  role: Spliced aligner (>= 2.7.2d) used by scFusion to align simulated single-cell
    reads
  repo: https://github.com/alexdobin/STAR
- name: samtools
  role: Read processing (>= 1.0, tested on 1.10) within the scFusion pipeline on simulated
    BAMs
- name: bedtools
  role: Genomic interval processing (tested on 2.29.2) within the scFusion pipeline
- name: pyensembl
  role: Python module for Ensembl GTF/annotation handling in scFusion
  repo: https://github.com/openvax/pyensembl
- name: Arriba / STAR-Fusion / FusionCatcher / EricScript
  role: Competing bulk RNA-seq fusion callers benchmarked on the same simulated FASTQ
    files
- name: FusionSimulatorToolkit
  role: Generates simulated Smart-Seq scRNA-seq FASTQ files with 1% spiked artificial
    chimeric fusion reads and ground truth
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:e1da4e8540eb3862710c5c2bba6d7c7f4d43d84dc0b0687c032a020f8261dec5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZijieJin/scFusion
  tools:
  - Arriba / STAR-Fusion / FusionCatcher / EricScript
  - FusionSimulatorToolkit
  - Python
  - R
  - STAR
  - bedtools
  - pyensembl
  - samtools
  - scFusion
  - stringr
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# fusion-caller-benchmark-evaluation

## Summary

Benchmark gene fusion detection methods by running them on simulated Smart-Seq scRNA-seq FASTQ data with spiked-in artificial chimeric reads and scoring precision, recall, and F-score against simulation ground truth, as done to compare scFusion against bulk callers (Arriba, STAR-Fusion, FusionCatcher, EricScript).

## When to use

Use when evaluating or comparing RNA-seq fusion callers on data with known ground-truth fusions, especially simulated single-cell datasets containing technical chimeric reads, to quantify precision/recall/F-score differences.

## When NOT to use

- Data are bulk RNA-seq without single-cell structure — scFusion's joint analysis across cells and the single-cell simulation design do not apply
- Input is 10X scRNA-seq rather than Smart-Seq data — scFusion is designed for Smart-Seq FASTQ input only
- No ground-truth fusion list exists, so precision/recall/F-score against truth cannot be computed

## Inputs

- Simulated paired-end Smart-Seq scRNA-seq FASTQ files (*_1.fastq, *_2.fastq) with 1% spiked chimeric fusion reads and known ground-truth fusion list
- Reference genome FASTQ/FASTA (e.g. hg19.fa, ~3G)
- GTF annotation file (Ensembl, NCBI, or UCSC)
- hg19 mappability file (UCSC mappability table)
- Detected fusion calls from each evaluated caller (scFusion output plus Arriba, STAR-Fusion, FusionCatcher, EricScript outputs)

## Outputs

- Per-method, per-setting precision, recall, and F-score table aggregated across 10 replicates (benchmark_results.csv)
- Ablation comparison showing precision gain and sensitivity loss when the deep-learning model is disabled

## How to apply

Generate simulated paired-end Smart-Seq scRNA-seq FASTQ files with FusionSimulatorToolkit for 500- and 1000-cell datasets at 2, 3, and 4 million reads per cell, spiking 1% artificial chimeric fusion reads with 10 replicates per setting. Run each caller — scFusion via scFusion.py with the hg19 reference genome, GTF annotation, and hg19 mappability file (STAR >= 2.7.2d for alignment, samtools/bedtools for read processing), plus an ablation variant with the deep-learning (tensorflow/keras) model turned off — and run the competing bulk callers Arriba, STAR-Fusion, FusionCatcher, and EricScript on the same FASTQ files. Compare each caller's detected fusions to the simulation ground truth to compute precision, recall, and F-score per method per setting, then aggregate across the 10 replicates. In the reference benchmark, scFusion achieved precision 0.921 and F-score 0.925 versus bulk-method precisions of 0.434-0.503 and F-scores of 0.574-0.667; the deep-learning ablation check should reproduce a precision gain of 0.049-0.168 with sensitivity loss below 0.04.

## Related tools

- **scFusion** (Single-cell fusion detection method under evaluation, run via scFusion.py with a statistical model and deep-learning model) — https://github.com/ZijieJin/scFusion
- **STAR** (Spliced aligner (>= 2.7.2d) used by scFusion to align simulated single-cell reads) — https://github.com/alexdobin/STAR
- **samtools** (Read processing (>= 1.0, tested on 1.10) within the scFusion pipeline on simulated BAMs)
- **bedtools** (Genomic interval processing (tested on 2.29.2) within the scFusion pipeline)
- **pyensembl** (Python module for Ensembl GTF/annotation handling in scFusion) — https://github.com/openvax/pyensembl
- **Arriba / STAR-Fusion / FusionCatcher / EricScript** (Competing bulk RNA-seq fusion callers benchmarked on the same simulated FASTQ files)
- **FusionSimulatorToolkit** (Generates simulated Smart-Seq scRNA-seq FASTQ files with 1% spiked artificial chimeric fusion reads and ground truth)

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- scFusion precision at similar recall exceeds bulk callers: reference values are precision 0.921 / F-score 0.925 vs bulk precisions 0.434-0.503 and F-scores 0.574-0.667
- Deep-learning ablation check: turning off the tensorflow/keras model reproduces a precision reduction of 0.049-0.168 with sensitivity loss below 0.04
- Results are stable across all simulation settings (500 vs 1000 cells; 2, 3, 4 million reads per cell) and 10 replicates per setting aggregate into benchmark_results.csv without divergence
- Each caller's detected fusions match simulation ground truth labels, so precision/recall/F-score are computable per method per setting

## Limitations

- scFusion has reduced sensitivity for fusions with low expression, as with all RNA-based methods
- Detection power is limited for rare fusions in highly heterogeneous tumor samples; overcoming this requires sequencing more cells and/or deeper per-cell sequencing
- Heavy PCR amplification generates artificial chimeric reads that create false-positive fusion candidates, motivating the simulation design
- Recommend 64 GB memory or more and 8 CPU cores or more per task; scFusion input requires *_1.fastq/*_2.fastq naming (*_1.fq is not allowed)

## Evidence

- [results] High precision of scFusion over bulk methods on simulation: [evidence span withheld — anchor status: not_found]
- [intro] False positives from amplification motivate simulation design: "the heavy amplification step may generate artificial chimeric reads, leading to false-positive fusion candidates"
- [discussion] Deep-learning model contributes precision: "When deep-learning model was turned off, the precisions of scFusion reduced"
- [readme] scFusion software prerequisites: "[STAR](https://github.com/alexdobin/STAR) >= 2.7.2d (tested on 2.7.2d and 2.7.8a)
- samtools >= 1.0 (tested on version 1.10)
- bedtools (tested on version 2.29.2)"
- [readme] Input format requirement for single-cell FASTQ files: "Single cell RNA sequencing files from Smart-Seq protocol. File names should be *_1.fastq, *_2.fastq"
- [abstract] Low false discovery rate claim: "We evaluate the performance of scFusion using simulated and five real scRNA-seq datasets and find that scFusion can efficiently and sensitively detect fusions with a low false discovery rate."
