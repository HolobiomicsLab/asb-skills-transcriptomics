---
name: spike-in-benchmark-fusion-recovery-analysis
description: Use when evaluating a single-cell fusion detection pipeline against a
  spike-in ground truth, to quantify sensitivity (e.g., 27/27 spike-ins recovered)
  and false discovery via bulk chimeric-read support fractions.
when_to_use_negative:
- Data are bulk RNA-seq or 10X droplet scRNA-seq rather than Smart-Seq single-cell
  FASTQ files
- There is no ground-truth fusion set (e.g., no spike-ins or known fusions) to evaluate
  recovery against
- FASTQ names use *_1.fq extension, which the scFusion pipeline explicitly does not
  allow
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: scFusion
  role: Single-cell gene fusion detection pipeline whose calls are benchmarked against
    the spike-in truth set
  repo: https://github.com/ZijieJin/scFusion
- name: STAR
  role: Aligns the Smart-Seq paired-end FASTQ reads to the hg19 genome (>= 2.7.2d,
    tested on 2.7.2d and 2.7.8a)
  repo: https://github.com/alexdobin/STAR
- name: samtools
  role: Processes and sorts the STAR alignment BAMs (>= 1.0, tested on 1.10)
- name: bedtools
  role: Interval operations supporting the scFusion pipeline (tested on 2.29.2)
- name: pyensembl
  role: Python module used by scFusion for gene/transcript annotation handling
  repo: https://github.com/openvax/pyensembl
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:1c10b132b0cf2867dac6f3329f14b6b79ba352315e4681e69b9fd7057268af34
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# spike-in-benchmark-fusion-recovery-analysis

## Summary

Benchmark a single-cell fusion caller by verifying recovery of known PCAWG spike-in fusions across 729 Smart-Seq single cells and comparing the caller's fusion counts and bulk-support fractions against four bulk RNA-seq fusion methods (as in Fig 5 of the scFusion paper).

## When to use

Use when evaluating a single-cell fusion detection pipeline against a spike-in ground truth, to quantify sensitivity (e.g., 27/27 spike-ins recovered) and false discovery via bulk chimeric-read support fractions.

## When NOT to use

- Data are bulk RNA-seq or 10X droplet scRNA-seq rather than Smart-Seq single-cell FASTQ files
- There is no ground-truth fusion set (e.g., no spike-ins or known fusions) to evaluate recovery against
- FASTQ names use *_1.fq extension, which the scFusion pipeline explicitly does not allow

## Inputs

- 729 paired-end Smart-Seq scRNA-seq FASTQ files (*_1.fastq, *_2.fastq)
- hg19 reference genome FASTA (hg19.fa, ~3G)
- GTF annotation file (Ensembl/NCBI/UCSC)
- hg19mappability file (unzipped into the data folder; obtainable from UCSC hgTables)
- List of 27 PCAWG spike-in fusions and the 27 single cells each was spiked into
- Fusion calls from four bulk RNA-seq methods with chimeric-read support

## Outputs

- scFusion fusion call list per single cell
- Spike-in recovery summary (27/27 fusions detected, each supported in its 27 target cells)
- fig5_fusion_comparison.csv: fusion counts and bulk-support fractions for scFusion vs bulk methods

## How to apply

Align the 729 paired-end Smart-Seq FASTQ files (*_1.fastq, *_2.fastq; *_1.fq not allowed) to hg19 with STAR >= 2.7.2d and sort/process BAMs with samtools >= 1.0, then run scFusion.py with the hg19 FASTA, GTF annotation, and unzipped hg19mappability file placed in the data folder, using the recommended 64 GB memory and 8 CPU cores per task. Collect scFusion fusion calls and check that all 27 PCAWG spike-in fusions are recovered, each supported in the 27 single cells it was spiked into. Tabulate the four bulk methods' fusion counts (310–9044) and bulk-support fractions (0.5–17.7%) alongside scFusion's ~80% bulk-support fraction (41 of 50 fusions being spike-ins or bulk-supported). Write the recovery summary and caller-vs-bulk comparison to fig5_fusion_comparison.csv.

## Related tools

- **scFusion** (Single-cell gene fusion detection pipeline whose calls are benchmarked against the spike-in truth set) — https://github.com/ZijieJin/scFusion
- **STAR** (Aligns the Smart-Seq paired-end FASTQ reads to the hg19 genome (>= 2.7.2d, tested on 2.7.2d and 2.7.8a)) — https://github.com/alexdobin/STAR
- **samtools** (Processes and sorts the STAR alignment BAMs (>= 1.0, tested on 1.10))
- **bedtools** (Interval operations supporting the scFusion pipeline (tested on 2.29.2))
- **pyensembl** (Python module used by scFusion for gene/transcript annotation handling) — https://github.com/openvax/pyensembl

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- All 27 PCAWG spike-in fusions are present in the scFusion calls (27/27 recovery)
- Each detected spike-in fusion is supported in the exact 27 single cells it was spiked into
- scFusion's bulk-support fraction is ~80% (41 of 50 calls), versus 0.5–17.7% for the bulk methods
- scFusion reports far fewer extra fusions (~24) than bulk methods (310–9044 total calls)
- fig5_fusion_comparison.csv contains counts and support fractions matching the Fig 5 comparison

## Limitations

- scFusion has reduced sensitivity for fusions with low expression, as for all RNA-based methods (shown in simulation)
- Detection power is limited for rare fusions in highly heterogeneous tumor samples; overcome only by sequencing more cells and/or deeper per cell
- scFusion is designed for Smart-Seq data rather than 10X, so this benchmark protocol is not directly transferable to droplet data

## Evidence

- [methods] Spike-in design: [evidence span withheld — anchor status: not_found]
- [results] Recovery and counts: [evidence span withheld — anchor status: not_found]
- [readme] Input format: "Single cell RNA sequencing files from Smart-Seq protocol. File names should be *_1.fastq, *_2.fastq"
- [readme] Recommended configuration: "64 GB memory or more for each task - 8 CPU cores or more for each task"
- [abstract] False discovery control rationale: [evidence span withheld — anchor status: not_found]
