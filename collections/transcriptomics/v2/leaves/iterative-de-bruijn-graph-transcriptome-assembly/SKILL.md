---
name: iterative-de-bruijn-graph-transcriptome-assembly
description: Use when bulk barcoded RNA-seq (e.g. SPIso-seq) paired-end FASTQ reads
  must be assembled into isoform contigs without a reference genome, and contigs are
  needed for downstream transcript-to-reference assignment.
when_to_use_negative:
- Input is whole-genome sequencing data from a large (mammalian-size) genome — the
  README states SPAdes 'is not intended for larger genomes' and should be used for
  such purposes 'at your own risk'
- A reference-based transcriptome reconstruction or quantification is required instead
  of a de novo isoform assembly
- Reads are already error-corrected by another tool — the error-correction module
  can be turned off, and rerunning BayesHammer is unnecessary
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: cloudrnaSPAdes
  role: 'Main assembler: RNA-Seq (spades.py --rna) mode assembling full-length isoforms
    reference-free from barcoded RNA-seq linked reads'
  repo: github.com/1dayac/cloudrnaSPAdes-release
- name: SPAdes
  role: Underlying iterative short-read assembly engine; K values selected automatically,
    supports --continue/--restart-from check-pointing
  repo: http://cab.spbu.ru/software/spades/
- name: BayesHammer
  role: Illumina read error-correction stage run before assembly to obtain high-quality
    assemblies
- name: MismatchCorrector
  role: Optional post-assembly module improving mismatch and short indel rates in
    contigs and scaffolds (off by default)
- name: IsoQuant
  role: Downstream assignment of assembled contigs >300 bp to reference transcripts
    (GENCODE v32/GRCh38) with class labels (unique, minor-difference, ambiguous, inconsistent,
    uninformative)
- name: Bandage
  role: GUI visualisation of the SPAdes assembly graph for inspecting contig connections
  repo: https://github.com/rrwick/Bandage
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:c06c28eb30365c089db8270ed0c9f00b58f6a9896b89c4090cbc5655bf80158b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bandage
  - BayesHammer
  - IsoQuant
  - MismatchCorrector
  - SPAdes
  - cloudrnaSPAdes
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 18
---

# Iterative de Bruijn graph transcriptome assembly

## Summary

Assemble full-length isoforms reference-free from barcoded RNA-seq linked reads using cloudrnaSPAdes, a feature release of the SPAdes assembler, which builds an iterative de Bruijn graph over automatically selected ascending k-mer sizes after BayesHammer read error correction.

## When to use

Use when bulk barcoded RNA-seq (e.g. SPIso-seq) paired-end FASTQ reads must be assembled into isoform contigs without a reference genome, and contigs are needed for downstream transcript-to-reference assignment.

## When NOT to use

- Input is whole-genome sequencing data from a large (mammalian-size) genome — the README states SPAdes 'is not intended for larger genomes' and should be used for such purposes 'at your own risk'
- A reference-based transcriptome reconstruction or quantification is required instead of a de novo isoform assembly
- Reads are already error-corrected by another tool — the error-correction module can be turned off, and rerunning BayesHammer is unnecessary

## Inputs

- Paired-end Illumina FASTQ reads from barcoded bulk RNA-seq (linked-read cDNA, e.g. SPIso-seq human brain: ~181.8M read pairs, 584,399 barcodes)
- Optional reference annotation + genome (e.g. GENCODE v32 GTF + GRCh38) only for the downstream IsoQuant assignment step, not for the assembly itself

## Outputs

- Assembled isoform contigs in <output_dir>/contigs.fasta
- Scaffolds and assembly graph files in the SPAdes output directory (e.g. FASTG/GFA graph for inspection in Bandage)
- Corrected reads from the BayesHammer error-correction stage
- Run logs showing the iterative K stages for validation

## How to apply

