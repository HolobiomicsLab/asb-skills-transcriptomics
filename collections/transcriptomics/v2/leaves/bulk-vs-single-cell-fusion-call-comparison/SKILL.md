---
name: bulk-vs-single-cell-fusion-call-comparison
description: Use when scRNA-seq (Smart-Seq) fusion calls from scFusion must be validated
  against known spike-in fusions or against fusion calls and chimeric-read support
  from bulk methods, e.g. to reproduce the Fig 5 comparison of fusion counts and bulk-support
  fractions.
when_to_use_negative:
- Input data are 10X droplet scRNA-seq rather than Smart-Seq paired-end FASTQ — the
  README states scFusion is 'For Smart-Seq data rather than 10X'
- The goal is to characterize fusion cellular heterogeneity or transcriptional impact
  in a real tumor rather than to benchmark call precision against a ground truth or
  bulk baseline
- No spike-in truth set or matched bulk supporting data exist, so the bulk-support-fraction
  comparison cannot be computed
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3053
tools:
- name: scFusion
  role: single-cell fusion caller whose calls are benchmarked against bulk methods
  repo: https://github.com/ZijieJin/scFusion
- name: STAR
  role: aligns paired-end Smart-Seq FASTQ reads to hg19 to produce BAMs for scFusion
    input
  repo: https://github.com/alexdobin/STAR
- name: samtools
  role: processes sorted BAM alignments from STAR before fusion calling
- name: pyensembl
  role: Python interface for Ensembl GTF/FASTA reference metadata used by the pipeline
  repo: https://github.com/openvax/pyensembl
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:9994d2fefca23382ddfa440c45aa76278ea27861508f6a7be84924c4b7bcf750
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# bulk-vs-single-cell-fusion-call-comparison

## Summary

Benchmark scFusion single-cell fusion calls against bulk RNA-seq fusion detection methods on a spike-in ground-truth dataset, quantifying how many calls are supported by spike-ins or bulk chimeric reads and how many extra fusions each method reports.

## When to use

Use when scRNA-seq (Smart-Seq) fusion calls from scFusion must be validated against known spike-in fusions or against fusion calls and chimeric-read support from bulk methods, e.g. to reproduce the Fig 5 comparison of fusion counts and bulk-support fractions.

## When NOT to use

- Input data are 10X droplet scRNA-seq rather than Smart-Seq paired-end FASTQ — the README states scFusion is 'For Smart-Seq data rather than 10X'
- The goal is to characterize fusion cellular heterogeneity or transcriptional impact in a real tumor rather than to benchmark call precision against a ground truth or bulk baseline
- No spike-in truth set or matched bulk supporting data exist, so the bulk-support-fraction comparison cannot be computed

## Inputs

- 729 paired-end Smart-Seq FASTQ files (*_1.fastq, *_2.fastq; *_1.fq not allowed)
- hg19 reference genome FASTA (~3G, e.g. hg19.fa)
- GTF annotation file (*.gtf) from Ensembl, NCBI, or UCSC
- unzipped hg19mappability file (UCSC mappability) placed in the scFusion data folder
- list of 27 PCAWG spike-in fusions and the 27 single cells each was spiked into
- fusion call sets from four bulk RNA-seq fusion detection methods

## Outputs

- scFusion fusion call list per single cell (STAR/samtools-derived alignments plus scFusion calls)
- fig5_fusion_comparison.csv with spike-in recovery (27/27) and scFusion-vs-bulk fusion counts and bulk-support fractions

## How to apply

Align the 729 paired-end Smart-Seq FASTQ files (*_1.fastq, *_2.fastq) to hg19 with STAR (>= 2.7.2d) and sort/process BAMs with samtools (>= 1.0), then run scFusion.py with the hg19 genome FASTA, a GTF annotation (Ensembl/NCBI/UCSC), and the unzipped hg19mappability file placed in the data folder; the README recommends 64 GB memory and 8 CPU cores per task. Collect scFusion fusion calls and check that all 27 PCAWG spike-in fusions are recovered, each supported in the 27 single cells it was spiked into. Tabulate the bulk methods' reported fusion counts (310–9044 in the paper) and their bulk-support fractions (0.5–17.7%) alongside scFusion's ~80% bulk-support fraction (41 of scFusion's fusions were spike-ins or had bulk supporting chimeric reads), mirroring Fig 5. The rationale is that scFusion's joint analysis of multiple single cells and its statistical plus deep-learning filters keep total calls near the spike-in truth set (only 24 additional fusions), whereas bulk methods on the same data report orders of magnitude more candidates with little external support.

## Related tools

- **scFusion** (single-cell fusion caller whose calls are benchmarked against bulk methods) — https://github.com/ZijieJin/scFusion
- **STAR** (aligns paired-end Smart-Seq FASTQ reads to hg19 to produce BAMs for scFusion input) — https://github.com/alexdobin/STAR
- **samtools** (processes sorted BAM alignments from STAR before fusion calling)
- **pyensembl** (Python interface for Ensembl GTF/FASTA reference metadata used by the pipeline) — https://github.com/openvax/pyensembl

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- All 27 PCAWG spike-in fusions are recovered by scFusion, each supported in the 27 single cells it was spiked into
- scFusion reports only ~24 fusions beyond the spike-ins, versus 310–9044 for each of the four bulk methods
- ~80% (41) of scFusion's detected fusions are spike-in fusions or have bulk supporting chimeric reads, versus 0.5–17.7% for bulk methods
- The comparison table reproduces the Fig 5 structure: fusion counts and bulk-support fractions for scFusion and the four bulk methods
- Run completes under the recommended configuration (64 GB memory, 8 CPU cores) with the hg19mappability filter active

## Limitations

- Turning off the deep-learning model reduced the precision of scFusion, and many potential artifacts were also filtered by the ad hoc filters
- If no mappability file is provided, scFusion turns off the mappability filter, weakening artifact removal
- Reduced sensitivity for fusions with low expression, common to all RNA-based methods
- Detection power is limited for rare fusions in highly heterogeneous tumor samples; inherent limitations can be overcome only by sequencing more cells and/or deeper per cell

## Evidence

- [readme] scFusion is designed for Smart-Seq paired-end input: [evidence span withheld — anchor status: external_doc]
- [methods] Smart-Seq FASTQ naming requirement: "Single cell RNA sequencing files from Smart-Seq protocol. File names should be *_1.fastq, *_2.fastq"
- [methods] Pipeline setup requires mappability file and scFusion.py: "Download all scripts, unzip the hg19mappability file in the data folder, and run scFusion.py."
- [methods] STAR version requirement used for alignment: "STAR (https://github.com/alexdobin/STAR) >= 2.7.2d (tested on 2.7.2d and 2.7.8a)"
- [abstract] scFusion's low false discovery rate is the premise of the benchmark: "We evaluate the performance of scFusion using simulated and five real scRNA-seq datasets and find that scFusion can efficiently and sensitively detect fusions with a low false discovery rate."
