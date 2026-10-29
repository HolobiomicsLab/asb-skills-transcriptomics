---
name: isoform-abundance-assignment
description: Use when RNA-seq data contain circRNA back-spliced junctions detected
  by CIRCexplorer2/CIRI2/UROBORUS and you need isoform-level relative expression (read
  counts, FPKM, TPM) rather than locus-level junction counts.
when_to_use_negative:
- Samples were not RNase R-treated (article states CircAST applies only to RNase R-treated
  samples; remnant linear mRNA causes false transcripts)
- The circRNAs of interest are intronic or intergenic — the current pipeline assembles
  only exon-annotated circular transcripts
- Only locus-level junction counting is needed, e.g. for differential BSJ testing,
  where full isoform assembly adds unnecessary complexity
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: CircAST
  role: 'Core tool: builds splice graphs from BSJ signatures, assembles circular isoforms
    with EMPC, and assigns reads via maximum likelihood to produce FPKM/TPM'
  repo: https://github.com/xiaofengsong/CircAST
- name: tophat
  role: Paired-end read alignment against the UCSC hg19 Bowtie index with the same
    GTF used by CircAST (-G genes_homo.gtf)
- name: samtools
  role: Sorts the tophat BAM by read name ('samtools sort -n') and converts it to
    SAM for CircAST input
- name: CircFullSeq.py
  role: Companion script that extracts full sequences of assembled circular transcripts
    from the CircAST result using UCSC chromFa FASTA files
  repo: https://github.com/xiaofengsong/CircAST
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:83716fa0812a8c1eaa7dcf8f18d24805c08b9a212a6051469381dbb01c89d19a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CircAST
  - CircFullSeq.py
  - samtools
  - tophat
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# isoform-abundance-assignment

## Summary

Assigns paired-end RNA-seq reads to specific circular RNA isoforms using CircAST's multiple splice graphs and maximum likelihood (EMPC) algorithm, yielding per-isoform junction read counts, FPKM and TPM estimates. It addresses the fact that circRNAs at one locus can have multiple alternatively spliced isoforms sharing the same back-spliced junction.

## When to use

Use when RNA-seq data contain circRNA back-spliced junctions detected by CIRCexplorer2/CIRI2/UROBORUS and you need isoform-level relative expression (read counts, FPKM, TPM) rather than locus-level junction counts.

## When NOT to use

- Samples were not RNase R-treated (article states CircAST applies only to RNase R-treated samples; remnant linear mRNA causes false transcripts)
- The circRNAs of interest are intronic or intergenic — the current pipeline assembles only exon-annotated circular transcripts
- Only locus-level junction counting is needed, e.g. for differential BSJ testing, where full isoform assembly adds unnecessary complexity

## Inputs

- GTF gene annotation file (UCSC-compatible format, e.g. UCSC hg19 genes_homo.gtf)
- Name-sorted SAM of paired-end alignments produced by tophat2 and 'samtools sort -n' (accepted_hits.sorted.sam)
- CircRNA back-spliced junction list from CIRCexplorer2/CIRI2/UROBORUS (or converted from another detector)
- Read length (integer, e.g. 100 for 100-bp paired-end reads)

## Outputs

- Per-circular-transcript table with chromosomal coordinates, transcript and parental gene names, strand, exon count/sizes/offsets, junction read counts, assigned read counts, and FPKM/TPM expression levels
- Full-length circular transcript sequences via the CircFullSeq.py helper script (requires UCSC chromFa.tar.gz chromosome FASTA folder)

## How to apply

Run tophat2 paired-end alignment against a UCSC hg19 GTF, sort the BAM by read name with 'samtools sort -n', and convert it to SAM with 'samtools view'. Provide CircAST with three inputs: the GTF annotation, the name-sorted SAM, and a circRNA junction list from an upstream detector; set the read length (-L, e.g. 100 bp for the simulated paired-end 100-bp datasets) and optionally a BSJ-supporting-read threshold (-T, default 10; the README example uses -T 5). CircAST constructs multiple splice graphs at each gene locus from BSJ signatures, uses the EMPC algorithm to assemble alternative circular transcripts (introducing skipped-exon variants), then allocates reads to isoforms with a maximum likelihood expectation–maximization approach. Validation in the paper used simulated data from 4 human loci (NOLC1, PAFAH1B2, DHPS, EXOSC10) with 10 isoforms under 8 conditions, achieving theoretical-vs-estimated correlation coefficients of 0.85–0.99 across depths of 0.09M–5.92M read pairs.

## Related tools

- **CircAST** (Core tool: builds splice graphs from BSJ signatures, assembles circular isoforms with EMPC, and assigns reads via maximum likelihood to produce FPKM/TPM) — https://github.com/xiaofengsong/CircAST
- **tophat** (Paired-end read alignment against the UCSC hg19 Bowtie index with the same GTF used by CircAST (-G genes_homo.gtf))
- **samtools** (Sorts the tophat BAM by read name ('samtools sort -n') and converts it to SAM for CircAST input)
- **CircFullSeq.py** (Companion script that extracts full sequences of assembled circular transcripts from the CircAST result using UCSC chromFa FASTA files) — https://github.com/xiaofengsong/CircAST

## Examples

```
python CircAST.py -L 100 -T 5 -G /home/***/circRNA/Gene/genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt
```

## Evaluation signals

- Output table has all 13 columns (chromosome, start, end, transcript name, parental gene, strand, exon count, exon sizes, offsets, junction read counts, FPKM, TPM, assigned reads) tab-delimited
- Quantification accuracy: correlation between theoretical and estimated abundances falls in the reported 0.85–0.99 range on simulated datasets with known ground truth
- Sum of assigned reads per locus is consistent with available junction-spanning and alignment evidence; EM allocation respects the splice graph structure
- Assembly quality matches simulation benchmarks: sensitivity 85.63%–94.32% and precision 81.96%–87.55% for full-length sequences
- The junction list, GTF, and SAM derive from the same reference annotation (README requires the same GTF for CircAST and upstream software)

## Limitations

- Only applicable to RNase R-treated samples; remnant linear mRNA can degrade assembly performance and generate false transcripts
- Cannot assemble intronic or intergenic circular transcripts — only exon-annotated ones
- Other BSJ detection tools require output conversion to CircAST's junction-list input format
- CircAST is implemented in Python 2 (Anaconda 2.5.0+, Python 2.7 recommended), which may pose environment constraints

## Evidence

- [abstract] EM allocation of reads to isoforms: "we developed CircAST. This tool uses multiple splice graphs as a circRNA model to reconstruct the full-length sequence and estimate expression levels by allocating reads to different isoforms with an"
- [abstract] Quantification accuracy on simulations: "CircAST quantified the expression of circRNA isoforms with correlation coefficients of 0.85-0.99 between theoretical and estimated values"
- [methods] Methods pipeline commands: "samtools sort -n -o accepted_hits.sorted.bam accepted_hits.bam"
- [readme] Input requirements from README: "CircAST works with three input files. A GTF annotation file, a sorted Sequence Alignment/Map (SAM) of pair-end reads generated by tophat2 and samtools, and a circRNA junction list"
- [discussion] Splice graph construction and EMPC: "We constructed multiple splice graphs at each gene locus on the basis of the BSJ signatures of sequencing data without relying on any prior assumptions"
