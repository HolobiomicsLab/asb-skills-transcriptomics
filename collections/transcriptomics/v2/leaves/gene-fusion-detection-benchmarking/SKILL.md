---
name: gene-fusion-detection-benchmarking
description: Use when comparing computational time of scFusion against bulk fusion
  detection methods (Arriba, STAR-Fusion, EricScript, FusionCatcher) on real Smart-Seq
  scRNA-seq datasets.
when_to_use_negative:
- Input FASTQ files use the *_1.fq naming convention or originate from a 10X protocol
  — scFusion targets Smart-Seq data with *_1.fastq/*_2.fastq naming only
- The goal is benchmarking detection sensitivity or false discovery rate on simulated
  fusions rather than runtime comparison on real scRNA-seq data
- Computational resources fall below the recommended 64 GB memory and 8 CPU cores
  per task, making runtime comparisons across methods incomparable
edam_operation: http://edamontology.org/operation_2934
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: scFusion
  role: Single-cell gene fusion detection pipeline whose end-to-end runtime (scFusion.py
    driving STAR, samtools/bedtools, R/stringr, Python/pyensembl) is measured against
    bulk methods
  repo: https://github.com/ZijieJin/scFusion
- name: STAR
  role: Spliced aligner (>= 2.7.2d) invoked by scFusion.py for single-cell RNA-seq
    alignment; a required prerequisite in PATH
  repo: https://github.com/alexdobin/STAR
- name: samtools
  role: SAM/BAM processing step (>= 1.0, tested on 1.10) within the scFusion runtime
    pipeline
- name: bedtools
  role: Genomic interval processing step (tested on 2.29.2) within the scFusion runtime
    pipeline
- name: pyensembl
  role: Python module used in the scFusion pipeline for Ensembl annotation access
    during runtime measurement
  repo: https://github.com/openvax/pyensembl
- name: stringr
  role: R package (R >= 3.5) used in the string-processing step of the scFusion pipeline
- name: Arriba
  role: Bulk RNA-seq fusion caller benchmarked against scFusion for CPU time on the
    same five datasets
- name: STAR-Fusion
  role: Bulk RNA-seq fusion caller benchmarked against scFusion for CPU time
- name: EricScript
  role: Bulk RNA-seq fusion caller benchmarked against scFusion for CPU time
- name: FusionCatcher
  role: Bulk RNA-seq fusion caller benchmarked against scFusion for CPU time
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:ef4d34c297260c16af3ca8fa2b8cf613a280de7c0d61a3df1caebf4f111fa775
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZijieJin/scFusion
  tools:
  - Arriba
  - EricScript
  - FusionCatcher
  - Python
  - R
  - STAR
  - STAR-Fusion
  - bedtools
  - pyensembl
  - samtools
  - scFusion
  - stringr
  edam_operation: http://edamontology.org/operation_2934
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# gene-fusion-detection-benchmarking

## Summary

Benchmark the computational runtime of the scFusion single-cell gene fusion detection pipeline (STAR alignment, samtools/bedtools processing, R/stringr and Python/pyensembl steps) against bulk RNA-seq fusion callers (Arriba, STAR-Fusion, EricScript, FusionCatcher) across five Smart-Seq scRNA-seq datasets, reproducing the reported Fig 4a CPU-hour comparison.

## When to use

Use when comparing computational time of scFusion against bulk fusion detection methods (Arriba, STAR-Fusion, EricScript, FusionCatcher) on real Smart-Seq scRNA-seq datasets.

## When NOT to use

- Input FASTQ files use the *_1.fq naming convention or originate from a 10X protocol — scFusion targets Smart-Seq data with *_1.fastq/*_2.fastq naming only
- The goal is benchmarking detection sensitivity or false discovery rate on simulated fusions rather than runtime comparison on real scRNA-seq data
- Computational resources fall below the recommended 64 GB memory and 8 CPU cores per task, making runtime comparisons across methods incomparable

## Inputs

- Paired-end Smart-Seq scRNA-seq FASTQ files per cell (named *_1.fastq, *_2.fastq; *_1.fq not allowed) for each of five datasets
- Reference genome FASTA (e.g. hg19.fa, ~3G)
- GTF annotation file (*.gtf) from Ensembl, NCBI, or UCSC
- Unzipped hg19mappability file placed in the data folder (optional; if absent the mappability filter is turned off)
- Installed bulk fusion callers: Arriba, STAR-Fusion, EricScript, FusionCatcher

## Outputs

- Wall-clock and CPU time recorded for each method-dataset pair
- cpu_hours_comparison.csv: total CPU hours per method (scFusion, Arriba, STAR-Fusion, EricScript, FusionCatcher) summed across the five datasets

## How to apply

Assemble per-dataset inputs: paired-end *_1.fastq/*_2.fastq per-cell files from the Smart-Seq protocol, an hg19 reference genome FASTA (~3G), a GTF annotation from Ensembl/NCBI/UCSC, and the unzipped hg19mappability file in the data folder (omitting it disables the mappability filter). Run scFusion end-to-end via scFusion.py, which drives STAR (>= 2.7.2d) alignment followed by samtools (>= 1.0), bedtools (2.29.2), R (>= 3.5, stringr) and Python 3 (pyensembl) steps, recording wall-clock and CPU time. Run each bulk caller on the same five datasets under the recommended configuration of 64 GB memory and 8 CPU cores per task. Sum recorded CPU times per method across all five datasets into total CPU hours, tabulate per method, and write the comparison table to cpu_hours_comparison.csv.

## Related tools

- **scFusion** (Single-cell gene fusion detection pipeline whose end-to-end runtime (scFusion.py driving STAR, samtools/bedtools, R/stringr, Python/pyensembl) is measured against bulk methods) — https://github.com/ZijieJin/scFusion
- **STAR** (Spliced aligner (>= 2.7.2d) invoked by scFusion.py for single-cell RNA-seq alignment; a required prerequisite in PATH) — https://github.com/alexdobin/STAR
- **samtools** (SAM/BAM processing step (>= 1.0, tested on 1.10) within the scFusion runtime pipeline)
- **bedtools** (Genomic interval processing step (tested on 2.29.2) within the scFusion runtime pipeline)
- **pyensembl** (Python module used in the scFusion pipeline for Ensembl annotation access during runtime measurement) — https://github.com/openvax/pyensembl
- **stringr** (R package (R >= 3.5) used in the string-processing step of the scFusion pipeline)
- **Arriba** (Bulk RNA-seq fusion caller benchmarked against scFusion for CPU time on the same five datasets)
- **STAR-Fusion** (Bulk RNA-seq fusion caller benchmarked against scFusion for CPU time)
- **EricScript** (Bulk RNA-seq fusion caller benchmarked against scFusion for CPU time)
- **FusionCatcher** (Bulk RNA-seq fusion caller benchmarked against scFusion for CPU time)

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- scFusion's total CPU hours across the five datasets are approximately half of Arriba's
- scFusion's total CPU hours are approximately 30% or less of each of STAR-Fusion, EricScript, and FusionCatcher
- cpu_hours_comparison.csv contains one total CPU-hour value per method, each summed over all five datasets
- Every method-dataset pair was run under the same recommended configuration (64 GB memory, 8 CPU cores per task) so runtimes are comparable
- scFusion runs completed end-to-end (STAR alignment through samtools/bedtools, R and Python steps) without the mappability filter being silently disabled

## Limitations

- scFusion requires Smart-Seq FASTQ files named *_1.fastq and *_2.fastq; *_1.fq is not allowed
- If the hg19mappability file is not provided, scFusion turns off the mappability filter
- scFusion has reduced sensitivity for fusions with low expression, as is the case for all RNA-based methods
- Detection power is limited for rare fusions in highly heterogeneous tumor samples; overcoming this requires sequencing more cells and/or deeper per-cell sequencing
- A job scheduler (e.g. Slurm) is optional; without it, coordinating many per-cell FASTQ tasks may be cumbersome

## Evidence

- [results] scFusion is more computationally efficient than bulk methods; runtime is about half of Arriba and ~30% or less of other bulk methods: [evidence span withheld — anchor status: not_found]
- [methods] scFusion.py pipeline invocation steps: "Download all scripts, unzip the hg19mappability file in the data folder, and run scFusion.py."
- [methods] Docker command line for scFusion: "docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]"
- [readme] Recommended hardware configuration from README: "64 GB memory or more for each task

- 8 CPU cores or more for each task"
- [readme] FASTQ naming requirements from README: "Single cell RNA sequencing files from Smart-Seq protocol. File names should be *_1.fastq, *_2.fastq (e.g. 1_1.fastq, 1_2.fastq, 2_1.fastq, 2_2.fastq) ** \*_1.fq is not allowed.**"
