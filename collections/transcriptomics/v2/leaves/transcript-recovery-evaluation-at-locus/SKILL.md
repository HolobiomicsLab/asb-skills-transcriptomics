---
name: transcript-recovery-evaluation-at-locus
description: Use when assessing whether a specific annotated RefSeq transcript (e.g.,
  XR_929880.3) missed by single-aligner assemblers can be recovered by integrating
  alignments from multiple aligners such as Hisat2 and Star.
when_to_use_negative:
- Input is long-read RNA-seq data (PacBio or ONT) — the current Tiglon version is
  not compatible with long-read datasets
- Goal is annotation-guided assembly — the current Tiglon version has no option for
  using known transcript annotations to guide assembly
- The transcript's splice junctions are already fully covered by a single aligner's
  mapping, making multi-aligner integration unnecessary
edam_operation: http://edamontology.org/operation_3258
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: Hisat2
  role: Aligns SRR307911 reads to GRCh38 with --dta to produce Hisat.bam, one of the
    two aligner mappings integrated
  repo: https://github.com/DaehwanKimLab/hisat2
- name: Star
  role: Produces the second alignment (Star.bam) of the same reads; its junction set
    differs from Hisat2's, which is what enables recovery
  repo: https://github.com/alexdobin/STAR
- name: StringTie2
  role: Single-aligner assembler run on each BAM as the negative control expected
    to miss XR_929880.3
  repo: https://github.com/gpertea/stringtie
- name: Scallop
  role: Second single-aligner assembler run on each BAM as negative control expected
    to miss XR_929880.3
  repo: https://github.com/Kingsford-Group/scallop
- name: Tiglon
  role: Integrates Hisat.bam and Star.bam via labeled splice graph and label-based
    dynamic path extension to recover XR_929880.3
  repo: https://github.com/yutingsdu/Tiglon
- name: bamtools
  role: Prerequisite library for building Tiglon (BAM handling toolkit)
  repo: https://github.com/pezmaster31/bamtools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2022.104067
    title: Tiglon enables accurate transcriptome assembly via integrating mappings
      of different aligners
  source_content_hash: sha256:adcd41712fc7e16a9a389106925991bb5f9a54d478665a55edaf0cd77b1b5e22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaehwanKimLab/hisat2
  tools:
  - Hisat2
  - Scallop
  - Star
  - StringTie2
  - Tiglon
  - bamtools
  edam_operation: http://edamontology.org/operation_3258
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# transcript-recovery-evaluation-at-locus

## Summary

Reproduce the Tiglon paper's motivating case study: assemble RNA-seq sample SRR307911 (human GRCh38) with single-aligner assemblers (StringTie2, Scallop on Hisat2 and Star BAMs) and with Tiglon on the merged alignments, then check via Cuffcompare whether the RefSeq transcript XR_929880.3 is recovered at its locus.

## When to use

Use when assessing whether a specific annotated RefSeq transcript (e.g., XR_929880.3) missed by single-aligner assemblers can be recovered by integrating alignments from multiple aligners such as Hisat2 and Star.

## When NOT to use

- Input is long-read RNA-seq data (PacBio or ONT) — the current Tiglon version is not compatible with long-read datasets
- Goal is annotation-guided assembly — the current Tiglon version has no option for using known transcript annotations to guide assembly
- The transcript's splice junctions are already fully covered by a single aligner's mapping, making multi-aligner integration unnecessary

## Inputs

- SRR307911 RNA-seq reads from SRA (FASTQ)
- Human GRCh38 reference genome and NCBI RefSeq annotation (GTF/GFF)
- Hisat.bam (Hisat2 --dta alignments) and Star.bam (STAR alignments), coordinate-sorted BAM
- Single-aligner assembly GTFs from StringTie2 and Scallop on each BAM

## Outputs

- Tiglon_outdir/Tiglon.gtf (integrated assembly from labeled splice graph)
- Cuffcompare comparison outputs per assembler restricted to the XR_929880.3 locus
- xr_case_report.tsv recording whether XR_929880.3 is recovered by StringTie2, Scallop, and Tiglon

## How to apply

