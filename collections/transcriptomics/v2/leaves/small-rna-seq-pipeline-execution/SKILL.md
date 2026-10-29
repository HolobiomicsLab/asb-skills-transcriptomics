---
name: small-rna-seq-pipeline-execution
description: Use when raw small RNA-seq FASTQ files (e.g., 50 nt serum reads) must
  be processed end-to-end into per-sample RNA-type count profiles, including case-vs-control
  differential expression on RNAs <200 nt.
when_to_use_negative:
- Input is already an annotated count matrix or feature table — use the Function module
  alone (-fun -fm) rather than re-running QC/Alignment/Annotation
- Reads are long (>200 nt) RNA-seq data; COMPSRA targets small RNAs (<200 nt) and
  its QC trimming and small RNA reference databases assume short reads
- No local Java/STAR environment and only a web workflow is acceptable — the authors
  note web-based tools like exceRpt require uploading original sequencing files, which
  is un-workable for big data; COMPSRA is a stand-alone alternative requiring local
  installation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: COMPSRA
  role: Stand-alone Java platform orchestrating the five-module pipeline (QC, Alignment,
    Annotation, Microbe, Function)
  repo: https://github.com/cougarlj/COMPSRA
- name: STAR
  role: Default RNA sequence aligner mapping qualified reads to hg19/hg38 in the Alignment
    module
- name: BLAST
  role: Sequence comparison tool aligning unmapped reads to the UCSC nt database in
    the Microbe module
- name: Java
  role: Runtime environment (JRE 8 or up) required to execute COMPSRA.jar
- name: exceRpt
  role: Comparable web-based pipeline used as a benchmark; COMPSRA was run with the
    same parameters as exceRpt for comparison
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:9695672e5fbda3bbea1d1b8f6a9dc8fde31305967a7fcc8c4968e3f313fa0ea4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - BLAST
  - COMPSRA
  - Java
  - STAR
  - exceRpt
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# small-rna-seq-pipeline-execution

## Summary

Execute the COMPSRA v1.0 stand-alone Java pipeline for small RNA-seq data, running its five modules (Quality Control, Alignment, Annotation, Microbe, Function) to identify and quantify miRNA, piRNA, snRNA, snoRNA, tRNA, circRNA and fragmental microbial RNA from FASTQ inputs.

## When to use

Use when raw small RNA-seq FASTQ files (e.g., 50 nt serum reads) must be processed end-to-end into per-sample RNA-type count profiles, including case-vs-control differential expression on RNAs <200 nt.

## When NOT to use

- Input is already an annotated count matrix or feature table — use the Function module alone (-fun -fm) rather than re-running QC/Alignment/Annotation
- Reads are long (>200 nt) RNA-seq data; COMPSRA targets small RNAs (<200 nt) and its QC trimming and small RNA reference databases assume short reads
- No local Java/STAR environment and only a web workflow is acceptable — the authors note web-based tools like exceRpt require uploading original sequencing files, which is un-workable for big data; COMPSRA is a stand-alone alternative requiring local installation

## Inputs

- FASTQ files from small RNA sequencing of biological samples (e.g., 50 nt serum reads)
- Sample list file (.list, one FASTQ path per line) for multi-sample runs
- Prebuilt reference resources: star_hg38 genome index, miRNA_hg38/piRNA_hg38/tRNA_hg38/snoRNA_hg38/snRNA_hg38/circRNA_hg38 databases, blast taxonomy and blast_archaea/bacteria/fungi/viruses databases
- SAM/BAM alignment files (alternative -in input format)

## Outputs

- Tab-delimited read-count files per RNA type per sample from the Annotation module
- Count matrix (RNA molecules as rows, samples as columns) from the Function module merge (-fm)
- CpM-normalized count matrix and Mann-Whitney U-test differential expression results per RNA molecule
- Unmapped reads (FASTA/BAM) from the Alignment module feeding the Microbe module
- Microbial read counts per taxon (archaea, bacteria, fungi, viruses) from the Microbe module

## How to apply

