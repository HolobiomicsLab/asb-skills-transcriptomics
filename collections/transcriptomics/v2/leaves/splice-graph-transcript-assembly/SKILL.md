---
name: splice-graph-transcript-assembly
description: Use when RNase R-treated paired-end RNA-seq data have back-spliced junctions
  detected by CIRCexplorer2, CIRI2, or UROBORUS and you need full-length circRNA transcript
  sequences with FPKM/TPM isoform-level quantification.
when_to_use_negative:
- Samples are not RNase R-treated — CircAST can be applied only to RNase R-treated
  samples, where residual linear mRNA would otherwise confound assembly
- Target circRNAs are intronic or intergenic — the pipeline assembles and quantifies
  only exon-annotated circular transcripts
- Input is raw unaligned FASTQ without prior TopHat2 alignment and BSJ detection by
  CIRCexplorer2/CIRI2/UROBORUS (or format-converted equivalents)
edam_operation: http://edamontology.org/operation_3258
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: CircAST
  role: Constructs multiple splice graphs from BSJ signatures, assembles circular
    isoforms with the EMPC algorithm, and estimates FPKM/TPM expression via maximum-likelihood
    read allocation
  repo: https://github.com/xiaofengsong/CircAST
- name: TopHat2
  role: Paired-end alignment against UCSC genome index with -p 12 and the UCSC GTF,
    producing accepted_hits.bam
- name: Samtools
  role: Name-sorts the BAM ('samtools sort -n') and converts it to accepted_hits.sorted.sam
    for CircAST input
- name: CIRCexplorer2
  role: Upstream back-spliced junction detection supplying circRNA lists in CircAST
    input format
- name: CIRI2
  role: Upstream back-spliced junction detection supplying result.txt-format circRNA
    lists
- name: UROBORUS
  role: Upstream back-spliced junction detection supplying circRNA lists (circ_out/denovo/annotated_circ.txt)
- name: CircFullSeq.py
  role: Companion script extracting full sequences of assembled circular transcripts
    from CircAST_result.txt using UCSC chromFa FASTA files
  repo: https://github.com/xiaofengsong/CircAST
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:a94db11702bf8a24522ba9bdd33f2279e196f31a0441c23ce431ca8b93cb61cf
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
  edam_operation: http://edamontology.org/operation_3258
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# splice-graph-transcript-assembly

## Summary

Assemble full-length circular RNA isoform sequences and quantify their expression by constructing multiple splice graphs at each gene locus from back-spliced junction (BSJ) signatures, then allocating paired-end reads to isoforms with CircAST's EMPC/maximum-likelihood algorithm.

## When to use

Use when RNase R-treated paired-end RNA-seq data have back-spliced junctions detected by CIRCexplorer2, CIRI2, or UROBORUS and you need full-length circRNA transcript sequences with FPKM/TPM isoform-level quantification.

## When NOT to use

- Samples are not RNase R-treated — CircAST can be applied only to RNase R-treated samples, where residual linear mRNA would otherwise confound assembly
- Target circRNAs are intronic or intergenic — the pipeline assembles and quantifies only exon-annotated circular transcripts
- Input is raw unaligned FASTQ without prior TopHat2 alignment and BSJ detection by CIRCexplorer2/CIRI2/UROBORUS (or format-converted equivalents)

## Inputs

- GTF gene annotation file in UCSC-compatible format (same file used by upstream circRNA detectors)
- Name-sorted SAM file of paired-end TopHat2 alignments (accepted_hits.sorted.sam)
- circRNA back-spliced junction list from CIRCexplorer2, CIRI2, or UROBORUS (or converted to CircAST format), e.g. circRNA_list.txt
- Read length parameter (-L) and junction supporting-reads threshold (-T, default 10)

## Outputs

- CircAST_result.txt: 13-column tab-delimited table (chromosome, circRNA start/end, transcript name, parent gene, strand, exon count, exon sizes, exon offsets, junction read counts, FPKM, TPM, assigned read counts) per circular transcript
- Full-length circular transcript sequences via CircFullSeq.py (requires UCSC chromFa.tar.gz decompressed FASTA folder)

## How to apply

