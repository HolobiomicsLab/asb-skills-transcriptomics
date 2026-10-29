---
name: sam-alignment-name-sorting-preprocessing
description: 'Use when running CircAST downstream of tophat2 alignments: the pipeline
  requires accepted_hits.bam sorted by read name and converted to accepted_hits.sorted.sam
  before isoform assembly and quantification.'
when_to_use_negative:
- Alignments were produced by an aligner other than tophat2 — CircAST expects tophat2-generated
  SAM with the pairing conventions it parses
- The target analysis does not involve CircAST circRNA isoform assembly/quantification
  — coordinate-sorted BAMs suffice for most other tools, and this conversion is unnecessary
- Only a circRNA junction list is available without matched alignment data — CircAST
  cannot allocate reads without the sorted SAM alignment
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: Tophat2
  role: Performs paired-end RNA-seq alignment against a Bowtie genome index with GTF-guided
    spliced alignment, producing accepted_hits.bam
- name: Samtools
  role: Sorts accepted_hits.bam by read name ('sort -n') and converts it to SAM text
    ('view'), yielding CircAST's required -F input
- name: CircAST
  role: Downstream consumer of the name-sorted SAM, using it with the junction list
    and GTF to assemble full-length circRNA isoforms and estimate FPKM/TPM expression
  repo: https://github.com/xiaofengsong/CircAST
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:92de6d630f289c60299fc76b5738af6f3ad6560a1266936f9374588a32d1860c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CIRCexplorer2
  - CIRI2
  - CircAST
  - Samtools
  - Tophat2
  - UROBORUS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# SAM alignment name-sorting preprocessing

## Summary

Prepares tophat2-generated BAM alignments for CircAST's full-length circRNA isoform assembly by name-sorting and converting them to SAM text, a required input format for the splice-graph/EM quantification engine.

## When to use

Use when running CircAST downstream of tophat2 alignments: the pipeline requires accepted_hits.bam sorted by read name and converted to accepted_hits.sorted.sam before isoform assembly and quantification.

## When NOT to use

- Alignments were produced by an aligner other than tophat2 — CircAST expects tophat2-generated SAM with the pairing conventions it parses
- The target analysis does not involve CircAST circRNA isoform assembly/quantification — coordinate-sorted BAMs suffice for most other tools, and this conversion is unnecessary
- Only a circRNA junction list is available without matched alignment data — CircAST cannot allocate reads without the sorted SAM alignment

## Inputs

- paired-end FASTQ read files (reads_1.fastq, reads_2.fastq)
- UCSC-format gene annotation GTF (e.g. genes_homo.gtf from hg19 or mm10)
- Bowtie genome index (e.g. UCSC hg19 Homo_sapiens/Sequence/BowtieIndex/genome)
- tophat2 output BAM file (accepted_hits.bam)

## Outputs

- name-sorted BAM (accepted_hits.sorted.bam)
- name-sorted SAM text file (accepted_hits.sorted.sam) ready as CircAST's -F input

## How to apply

Align paired-end reads with tophat2 using a gene annotation GTF and a Bowtie genome index (e.g. UCSC hg19). Sort the resulting accepted_hits.bam by read name (not coordinate) using 'samtools sort -n -o accepted_hits.sorted.bam accepted_hits.bam', then convert to text with 'samtools view accepted_hits.sorted.bam > accepted_hits.sorted.sam'. Name-sorting is essential because CircAST pairs mate reads from the sorted SAM to allocate reads to circular isoforms across conditions. Use the same GTF annotation file when running tophat2, CircAST, and the upstream BSJ detector (CIRCexplorer2, CIRI2, or UROBORUS) to keep coordinate systems consistent.

## Related tools

- **Tophat2** (Performs paired-end RNA-seq alignment against a Bowtie genome index with GTF-guided spliced alignment, producing accepted_hits.bam)
- **Samtools** (Sorts accepted_hits.bam by read name ('sort -n') and converts it to SAM text ('view'), yielding CircAST's required -F input)
- **CircAST** (Downstream consumer of the name-sorted SAM, using it with the junction list and GTF to assemble full-length circRNA isoforms and estimate FPKM/TPM expression) — https://github.com/xiaofengsong/CircAST

## Examples

```
tophat -o results -p 12 -G genes_homo.gtf /home/index/Homo_sapiens/UCSC/hg19/Sequence/BowtieIndex/genome reads_1.fastq reads_2.fastq; samtools sort -n -o accepted_hits.sorted.bam accepted_hits.bam; samtools view accepted_hits.sorted.bam > accepted_hits.sorted.sam
```

## Evaluation signals

- accepted_hits.sorted.sam exists and is non-empty after 'samtools view' conversion
- Read order in the sorted SAM/BAM is by read name (QNAME), not genomic coordinate — verify with 'samtools view -H' followed by inspecting initial QNAMEs
- The same GTF file was used for tophat2 alignment, BSJ detection, and the subsequent CircAST -G argument
- Running CircAST with the resulting SAM (e.g. -L 100 -T 5) produces an output table with junction read counts, FPKM, and TPM per circular transcript

## Limitations

- The workflow is tied to tophat2 alignments; SAMs from other aligners may not satisfy CircAST's parsing expectations without adaptation
- CircAST quantifies exon-annotated circular transcripts and may miss intronic or intergenic circular transcripts regardless of alignment preprocessing quality
- The GTF used here must be identical to the one used by CircAST and the upstream circRNA detector; mismatches break locus-level assembly

## Evidence

- [methods] sort BAM file by read name: "samtools sort -n -o accepted_hits.sorted.bam accepted_hits.bam"
- [methods] convert sorted BAM to SAM: "samtools view accepted_hits.sorted.bam > accepted_hits.sorted.sam"
- [readme] README input requirement: "a sorted Sequence Alignment/Map (SAM) of pair-end reads generated by tophat2 and samtools"
- [readme] README GTF consistency note: "The GTF annotation file should be the same one when running CircAST and its upstream softwares."
