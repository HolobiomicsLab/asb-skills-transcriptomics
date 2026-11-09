---
name: alternative-splicing-event-comparison
description: Use when CircAST_result.txt full-length circRNA assemblies exist for
  an RNase R-treated RNA-seq dataset and you need to count transcripts whose AS events
  differ from linear cognates or identify novel AS events.
when_to_use_negative:
- Input samples are not RNase R-treated — CircAST applies only to RNase R-treated
  samples where remnant linear mRNA can generate false circular transcripts, distorting
  the AS comparison
- Circular transcripts of interest are intronic or intergenic — the CircAST pipeline
  assembles only exon-annotated circular transcripts, so AS comparison against GTF-annotated
  linear cognates will miss them
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: CircAST
  role: Assembles full-length alternatively spliced circRNA transcripts via multiple
    splice graphs and the EMPC algorithm; its CircAST_result.txt exon structure is
    the basis of the AS comparison
  repo: https://github.com/xiaofengsong/CircAST
- name: CircFullSeq.py
  role: Companion script that extracts the full sequence of each circular transcript
    from the CircAST result using UCSC chromosome FASTA files, enabling direct sequence-based
    AS event inspection
  repo: https://github.com/xiaofengsong/CircAST
- name: CIRCexplorer2
  role: Optional upstream detector of back-spliced junctions feeding circRNA_list.txt
    into CircAST
- name: CIRI2
  role: Optional upstream detector of back-spliced junctions feeding circRNA_list.txt
    into CircAST
- name: UROBORUS
  role: Optional upstream detector of back-spliced junctions feeding circRNA_list.txt
    into CircAST
- name: TopHat2
  role: Aligns paired-end RNase R-treated reads to the UCSC genome index to produce
    accepted_hits.bam required by CircAST
- name: Samtools
  role: Name-sorts accepted_hits.bam and converts it to accepted_hits.sorted.sam for
    CircAST input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:bcd45447a5b242bea89a795ecef490c0b79ccae61d7e0150130b1000cb84ee76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CIRCexplorer2
  - CIRI
  - CIRI2
  - CircAST
  - CircFullSeq.py
  - Samtools
  - TopHat2
  - UROBORUS
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# alternative-splicing-event-comparison

## Summary

Compares alternative splicing (AS) events of CircAST-assembled full-length circular transcripts against their cognate linear transcripts to flag circRNAs whose internal splicing differs, and tallies novel AS events, using the CircAST_result.txt 13-column output.

## When to use

Use when CircAST_result.txt full-length circRNA assemblies exist for an RNase R-treated RNA-seq dataset and you need to count transcripts whose AS events differ from linear cognates or identify novel AS events.

## When NOT to use

- Input samples are not RNase R-treated — CircAST applies only to RNase R-treated samples where remnant linear mRNA can generate false circular transcripts, distorting the AS comparison
- Circular transcripts of interest are intronic or intergenic — the CircAST pipeline assembles only exon-annotated circular transcripts, so AS comparison against GTF-annotated linear cognates will miss them

## Inputs

- CircAST_result.txt (13-column tab-delimited circRNA transcript assembly output with exon sizes/offsets, FPKM, TPM, read counts)
- UCSC-compatible GTF gene annotation file (same file used for CircAST and upstream BSJ detection)
- circRNA junction list (circRNA_list.txt) from CIRCexplorer2/CIRI2/UROBORUS with >=10 supporting back-spliced reads
- Decompressed UCSC chromFa.tar.gz genome FASTA folder for CircFullSeq.py sequence extraction

## Outputs

- Count of full-length circular transcripts with AS events distinct from linear cognates (expected 593/3464, ~17%, in the mouse testis benchmark)
- Count of novel AS events (expected 380, matching Table S3)
- Per-transcript isoform AS structure derived from exon sizes/offsets or full sequences

## How to apply

After running CircAST on a sorted SAM from TopHat2 and a back-spliced junction list from CIRCexplorer2/CIRI2/UROBORUS (threshold -T, default 10 supporting reads), parse CircAST_result.txt, which holds one record per assembled circular transcript across 13 tab-delimited columns (chromosome, start, end, transcript name, parental gene, strand, exon count, exon sizes/offsets, junction read counts, FPKM, TPM, assigned reads). Use the exon sizes and offsets (or extracted full sequences via CircFullSeq.py) to infer each circular transcript's internal AS structure and compare it with the AS structure of its cognate linear transcript derived from the UCSC-compatible GTF annotation used by both CircAST and the upstream detector. Tally transcripts whose AS events differ from the linear cognate (in the mouse testis RNase R dataset this was 593 of 3464 assembled transcripts, ~17%) and count novel AS events absent from linear annotation (expected 380).

## Related tools

- **CircAST** (Assembles full-length alternatively spliced circRNA transcripts via multiple splice graphs and the EMPC algorithm; its CircAST_result.txt exon structure is the basis of the AS comparison) — https://github.com/xiaofengsong/CircAST
- **CircFullSeq.py** (Companion script that extracts the full sequence of each circular transcript from the CircAST result using UCSC chromosome FASTA files, enabling direct sequence-based AS event inspection) — https://github.com/xiaofengsong/CircAST
- **CIRCexplorer2** (Optional upstream detector of back-spliced junctions feeding circRNA_list.txt into CircAST)
- **CIRI2** (Optional upstream detector of back-spliced junctions feeding circRNA_list.txt into CircAST)
- **UROBORUS** (Optional upstream detector of back-spliced junctions feeding circRNA_list.txt into CircAST)
- **TopHat2** (Aligns paired-end RNase R-treated reads to the UCSC genome index to produce accepted_hits.bam required by CircAST)
- **Samtools** (Name-sorts accepted_hits.bam and converts it to accepted_hits.sorted.sam for CircAST input)

## Examples

```
python CircAST.py -L 100 -T 5 -G /home/***/circRNA/Gene/genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt; python CircFullSeq.py -F /home/**/Chromosomes_homo -I CircAST_result.txt
```

## Evaluation signals

- CircAST_result.txt exists and is 13-column tab-delimited with exactly 3464 circular transcript records for the mouse testis RNase R dataset
- 593 transcripts (of 3464) are identified whose AS events differ from their linear cognates (~17%), matching Table S3
- 380 novel AS events are tallied, matching the published count
- Exon sizes/offsets in output columns 8-9 parse consistently with column 7 (number of exons) for each record
- RT-PCR + Sanger validation benchmark: 32 of 37 randomly selected isoforms should be reproducible in wet-lab checks

## Limitations

- CircAST can be applied only to RNase R-treated samples, in which remnant linear mRNA may be left over and affect assembly performance, generating false transcripts
- The current CircAST pipeline may miss intronic or intergenic circular transcripts, limiting the scope of AS comparison to exon-annotated circular transcripts
- CircAST version number is not stated in the article, complicating exact reproducibility
- Other BSJ detection tools require output file format conversion to CircAST input format before the comparison workflow can run

## Evidence

- [abstract] identified 380 circRNAs with full-length sequences different from those of their corresponding cognate linear RNAs: "identified 380 circRNAs with full-length sequences different from those of their corresponding cognate linear RNAs"
- [results] CircAST assembled 3464 transcripts with 593 AS-different isoforms: [evidence span withheld — anchor status: not_found]
- [readme] The first 3 columns are the same with bed file format.: "The first 3 columns are the same with bed file format."
- [discussion] CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over.: "CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over."
- [readme] It is designed for extracting the full sequence of circular transctipts in the CircAST result.: "It is designed for extracting the full sequence of circular transctipts in the CircAST result."
