---
name: assembly-sensitivity-precision-f1-evaluation
description: Use when simulated RNA-seq datasets with known ground-truth circRNA isoforms
  are available and you must quantify how well an assembly tool reconstructs full-length
  circular transcripts (sensitivity, precision, F1) across depths, read lengths, or
  RI bins.
when_to_use_negative:
- Real biological datasets without simulated ground-truth isoforms — sensitivity/precision
  cannot be computed and validation must rely on RT-PCR/Sanger instead
- Samples not treated with RNase R or with substantial remnant linear mRNA, which
  can generate false transcripts and distort precision estimates
- Intronic or intergenic circRNAs, which the current CircAST pipeline may miss, biasing
  sensitivity against the tool
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: CircAST
  role: Primary assembler under evaluation; reconstructs full-length circular isoforms
    via multiple splice graphs and EMPC and estimates expression by EM read allocation
  repo: https://github.com/xiaofengsong/CircAST
- name: CIRCexplorer2
  role: Comparator assembler (Cufflinks-based mode) run on identical alignments and
    junction lists, and source of back-spliced junction lists
- name: TopHat2
  role: Aligns simulated paired-end reads to the UCSC hg19 Bowtie index with the GTF
    (-p 12, -G genes_homo.gtf)
- name: SAMtools
  role: Sorts accepted_hits.bam by read name and converts it to accepted_hits.sorted.sam
    for CircAST input
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:5c49126f66320700147194ea80f5764aec5f2b07f6494244fba8ac1d8bacc9b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CIRCexplorer2
  - CircAST
  - Cufflinks
  - SAMtools
  - TopHat2
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# assembly-sensitivity-precision-f1-evaluation

## Summary

Benchmark full-length circRNA transcript assembly by comparing assembled circular isoforms against simulation ground truth to compute sensitivity (TP/(TP+FN)), precision (TP/(TP+FP)), and F1 per dataset. Used in CircAST's simulated-data evaluation against CIRCexplorer2 and for depth/RI-stratified performance analysis.

## When to use

Use when simulated RNA-seq datasets with known ground-truth circRNA isoforms are available and you must quantify how well an assembly tool reconstructs full-length circular transcripts (sensitivity, precision, F1) across depths, read lengths, or RI bins.

## When NOT to use

- Real biological datasets without simulated ground-truth isoforms — sensitivity/precision cannot be computed and validation must rely on RT-PCR/Sanger instead
- Samples not treated with RNase R or with substantial remnant linear mRNA, which can generate false transcripts and distort precision estimates
- Intronic or intergenic circRNAs, which the current CircAST pipeline may miss, biasing sensitivity against the tool

## Inputs

- Simulated paired-end FASTQ read sets (depth and read-length series) with ground-truth circRNA isoform sequences
- UCSC hg19 Bowtie index and UCSC-compatible gene annotation GTF (genes_homo.gtf / genes.gtf)
- accepted_hits.sorted.sam (tophat2 alignments sorted by read name)
- Back-spliced junction list (circRNA_list.txt: chromosome, start, end) from CIRCexplorer2, CIRI2, or UROBORUS
- CircAST result table (chromosome, start, end, transcript name, junction read counts, FPKM, TPM, assigned reads)

## Outputs

- Per-dataset sensitivity, precision, and F1 values (e.g., assembly_sensitivity_precision_comparison.csv)
- Depth-stratified performance table (best F1 at 1.48M reads; sensitivity 85.63%-94.32%, precision 81.96%-87.55%)
- RI-stratified metrics (ri_performance_metrics.csv) with per-site RI assignments and RI-bin sensitivity/precision
- Quantification accuracy: correlation coefficients between theoretical and estimated FPKM/TPM per RI bin

## How to apply

Align each simulated paired-end FASTQ set to the UCSC hg19 Bowtie index with tophat2 (-p 12, -G genes_homo.gtf), sort accepted_hits.bam by read name with 'samtools sort -n', and convert to accepted_hits.sorted.sam via 'samtools view'. Detect back-spliced junctions with CIRCexplorer2/CIRI2/UROBORUS, converting output to CircAST's circRNA_list.txt format (chromosome, start, end), keeping the same GTF for mapping and assembly. Run 'python CircAST.py -L <read length> -T 10 -G <gtf> -F accepted_hits.sorted.sam -J <junction list>' to build multiple splice graphs and assemble circular isoforms with EMPC; run the comparator (e.g., CIRCexplorer2's Cufflinks-based mode) on the identical alignments and junction lists. Count correct full-length reconstructions against the simulated ground truth to compute sensitivity (TP/(TP+FN)) and precision (TP/(TP+FP)) per dataset, then derive F1 and stratify by sequencing depth (peaking at 1.48M reads) or relation index (RI 1-4 vs RI > 4). For quantification accuracy, correlate theoretical vs CircAST-estimated FPKM/TPM (reported Pearson correlations 0.85-0.99).

## Related tools

- **CircAST** (Primary assembler under evaluation; reconstructs full-length circular isoforms via multiple splice graphs and EMPC and estimates expression by EM read allocation) — https://github.com/xiaofengsong/CircAST
- **CIRCexplorer2** (Comparator assembler (Cufflinks-based mode) run on identical alignments and junction lists, and source of back-spliced junction lists)
- **TopHat2** (Aligns simulated paired-end reads to the UCSC hg19 Bowtie index with the GTF (-p 12, -G genes_homo.gtf))
- **SAMtools** (Sorts accepted_hits.bam by read name and converts it to accepted_hits.sorted.sam for CircAST input)

## Examples

```
python CircAST.py -L 100 -T 5 -G /home/***/circRNA/Gene/genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt
```

## Evaluation signals

- CircAST sensitivity falls within the reported 85.63%-94.32% range and precision within 81.96%-87.55% across the eight simulated datasets
- CircAST outperforms CIRCexplorer2 (reported sensitivity 74.75%-75.61%; precision 58.42%-60.42%) on both metrics
- Best F1 occurs at 1.48M simulated read depth, >=86% of circRNAs assembled at 0.37M depth, and precision >81% at 5.92M depth
- RI-stratified pattern reproduced: sensitivity above 93% for RI 1-4, dropping to about 86% for RI > 4
- Theoretical-vs-estimated expression correlations (FPKM/TPM) fall in the 0.85-0.99 range

## Limitations

- CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over
- The remnant linear mRNA in the samples can affect the performance of circular transcript assembly and generate false transcripts to some extent
- The current version of the CircAST pipeline can assemble full-length sequences and quantify exon-annotated circular transcripts, but may miss intronic or intergenic circular transcripts

## Evidence

- [abstract] Sensitivity range in simulation: "CircAST correctly assembled the full sequences of circRNAs with a sensitivity of 85.63%-94.32% and a precision of 81.96%-87.55%"
- [abstract] Quantification accuracy range: "CircAST quantified the expression of circRNA isoforms with correlation coefficients of 0.85-0.99 between theoretical and estimated values"
- [discussion] RNase R limitation: "CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over."
- [discussion] EMPC assembly algorithm: "used the EMPC algorithm on these multiple splice graphs to assemble the multiple circular transcripts"
- [readme] CLI usage: "python CircAST.py -L length -T thresh -G GTF -F accepted_hits.sorted.sam -J junction.txt"
- [readme] GTF consistency requirement: "The GTF annotation file should be the same one when running CircAST and its upstream softwares."
