---
name: rna-seq-paired-end-read-simulation
description: Use when benchmarking a circRNA full-length assembly or isoform quantification
  method against known ground truth, requiring simulated paired-end reads with controlled
  depth, read length, and per-condition isoform abundances.
when_to_use_negative:
- The goal is real-data circRNA discovery on RNase R-treated RNA-seq rather than controlled
  benchmarking against known truth
- Circular transcripts of interest are intronic or intergenic, which the CircAST pipeline
  (and hence this hg19-annotation-based simulation design) cannot model
- Only single-end or non-paired-end data are available, since the design and CircAST
  itself assume paired-end SAM/BAM alignments from tophat2
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3511
tools:
- name: CircAST
  role: Downstream method under evaluation; also defines the junction-list input format
    the simulation must reproduce
  repo: https://github.com/xiaofengsong/CircAST
- name: tophat
  role: Paired-end alignment step applied to the simulated FASTQ reads before CircAST
    assembly and quantification
- name: samtools
  role: Sorts simulated-read BAM by read name and converts to SAM for CircAST input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:822ea9ae7785f87a09484709c0b7ccef56a5d4a29e281209f12be1c0d44a04ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CircAST
  - samtools
  - tophat
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# rna-seq-paired-end-read-simulation

## Summary

Generates simulated paired-end RNA-seq datasets with known ground-truth transcript abundances to benchmark circRNA assembly and relative quantification, as done in the CircAST study with 2923 Hs68-derived circular transcripts and a 4-loci (NOLC1, PAFAH1B2, DHPS, EXOSC10), 10-isoform, 8-condition panel.

## When to use

Use when benchmarking a circRNA full-length assembly or isoform quantification method against known ground truth, requiring simulated paired-end reads with controlled depth, read length, and per-condition isoform abundances.

## When NOT to use

- The goal is real-data circRNA discovery on RNase R-treated RNA-seq rather than controlled benchmarking against known truth
- Circular transcripts of interest are intronic or intergenic, which the CircAST pipeline (and hence this hg19-annotation-based simulation design) cannot model
- Only single-end or non-paired-end data are available, since the design and CircAST itself assume paired-end SAM/BAM alignments from tophat2

## Inputs

- Back-spliced junction records in CircAST junction-list format (e.g., 2923 records from SRA:SRR444974)
- UCSC hg19 GTF gene annotation (UCSC-compatible GTF used consistently across CircAST and upstream tools)
- Ground-truth abundance assignments for each circular transcript (e.g., all 2923 transcripts)
- Design parameters: read length (100 bp default; 50–150 bp series), depth (0.09M–5.92M read pairs), panel structure (10 isoforms, 4 loci, 8 conditions)

## Outputs

- Simulated paired-end FASTQ datasets (depth series and read-length series)
- 4-loci, 10-isoform, 8-condition simulation panel with per-condition isoform abundance settings
- Ground-truth abundance table for evaluating assembly (sensitivity/precision) and quantification (correlation between theoretical and estimated expression)

## How to apply

Extract back-spliced junction records in CircAST-compatible junction-list format (e.g., 2923 records from Hs68 SRA:SRR444974), build circular transcript structures on the UCSC hg19 GTF annotation, and introduce randomly skipped exons to create isoform variants. Assign theoretical expression abundances as ground truth, then simulate paired-end reads — for CircAST, 100-bp paired-end reads across a depth series of 0.09M to 5.92M read pairs, and a read-length series spanning 50–150 bp at a fixed 2.96M read pairs. Build a dedicated panel (e.g., 10 circular isoforms from NOLC1, PAFAH1B2, DHPS, and EXOSC1 across 8 conditions) to test relative quantification under dynamic multi-isoform expression. Judge correctness by verifying each simulated dataset matches its design parameters (transcript count, read length, depth, isoform/condition structure).

## Related tools

- **CircAST** (Downstream method under evaluation; also defines the junction-list input format the simulation must reproduce) — https://github.com/xiaofengsong/CircAST
- **tophat** (Paired-end alignment step applied to the simulated FASTQ reads before CircAST assembly and quantification)
- **samtools** (Sorts simulated-read BAM by read name and converts to SAM for CircAST input)

## Examples

```
tophat -o results -p 12 -G /home/***/genes_homo.gtf /home/***/index/Homo_sapiens/UCSC/hg19/Sequence/BowtieIndex/genome reads_1.fastq reads_2.fastq
```

## Evaluation signals

- Each simulated dataset matches its reported design parameters: 2923 circular transcripts, 100-bp paired-end reads, 0.09M–5.92M read-pair depth series
- Read-length series runs 50–150 bp at a fixed 2.96M read pairs
- Panel contains 10 isoforms across 8 conditions from exactly 4 loci (NOLC1, PAFAH1B2, DHPS, EXOSC10)
- Theoretical abundances recover correlation coefficients of 0.85–0.99 between theoretical and estimated values when quantified by CircAST
- Assembly sensitivity falls in the 85.63%–94.32% range and precision in the 81.96%–87.55% range reported for CircAST in simulation

## Limitations

- Simulation and CircAST evaluation are restricted to exon-annotated circular transcripts on the hg19 GTF; intronic or intergenic circular transcripts may be missed
- Remnant linear mRNA can affect assembly performance and generate false transcripts, so simulated or real samples ideally reflect RNase R treatment
- CircAST can be applied only to RNase R-treated samples, limiting what the simulated benchmarks represent for non-treated data

## Evidence

- [abstract] CircAST correctly assembled the full sequences of circRNAs with a sensitivity of 85.63%-94.32% and a precision of 81.96%-87.55%: "CircAST correctly assembled the full sequences of circRNAs with a sensitivity of 85.63%-94.32% and a precision of 81.96%-87.55%"
- [abstract] CircAST quantified the expression of circRNA isoforms with correlation coefficients of 0.85-0.99 between theoretical and estimated values: "CircAST quantified the expression of circRNA isoforms with correlation coefficients of 0.85-0.99 between theoretical and estimated values"
- [results] A dedicated simulation panel was generated consisting of 10 circular isoforms from 4 human gene loci (NOLC1, PAFAH1B2, DHPS, and EXOSC10), each expressed under 8 conditions, to test relative quantification.: [evidence span withheld — anchor status: not_found]
- [methods] Simulate paired-end 100-bp reads from the assigned transcripts across a depth series ranging from 0.09M to 5.92M read pairs per dataset.: [evidence span withheld — anchor status: not_found]
- [readme] CircAST works with three input files. A GTF annotation file, a sorted Sequence Alignment/Map (SAM) of pair-end reads generated by tophat2 and samtools, and a circRNA junction list: "CircAST works with three input files. A GTF annotation file, a sorted Sequence Alignment/Map (SAM) of pair-end reads generated by tophat2 and samtools, and a circRNA junction list containing the"
