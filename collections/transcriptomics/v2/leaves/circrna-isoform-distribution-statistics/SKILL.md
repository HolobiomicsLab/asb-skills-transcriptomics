---
name: circrna-isoform-distribution-statistics
description: Use when CircAST output tables (per-transcript junction reads, assigned
  reads, FPKM/TPM) are available for multiple samples and the goal is to characterize
  how many and how diverse the circular isoforms are across cell lines or tissues.
when_to_use_negative:
- The samples were not RNase R-treated — the article states 'CircAST can be applied
  only to RNase R-treated samples', so diversity statistics on non-enriched libraries
  would be unreliable.
- The target circRNAs are intronic or intergenic — the current CircAST pipeline may
  miss these, so exon-annotated transcript counts and exon-number distributions would
  be incomplete.
- The alignment or BSJ detection used a different GTF than CircAST — counts of assembled
  exonic transcripts would be inconsistent with the junction lists and not comparable
  across samples.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: CircAST
  role: Assembles alternatively spliced circular transcripts on multiple splice graphs
    and quantifies them (junction reads, assigned reads, FPKM, TPM) via EM-based maximum-likelihood
    read assignment, providing the per-sample tables the statistics aggregate.
  repo: https://github.com/xiaofengsong/CircAST
- name: TopHat
  role: Performs paired-end alignment of the SRA FASTQ reads against the UCSC hg19
    Bowtie index using the gene GTF before CircAST assembly.
- name: samtools
  role: Name-sorts the TopHat2 accepted_hits.bam ('samtools sort -n') and converts
    it to accepted_hits.sorted.sam, the -F input CircAST requires.
- name: CIRCexplorer2
  role: One of the upstream tools that detects back-spliced junctions to build the
    circRNA junction list (-J) per sample.
- name: CIRI2
  role: Alternative upstream BSJ detector producing circRNA_list.txt input for CircAST.
- name: UROBORUS
  role: Alternative upstream BSJ detector producing the CircAST junction list; its
    annotated/denovo outputs are directly compatible.
- name: CircFullSeq.py
  role: Companion script that extracts full sequences of the circular transcripts
    from CircAST results, needed for full-length distribution statistics.
  repo: https://github.com/xiaofengsong/CircAST
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:bba4abad0063e3db805f2d3846b9baaf930d17366da1bdb9f37869ef6412ae2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CIRCexplorer2
  - CIRI2
  - CircAST
  - CircFullSeq.py
  - TopHat
  - UROBORUS
  - samtools
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# circrna-isoform-distribution-statistics

## Summary

Aggregate CircAST per-sample outputs to quantify circRNA isoform diversity: count exonic circular transcripts, examine exon-number and full-length distributions, and compute the fraction of back-splice junctions assigned to more than one isoform.

## When to use

Use when CircAST output tables (per-transcript junction reads, assigned reads, FPKM/TPM) are available for multiple samples and the goal is to characterize how many and how diverse the circular isoforms are across cell lines or tissues.

## When NOT to use

- The samples were not RNase R-treated — the article states 'CircAST can be applied only to RNase R-treated samples', so diversity statistics on non-enriched libraries would be unreliable.
- The target circRNAs are intronic or intergenic — the current CircAST pipeline may miss these, so exon-annotated transcript counts and exon-number distributions would be incomplete.
- The alignment or BSJ detection used a different GTF than CircAST — counts of assembled exonic transcripts would be inconsistent with the junction lists and not comparable across samples.

## Inputs

- CircAST result tables per sample (tab-delimited: chromosome, start, end, circular transcript name, parental gene, strand, number of exons, exon sizes, exon offsets, junction read counts, FPKM, TPM, assigned reads)
- Per-sample circRNA junction lists from CIRCexplorer2/CIRI2/UROBORUS (or converted from other BSJ tools) used as CircAST -J input
- Name-sorted paired-end SAM alignments (accepted_hits.sorted.sam from TopHat2 + samtools) and the UCSC hg19-compatible GTF used in all steps

## Outputs

- Per-sample counts of exonic circular transcripts (e.g. 5253, 4533, 11412) and their originating back-spliced circRNAs (e.g. 4549, 4201, 9982)
- Exon-number distribution and full-length transcript distribution per sample (Figure 4 statistics)
- Per-sample fraction of back-splice junctions assigned to more than one circular isoform (expected within the reported 7%-22% range)
- Cross-sample diversity comparison showing isoform-abundance-ratio patterns across HeLa, HEK293, and Hs68

