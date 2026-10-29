---
name: junction-read-count-benchmark-comparison
description: Use when evaluating whether a circRNA isoform quantification method outperforms
  naive junction-read counting, using simulated loci with two or more splicing isoforms
  and known theoretical expression values across conditions.
when_to_use_negative:
- Samples are not RNase R-treated (remnant linear mRNA can affect assembly and generate
  false transcripts)
- Target circular transcripts are intronic or intergenic (CircAST handles only exon-annotated
  circular transcripts)
- Only a single condition or single-isoform loci are available, so no correlation
  across conditions/isoforms can be computed
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CircAST
  role: splice-graph assembly and EM-based isoform quantification producing FPKM/TPM
    per circular transcript
  repo: https://github.com/xiaofengsong/CircAST
- name: Tophat2
  role: paired-end alignment of simulated reads to generate accepted_hits.bam
- name: Samtools
  role: name-sorting the BAM and converting to accepted_hits.sorted.sam required by
    CircAST
- name: CIRCexplorer2
  role: upstream back-splice junction detection supplying CircAST-format junction
    list
- name: CIRI2
  role: alternative upstream back-splice junction detector
- name: UROBORUS
  role: alternative upstream back-splice junction detector
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:4d979e6ac4863237c586b54c13fa08560711ec2ba313e0c6a14422e28c7e70ab
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# junction-read-count-benchmark-comparison

## Summary

Benchmark CircAST's splice-graph/EM isoform quantification against raw back-splice junction read counting on simulated multi-isoform circRNA loci, using Pearson correlation between estimated and theoretical relative abundances across conditions.

## When to use

Use when evaluating whether a circRNA isoform quantification method outperforms naive junction-read counting, using simulated loci with two or more splicing isoforms and known theoretical expression values across conditions.

## When NOT to use

- Samples are not RNase R-treated (remnant linear mRNA can affect assembly and generate false transcripts)
- Target circular transcripts are intronic or intergenic (CircAST handles only exon-annotated circular transcripts)
- Only a single condition or single-isoform loci are available, so no correlation across conditions/isoforms can be computed

## Inputs

- Simulated paired-end RNA-seq alignments per condition (tophat2 accepted_hits.bam)
- CircRNA back-splice junction lists per condition (CIRCexplorer2/CIRI2/UROBORUS output in CircAST format)
- Gene annotation GTF file (UCSC-compatible, same as used by upstream detector)
- Theoretical (ground-truth) isoform relative abundances per locus per condition

## Outputs

- Per-locus CircAST-estimated relative isoform fractions across 8 conditions
- Pearson correlation coefficients between theoretical and CircAST-estimated expression (expected range 0.849-0.993)
- Pearson correlations from junction-read counts alone (baseline for comparison)

## How to apply

Simulate RNA-seq reads for four circRNA loci (each with two or more isoforms) under eight expression conditions, align with tophat2, and preprocess by 'samtools sort -n' followed by 'samtools view' to obtain accepted_hits.sorted.sam. Generate the back-splice junction list per condition with CIRCexplorer2, CIRI2, or UROBORUS in CircAST format, then run CircAST (e.g., python CircAST.py -L <read_length> -T 10 -G genes.gtf -F accepted_hits.sorted.sam -J junction.txt) so the splice-graph/EM engine allocates reads to circular isoforms. Parse the CircAST output table (junction read counts, assigned read counts, FPKM, TPM), compute per-locus relative isoform fractions, and calculate Pearson correlations (scipy.stats.pearsonr) between theoretical and estimated values across the 8 conditions. Repeat the correlation computation using raw junction read counts alone and compare, since CircAST's advantage comes from allocating all informative reads (not just BSJ reads) to isoforms via maximum likelihood.

## Related tools

- **CircAST** (splice-graph assembly and EM-based isoform quantification producing FPKM/TPM per circular transcript) — https://github.com/xiaofengsong/CircAST
- **Tophat2** (paired-end alignment of simulated reads to generate accepted_hits.bam)
- **Samtools** (name-sorting the BAM and converting to accepted_hits.sorted.sam required by CircAST)
- **CIRCexplorer2** (upstream back-splice junction detection supplying CircAST-format junction list)
- **CIRI2** (alternative upstream back-splice junction detector)
- **UROBORUS** (alternative upstream back-splice junction detector)

## Examples

```
python CircAST.py -L 100 -T 5 -G /home/***/circRNA/Gene/genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt
```

## Evaluation signals

- Per-locus Pearson correlation between CircAST estimates and theoretical values falls within the reported 0.849-0.993 range
- CircAST correlations exceed the correlations obtained from junction-read counts alone for the same loci/conditions
- CircAST output table contains all 13 expected columns including junction read counts, assigned read counts, FPKM, and TPM
- Junction-support threshold (-T, default 10) is applied consistently across conditions
- The same GTF annotation file is used for CircAST and its upstream junction detector

## Limitations

- CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over
- Remnant linear mRNA in the samples can affect assembly performance and generate false transcripts
- The current CircAST pipeline may miss intronic or intergenic circular transcripts
- Other BSJ detection tools require output file format conversion to CircAST input format

## Evidence

- [abstract] CircAST quantified the expression of circRNA isoforms with correlation coefficients of 0.85-0.99 between theoretical and estimated values: "CircAST quantified the expression of circRNA isoforms with correlation coefficients of 0.85-0.99 between theoretical and estimated values"
- [intro] estimate expression levels by allocating reads to isoforms with an expectation-maximization algorithm: "This tool uses multiple splice graphs as a circRNA model to reconstruct the full-length sequence and estimate expression levels by allocating reads to different isoforms with an"
- [readme] samtools sort by read name workflow: "samtools sort -n -o accepted_hits.sorted.bam accepted_hits.bam"
- [methods] set threshold for back-splicing junction supporting reads: "set threshold for back-splicing junction supporting reads(optional, default: 10)"
- [discussion] CircAST applicable only to RNase R-treated samples: "CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over."
