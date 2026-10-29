---
name: cross-method-fusion-comparison
description: Use when scRNA-seq fusion calls from scFusion must be benchmarked against
  bulk callers (Arriba, STAR-Fusion, EricScript, FusionCatcher) for sensitivity or
  CPU-time comparison across Smart-Seq datasets.
when_to_use_negative:
- The data are bulk RNA-seq with no per-cell resolution — the single-cell sensitivity
  claims (TCR recombinations detected per cell) cannot be evaluated.
- The scRNA-seq data are 10X rather than Smart-Seq — scFusion is documented as a pipeline
  'for detecting gene fusions at single-cell resolution (For Smart-Seq data rather
  than 10X)'.
- The goal is calling a single known fusion in one sample, not method benchmarking
  — a single-caller run suffices and the cross-method runtime/agreement comparison
  adds overhead.
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scFusion
  role: Single-cell fusion detection pipeline under evaluation; run via scFusion.py
    (STAR alignment, samtools/bedtools, statistical + deep-learning filters)
  repo: https://github.com/ZijieJin/scFusion
- name: STAR
  role: Aligner (>= 2.7.2d) invoked inside scFusion.py to align single-cell reads
    for the fusion calls being compared
  repo: https://github.com/alexdobin/STAR
- name: pyensembl
  role: Python module used by scFusion for annotation handling of fusion candidates
  repo: https://github.com/openvax/pyensembl
- name: Arriba / STAR-Fusion / EricScript / FusionCatcher
  role: Bulk RNA-seq fusion callers used as comparison baselines on the same five
    scRNA-seq datasets
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:d037a33a43c1af6c8ff0da27316168a33c50ea9a635977208018fd4674d2fa0f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZijieJin/scFusion
  tools:
  - Arriba / STAR-Fusion / EricScript / FusionCatcher
  - Python
  - R
  - STAR
  - bedtools
  - numpy
  - pyensembl
  - pysam
  - samtools
  - scFusion
  - scipy
  - stringr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# cross-method-fusion-comparison

## Summary

Benchmarks single-cell fusion detection (scFusion) against bulk RNA-seq fusion callers (Arriba, STAR-Fusion, EricScript, FusionCatcher) on the same Smart-Seq scRNA-seq datasets, comparing both detection sensitivity (e.g., V(D)J recombinations) and computational cost.

## When to use

Use when scRNA-seq fusion calls from scFusion must be benchmarked against bulk callers (Arriba, STAR-Fusion, EricScript, FusionCatcher) for sensitivity or CPU-time comparison across Smart-Seq datasets.

## When NOT to use

- The data are bulk RNA-seq with no per-cell resolution — the single-cell sensitivity claims (TCR recombinations detected per cell) cannot be evaluated.
- The scRNA-seq data are 10X rather than Smart-Seq — scFusion is documented as a pipeline 'for detecting gene fusions at single-cell resolution (For Smart-Seq data rather than 10X)'.
- The goal is calling a single known fusion in one sample, not method benchmarking — a single-caller run suffices and the cross-method runtime/agreement comparison adds overhead.

## Inputs

- Smart-Seq paired-end scRNA-seq FASTQ files (*_1.fastq, *_2.fastq; *_1.fq not allowed) from five real datasets
- Reference genome FASTA (e.g., hg19.fa, ~3G)
- GTF annotation file (Ensembl/NCBI/UCSC)
- Unzipped hg19mappability file in the data folder
- Installed bulk fusion callers: Arriba, STAR-Fusion, EricScript, FusionCatcher

## Outputs

- Per-method fusion call tables over the same single-cell datasets
- V(D)J recombination counts (e.g., TRAJ33-TRAV1-2: 126 cells; TRAJ12-TRAV1-2: 20 cells) and SLC4A10 Fisher's exact test P-value
- STAR-Fusion candidate V(D)J fraction (expected 1.8%)
- cpu_hours_comparison.csv with total CPU hours per method across the five datasets

## How to apply

Run scFusion end-to-end by executing scFusion.py on Smart-Seq paired FASTQ files (*_1.fastq/*_2.fastq), which drives STAR (>= 2.7.2d) alignment, samtools/bedtools processing, the hg19 mappability filter, and the statistical plus deep-learning (tensorflow/keras/numpy) filtering models. Run each bulk caller (Arriba, STAR-Fusion, EricScript, FusionCatcher) on the identical datasets under the recommended 64 GB memory / 8 CPU core configuration, recording runtime per method-dataset pair. For sensitivity comparison, compile fusion call tables per method and quantify method-specific events — e.g., in the 2355-cell T cell dataset scFusion detects 8 TCR recombinations (TRAJ33-TRAV1-2 in 126 cells, TRAJ12-TRAV1-2 in 20 cells) validated against SLC4A10 expression by Fisher's exact test (P < 10^-16), while only 1.8% of STAR-Fusion candidates are V(D)J recombinations and other bulk methods detect none. Sum CPU times per method across the five scRNA-seq datasets and tabulate total CPU hours, expecting scFusion at roughly half of Arriba and ~30% or less of the other bulk methods.

## Related tools

- **scFusion** (Single-cell fusion detection pipeline under evaluation; run via scFusion.py (STAR alignment, samtools/bedtools, statistical + deep-learning filters)) — https://github.com/ZijieJin/scFusion
- **STAR** (Aligner (>= 2.7.2d) invoked inside scFusion.py to align single-cell reads for the fusion calls being compared) — https://github.com/alexdobin/STAR
- **pyensembl** (Python module used by scFusion for annotation handling of fusion candidates) — https://github.com/openvax/pyensembl
- **Arriba / STAR-Fusion / EricScript / FusionCatcher** (Bulk RNA-seq fusion callers used as comparison baselines on the same five scRNA-seq datasets)

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- scFusion's fusion table reproduces the 8 TCR recombinations with per-fusion cell counts (TRAJ33-TRAV1-2: 126 cells; TRAJ12-TRAV1-2: 20 cells)
- Fisher's exact test of SLC4A10 expression vs TCR fusion detection yields P < 10^-16, matching Table 1
- STAR-Fusion candidate V(D)J recombination fraction equals ~1.8% and no other bulk method detects any V(D)J recombination
- scFusion total CPU hours are approximately half of Arriba's and ~30% or less of STAR-Fusion, EricScript, and FusionCatcher across the five datasets
- Turning off the deep-learning model lowers precision — run the full statistical + deep-learning filter stack so comparisons reflect the published performance

## Limitations

- scFusion has reduced sensitivity for fusions with low expression, common to all RNA-based methods, so cross-method comparison on low-expression fusions is inconclusive.
- Detection power is limited for rare fusions in highly heterogeneous tumor samples; e.g., TRAJ20-TRAV1-2 was not identified, possibly because it was rare in MAIT cells.
- Comparison requires substantial resources: 64 GB memory and 8 CPU cores per task, with a job scheduler (e.g., Slurm) recommended.

## Evidence

- [abstract] abstract: "In a T cell dataset, scFusion detects the invariant TCR gene recombinations in mucosal-associated invariant T cells that many methods developed for bulk data fail to detect"
- [methods] methods-docker: "docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]"
- [readme] readme-config: "- 64 GB memory or more for each task

- 8 CPU cores or more for each task"
- [discussion] discussion-dl: "When deep-learning model was turned off, the precisions of scFusion reduced"
- [abstract] abstract-eval: "We evaluate the performance of scFusion using simulated and five real scRNA-seq datasets and find that scFusion can efficiently and sensitively detect fusions with a low false discovery rate."