Download SRR307911 reads from SRA and align to the human GRCh38 reference with Hisat2 (--dta) to get Hisat.bam, and with STAR to get Star.bam. Run StringTie2 and Scallop separately on each single-aligner BAM to obtain four single-aligner assembly GTFs; this is the negative control since neither alignment covers all splice junctions of XR_929880.3 even though both cover all its exons. Merge Hisat.bam and Star.bam with samtools merge, list each BAM path one per line in a bamFile_list file, and run Tiglon -B bamFile_list -o Tiglon_outdir, which builds a labeled splice graph (edges labeled 2 when captured by all aligners, 1 otherwise, plus 2+/2- identical-read labels) and applies label-based dynamic path extension to emit Tiglon_outdir/Tiglon.gtf. Compare every assembled GTF against the NCBI RefSeq annotation with Cuffcompare, restricting inspection to the XR_929880.3 locus, and compile recovery per assembler into xr_case_report.tsv.

## Related tools

- **Hisat2** (Aligns SRR307911 reads to GRCh38 with --dta to produce Hisat.bam, one of the two aligner mappings integrated) — https://github.com/DaehwanKimLab/hisat2
- **Star** (Produces the second alignment (Star.bam) of the same reads; its junction set differs from Hisat2's, which is what enables recovery) — https://github.com/alexdobin/STAR
- **StringTie2** (Single-aligner assembler run on each BAM as the negative control expected to miss XR_929880.3) — https://github.com/gpertea/stringtie
- **Scallop** (Second single-aligner assembler run on each BAM as negative control expected to miss XR_929880.3) — https://github.com/Kingsford-Group/scallop
- **Tiglon** (Integrates Hisat.bam and Star.bam via labeled splice graph and label-based dynamic path extension to recover XR_929880.3) — https://github.com/yutingsdu/Tiglon
- **bamtools** (Prerequisite library for building Tiglon (BAM handling toolkit)) — https://github.com/pezmaster31/bamtools

## Examples

```
samtools merge merged.bam Hisat.bam Star.bam; printf 'merged.bam\n' > bamFile_list; Tiglon -B bamFile_list -o Tiglon_outdir
```

## Evaluation signals

- xr_case_report.tsv shows substring 'XR_929880' present in Tiglon's Cuffcompare output but absent from all four StringTie2 and Scallop single-aligner comparison outputs
- BAM coverage check confirms both Hisat2 and Star alignments cover all exons of XR_929880.3 while neither covers all of its splice junctions
- samtools merge produced a single valid BAM containing both aligners' reads, and bamFile_list lists exactly one BAM path per line
- Tiglon_outdir/Tiglon.gtf contains a transcript matching the XR_929880.3 exon chain at its GRCh38 locus
- All four single-aligner GTFs and Tiglon.gtf are well-formed GTFs accepted by Cuffcompare against the RefSeq annotation

## Limitations

- The current Tiglon version accepts only BAM files generated by Hisat(2) and Star
- Tiglon has no annotation-guided assembly option, so known annotations cannot steer the assembly
- Tiglon is not compatible with long-read RNA-seq datasets (PacBio, Oxford Nanopore)
- Recovery depends on junction coverage: if even the integrated alignments fail to capture a junction of the transcript, it cannot be reconstructed

## Evidence

- [intro] both the Hisat2 and Star (two of the best current aligners) alignments cover the whole exons of this transcript, but neither of them covers the entire junctions: "both the Hisat2 and Star (two of the best current aligners) alignments cover the whole exons of this transcript, but neither of them covers the entire junctions"
- [intro] Tiglon further labels the edges (junctions) with 1 and 2, where label 2 indicates that they are captured by all the employed aligners and label 1 otherwise.: "Tiglon further labels the edges (junctions) with 1 and 2, where label 2 indicates that they are captured by all the employed aligners and label 1 otherwise."
- [intro] in NCBI RefSeq from human reference genome GRCh38 is captured by the RNA-seq sample SRA: SRR307911: "in NCBI RefSeq from human reference genome GRCh38 is captured by the RNA-seq sample SRA: SRR307911"
- [intro] Tiglon employs a newly developed labeled-based dynamic path extension program to recover the expressed transcripts accurately.: "Tiglon employs a newly developed labeled-based dynamic path extension program to recover the expressed transcripts accurately."
- [readme] The current version of Tiglon can accepts BAM files generated by Hisat(2) and Star.: "The current version of Tiglon can accepts BAM files generated by Hisat(2) and Star."
