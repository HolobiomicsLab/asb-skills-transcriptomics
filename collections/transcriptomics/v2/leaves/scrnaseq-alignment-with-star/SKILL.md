---
name: scrnaseq-alignment-with-star
description: Use when input is paired Smart-Seq single-cell RNA-seq data named *_1.fastq/*_2.fastq
  and a genome reference (.fa) plus GTF are available, and per-cell alignments are
  needed for fusion or splice-aware analysis.
when_to_use_negative:
- Input is 10X Genomics droplet scRNA-seq data — scFusion and this alignment step
  target Smart-Seq data rather than 10X
- FASTQ files follow the *_1.fq / *_2.fq naming pattern, which the scFusion pipeline
  explicitly disallows
- Data are bulk RNA-seq samples rather than single-cell Smart-Seq libraries, where
  scFusion's joint single-cell model does not apply
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: STAR
  role: Splice-aware aligner used to map each paired Smart-Seq FASTQ pair to the hg19
    reference genome; version >= 2.7.2d required
  repo: https://github.com/alexdobin/STAR
- name: samtools
  role: Post-alignment processing of STAR output (>= 1.0, tested on 1.10) before candidate-read
    extraction
- name: bedtools
  role: Extraction of chimeric/fusion-supporting candidate reads from the processed
    alignments (tested on 2.29.2)
- name: scFusion
  role: Downstream pipeline (run via scFusion.py) that consumes the alignments for
    gene fusion detection
  repo: https://github.com/ZijieJin/scFusion
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:20237265a1935dc4ada629cf1f9b6b9925718b78506829fb9cfaae1a3b2f69d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexdobin/STAR
  tools:
  - Keras
  - Python
  - R
  - STAR
  - TensorFlow
  - bedtools
  - numpy
  - pyensembl
  - pysam
  - samtools
  - scFusion
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# scrnaseq-alignment-with-star

## Summary

Align single-cell Smart-Seq scRNA-seq FASTQ pairs to a reference genome (e.g., hg19) using STAR (>= 2.7.2d), producing BAM alignments that feed downstream chimeric/fusion candidate extraction in the scFusion pipeline.

## When to use

Use when input is paired Smart-Seq single-cell RNA-seq data named *_1.fastq/*_2.fastq and a genome reference (.fa) plus GTF are available, and per-cell alignments are needed for fusion or splice-aware analysis.

## When NOT to use

- Input is 10X Genomics droplet scRNA-seq data — scFusion and this alignment step target Smart-Seq data rather than 10X
- FASTQ files follow the *_1.fq / *_2.fq naming pattern, which the scFusion pipeline explicitly disallows
- Data are bulk RNA-seq samples rather than single-cell Smart-Seq libraries, where scFusion's joint single-cell model does not apply

## Inputs

- Paired Smart-Seq single-cell FASTQ files named *_1.fastq and *_2.fastq (e.g., 1_1.fastq, 1_2.fastq)
- Reference genome FASTA file (*.fa, e.g., hg19.fa, ~3 GB)
- GTF annotation file (*.gtf) from Ensembl, NCBI, or UCSC

## Outputs

- Per-cell splice-aware genomic alignments (SAM/BAM) suitable for samtools processing
- Sorted/processed alignments from which chimeric/fusion-supporting candidate reads are extracted with bedtools

## How to apply

Collect the paired-end Smart-Seq FASTQ files, ensuring names follow the *_1.fastq / *_2.fastq convention ('*_1.fq is not allowed'). Use STAR version >= 2.7.2d (tested on 2.7.2d and 2.7.8a) with the hg19 reference genome file (hg19.fa, ~3G) and a GTF annotation obtained from Ensembl, NCBI, or UCSC. Process resulting SAM/BAM outputs with samtools (>= 1.0, tested on 1.10) so that chimeric/fusion-supporting candidate reads can be extracted with bedtools (tested on 2.29.2). Plan compute resources per the README: 64 GB memory and 8 CPU cores per task are recommended, and jobs may be dispatched via a scheduler such as Slurm. Downstream steps (pyensembl annotation, deep-learning scoring) depend on these alignments, so alignment correctness directly gates fusion-candidate quality.

## Related tools

- **STAR** (Splice-aware aligner used to map each paired Smart-Seq FASTQ pair to the hg19 reference genome; version >= 2.7.2d required) — https://github.com/alexdobin/STAR
- **samtools** (Post-alignment processing of STAR output (>= 1.0, tested on 1.10) before candidate-read extraction)
- **bedtools** (Extraction of chimeric/fusion-supporting candidate reads from the processed alignments (tested on 2.29.2))
- **scFusion** (Downstream pipeline (run via scFusion.py) that consumes the alignments for gene fusion detection) — https://github.com/ZijieJin/scFusion

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- Input FASTQ filenames follow the *_1.fastq / *_2.fastq convention with no *_1.fq files present
- STAR version in use is >= 2.7.2d (tested versions: 2.7.2d and 2.7.8a)
- Downstream bedtools candidate extraction yields chimeric/fusion-supporting reads from the processed alignments
- Alignment tasks ran with recommended resources (64 GB RAM, >= 8 CPU cores) without memory failures
- The full scFusion.py pipeline completes and produces fusion candidates annotatable by pyensembl

## Limitations

- Fusions with low expression show reduced sensitivity, common to all RNA-based methods, which limits what alignment quality can recover
- Detection power is limited for rare fusions in highly heterogeneous tumor samples; overcome only by sequencing more and/or deeper cells
- Runs require substantial resources — 64 GB memory and 8 CPU cores per task are recommended

## Evidence

- [methods] STAR version requirement: "STAR (https://github.com/alexdobin/STAR) >= 2.7.2d (tested on 2.7.2d and 2.7.8a)"
- [readme] Smart-Seq FASTQ naming: "Single cell RNA sequencing files from Smart-Seq protocol. File names should be *_1.fastq, *_2.fastq"
- [methods] Reference and annotation inputs: "Reference genome file (*.fa)(like hg19.fa, file size = ~3G)"
- [readme] Downstream processing tools: "samtools >= 1.0 (tested on version 1.10)"
- [readme] Smart-Seq scope restriction: [evidence span withheld — anchor status: external_doc]
