---
name: barcoded-rnaseq-isoform-assembly
description: Use when input is paired-end barcoded RNA-seq FASTQ (e.g. SPIso-seq human
  brain, ~181.8M read pairs / 584,399 barcodes) and full-length isoform contigs must
  be assembled without a transcript reference.
when_to_use_negative:
- Input is long-read (PacBio Iso-Seq / Oxford Nanopore) transcript data — cloudrnaSPAdes
  targets barcoded short-read linked RNA-seq, though SPAdes itself can use PacBio
  CLR and Oxford Nanopore reads only for gap closure and repeat resolution
- The goal is reference-based transcript quantification or alignment of annotated
  transcripts rather than reference-free isoform assembly
- The data are standard short-read RNA-seq without barcodes/linked-read structure,
  where plain rnaSPAdes rather than the cloudrnaSPAdes barcoded pipeline is appropriate
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: cloudrnaSPAdes
  role: 'Primary assembler: assembles full-length isoforms from barcoded RNA-seq linked-read
    data in a reference-free fashion; invoked in RNA-Seq mode (spades.py --rna)'
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: SPAdes 3.13.1
  role: Underlying assembly toolkit providing the iterative short-read assembly module
    with automatically selected K values and check-pointing (--continue / --restart-from
    last)
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: BayesHammer
  role: Read error correction for Illumina reads, run before assembly to obtain high-quality
    assemblies
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: IsoQuant
  role: 'Post-assembly transcript assignment: classifies each >300 bp contig as unique,
    minor-difference, ambiguous, inconsistent, or uninformative against GENCODE v32/GRCh38'
- name: Bandage
  role: Optional GUI visualisation of the SPAdes assembly graph (contigs and connections)
    during QC of isoform assembly
  repo: https://rrwick.github.io/Bandage/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:ea1c80eb3b9f1651bbe3375632080232480e68be6c960ecf5819674f459f8caf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/cloudrnaSPAdes-release
  tools:
  - Bandage
  - BayesHammer
  - IsoQuant
  - SPAdes
  - SPAdes 3.13.1
  - cloudrnaSPAdes
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# barcoded-rnaseq-isoform-assembly

## Summary

Reference-free assembly of full-length isoforms from bulk barcoded (linked-read) Illumina RNA-seq data using cloudrnaSPAdes, a feature release of the SPAdes assembler, followed by contig filtering and IsoQuant-based assignment against a reference annotation (e.g. GENCODE v32 on GRCh38).

## When to use

Use when input is paired-end barcoded RNA-seq FASTQ (e.g. SPIso-seq human brain, ~181.8M read pairs / 584,399 barcodes) and full-length isoform contigs must be assembled without a transcript reference.

## When NOT to use

- Input is long-read (PacBio Iso-Seq / Oxford Nanopore) transcript data — cloudrnaSPAdes targets barcoded short-read linked RNA-seq, though SPAdes itself can use PacBio CLR and Oxford Nanopore reads only for gap closure and repeat resolution
- The goal is reference-based transcript quantification or alignment of annotated transcripts rather than reference-free isoform assembly
- The data are standard short-read RNA-seq without barcodes/linked-read structure, where plain rnaSPAdes rather than the cloudrnaSPAdes barcoded pipeline is appropriate

## Inputs

- Paired-end barcoded RNA-seq FASTQ read pairs (e.g. SPIso-seq human brain cDNA, ~181.8M pairs, 584,399 barcodes)
- Reference transcript annotation GTF (GENCODE v32)
- Reference genome FASTA (GRCh38)

## Outputs

- Assembled isoform contigs in <output_dir>/contigs.fasta
- BayesHammer-corrected reads (intermediate)
- Filtered set of contigs longer than 300 bp
- IsoQuant assignment classes per contig (unique, minor-difference, ambiguous, inconsistent, uninformative)
- isoform_assignment_counts.csv with total contig count and per-class counts

## How to apply

