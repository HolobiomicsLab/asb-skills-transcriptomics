---
name: linked-read-isoform-assembly
description: Use when input is barcoded bulk RNA-seq (e.g. SPIso-seq-style linked
  reads in FASTQ) and the goal is de novo full-length isoform reconstruction, especially
  for highly alternatively spliced genes such as GYPC, MAPT, or BIN1.
when_to_use_negative:
- Input is standard (non-barcoded) RNA-seq — rnaSPAdes, not cloudrnaSPAdes, is the
  de novo transcriptome assembler for that case
- The goal is whole-genome assembly of a large (mammalian-size) genome — SPAdes is
  not intended for larger genomes
- Reads are already long full-length transcripts (PacBio CLR/ONT) — SPAdes uses those
  only for gap closure and repeat resolution, not as the primary barcoded input
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: cloudrnaSPAdes
  role: Feature release of SPAdes that assembles full-length isoforms reference-free
    from barcoded RNA-seq linked reads; performs per-read-cloud subgraph simplification
    (tip clipping, bulge removal, gap closing)
  repo: github.com/1dayac/cloudrnaSPAdes-release
- name: SPAdes
  role: Assembly toolkit providing graph construction (spades-gbuilder) and the iterative
    short-read assembly engine underlying cloudrnaSPAdes
  repo: github.com/1dayac/cloudrnaSPAdes-release
- name: BayesHammer
  role: Read error correction module run on the barcoded FASTQ before assembly
- name: InSilicoSeq
  role: Simulates barcoded SPIso-seq-style paired-end reads from GRCh38/GENCODE v32
    gene loci for benchmarking
- name: IsoQuant
  role: Assigns assembled contigs to annotated GENCODE v32 isoforms to compute precision,
    recall, and fixed-recall metrics
- name: Bandage
  role: GUI visualization of the SPAdes assembly graph and its GFA representation
    for inspecting per-barcode subgraphs
  repo: https://github.com/rrwick/Bandage
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:e7bda43677f3c7fcd9403d4ea69742d692efe0af504a27a952286176f671b975
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bandage
  - BayesHammer
  - InSilicoSeq
  - IsoQuant
  - SPAdes
  - cloudrnaSPAdes
  - spades-gbuilder
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# linked-read-isoform-assembly

## Summary

Reference-free assembly of full-length isoforms from barcoded (linked-read) bulk RNA-seq using cloudrnaSPAdes, a SPAdes feature release that resolves isoforms per read cloud via barcode-specific subgraph simplification and barcode-aware filtering.

## When to use

Use when input is barcoded bulk RNA-seq (e.g. SPIso-seq-style linked reads in FASTQ) and the goal is de novo full-length isoform reconstruction, especially for highly alternatively spliced genes such as GYPC, MAPT, or BIN1.

## When NOT to use

- Input is standard (non-barcoded) RNA-seq — rnaSPAdes, not cloudrnaSPAdes, is the de novo transcriptome assembler for that case
- The goal is whole-genome assembly of a large (mammalian-size) genome — SPAdes is not intended for larger genomes
- Reads are already long full-length transcripts (PacBio CLR/ONT) — SPAdes uses those only for gap closure and repeat resolution, not as the primary barcoded input

## Inputs

- Barcoded paired-end Illumina RNA-seq FASTQ files (SPIso-seq-style linked reads)
- Reference gene annotation for isoform assignment (e.g. GRCh38/GENCODE v32 GTF/GFF)
- Optional: simulated reads generated with InSilicoSeq from known gene loci for benchmarking

## Outputs

- per-gene contigs.fasta of assembled isoform sequences
- per-barcode simplified subgraphs and extracted simple isoform paths
- IsoQuant isoform-assignment report (gene_isoform_assignment_metrics.tsv with precision, recall, fixed-recall)

## How to apply