Align paired-end reads with TopHat2 (-p 12) against a UCSC genome index using a UCSC-compatible GTF, then sort the accepted_hits.bam by read name ('samtools sort -n') and convert to SAM for CircAST input. Obtain the back-spliced junction list from CIRCexplorer2, CIRI2, or UROBORUS (or convert another detector's output to CircAST's circRNA_list.txt format), keeping the same GTF used upstream. Run CircAST, which constructs multiple splice graphs at each circRNA locus from BSJ signatures without prior assumptions, assembles circular transcripts with the EMPC algorithm, and estimates expression via maximum likelihood (EM) read allocation. Set -L to the read length (e.g. 100) and -T to the back-splicing junction supporting-reads threshold (default 10; the mouse testis run used T=5) to control which BSJs enter assembly. Judge success by the 13-column tab-delimited CircAST_result.txt (chromosome, start, end, transcript name, parent gene, strand, exon count/sizes/offsets, junction read counts, FPKM, TPM, assigned reads), optionally extracting full sequences with CircFullSeq.py against UCSC chromFa files.

## Related tools

- **CircAST** (Constructs multiple splice graphs from BSJ signatures, assembles circular isoforms with the EMPC algorithm, and estimates FPKM/TPM expression via maximum-likelihood read allocation) — https://github.com/xiaofengsong/CircAST
- **TopHat2** (Paired-end alignment against UCSC genome index with -p 12 and the UCSC GTF, producing accepted_hits.bam)
- **Samtools** (Name-sorts the BAM ('samtools sort -n') and converts it to accepted_hits.sorted.sam for CircAST input)
- **CIRCexplorer2** (Upstream back-spliced junction detection supplying circRNA lists in CircAST input format)
- **CIRI2** (Upstream back-spliced junction detection supplying result.txt-format circRNA lists)
- **UROBORUS** (Upstream back-spliced junction detection supplying circRNA lists (circ_out/denovo/annotated_circ.txt))
- **CircFullSeq.py** (Companion script extracting full sequences of assembled circular transcripts from CircAST_result.txt using UCSC chromFa FASTA files) — https://github.com/xiaofengsong/CircAST

## Examples

```
python CircAST.py -L 100 -T 5 -G /home/***/circRNA/Gene/genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt
```

## Evaluation signals

- CircAST_result.txt exists with 13 tab-delimited columns matching the documented output schema
- On the adult mouse testis RNase R dataset (GSA: CRA002302), CircAST assembles 3464 full-length circRNA transcripts from 2883 back-spliced circRNAs at ≥10 supporting back-spliced reads
- 593 of 3464 transcripts (~17%) show AS events distinct from their linear cognate transcripts and 380 novel AS events are tallied, matching Table S3
- In simulation, per-locus Pearson correlations between theoretical and estimated isoform abundances fall within 0.849–0.993 and exceed junction-read-count-only quantification
- RT-PCR/Sanger validation benchmark: 32 of 37 randomly selected isoforms validate, providing an expected ~86% validation rate

## Limitations

- CircAST can be applied only to RNase R-treated samples; remnant linear mRNA can affect assembly performance and generate false transcripts
- The current pipeline assembles and quantifies exon-annotated circular transcripts only and may miss intronic or intergenic circular transcripts
- Other BSJ detection tools require output file format conversion to CircAST input format before use

## Evidence

- [abstract] multiple splice graphs: "we developed CircAST, a tool to assemble alternatively spliced circRNA transcripts and estimate their expression by using multiple splice graphs"
- [discussion] EMPC on splice graphs: "used the EMPC algorithm on these multiple splice graphs to assemble the multiple circular transcripts"
- [methods] tophat2 command: "tophat -o results -p 12 -G /home/***/genes_homo.gtf /home/***/index/Homo_sapiens/UCSC/hg19/Sequence/BowtieIndex/genome reads_1.fastq reads_2.fastq"
- [readme] threshold option: "set threshold for back-splicing junction supporting reads(optional, default: 10)"
- [discussion] RNase R restriction: "CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over."
- [readme] compatible detectors: "CircAST is compatible with popular upstream circRNA detection tools (CIRCexplorer2, CIRI2, UROBORUS)."
