---
name: exon-skipping-variation-modeling
description: Use when you need simulated circRNA isoform panels with known full-length
  sequences and expression levels — e.g. benchmarking assembly or quantification tools
  like CircAST across depth, read-length, or multi-condition series.
when_to_use_negative:
- The goal is to detect or quantify circRNAs from real RNA-seq data rather than to
  build simulated ground truth for benchmarking
- The circRNAs of interest are intronic or intergenic — the CircAST pipeline assembles
  and quantifies only exon-annotated circular transcripts
- Samples are not RNase R-treated, since CircAST can be applied only to RNase R-treated
  samples in which remnant linear mRNA may still affect assembly
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CircAST
  role: Assembles alternatively spliced circRNA isoform full-length sequences and
    estimates expression against the simulated ground truth
  repo: https://github.com/xiaofengsong/CircAST
- name: tophat
  role: Paired-end alignment of simulated reads to the UCSC hg19 Bowtie index before
    splice-graph construction
- name: samtools
  role: Sorts tophat2's accepted_hits.bam by read name and converts it to the sorted
    SAM required by CircAST
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:266b286766151e6e4172c3016ad3674fd682a40b3b9ffffed2e1426e2c4ee8e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CircAST
  - samtools
  - tophat
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# exon-skipping-variation-modeling

## Summary

Model exon-skipping variation in circular transcripts by building circular transcript structures on a UCSC hg19 GTF annotation and randomly skipping exons to generate isoform variants, then assigning theoretical abundances so that CircAST's assembly and relative quantification (correlation 0.85-0.99) can be evaluated against known ground truth.

## When to use

Use when you need simulated circRNA isoform panels with known full-length sequences and expression levels — e.g. benchmarking assembly or quantification tools like CircAST across depth, read-length, or multi-condition series.

## When NOT to use

- The goal is to detect or quantify circRNAs from real RNA-seq data rather than to build simulated ground truth for benchmarking
- The circRNAs of interest are intronic or intergenic — the CircAST pipeline assembles and quantifies only exon-annotated circular transcripts
- Samples are not RNase R-treated, since CircAST can be applied only to RNase R-treated samples in which remnant linear mRNA may still affect assembly

## Inputs

- Back-spliced junction records in CircAST-compatible junction-list format (e.g. 2923 Hs68 BSJ records from SRA: SRR444974)
- UCSC hg19 GTF gene annotation file
- Gene loci selected for the isoform panel (NOLC1, PAFAH1B2, DHPS, EXOSC10)
- Per-condition isoform abundance settings for the 8-condition panel
- Simulation design parameters (depth 0.09M-5.92M read pairs; read lengths 50-150 bp; paired-end 100 bp)

## Outputs

- Circular transcript structures with randomly skipped exons (isoform variants)
- Ground-truth theoretical expression abundances per circular transcript
- Simulated paired-end FASTQ datasets (100-bp depth series; 50-150 bp read-length series)
- 10-isoform, 8-condition, 4-loci simulation panel with per-condition abundances

## How to apply

Extract back-spliced junction records in CircAST-compatible junction-list format (e.g. the 2923 Hs68 records from SRA: SRR444974) and construct circular transcript structures on the UCSC hg19 GTF annotation, introducing randomly skipped exons to create isoform variants. Assign theoretical expression abundances to each circular transcript to serve as ground truth. Simulate paired-end 100-bp reads across a depth series (0.09M-5.92M read pairs per dataset) and a read-length series (50-150 bp at fixed 2.96M read pairs). For multi-isoform benchmarking, build a dedicated panel of 10 circular isoforms from 4 human gene loci (NOLC1, PAFAH1B2, DHPS, EXOSC10) expressed under 8 conditions with per-condition abundance settings. Align simulated reads with tophat2, sort the BAM by name with samtools, and run CircAST to compare estimated vs theoretical expression.

## Related tools

- **CircAST** (Assembles alternatively spliced circRNA isoform full-length sequences and estimates expression against the simulated ground truth) — https://github.com/xiaofengsong/CircAST
- **tophat** (Paired-end alignment of simulated reads to the UCSC hg19 Bowtie index before splice-graph construction)
- **samtools** (Sorts tophat2's accepted_hits.bam by read name and converts it to the sorted SAM required by CircAST)

## Examples

```
python CircAST.py -L 100 -T 5 -G /home/***/circRNA/Gene/genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt
```

## Evaluation signals

- Simulated datasets match design parameters: 2923 circular transcripts, 100-bp paired-end reads at 0.09M-5.92M depth, 50-150 bp at 2.96M depth
- Simulation panel contains exactly 10 circular isoforms from the 4 named loci (NOLC1, PAFAH1B2, DHPS, EXOSC10) across 8 conditions
- CircAST assembly achieves sensitivity in the reported 85.63%-94.32% range and precision in the 81.96%-87.55% range
- Pearson correlation between theoretical and estimated isoform expression falls within the reported 0.85-0.99 range
- Each simulated transcript's structure is checkable against the hg19 GTF with skipped exons consistent with the parent gene's exon coordinates

## Limitations

- CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over
- Remnant linear mRNA in samples can affect the performance of circular transcript assembly and generate false transcripts
- The current CircAST pipeline may miss intronic or intergenic circular transcripts and quantifies only exon-annotated circular transcripts
- Junction lists from other BSJ detection tools require output-format conversion before CircAST can consume them

## Evidence

- [other] construct circular transcript structures on the UCSC hg19 GTF annotation, introducing randomly skipped exons to create isoform variants: [evidence span withheld — anchor status: not_found]
- [other] A dedicated simulation panel was generated consisting of 10 circular isoforms from 4 human gene loci (NOLC1, PAFAH1B2, DHPS, and EXOSC10), each expressed under 8 conditions, to test relative quantification.: [evidence span withheld — anchor status: not_found]
- [abstract] we developed CircAST, a tool to assemble alternatively spliced circRNA transcripts and estimate their expression by using multiple splice graphs: "we developed CircAST, a tool to assemble alternatively spliced circRNA transcripts and estimate their expression by using multiple splice graphs"
- [readme] CircAST is compatible with popular upstream circRNA detection tools (CIRCexplorer2, CIRI2, UROBORUS).: "CircAST is compatible with popular upstream circRNA detection tools (CIRCexplorer2, CIRI2, UROBORUS)."