## How to apply

Run CircAST per sample with the same UCSC-compatible GTF used for TopHat2 alignment and upstream BSJ detection, so each circRNA junction yields assembled circular transcripts with the README output columns (chromosome/start/end, transcript name, parental gene, strand, number of exons, exon sizes/offsets, junction read counts, FPKM, TPM, assigned reads). Then aggregate across samples: (1) count exonic circular transcripts per sample — e.g. the human cell-line datasets HeLa SRR1636985/SRR1636986/SRR3476956, HEK293 SRR3479244, and Hs68 SRR444974 yielded 5253, 4533, and 11,412 transcripts from 4549, 4201, and 9982 back-spliced circRNAs respectively; (2) tabulate the number-of-exons and full-length-sequence distributions; (3) compute, per sample, the fraction of back-splice junctions to which CircAST's EM-based maximum-likelihood read assignment allocated more than one isoform. These per-sample statistics reproduce the Figure 4 patterns and support the conclusion that circular transcripts are highly diverse and highly regulated across cell lines.

## Related tools

- **CircAST** (Assembles alternatively spliced circular transcripts on multiple splice graphs and quantifies them (junction reads, assigned reads, FPKM, TPM) via EM-based maximum-likelihood read assignment, providing the per-sample tables the statistics aggregate.) — https://github.com/xiaofengsong/CircAST
- **TopHat** (Performs paired-end alignment of the SRA FASTQ reads against the UCSC hg19 Bowtie index using the gene GTF before CircAST assembly.)
- **samtools** (Name-sorts the TopHat2 accepted_hits.bam ('samtools sort -n') and converts it to accepted_hits.sorted.sam, the -F input CircAST requires.)
- **CIRCexplorer2** (One of the upstream tools that detects back-spliced junctions to build the circRNA junction list (-J) per sample.)
- **CIRI2** (Alternative upstream BSJ detector producing circRNA_list.txt input for CircAST.)
- **UROBORUS** (Alternative upstream BSJ detector producing the CircAST junction list; its annotated/denovo outputs are directly compatible.)
- **CircFullSeq.py** (Companion script that extracts full sequences of the circular transcripts from CircAST results, needed for full-length distribution statistics.) — https://github.com/xiaofengsong/CircAST

## Examples

```
python CircAST.py -L 100 -T 5 -G /home/***/circRNA/Gene/genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt
```

## Evaluation signals

- Per-sample exonic transcript counts reproduce the reported pattern (5253/4533/11412 from 4549/4201/9982 BSJs for the three human cell-line datasets).
- The per-sample multi-isoform BSJ fraction falls within the reported 7%-22% range (Figure 4 reproduction criterion).
- Every aggregated transcript row carries all 13 README output fields, including junction read counts, FPKM, TPM, and assigned reads; transcript counts are consistent with the junction-list sizes.
- Exon-number and full-length distributions and isoform-abundance-ratio distributions match the reported Figure 4 patterns.
- Cross-sample comparison shows substantial diversity in circular transcripts across cell lines, as reported.

## Limitations

- CircAST can be applied only to RNase R-treated samples; remnant linear mRNA in samples can affect assembly performance and generate false transcripts, inflating apparent isoform diversity.
- The current CircAST pipeline assembles and quantifies only exon-annotated circular transcripts and may miss intronic or intergenic circular transcripts.
- Junction lists from BSJ tools other than CIRCexplorer2/CIRI2/UROBORUS must first be converted to CircAST input format, and the same GTF must be used in alignment, BSJ detection, and assembly.

## Evidence

- [readme] Per-sample transcript counts and multi-isoform fractions: "Read counts (junction reads)"
- [abstract] CircAST quantifies isoform expression via read assignment, giving the FPKM/TPM and assigned-read columns aggregated here: "By assigning reads to specific isoforms, CircAST quantified the expression of circRNA isoforms"
- [discussion] Per-sample multi-isoform BSJ fraction is computed from EM-based maximum-likelihood assignment: "We then used a maximum likelihood estimation algorithm to calculate the expression levels."
- [discussion] Thousands of exonic circular transcripts were reconstructed and found highly diverse among cell lines or tissues: "thousands of exonic circular transcripts were successfully reconstructed. Interestingly they were found to be highly diverse among different cell lines or tissues"
- [abstract] Substantial cross-sample diversity of circular transcripts: "observed substantial diversity in circular transcripts across samples, thus suggesting that circRNA expression is highly regulated"