Run cloudrnaSPAdes in RNA-Seq mode via spades.py --rna on the paired-end FASTQ reads, keeping BayesHammer read error correction enabled since SPAdes recommends BayesHammer/IonHammer to obtain high-quality assemblies, and let the assembler pick odd k-mer sizes automatically (values of K are selected automatically based on read length and data set type). If the large-scale human-brain-scale run is interrupted, resume using SPAdes check-pointing (--continue or --restart-from last). Extract isoform contigs from <output_dir>/contigs.fasta and filter to contigs longer than 300 bp. Assign the filtered contigs to reference transcripts with IsoQuant against e.g. GENCODE v32 on GRCh38, which classifies each contig as unique, minor-difference, ambiguous, inconsistent, or uninformative. Tabulate total contigs >300 bp plus per-class counts (e.g. 1,463,514 unique and 55,611 minor-difference assignments in the SPIso-seq human brain run) into isoform_assignment_counts.csv, checking that class counts sum consistently with the assembler/IsoQuant run logs.

## Related tools

- **cloudrnaSPAdes** (Primary assembler: assembles full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion; invoked in RNA-Seq mode (spades.py --rna)) — https://github.com/1dayac/cloudrnaSPAdes-release
- **SPAdes 3.13.1** (Underlying assembly toolkit providing the iterative short-read assembly module with automatically selected K values and check-pointing (--continue / --restart-from last)) — https://github.com/1dayac/cloudrnaSPAdes-release
- **BayesHammer** (Read error correction for Illumina reads, run before assembly to obtain high-quality assemblies) — https://github.com/1dayac/cloudrnaSPAdes-release
- **IsoQuant** (Post-assembly transcript assignment: classifies each >300 bp contig as unique, minor-difference, ambiguous, inconsistent, or uninformative against GENCODE v32/GRCh38)
- **Bandage** (Optional GUI visualisation of the SPAdes assembly graph (contigs and connections) during QC of isoform assembly) — https://rrwick.github.io/Bandage/

## Examples

```
spades.py --rna -1 brain_R1.fastq.gz -2 brain_R2.fastq.gz -o spades_out && seqkit seq -m 300 spades_out/contigs.fasta > contigs_gt300bp.fasta
```

## Evaluation signals

- isoform_assignment_counts.csv contains the total contig count (>300 bp) plus one count for each of the five IsoQuant assignment classes
- The five IsoQuant class counts sum consistently with the assembler and IsoQuant run logs
- The run completed all automatically selected odd k-mer stages in ascending order; on interruption, --continue / --restart-from last resumes from the last SPAdes check-point
- BayesHammer error correction completed before assembly (high-quality assemblies are expected only when the correction stage ran)
- For the real SPIso-seq human brain dataset, the unique-assignment count (1,463,514) and minor-difference count (55,611) against GENCODE v32/GRCh38 should be reproduced

## Limitations

- SPAdes was initially designed for small genomes and 'is not intended for larger genomes (e.g. mammalian size genomes)', so mammalian-scale runs such as human brain carry runtime/memory risk and may need check-pointing
- MismatchCorrector (which improves mismatch and short indel rates in contigs) uses BWA and is turned off by default
- The source documentation lacks a Discussion-section discussion and no changelog was found for the release
- Assignment results depend on the reference annotation version (GENCODE v32 with GRCh38 in the benchmark); other annotations may shift class counts

## Evidence

- [abstract] cloudrnaSPAdes assembles full-length isoforms from barcoded RNA-seq linked-read data: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [methods] Read error correction is recommended before assembly: "We recommend to run SPAdes with BayesHammer/IonHammer to obtain high-quality assemblies."
- [methods] K-mer sizes are chosen automatically by the iterative assembly module: "SPAdes – iterative short-read genome assembly module; values of K are selected automatically based on the read length and data set type."
- [methods] BayesHammer is the Illumina read error correction tool: "BayesHammer – read error correction tool for Illumina reads, which works well on both single-cell and standard data sets."
- [readme] rnaSPAdes is the de novo transcriptome assembler pipeline within SPAdes: "rnaSPAdes &ndash; a *de novo* transcriptome assembler from RNA-Seq data (see [rnaSPAdes manual](rnaspades_manual.html))."
- [readme] SPAdes is not intended for mammalian-size genomes: "SPAdes is not intended for larger genomes (e.g. mammalian size genomes). For such purposes you can use it at your own risk."
