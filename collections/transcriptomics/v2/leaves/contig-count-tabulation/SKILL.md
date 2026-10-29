---
name: contig-count-tabulation
description: 'Use when auditing or reproducing assembler-vs-annotation statistics
  for a barcoded RNA-seq assembly: counting >300 bp isoform contigs from contigs.fasta
  and their per-class IsoQuant assignment counts against GENCODE v32/GRCh38.'
when_to_use_negative:
- Input is plain whole-genome short-read data from a mammalian-size genome — the README
  states 'SPAdes is not intended for larger genomes (e.g. mammalian size genomes)';
  this skill targets barcoded RNA-seq linked-read isoform assembly.
- Goal is reference-based transcript quantification from reads rather than reference-free
  assembly followed by contig-to-isoform assignment counting.
- Contigs are already annotated/quantified and no per-class assignment tabulation
  against GENCODE v32/GRCh38 is needed.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: cloudrnaSPAdes
  role: Reference-free assembler producing the full-length isoform contigs from barcoded
    RNA-seq linked reads; its contigs.fasta is the object counted and filtered at
    >300 bp
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: SPAdes
  role: Host toolkit run via spades.py --rna with BayesHammer error correction and
    automatic odd k-mer selection; supports --continue/--restart-from check-pointing
    for the large run
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: IsoQuant
  role: Assigns each >300 bp contig to a GENCODE v32/GRCh38 transcript and classifies
    it as unique, minor-difference, ambiguous, inconsistent, or uninformative
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:aa2ba4eafdc02794e2eab76e38e44e7368d14e3695d92c0d8bc5841c1aced3cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/cloudrnaSPAdes-release
  tools:
  - IsoQuant
  - SPAdes
  - cloudrnaSPAdes
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# contig-count-tabulation

## Summary

Reproduces the cloudrnaSPAdes Section 3.2 statistics: total contigs longer than 300 bp assembled from the barcoded SPIso-seq human brain RNA-seq data, and the counts of those contigs across the five IsoQuant assignment classes against GENCODE v32 + GRCh38, tabulated into isoform_assignment_counts.csv.

## When to use

Use when auditing or reproducing assembler-vs-annotation statistics for a barcoded RNA-seq assembly: counting >300 bp isoform contigs from contigs.fasta and their per-class IsoQuant assignment counts against GENCODE v32/GRCh38.

## When NOT to use

- Input is plain whole-genome short-read data from a mammalian-size genome — the README states 'SPAdes is not intended for larger genomes (e.g. mammalian size genomes)'; this skill targets barcoded RNA-seq linked-read isoform assembly.
- Goal is reference-based transcript quantification from reads rather than reference-free assembly followed by contig-to-isoform assignment counting.
- Contigs are already annotated/quantified and no per-class assignment tabulation against GENCODE v32/GRCh38 is needed.

## Inputs

- SPIso-seq human brain cDNA paired-end FASTQ reads (~181.8M read pairs, 584,399 barcodes)
- GENCODE v32 GTF annotation
- GRCh38 genome reference (FASTA)
- Assembled isoform contigs (contigs.fasta from the cloudrnaSPAdes output directory)

## Outputs

- Filtered FASTA of isoform contigs >300 bp
- isoform_assignment_counts.csv with the total contig count (>300 bp) and one count for each of the five IsoQuant assignment classes (unique, minor-difference, ambiguous, inconsistent, uninformative)

## How to apply

Run cloudrnaSPAdes in RNA-Seq mode (spades.py --rna) on the ~181.8M paired-end SPIso-seq human brain FASTQ pairs (584,399 barcodes) with BayesHammer read error correction enabled and automatically selected odd k-mer sizes; if the large-scale run is interrupted, resume via SPAdes check-pointing (--continue or --restart-from last). Extract <output_dir>/contigs.fasta and retain only contigs longer than 300 bp. Run IsoQuant on the filtered contigs against the GENCODE v32 GTF on GRCh38 so each contig is assigned to a reference transcript and classified as unique, minor-difference, ambiguous, inconsistent, or uninformative. Tabulate the total >300 bp contig count plus one count per assignment class into isoform_assignment_counts.csv. A correct run should yield 1,463,514 contigs with a unique isoform assignment and 55,611 contigs with only minor differences, with class counts summing consistently with the assembler and IsoQuant run logs.

## Related tools

- **cloudrnaSPAdes** (Reference-free assembler producing the full-length isoform contigs from barcoded RNA-seq linked reads; its contigs.fasta is the object counted and filtered at >300 bp) — https://github.com/1dayac/cloudrnaSPAdes-release
- **SPAdes** (Host toolkit run via spades.py --rna with BayesHammer error correction and automatic odd k-mer selection; supports --continue/--restart-from check-pointing for the large run) — https://github.com/1dayac/cloudrnaSPAdes-release
- **IsoQuant** (Assigns each >300 bp contig to a GENCODE v32/GRCh38 transcript and classifies it as unique, minor-difference, ambiguous, inconsistent, or uninformative)

## Examples

```
spades.py --rna -1 brain_R1.fastq.gz -2 brain_R2.fastq.gz -o spades_rna_out --restart-from last
```

## Evaluation signals

- isoform_assignment_counts.csv contains exactly one total count (>300 bp contigs) plus one count for each of the five IsoQuant assignment classes
- Assignment-class counts sum consistently with the assembler and IsoQuant run logs
- Reported reproduction values are reached: 1,463,514 uniquely assigned contigs and 55,611 minor-difference contigs against GENCODE v32 + GRCh38
- Only contigs longer than 300 bp from <output_dir>/contigs.fasta are included in the totals (shorter contigs excluded)
- spades run logs show all automatically selected odd k-mer stages completed (or were resumed via --continue/--restart-from last) before counting

## Limitations

- The large-scale human assembly can be interrupted; completion relies on SPAdes check-pointing (--continue / --restart-from last) and logs must be checked for missing k-mer stages before counting.
- SPAdes/cloudrnaSPAdes is documented as not intended for mammalian-size genomes in WGS mode; here it is applied to barcoded RNA-seq reads, so misuse on genomic data is a failure mode.
- MismatchCorrector (mismatch/short-indel improvement of contigs) is turned off by default, so contig sequences used for IsoQuant assignment reflect the default pipeline settings.

## Evidence

- [abstract] we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [methods] SPAdes – iterative short-read genome assembly module; values of K are selected automatically based on the read length and data set type.: "SPAdes – iterative short-read genome assembly module; values of K are selected automatically based on the read length and data set type."
- [methods] BayesHammer – read error correction tool for Illumina reads, which works well on both single-cell and standard data sets.: "BayesHammer – read error correction tool for Illumina reads, which works well on both single-cell and standard data sets."
- [readme] We recommend to run SPAdes with BayesHammer/IonHammer to obtain high-quality assemblies.: "We recommend to run SPAdes with BayesHammer/IonHammer to obtain high-quality assemblies."
- [readme] Version 3.13.1 of SPAdes supports paired-end reads, mate-pairs and unpaired reads. SPAdes can take as input several paired-end and mate-pair libraries simultaneously.: "Version 3.13.1 of SPAdes supports paired-end reads, mate-pairs and unpaired reads. SPAdes can take as input several paired-end and mate-pair libraries simultaneously."
- [readme] SPAdes is not intended for larger genomes (e.g. mammalian size genomes). For such purposes you can use it at your own risk.: "SPAdes is not intended for larger genomes (e.g. mammalian size genomes). For such purposes you can use it at your own risk."
