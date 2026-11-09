---
name: rna-seq-de-novo-transcriptome-assembly
description: Use when you have Illumina paired-end RNA-seq reads (e.g. 2x100bp FASTQ,
  possibly with 10x/barcode structure) and need reference-free recovery of full-length
  isoforms from high-isoform-diversity genes like GYPC, MAPT, or BIN1.
when_to_use_negative:
- When a high-quality reference transcriptome exists and genome-guided alignment or
  barcode-aware reconstruction (cloudrnaSPAdes) is available — standard short-read
  assemblers left most simulated isoforms undetected
- When input is large-genome whole-genome sequencing data rather than transcriptomic
  reads — SPAdes was designed for small genomes and offers separate pipelines (metaSPAdes,
  plasmidSPAdes) for those cases
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: rnaSPAdes
  role: de novo transcriptome assembler run via rnaspades.py with automatic k-mer
    selection from read length
  repo: http://cab.spbu.ru/software/spades/
- name: cloudrnaSPAdes
  role: barcode-aware isoform assembly from bulk barcoded RNA-seq linked reads, as
    a feature release of SPAdes
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: SPAdes 3.13.1
  role: assembly toolkit whose pipeline (BayesHammer error correction, iterative assembly
    with automatic K selection) underlies the transcriptome runs
- name: Trinity
  role: standard bulk short-read RNA assembler baseline run ignoring barcode information
- name: RNA-Bloom
  role: standard de novo transcriptome assembler baseline with default settings
- name: IsoQuant
  role: assigns contigs from each assembler's contigs.fasta to genes and isoforms
    using the reference annotation
- name: Bandage
  role: GUI visualization of the SPAdes assembly graphs during inspection
  repo: https://github.com/rrwick/Bandage
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:b3214acdf8e60d039aa21d32cf82e12e95bdfc037762a2a7293eaa1e7d86bf7c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bandage
  - IsoQuant
  - RNA-Bloom
  - SPAdes
  - SPAdes 3.13.1
  - Trinity
  - cloudrnaSPAdes
  - rnaSPAdes
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# rna-seq-de-novo-transcriptome-assembly

## Summary

Assemble full-length transcript isoforms de novo from RNA-seq (including bulk barcoded linked-read) data using SPAdes-family assemblers — specifically rnaSPAdes via rnaspades.py and cloudrnaSPAdes — and benchmark the resulting contigs against barcode-aware pipelines.

## When to use

Use when you have Illumina paired-end RNA-seq reads (e.g. 2x100bp FASTQ, possibly with 10x/barcode structure) and need reference-free recovery of full-length isoforms from high-isoform-diversity genes like GYPC, MAPT, or BIN1.

## When NOT to use

- When a high-quality reference transcriptome exists and genome-guided alignment or barcode-aware reconstruction (cloudrnaSPAdes) is available — standard short-read assemblers left most simulated isoforms undetected
- When input is large-genome whole-genome sequencing data rather than transcriptomic reads — SPAdes was designed for small genomes and offers separate pipelines (metaSPAdes, plasmidSPAdes) for those cases

## Inputs

- Paired-end Illumina RNA-seq FASTQ reads per target gene (simulated or real, e.g. GYPC, MAPT, BIN1 read sets)
- Barcoded bulk RNA-seq linked-read data for barcode-aware assembly with cloudrnaSPAdes
- Reference genome + GTF annotation for IsoQuant-based isoform assignment

## Outputs

- contigs.fasta transcript contigs per assembler run
- Per-gene per-assembler uniquely-assigned full-length isoform counts (e.g. isoform_counts_per_gene.tsv)

## How to apply

Run rnaSPAdes through the rnaspades.py executable on each gene-specific read set (e.g. GYPC, MAPT, BIN1), letting k-mer sizes be selected automatically from the read length (default SPAdes behavior); note that --careful is not supported in rnaSPAdes. Optionally first run BayesHammer read error correction, which SPAdes documentation recommends for high-quality assemblies. Collect each assembler's contigs.fasta output (rival assemblers include Trinity and RNA-Bloom run in standard bulk mode ignoring barcodes). Assign contigs to genes and isoforms with IsoQuant against a reference annotation, then tally isoforms receiving a unique full-length contig assignment per gene per assembler, e.g. into a TSV such as isoform_counts_per_gene.tsv. In the published comparison, standard short-read assemblers left most simulated isoforms undetected, Trinity outperformed rnaSPAdes on these datasets, and barcode-aware pipelines (cloudrnaSPAdes) recovered far more isoforms even for high-isoform-diversity genes.

## Related tools

- **rnaSPAdes** (de novo transcriptome assembler run via rnaspades.py with automatic k-mer selection from read length) — http://cab.spbu.ru/software/spades/
- **cloudrnaSPAdes** (barcode-aware isoform assembly from bulk barcoded RNA-seq linked reads, as a feature release of SPAdes) — https://github.com/1dayac/cloudrnaSPAdes-release
- **SPAdes 3.13.1** (assembly toolkit whose pipeline (BayesHammer error correction, iterative assembly with automatic K selection) underlies the transcriptome runs)
- **Trinity** (standard bulk short-read RNA assembler baseline run ignoring barcode information)
- **RNA-Bloom** (standard de novo transcriptome assembler baseline with default settings)
- **IsoQuant** (assigns contigs from each assembler's contigs.fasta to genes and isoforms using the reference annotation)
- **Bandage** (GUI visualization of the SPAdes assembly graphs during inspection) — https://github.com/rrwick/Bandage

## Examples

```
rnaspades.py -1 GYPC_R1.fastq.gz -2 GYPC_R2.fastq.gz -o gypc_rnaspades_out  # k-mer sizes auto-selected; --careful unsupported
```

## Evaluation signals

- isoform_counts_per_gene.tsv shows Trinity recovering more uniquely-assigned full-length isoforms than rnaSPAdes on the simulated GYPC/MAPT/BIN1 datasets
- Barcode-aware pipelines detect far more full-length isoforms than any of Trinity, RNA-Bloom, or rnaSPAdes
- Each assembler produces a well-formed contigs.fasta whose contigs receive unique full-length gene/isoform assignments from IsoQuant
- rnaSPAdes runs complete with automatically selected k-mer sizes and without the unsupported --careful flag

## Limitations

- Standard short-read RNA assemblers ignore barcode information and left most simulated isoforms undetected on complex genes
- The --careful option is not supported in rnaSPAdes
- Trinity outperformed rnaSPAdes on the simulated datasets tested

## Evidence

- [abstract] we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [abstract] cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity: "Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity."
- [readme] rnaSPAdes a de novo transcriptome assembler from RNA-Seq data: [evidence span withheld — anchor status: not_found]
- [readme] SPAdes iterative assembly selects K automatically based on read length and data set type: "SPAdes &ndash; iterative short-read genome assembly module; values of K are selected automatically based on the read length and data set type."
- [readme] We recommend to run SPAdes with BayesHammer/IonHammer to obtain high-quality assemblies.: "We recommend to run SPAdes with BayesHammer/IonHammer to obtain high-quality assemblies."