Run COMPSRA as a single command combining modules: Quality Control trims the adapter (e.g., TGGAATTCTCGGGTGCCAAGG) and 4 random bases at both 5' and 3' ends with quality thresholds -rh 20 -rt 20 -rr 20 and length grouping such as -rlh 8,17; the Alignment module maps qualified reads to hg19/hg38 with STAR (build the index once with -mbi, ~3 hours, using star_hg38 downloaded via the ToolKit); the Annotation module annotates mapped reads against prebuilt databases (miRBase miRNA, piRNABank/piRBase/piRNACluster piRNA, gtRNAdb tRNA, GENCODE release 27 snRNA/snoRNA, circBase circRNA, selected with -ac 1,2,3,4,5,6) with automatic LiftOver for genome-version consistency; the optional Microbe module BLASTs unmapped reads against the UCSC nt database for archaea, bacteria, fungi and viruses; the Function module merges counts and performs CpM normalization plus Mann-Whitney rank sum differential expression with -fdcase/-fdctrl sample ranges. Expect roughly 10 minutes per sample for the main pipeline (30 GB RAM server) and ~3.5 hours per sample for the microbiome module. Use -inf with a sample list file when processing multiple FASTQs in one run.

## Related tools

- **COMPSRA** (Stand-alone Java platform orchestrating the five-module pipeline (QC, Alignment, Annotation, Microbe, Function)) — https://github.com/cougarlj/COMPSRA
- **STAR** (Default RNA sequence aligner mapping qualified reads to hg19/hg38 in the Alignment module)
- **BLAST** (Sequence comparison tool aligning unmapped reads to the UCSC nt database in the Microbe module)
- **Java** (Runtime environment (JRE 8 or up) required to execute COMPSRA.jar)
- **exceRpt** (Comparable web-based pipeline used as a benchmark; COMPSRA was run with the same parameters as exceRpt for comparison)

## Examples

```
java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -aln -mt star -ann -ac 1,2,3,4,5,6 -inf ./example/sample.list -out ./example_out/
```

## Evaluation signals

- On the 12 healthy serum samples, COMPSRA should detect 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs at a read count threshold of 5
- Microbe module should report 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses across the 12 serum samples
- After QC, read length should vary from 0 nt to 42 nt for raw 50 nt reads trimmed of adapters and 4 random bases at both ends
- Per-sample runtime of the main pipeline should be approximately 10 minutes (vs ~20 minutes for exceRpt) on a 30 GB RAM server; microbiome module averages ~3.5 hours per sample
- First STAR run must build the index (-mbi) successfully — COMPSRA builds it on first execution and the process costs about 3 hours; chmod the STAR binary if the build fails

## Limitations

- Read quality thresholds, adapter sequence, and random-base trimming must be supplied on the command line (-ra, -rb, -rh 20, -rt 20, -rr 20) and are kit-dependent (e.g., NEXTflexTM Small RNA-Seq kit randomized bases)
- Prebuilt annotation databases are tied to hg19/hg38 genome versions and rely on automatic UCSC LiftOver for consistency; rat (rno5/rno6) support was only added in V1.0.1/V1.0.2 and mouse genome download was only fixed in V1.0.3
- The Microbe module is optional, requires separate BLAST and taxonomy resource downloads, and averages ~3.5 hours per sample across the four taxons
- STAR index building on first run takes about 3 hours and may fail with a non-executable STAR binary requiring chmod

## Evidence

- [methods] composed of five functionally independent and customizable modules: "composed of five functionally independent and customizable modules: Quality Control (QC), Alignment, Annotation, Microbe and Function"
- [results] QC trimming behavior: "The length of raw reads was 50 nt and after trimming adapters and 4 random bases at both 5′ and 3′ ends, the read length varied from 0 nt to 42 nt"
- [results] runtime comparison: "COMPSRA takes ~10 minutes per sample, but exceRpt will cost ~20 minutes for each sample (see Supplementary Fig. S1)"
- [results] detected molecule counts: "COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs, as well as 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses"
- [readme] pipeline invocation example: "java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -aln -mt star -ann -ac 1,2,3,4,5,6 -in ./example/sample01.fastq -out ./example_out/"
- [readme] STAR index build note: "Note: -mbi was only needed for the first time when you run COMPSRA and built the index files for STAR. This process will cost about 3 hours."
