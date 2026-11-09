---
name: em-based-isoform-quantification
description: Use when RNase R-treated paired-end RNA-seq reads have been aligned (TopHat2),
  back-spliced junctions are listed from CIRCexplorer2/CIRI2/UROBORUS, and you need
  full-length circRNA isoform sequences plus per-isoform FPKM/TPM expression.
when_to_use_negative:
- 'Samples were not RNase R-treated: ''CircAST can be applied only to RNase R-treated
  samples, in which linear mRNA may be left over'''
- 'The targets are intronic or intergenic circular transcripts: CircAST ''may miss
  intronic or intergenic circular transcripts'' and quantifies only exon-annotated
  circular transcripts'
- The task is de novo full-length circRNA discovery without a BSJ list from an upstream
  detector — CircAST operates downstream of CIRCexplorer2/CIRI2/UROBORUS and requires
  their junction lists
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CircAST
  role: 'Core tool: builds multiple splice graphs from BSJ signatures, assembles circular
    isoforms with EMPC, and quantifies expression via EM maximum-likelihood read assignment'
  repo: https://github.com/xiaofengsong/CircAST
- name: TopHat2
  role: Paired-end alignment of RNA-seq reads against the UCSC genome index with the
    gene GTF to produce accepted_hits.bam
- name: Samtools
  role: Name-sorts the TopHat2 BAM ('samtools sort -n') and converts it to the SAM
    input required by CircAST
- name: CIRCexplorer2
  role: Upstream back-spliced junction detection producing the CircAST junction input
    list
- name: CIRI2
  role: Alternative upstream BSJ detector (result.txt junction list format)
- name: UROBORUS
  role: Alternative upstream BSJ detector (circ_out/denovo/annotated_circ.txt junction
    list format)
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:dadfab1fb9eb9e849607092fd55885c22e89286a983bf32a87ce7c64c7f405ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CIRCexplorer2
  - CIRI
  - CIRI2
  - CircAST
  - Samtools
  - TopHat2
  - UROBORUS
  - samtools
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# EM-based isoform quantification

## Summary

CircAST-style assembly and quantification of full-length alternatively spliced circular RNA isoforms: it builds multiple splice graphs from back-splice junction (BSJ) signatures, assembles circular transcripts with the EMPC algorithm, and estimates per-isoform FPKM/TPM by assigning reads to isoforms via an expectation-maximization maximum-likelihood algorithm.

## When to use

Use when RNase R-treated paired-end RNA-seq reads have been aligned (TopHat2), back-spliced junctions are listed from CIRCexplorer2/CIRI2/UROBORUS, and you need full-length circRNA isoform sequences plus per-isoform FPKM/TPM expression.

## When NOT to use

- Samples were not RNase R-treated: 'CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over'
- The targets are intronic or intergenic circular transcripts: CircAST 'may miss intronic or intergenic circular transcripts' and quantifies only exon-annotated circular transcripts
- The task is de novo full-length circRNA discovery without a BSJ list from an upstream detector — CircAST operates downstream of CIRCexplorer2/CIRI2/UROBORUS and requires their junction lists

## Inputs

- UCSC-compatible gene annotation GTF file (same file used for alignment and BSJ detection)
- Read-name-sorted SAM file of paired-end alignments from TopHat2 (accepted_hits.sorted.sam)
- circRNA back-spliced junction list (circRNA_list.txt) from CIRCexplorer2, CIRI2, or UROBORUS, or converted from other BSJ tools
- Read length (e.g. 100) and optional BSJ supporting-read threshold (default 10)
- Decompressed UCSC chromFa.tar.gz chromosome FASTA folder (for CircFullSeq.py)

## Outputs

- CircAST_result.txt: 13-column tab-delimited table (chromosome, circular RNA start/end, circular transcript name, parental gene, strand, number of exons, exon sizes, exon offsets, junction read counts, FPKM, TPM, assigned reads)
- Full-length circular transcript sequences via CircFullSeq.py
- Per-isoform expression (FPKM/TPM) enabling AS-event comparison to cognate linear transcripts

## How to apply

Align paired-end FASTQ with TopHat2 against a UCSC Bowtie index using the gene GTF (-p 12), then name-sort the accepted_hits.bam with 'samtools sort -n' and convert to SAM via 'samtools view'. Detect BSJs with CIRCexplorer2, CIRI2, or UROBORUS and format the output as circRNA_list.txt, ensuring the same UCSC-compatible GTF used in alignment is supplied to both the BSJ caller and CircAST. Run CircAST with the read length (-L), an optional BSJ supporting-read threshold (-T, default 10), the GTF (-G), the sorted SAM (-F), and the junction list (-J); CircAST constructs multiple splice graphs per gene locus from BSJ signatures without prior assumptions and applies the EMPC algorithm plus maximum-likelihood read assignment to assemble circular isoforms and quantify them. Inspect the tab-delimited CircAST_result.txt (13 columns: chromosome, start, end, transcript name, parental gene, strand, exon count/sizes/offsets, junction read counts, FPKM, TPM, assigned reads) and use CircFullSeq.py with the UCSC chromFa folder to extract full circular transcript sequences. On the RNase R-treated mouse testis data (GSA: CRA002302) this yielded 3464 full-length circular transcripts from 2883 BSJs at the default threshold; in simulations assembly sensitivity was 85.63%-94.32% and precision 81.96%-87.55%, with isoform quantification correlations of 0.85-0.99.

## Related tools

- **CircAST** (Core tool: builds multiple splice graphs from BSJ signatures, assembles circular isoforms with EMPC, and quantifies expression via EM maximum-likelihood read assignment) — https://github.com/xiaofengsong/CircAST
- **TopHat2** (Paired-end alignment of RNA-seq reads against the UCSC genome index with the gene GTF to produce accepted_hits.bam)
- **Samtools** (Name-sorts the TopHat2 BAM ('samtools sort -n') and converts it to the SAM input required by CircAST)
- **CIRCexplorer2** (Upstream back-spliced junction detection producing the CircAST junction input list)
- **CIRI2** (Alternative upstream BSJ detector (result.txt junction list format))
- **UROBORUS** (Alternative upstream BSJ detector (circ_out/denovo/annotated_circ.txt junction list format))

## Examples

```
python CircAST.py -L 100 -T 5 -G /home/***/circRNA/Gene/genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt
```

## Evaluation signals

- CircAST_result.txt exists and is a 13-column tab-delimited file whose rows include chromosome, start, end, transcript name, parental gene, strand, exon structure, junction reads, FPKM, TPM, and assigned reads
- On mouse testis RNase R data (GSA: CRA002302), 3464 full-length circular transcripts are assembled from 2883 back-spliced circRNAs at the default threshold, with 593 transcripts whose AS events differ from cognate linear RNAs and 380 novel AS events (Table S3)
- Per-sample multi-isoform BSJ fractions across human cell line datasets fall within the reported 7%-22% range, with reconstructed exonic circular transcripts numbering 5253, 4533, and 11412 (from 4549, 4201, 9982 BSJs)
- Simulated-data invariants: full-sequence assembly sensitivity 85.63%-94.32% and precision 81.96%-87.55%; quantification correlation 0.85-0.99 between theoretical and estimated expression
- RT-PCR/Sanger validation benchmark: ~86% of randomly selected isoforms (32/37) confirm the assembled sequences

## Limitations

- CircAST can be applied only to RNase R-treated samples, and remnant linear mRNA in samples can affect assembly performance and generate false transcripts
- The current CircAST pipeline may miss intronic or intergenic circular transcripts; it handles only exon-annotated circular transcripts
- The same GTF annotation file must be used when running CircAST and its upstream software; other BSJ detection tools require output conversion to the CircAST input format
- CircAST is implemented in Python 2 under Linux (Anaconda 2.5.0+ / Python 2.7 recommended); a version number is not stated in the article

## Evidence

- [abstract] EM read assignment for isoform expression: "By assigning reads to specific isoforms, CircAST quantified the expression of circRNA isoforms"
- [discussion] Multiple splice graphs and EMPC assembly: "used the EMPC algorithm on these multiple splice graphs to assemble the multiple circular transcripts"
- [discussion] Maximum likelihood quantification: "We then used a maximum likelihood estimation algorithm to calculate the expression levels."
- [discussion] Splice graph construction from BSJ signatures: "We constructed multiple splice graphs at each gene locus on the basis of the BSJ signatures of sequencing data without relying on any prior assumptions"
- [readme] CircAST CLI options and threshold: "python CircAST.py -L length -T thresh -G GTF -F accepted_hits.sorted.sam -J junction.txt"
- [readme] Default BSJ supporting-read threshold: "set threshold for back-splicing junction supporting reads(optional, default: 10)"
