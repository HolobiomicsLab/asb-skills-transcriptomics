---
name: mirna-read-alignment-with-star
description: Use when quality-controlled small RNA-seq reads (e.g., serum FASTQ processed
  with adapter trimming) must be mapped to hg19/hg38 before annotating mapped reads
  to miRBase miRNAs and building per-sample miRNA count matrices.
when_to_use_negative:
- Input is raw FASTQ that has not passed COMPSRA QC (adapter/random-base trimming
  and quality filtering) — run the QC module first
- Reads are longer RNA-seq transcripts or the goal is transcript-level quantification
  rather than small RNA mapping to hg19/hg38
- The reference organism is not supported by the prebuilt COMPSRA indices (e.g., only
  hg19/hg38 and rat rno5/rno6 are prebuilt)
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: COMPSRA
  role: Pipeline wrapper invoking the Alignment module (-aln -mt star); provides toolkit
    download of STAR and genome index
  repo: https://github.com/cougarlj/COMPSRA
- name: STAR
  role: Default RNA-seq aligner performing local read-ends alignment of qualified
    small RNA reads to hg19/hg38 with one mismatch allowed
  repo: https://code.google.com/archive/p/rna-star/downloads
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:1644eafa7cc8b04b46c06016dc13d3161adca2a602b29756b5123c2f1a94959c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - STAR
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# miRNA read alignment with STAR

## Summary

Maps quality-controlled small RNA-seq reads to the human reference genome (hg19/hg38) using STAR in local read-ends alignment mode as implemented in the COMPSRA Alignment module, producing the aligned BAM that feeds miRNA annotation.

## When to use

Use when quality-controlled small RNA-seq reads (e.g., serum FASTQ processed with adapter trimming) must be mapped to hg19/hg38 before annotating mapped reads to miRBase miRNAs and building per-sample miRNA count matrices.

## When NOT to use

- Input is raw FASTQ that has not passed COMPSRA QC (adapter/random-base trimming and quality filtering) — run the QC module first
- Reads are longer RNA-seq transcripts or the goal is transcript-level quantification rather than small RNA mapping to hg19/hg38
- The reference organism is not supported by the prebuilt COMPSRA indices (e.g., only hg19/hg38 and rat rno5/rno6 are prebuilt)

## Inputs

- Length-filtered FASTQ from COMPSRA QC module (e.g., *_17to50_FitRead.fastq.gz)
- Prebuilt hg38 STAR genome index (downloaded via COMPSRA toolkit as star_hg38)
- STAR aligner executable installed in the COMPSRA plug directory

## Outputs

- Per-sample STAR alignment BAM (e.g., sample01_17to50_FitRead_STAR_Aligned.out.bam)
- Unmapped reads file for downstream Microbe module processing

## How to apply

After the COMPSRA QC module (adapter TGGAATTCTCGGGTGCCAAGG, -rb 4 random-base removal, -rh 20 -rt 20 -rr 20 quality filters, -rlh 8,17 read-length grouping), take the length-filtered FASTQ (e.g., sample01_17to50_FitRead.fastq.gz) and run the COMPSRA Alignment module with -aln -mt star. STAR performs local read-ends alignment against the prebuilt hg38 index, allowing one mismatch by default (--outFilterMismatchNmax 1); pass -mbi only on the first run so COMPSRA builds the STAR index (about 3 hours). Install STAR either manually into the COMPSRA plug directory or via the build-in toolkit (java -jar COMPSRA.jar -tk -dr -ck star; genome via -ck star_hg38). The output is a STAR_Aligned.out.bam per sample that is directly consumed by the Annotation module (-ann -ac 1) to quantify miRNAs against miRBase; reads unmapped to the human genome are saved to a FASTA/BAM for the optional Microbe module.

## Related tools

- **COMPSRA** (Pipeline wrapper invoking the Alignment module (-aln -mt star); provides toolkit download of STAR and genome index) — https://github.com/cougarlj/COMPSRA
- **STAR** (Default RNA-seq aligner performing local read-ends alignment of qualified small RNA reads to hg19/hg38 with one mismatch allowed) — https://code.google.com/archive/p/rna-star/downloads

## Examples

```
java -jar COMPSRA.jar -ref hg38 -aln -mt star -in ./example_out/sample01/sample01_17to50_FitRead.fastq.gz -out ./example_out/sample01/sample01_17to50_FitRead
```

## Evaluation signals

- STAR index build (-mbi) succeeds once and subsequent runs skip index building without errors
- Aligned output BAM named per convention (e.g., <sample>_17to50_FitRead_STAR_Aligned.out.bam) exists and is non-empty for each of the 12 input samples
- Alignment parameters match the reproducibility task: local read-ends alignment, --outFilterMismatchNmax 1, hg38 reference
- Downstream Annotation module (-ann -ac 1) consumes the BAM and yields miRNA counts whose top 30% set reproduces the SRP120169 benchmark sensitivity/specificity of 93%/98% (85%/87% overall)

## Limitations

- STAR index build via -mbi takes about 3 hours the first time; if it fails, enter the STAR installation directory and set the executable bit with chmod
- Only hg19/hg38 human genomes (plus prebuilt rat rno5/rno6) are supported as reference by the prebuilt COMPSRA toolkit resources
- Default parameters allow only one mismatch (--outFilterMismatchNmax 1), which may drop reads with more mismatches

## Evidence

- [readme] Qualified reads from the QC module output are first mapped to the human genome hg19/hg38, and then aligned reads are quantified and annotated in the Annotation Module.: "Qualified reads from the QC module output are first mapped to the human genome hg19/hg38, and then aligned reads are quantified and annotated in the Annotation Module."
- [intro] It currently uses STAR16 and BLAST17 for alignment and sequence comparison: "It currently uses STAR16 and BLAST17 for alignment and sequence comparison"
- [readme] -mbi was only needed for the first time when you run COMPSRA and built the index files for STAR. This process will cost about 3 hours.: "(Note: -mbi was only needed for the first time when you run COMPSRA and built the index files for STAR. This process will cost about 3 hours.)"
- [readme] java -jar COMPSRA.jar -ref hg38 -aln -mt star -mbi -in ./example_out/sample01/sample01_17to50_FitRead.fastq.gz: "java -jar COMPSRA.jar -ref hg38 -aln -mt star -mbi -in ./example_out/sample01/sample01_17to50_FitRead.fastq.gz -out ./example_out/sample01/sample01_17to50_FitRead"
- [results] COMPSRA achieved a sensitivity of 85% and specificity of 87% against the SRP120169 benchmark overall: [evidence span withheld — anchor status: not_found]
