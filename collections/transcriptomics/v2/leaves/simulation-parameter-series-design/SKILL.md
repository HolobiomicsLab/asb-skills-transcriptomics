---
name: simulation-parameter-series-design
description: Use when benchmarking a circRNA assembly or isoform-quantification method
  against known ground truth, or when validating that regenerated simulation datasets
  match a published design (depths, read lengths, loci, conditions).
when_to_use_negative:
- The goal is empirical benchmarking on real RNA-seq data rather than controlled simulation
  with known ground-truth abundances
- The target circular transcripts are intronic or intergenic — the CircAST pipeline
  quantifies only exon-annotated circular transcripts, so simulated panels must be
  built on annotated exonic loci
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: CircAST
  role: Tool under evaluation on simulated datasets; assembles full-length circular
    isoform sequences via multiple splice graphs and estimates expression (FPKM/TPM)
    by EM read assignment
  repo: https://github.com/xiaofengsong/CircAST
- name: tophat
  role: Paired-end alignment of simulated reads against the UCSC hg19 Bowtie index
    prior to CircAST input generation
- name: samtools
  role: Sorts the tophat2 BAM by read name and converts it to name-sorted SAM for
    CircAST input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:26baddca7cafd25814654fffb1c549182fbce7129b85be05922ebd66bfe51e75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CircAST
  - samtools
  - tophat
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# simulation-parameter-series-design

## Summary

Designs controlled simulation panels of circular RNA sequencing data — depth series, read-length series, and multi-locus/multi-condition isoform panels with assigned ground-truth abundances — to benchmark full-length assembly and relative quantification of circRNA isoform expression.

## When to use

Use when benchmarking a circRNA assembly or isoform-quantification method against known ground truth, or when validating that regenerated simulation datasets match a published design (depths, read lengths, loci, conditions).

## When NOT to use

- The goal is empirical benchmarking on real RNA-seq data rather than controlled simulation with known ground-truth abundances
- The target circular transcripts are intronic or intergenic — the CircAST pipeline quantifies only exon-annotated circular transcripts, so simulated panels must be built on annotated exonic loci

## Inputs

- Back-spliced junction records in CircAST-compatible junction-list format (e.g., 2923 Hs68 records from SRA: SRR444974)
- UCSC hg19 GTF gene annotation
- Ground-truth isoform abundance assignments (theoretical expression per circular transcript)
- Design parameters: read length series (50–150 bp), depth series (0.09M–5.92M read pairs), and condition/locus panel specification

## Outputs

- Simulated paired-end FASTQ read sets for each depth and read-length condition
- 4-loci (NOLC1, PAFAH1B2, DHPS, EXOSC10), 10-isoform, 8-condition simulation panel with per-condition isoform abundances
- Ground-truth expression table used to compute estimated-vs-theoretical correlations

## How to apply

First, build circular transcript structures from real back-spliced junction records (here, 2923 Hs68 BSJ records from SRA: SRR444974 in CircAST-compatible junction-list format) on the UCSC hg19 GTF annotation, introducing randomly skipped exons to create isoform variants. Assign theoretical expression abundances to each circular transcript to serve as ground truth, then simulate paired-end 100-bp reads (tophat2-style alignment) across a depth series of 0.09M–5.92M read pairs and a read-length series (50–150 bp at a fixed 2.96M read pairs). For relative-quantification testing, build a dedicated panel of 10 circular isoforms from 4 human gene loci (NOLC1, PAFAH1B2, DHPS, EXOSC10), each expressed under 8 conditions with per-condition abundance settings. Run the tool under evaluation (CircAST with default BSJ-support threshold 10) on each simulated dataset and judge correctness by confirming datasets match design parameters (2923 transcripts; 100-bp paired-end at 0.09–5.92M depth; 50–150 bp at 2.96M depth; 10 isoforms across 8 conditions) and by correlation between theoretical and estimated abundances (expected 0.85–0.99).

## Related tools

- **CircAST** (Tool under evaluation on simulated datasets; assembles full-length circular isoform sequences via multiple splice graphs and estimates expression (FPKM/TPM) by EM read assignment) — https://github.com/xiaofengsong/CircAST
- **tophat** (Paired-end alignment of simulated reads against the UCSC hg19 Bowtie index prior to CircAST input generation)
- **samtools** (Sorts the tophat2 BAM by read name and converts it to name-sorted SAM for CircAST input)

## Examples

```
python CircAST.py -L 100 -T 5 -G /home/***/circRNA/Gene/genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt
```

## Evaluation signals

- Each simulated dataset contains 2923 circular transcripts with 100-bp paired-end reads at depths spanning 0.09M–5.92M read pairs
- Read-length series datasets are all fixed at 2.96M read pairs with lengths spanning 50–150 bp
- The quantification panel contains exactly 10 circular isoforms across 8 conditions from the loci NOLC1, PAFAH1B2, DHPS, and EXOSC10
- Correlation coefficients between theoretical and estimated expression fall in the reported 0.85–0.99 range
- Assembly sensitivity and precision on simulated full sequences fall within the reported 85.63%–94.32% and 81.96%–87.55% ranges

## Limitations

- CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over
- Remnant linear mRNA in the samples can affect the performance of circular transcript assembly and generate false transcripts to some extent
- The current CircAST pipeline can assemble full-length sequences and quantify exon-annotated circular transcripts, but may miss intronic or intergenic circular transcripts

## Evidence

- [methods] depth series design: [evidence span withheld — anchor status: not_found]
- [results] multi-locus panel: [evidence span withheld — anchor status: not_found]
- [abstract] quantification benchmark: "CircAST quantified the expression of circRNA isoforms with correlation coefficients of 0.85-0.99 between theoretical and estimated values"
- [readme] CircAST threshold parameter: "set threshold for back-splicing junction supporting reads(optional, default: 10)"
