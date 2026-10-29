---
name: detection-count-reproducibility-comparison
description: Use when a published COMPSRA serum small RNA-seq result (molecule counts
  per RNA class at a read-count threshold of 5) must be re-derived from raw 50 nt
  FASTQ reads and validated against reported figures.
when_to_use_negative:
- The goal is microbial profiling (archaea/bacteria/fungi/viruses) — the Microbe module
  with BLAST against the UCSC nt database is a separate, optional module not part
  of this count-reproduction task
- The input is already a count matrix or annotated BAM — jump directly to the Function
  module merge instead of rerunning QC/Alignment/Annotation
- A case-versus-control differential expression analysis is needed — use -fd -fdcase/-fdctrl
  with the Mann-Whitney rank sum test instead of the simple -fm merge
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: COMPSRA v1.0
  role: 'End-to-end platform: QC, Alignment, Annotation and Function (merge) modules
    run on the 12 serum FASTQs to produce per-class molecule counts'
  repo: https://github.com/cougarlj/COMPSRA
- name: STAR
  role: Default RNA-seq aligner used by the Alignment module to map qualified reads
    to hg38 with small-RNA default parameters
  repo: https://github.com/cougarlj/COMPSRA
- name: Java
  role: Runtime environment (JRE 8+) required to execute the COMPSRA jar and its ToolKit
    commands
  repo: https://github.com/cougarlj/COMPSRA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:3f17365adefa56272fdfbad093541f8d2cd8dd9d4308c0f2919784ff4b2e2906
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - COMPSRA v1.0
  - Java
  - STAR
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# detection-count-reproducibility-comparison

## Summary

Reproduces the COMPSRA small RNA-seq detection counts on the 12 healthy human serum samples by running the QC, Alignment, Annotation and Function (merge) modules end-to-end with the article's parameters, then comparing the number of distinct miRNAs, piRNAs, snRNAs, snoRNAs, tRNAs and circRNAs against the published values (375/280/167/88/401/7285) within a small tolerance.

## When to use

Use when a published COMPSRA serum small RNA-seq result (molecule counts per RNA class at a read-count threshold of 5) must be re-derived from raw 50 nt FASTQ reads and validated against reported figures.

## When NOT to use

- The goal is microbial profiling (archaea/bacteria/fungi/viruses) — the Microbe module with BLAST against the UCSC nt database is a separate, optional module not part of this count-reproduction task
- The input is already a count matrix or annotated BAM — jump directly to the Function module merge instead of rerunning QC/Alignment/Annotation
- A case-versus-control differential expression analysis is needed — use -fd -fdcase/-fdctrl with the Mann-Whitney rank sum test instead of the simple -fm merge

## Inputs

- 12 small RNA-seq FASTQ files (raw reads 50 nt) from healthy human serum controls
- COMPSRA jar (v1.0.x) and Java runtime (JRE 8+)
- Prebuilt hg38 annotation databases: miRBase (miRNA), piRNABank/piRBase/piRNACluster (piRNA), gtRNAdb (tRNA), GENCODE release 27 (snRNA/snoRNA), circBase (circRNA)
- STAR aligner and hg38 reference genome with STAR index

## Outputs

- compsra_detection_counts.csv — number of distinct RNA molecules detected per class (miRNA, piRNA, snRNA, snoRNA, tRNA, circRNA) at read-count threshold of 5
- Per-sample read-count text files per RNA type from the Annotation module
- Merged RNA-molecule x sample count matrices from the Function module (CpM-normalized where applicable)

## How to apply

Install COMPSRA (JRE 8+) from github.com/cougarlj/COMPSRA and fetch the prebuilt hg38 annotation databases (miRNA_hg38, piRNA_hg38, tRNA_hg38, snoRNA_hg38, snRNA_hg38, circRNA_hg38), STAR and the hg38 genome via the ToolKit (java -jar COMPSRA.jar -tk -dr -ck ...), on a machine with at least 30 GB RAM because STAR index building needs ~3 hours. Run the QC module on each of the 12 FASTQ files with adapter TGGAATTCTCGGGTGCCAAGG, 4 random bases trimmed from both ends (-rb 4), quality trimming (-rh 20 -rt 20 -rr 20) and read-length grouping (-rlh 8,17). Align qualified reads to hg38 with STAR using default small-RNA parameters (--outFilterMismatchNmax 1, --outFilterMatchNmin 16), building the index once with -mbi, then annotate the BAMs against all six RNA classes (-ac 1,2,3,4,5,6) requiring full-read overlap (-ann_overlap 1). Merge per-sample counts with the Function module (-fun -fm -fms 1-12 -fdclass 1,2,3,4,5,6), apply the default Count-per-Million (CpM) normalization, and tabulate distinct molecules detected at the read-count threshold of 5 used in the article. Compare the reproduced counts to the reported 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs within a small tolerance to judge success.

## Related tools

- **COMPSRA v1.0** (End-to-end platform: QC, Alignment, Annotation and Function (merge) modules run on the 12 serum FASTQs to produce per-class molecule counts) — https://github.com/cougarlj/COMPSRA
- **STAR** (Default RNA-seq aligner used by the Alignment module to map qualified reads to hg38 with small-RNA default parameters) — https://github.com/cougarlj/COMPSRA
- **Java** (Runtime environment (JRE 8+) required to execute the COMPSRA jar and its ToolKit commands) — https://github.com/cougarlj/COMPSRA

## Examples

```
java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -aln -mt star -ann -ac 1,2,3,4,5,6 -fun -fm -fms 1-12 -fdclass 1,2,3,4,5,6 -fdann -pro ALL_MERGE -inf ./example/sample.list -out ./example_out/
```

## Evaluation signals

- Reproduced detection counts at the threshold of 5 match the reported values within a small tolerance: 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs
- Post-QC read lengths fall in the 0–42 nt range expected after trimming the adapter and 4 random bases from 50 nt raw reads
- Merged Function-module matrices have 12 sample columns (fms 1-12) and separate per-class files for classes 1,2,3,4,5,6
- STAR index build succeeds on 30 GB RAM (about 3 hours on first run with -mbi) and each sample completes in ~10 minutes for the main pipeline
- Normalization applied is Count-per-Million (CpM), i.e. each sample library size scaled to one million reads

## Limitations

- COMPSRA software version used in the article is not explicitly stated in the results section; only v1.0.x README releases are available for reproduction
- STAR index building requires at least 30 GB RAM and takes about 3 hours; the prebuilt annotation databases must be downloaded via the ToolKit before annotation
- The default annotation requires a full-read overlap with reference entries (-ann_overlap 1), so counts are sensitive to adapter/random-base trimming choices (adapter TGGAATTCTCGGGTGCCAAGG, -rb 4)
- Counts of unmapped microbial reads (608 archaea, 103825 bacteria, 45343 fungi, 208 viruses) require the separate Microbe module and additional BLAST resources not covered by this task

## Evidence

- [results] COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs: "COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs, as well as 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses"
- [readme] Five functional modules: "COMPSRA was composed of five functional modules: **Quality Control**, **Alignment**, **Annotation**, **Microb**e and **Function**."
- [methods] CpM normalization definition: "The default normalization method is Count-per-Million (CpM), which normalizes each sample library size into one million reads"
- [readme] QC parameters in the example command: "java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17"
- [results] Raw and post-trimming read lengths: "The length of raw reads was 50 nt and after trimming adapters and 4 random bases at both 5′ and 3′ ends, the read length varied from 0 nt to 42 nt"