First correct read errors with BayesHammer (the SPAdes default error-correction module for Illumina reads). Run cloudrnaSPAdes on the corrected barcoded paired-end FASTQ (optionally per gene locus) to build a shared assembly graph with spades-gbuilder and to partition reads into read clouds by barcode. For each barcode, extract a barcode-specific subgraph that leaves the shared graph unmodified, compute barcode-specific edge coverage along sequence-of-edge paths, then simplify the subgraph by clipping low-coverage tips (dead-end/dead-start edges), removing bulges by comparing barcode coverage of parallel edges between the same node pairs, and closing gaps across short low-coverage edges — because an edge erroneous for one barcode's isoform may be correct for another's. Extract simple isoform paths per subgraph as contigs, then assign contigs to annotated isoforms (e.g. GENCODE v32) with IsoQuant. To improve recall for complex genes (MAPT, 16 isoforms; BIN1, 15 isoforms), repeat after filtering read clouds that produce multiple isoforms for the same gene ('Fixed recall' procedure) or removing low-coverage reads.

## Related tools

- **cloudrnaSPAdes** (Feature release of SPAdes that assembles full-length isoforms reference-free from barcoded RNA-seq linked reads; performs per-read-cloud subgraph simplification (tip clipping, bulge removal, gap closing)) — github.com/1dayac/cloudrnaSPAdes-release
- **SPAdes** (Assembly toolkit providing graph construction (spades-gbuilder) and the iterative short-read assembly engine underlying cloudrnaSPAdes) — github.com/1dayac/cloudrnaSPAdes-release
- **BayesHammer** (Read error correction module run on the barcoded FASTQ before assembly)
- **InSilicoSeq** (Simulates barcoded SPIso-seq-style paired-end reads from GRCh38/GENCODE v32 gene loci for benchmarking)
- **IsoQuant** (Assigns assembled contigs to annotated GENCODE v32 isoforms to compute precision, recall, and fixed-recall metrics)
- **Bandage** (GUI visualization of the SPAdes assembly graph and its GFA representation for inspecting per-barcode subgraphs) — https://github.com/rrwick/Bandage

## Evaluation signals

- IsoQuant must assign >98% of assembled isoforms accurately to reference isoforms on the simulated GYPC, MAPT, and BIN1 datasets
- After the 'Fixed recall' barcode filtering procedure, recall for MAPT and BIN1 must increase over the unfiltered run and exceed the 'Separate strategy' competitor
- Each simplified per-barcode subgraph must yield extractable simple isoform paths consistent with the known isoform structure of the GYPC gene (5 annotated isoforms)
- The shared assembly graph must remain unmodified by per-barcode subgraph simplification (tip clipping acts only on the barcode-specific subgraph)
- gene_isoform_assignment_metrics.tsv must contain per-gene precision, recall, and fixed-recall values for both unfiltered and filtered assemblies

## Limitations

- SPAdes was initially designed for small genomes and 'is not intended for larger genomes (e.g. mammalian size genomes)'; per-gene locus assembly is used to keep the problem tractable
- Barcode-specific subgraph edits must be conservative: an edge erroneous for one barcode's isoform may be correct for another barcode's isoform, so simplification is applied per subgraph rather than to the shared graph
- MismatchCorrector is turned off by default, so mismatch and short indel rates in contigs are not corrected unless explicitly enabled
- Reported accuracy is demonstrated on simulated and real human data; performance depends on barcode quality and coverage of low-coverage isoforms

## Evidence

- [abstract] we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [abstract] Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity.: "Evaluating it on simulated and real human data, we found that cloudrnaSPAdes accurately assembles isoforms, even for genes with high isoform diversity."
- [methods] BayesHammer – read error correction tool for Illumina reads, which works well on both single-cell and standard data sets.: "BayesHammer – read error correction tool for Illumina reads, which works well on both single-cell and standard data sets."
- [methods] spades-read-filter is a tool for filtering reads with median kmer coverage less than threshold.: "spades-read-filter is a tool for filtering reads with median kmer coverage less than threshold."
- [readme] We recommend to run SPAdes with BayesHammer/IonHammer to obtain high-quality assemblies.: "We recommend to run SPAdes with BayesHammer/IonHammer to obtain high-quality assemblies."
- [readme] SPAdes is not intended for larger genomes (e.g. mammalian size genomes).: "SPAdes is not intended for larger genomes (e.g. mammalian size genomes)."