Run cloudrnaSPAdes in RNA-Seq mode (spades.py --rna) on the paired-end FASTQ input, keeping the default BayesHammer read error correction stage, since SPAdes recommends BayesHammer/IonHammer to obtain high-quality assemblies. Let the assembler pick the iterative K values automatically based on read length and data set type (odd k-mers in ascending order); do not fix them manually unless you have a specific reason. For long runs on large datasets (e.g. ~181.8M SPIso-seq human brain read pairs, 584,399 barcodes), use SPAdes check-pointing with --continue or --restart-from last if the run is interrupted. Extract the assembled isoform contigs from <output_dir>/contigs.fasta and apply the desired length threshold (e.g. keep contigs > 300 bp) before downstream annotation. Optionally enable MismatchCorrector to improve mismatch and short indel rates in the resulting contigs, noting it is turned off by default.

## Related tools

- **cloudrnaSPAdes** (Main assembler: RNA-Seq (spades.py --rna) mode assembling full-length isoforms reference-free from barcoded RNA-seq linked reads) — github.com/1dayac/cloudrnaSPAdes-release
- **SPAdes** (Underlying iterative short-read assembly engine; K values selected automatically, supports --continue/--restart-from check-pointing) — http://cab.spbu.ru/software/spades/
- **BayesHammer** (Illumina read error-correction stage run before assembly to obtain high-quality assemblies)
- **MismatchCorrector** (Optional post-assembly module improving mismatch and short indel rates in contigs and scaffolds (off by default))
- **IsoQuant** (Downstream assignment of assembled contigs >300 bp to reference transcripts (GENCODE v32/GRCh38) with class labels (unique, minor-difference, ambiguous, inconsistent, uninformative))
- **Bandage** (GUI visualisation of the SPAdes assembly graph for inspecting contig connections) — https://github.com/rrwick/Bandage

## Examples

```
spades.py --rna -1 brain_R1.fastq.gz -2 brain_R2.fastq.gz -o spades_iso_out
```

## Evaluation signals

- spades.log shows the automatic K values were used in ascending order and all iterative stages completed; interrupted runs resume correctly via --continue/--restart-from last
- Corrected reads from BayesHammer exist in the output directory, confirming the error-correction stage ran before assembly
- contigs.fasta exists and the count of contigs >300 bp is tabulated (e.g. for the SPIso-seq human brain run, 1,463,514 contigs got unique IsoQuant assignments and 55,611 had only minor differences against GENCODE v32 + GRCh38)
- IsoQuant class counts (unique, minor-difference, ambiguous, inconsistent, uninformative) sum consistently with the assembler/IsoQuant run logs in isoform_assignment_counts.csv
- Assembly graph opens/validates in a graph viewer such as Bandage without disconnected or malformed segments

## Limitations

- SPAdes was initially designed for small genomes and 'is not intended for larger genomes (e.g. mammalian size genomes)' — running it on human barcoded RNA-seq is done at one's own risk and needs check-pointing for long runs
- MismatchCorrector is turned off by default, so mismatch/short-indel rates in contigs may be higher unless explicitly enabled
- k-mer coverage read filtering (spades-read-filter, median k-mer coverage below a threshold) and --cov-cutoff are available but off by default; low-coverage spurious contigs may appear without them

## Evidence

- [abstract] we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [abstract] Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity.: "Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity."
- [readme] SPAdes – iterative short-read genome assembly module; values of K are selected automatically based on the read length and data set type.: "SPAdes – iterative short-read genome assembly module; values of K are selected automatically based on the read length and data set type."
- [readme] We recommend to run SPAdes with BayesHammer/IonHammer to obtain high-quality assemblies.: "We recommend to run SPAdes with BayesHammer/IonHammer to obtain high-quality assemblies."
- [readme] SPAdes is not intended for larger genomes (e.g. mammalian size genomes). For such purposes you can use it at your own risk.: "SPAdes is not intended for larger genomes (e.g. mammalian size genomes). For such purposes you can use it at your own risk."
- [readme] rnaSPAdes – a *de novo* transcriptome assembler from RNA-Seq data (see [rnaSPAdes manual](rnaspades_manual.html)).: "rnaSPAdes – a *de novo* transcriptome assembler from RNA-Seq data (see [rnaSPAdes manual](rnaspades_manual.html))."
