---
name: barcoded-rnaseq-read-simulation
description: Use when you need synthetic barcoded RNA-seq linked reads from genes
  with known isoform structure (e.g. GYPC, MAPT, BIN1) to evaluate reference-free
  isoform assembly and barcode-filtering strategies.
when_to_use_negative:
- The input is real SPIso-seq or other barcoded RNA-seq data — simulation is only
  needed to establish ground truth for benchmarking assemblers
- The target gene has a single annotated isoform and no alternative splicing, so isoform
  recall cannot be meaningfully measured
- The downstream assembler does not consume barcoded linked reads (e.g. plain short-read
  genome assemblers)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: InSilicoSeq
  role: Generates the simulated barcoded SPIso-seq-style paired-end FASTQ reads from
    the target gene loci
- name: cloudrnaSPAdes
  role: Reference-free isoform assembly of the simulated barcoded reads; feature release
    of SPAdes
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: BayesHammer
  role: SPAdes default read error correction applied to the simulated FASTQ files
    before assembly
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: IsoQuant
  role: Assigns assembled contigs to annotated GENCODE v32 isoforms to compute precision/recall
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:43e1e5f44972f861b94a1ab5bd7a59fda6ae1c2ed2d1c5ef7db920b7ca3259be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/cloudrnaSPAdes-release
  tools:
  - BayesHammer
  - InSilicoSeq
  - IsoQuant
  - SPAdes
  - cloudrnaSPAdes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# barcoded-rnaseq-read-simulation

## Summary

Simulates barcoded paired-end RNA-seq (SPIso-seq-style) reads from selected gene loci so that barcoded isoform assemblers like cloudrnaSPAdes can be benchmarked against annotated GENCODE isoforms with known ground truth.

## When to use

Use when you need synthetic barcoded RNA-seq linked reads from genes with known isoform structure (e.g. GYPC, MAPT, BIN1) to evaluate reference-free isoform assembly and barcode-filtering strategies.

## When NOT to use

- The input is real SPIso-seq or other barcoded RNA-seq data — simulation is only needed to establish ground truth for benchmarking assemblers
- The target gene has a single annotated isoform and no alternative splicing, so isoform recall cannot be meaningfully measured
- The downstream assembler does not consume barcoded linked reads (e.g. plain short-read genome assemblers)

## Inputs

- Genomic sequences of target gene loci (e.g. GYPC, MAPT, BIN1 from GRCh38)
- GENCODE v32 GTF/FASTA annotation listing annotated isoforms per gene (5, 16, and 15 isoforms respectively)
- Simulation parameters for barcoded paired-end SPIso-seq-style reads (read length, coverage, barcode structure)

## Outputs

- Simulated barcoded paired-end FASTQ files per gene
- Error-corrected reads (BayesHammer output) and per-gene contigs.fasta from cloudrnaSPAdes
- gene_isoform_assignment_metrics.tsv with per-gene precision, recall, and fixed-recall values from IsoQuant assignments

## How to apply

Extract the genomic sequences of the target gene loci from GRCh38 and their annotated isoforms from GENCODE (v32 in the benchmark: GYPC with 5, MAPT with 16, and BIN1 with 15 annotated isoforms), then generate barcoded paired-end FASTQ reads in silico (InSilicoSeq) matching the paper's simulated human data design. Feed the FASTQ files through BayesHammer read error correction (the SPAdes default), then assemble each gene separately with cloudrnaSPAdes, producing contigs.fasta per gene. Run the assembly twice — once unfiltered and once after barcode-based filtering of read clouds that yield multiple isoforms per gene — and assign contigs to GENCODE v32 isoforms with IsoQuant to compute per-gene precision, recall, and 'Fixed recall'. The known simulated isoform set is the ground truth that makes precision/recall measurable.

## Related tools

- **InSilicoSeq** (Generates the simulated barcoded SPIso-seq-style paired-end FASTQ reads from the target gene loci)
- **cloudrnaSPAdes** (Reference-free isoform assembly of the simulated barcoded reads; feature release of SPAdes) — https://github.com/1dayac/cloudrnaSPAdes-release
- **BayesHammer** (SPAdes default read error correction applied to the simulated FASTQ files before assembly) — https://github.com/1dayac/cloudrnaSPAdes-release
- **IsoQuant** (Assigns assembled contigs to annotated GENCODE v32 isoforms to compute precision/recall)

## Evaluation signals

- Per-gene isoform-assignment accuracy of assembled contigs should exceed the reported >98% when evaluated with IsoQuant against GENCODE v32
- The barcode-filtered ('Fixed recall') run should show a recall gain over the unfiltered run for the complex MAPT and BIN1 genes
- Recall after fixed-recall filtering should exceed the 'Separate strategy' competitor on MAPT and BIN1
- Contigs produced per gene should be assignable to annotated isoforms; unassignable contigs indicate simulation or assembly failure
- Corrected reads from BayesHammer should assemble without barcode-dependent fragmentation of single-isoform genes (e.g. GYPC)

## Limitations

- Simulated data may not capture real read-cloud coverage distributions, so recall gains from barcode filtering should be confirmed on real human data
- The benchmark's recall computation depends on the 'Fixed recall' filtering procedure that removes read clouds producing multiple isoforms for the same gene, which discards information
- cloudrnaSPAdes inherits SPAdes scope constraints; the manual notes SPAdes was designed for small genomes and 'is not intended for larger genomes (e.g. mammalian size genomes)'

## Evidence

- [abstract] Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity.: "Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity."
- [abstract] cloudrnaSPAdes assembles full-length isoforms from barcoded RNA-seq linked reads reference-free: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [readme] BayesHammer read error correction tool description: "BayesHammer – read error correction tool for Illumina reads, which works well on both single-cell and standard data sets."
- [readme] Recommendation to run error correction before assembly: "We recommend to run SPAdes with BayesHammer/IonHammer to obtain high-quality assemblies."
- [readme] SPAdes scope limitation for large genomes: "SPAdes was initially designed for small genomes. It was tested on bacterial (both single-cell MDA and standard isolates), fungal and other small genomes. SPAdes is not intended for larger genomes"
