---
name: single-cell-rnaseq-fusion-simulation
description: Use when evaluating or comparing fusion detection methods on simulated
  single-cell RNA-seq data, or when quantifying how deep-learning/statistical filtering
  affects scFusion's precision and F-score.
when_to_use_negative:
- Input is real 10X Genomics scRNA-seq data — scFusion is built for Smart-Seq data
  rather than 10X
- FASTQ files are named *_1.fq, which the scFusion pipeline does not accept
- No simulation ground truth is available, so precision/recall/F-score against known
  fusions cannot be computed
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: scFusion
  role: Single-cell fusion detection pipeline being benchmarked; run via scFusion.py
    with deep-learning ablation variant
  repo: https://github.com/ZijieJin/scFusion
- name: STAR
  role: Spliced aligner used to align simulated scRNA-seq reads (>= 2.7.2d)
  repo: https://github.com/alexdobin/STAR
- name: pyensembl
  role: Python module for Ensembl annotation used within the scFusion pipeline
  repo: https://github.com/openvax/pyensembl
- name: samtools
  role: Read processing in the scFusion pipeline (>= 1.0, tested on 1.10)
- name: bedtools
  role: Genomic interval operations in the scFusion pipeline (tested on 2.29.2)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:caeaddd647df1fa973cb3274fbea091e988b6c255ff9fd70ff34169cc0125f86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZijieJin/scFusion
  tools:
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
yes_rate: 0.5625
n_claims: 16
---

# single-cell-rnaseq-fusion-simulation

## Summary

Benchmark single-cell gene fusion callers by simulating Smart-Seq paired-end scRNA-seq FASTQs with spiked artificial chimeric fusion reads, then score precision, recall, and F-score against the simulation ground truth to compare scFusion against bulk methods (Arriba, STAR-Fusion, FusionCatcher, EricScript).

## When to use

Use when evaluating or comparing fusion detection methods on simulated single-cell RNA-seq data, or when quantifying how deep-learning/statistical filtering affects scFusion's precision and F-score.

## When NOT to use

- Input is real 10X Genomics scRNA-seq data — scFusion is built for Smart-Seq data rather than 10X
- FASTQ files are named *_1.fq, which the scFusion pipeline does not accept
- No simulation ground truth is available, so precision/recall/F-score against known fusions cannot be computed

## Inputs

- Simulated paired-end Smart-Seq scRNA-seq FASTQ files (*_1.fastq, *_2.fastq) with spiked chimeric fusion reads
- Simulation ground truth of spiked fusions per setting
- hg19 reference genome FASTA (~3G)
- GTF annotation file (Ensembl, NCBI, or UCSC)
- hg19 mappability file (UCSC)

## Outputs

- Per-replicate precision, recall, and F-score for each fusion caller and each simulation setting
- Aggregated benchmark_results.csv across 10 replicates
- Deep-learning ablation comparison quantifying precision gain (0.049-0.168) and sensitivity loss (<0.04)

## How to apply

Generate simulated Smart-Seq paired-end scRNA-seq FASTQ files (naming *_1.fastq/*_2.fastq, since *_1.fq is not accepted) with FusionSimulatorToolkit for 500- and 1000-cell datasets at 2, 3, and 4 million reads per cell, spiking ~1% artificial chimeric fusion reads with 10 replicates per setting to mimic amplification artifacts. Run scFusion.py on each dataset with the hg19 reference genome (.fa, ~3G), GTF annotation, and hg19 mappability file, using STAR >= 2.7.2d for alignment and samtools/bedtools for read processing (64 GB memory, 8+ cores recommended). Also run an ablation variant with the tensorflow/keras deep-learning model turned off, plus the bulk callers Arriba, STAR-Fusion, FusionCatcher, and EricScript on the same FASTQs. Match each caller's detections against the simulation ground truth to compute precision, recall, and F-score per method per setting, and aggregate over replicates into benchmark_results.csv.

## Related tools

- **scFusion** (Single-cell fusion detection pipeline being benchmarked; run via scFusion.py with deep-learning ablation variant) — https://github.com/ZijieJin/scFusion
- **STAR** (Spliced aligner used to align simulated scRNA-seq reads (>= 2.7.2d)) — https://github.com/alexdobin/STAR
- **pyensembl** (Python module for Ensembl annotation used within the scFusion pipeline) — https://github.com/openvax/pyensembl
- **samtools** (Read processing in the scFusion pipeline (>= 1.0, tested on 1.10))
- **bedtools** (Genomic interval operations in the scFusion pipeline (tested on 2.29.2))

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- In the 1000-cell, 4M-reads-per-cell simulation, scFusion reaches precision ~0.921 and F-score ~0.925, versus bulk-method precisions of 0.434-0.503
- Turning off the deep-learning model drops scFusion precision by 0.049-0.168 with sensitivity loss below 0.04
- Each caller's detected fusions must match the simulation's ground-truth spiked fusion list to compute precision/recall/F-score consistently
- Results are aggregated over 10 replicates per setting (500/1000 cells x 2/3/4 million reads per cell) so metric variance is captured
- Pipeline runs only with valid Smart-Seq file naming (*_1.fastq/*_2.fastq) and a provided mappability file; otherwise the mappability filter is silently turned off

## Limitations

- scFusion has reduced sensitivity for fusions with low expression, as for all RNA-based methods
- Detection power is limited for rare fusions in highly heterogeneous tumor samples; overcome only by sequencing more and/or deeper cells
- False positives from the heavy amplification step generating artificial chimeric reads motivate, and complicate, simulation-based evaluation

## Evidence

- [readme] Smart-Seq FASTQ input convention: "Single cell RNA sequencing files from Smart-Seq protocol. File names should be *_1.fastq, *_2.fastq"
- [discussion] Deep-learning ablation effect: "When deep-learning model was turned off, the precisions of scFusion reduced"
- [discussion] Low sensitivity limitation: "scFusion has reduced sensitivity for fusions with low expression (as shown in the simulation), just as is the case for all RNA-based methods"
- [methods] Mappability filter behavior: "If you do not provide this file, scFusion will turn off the mappability filter."
- [intro] False positives from amplification: "the heavy amplification step may generate artificial chimeric reads, leading to false-positive fusion candidates"
