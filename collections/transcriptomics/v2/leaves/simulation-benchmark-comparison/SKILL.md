---
name: simulation-benchmark-comparison
description: Use when evaluating a circRNA full-length assembly/quantification tool
  against simulated paired-end FASTQ reads with known ground-truth circular isoforms,
  to characterize how sequencing depth or read length affects assembly sensitivity,
  precision, and F1.
when_to_use_negative:
- Input is real experimental RNA-seq data without ground-truth circular isoform sequences
  — sensitivity/precision cannot be computed as in this simulation benchmark
- 'Data are not RNase R-treated: the article states CircAST can be applied only to
  RNase R-treated samples, so simulated/real benchmarks must model rRNA-depleted,
  RNase R-treated conditions'
- The goal is circRNA detection (back-splice site identification) rather than full-length
  isoform assembly — use upstream detectors like CIRI2 or CIRCexplorer2 instead of
  benchmarking CircAST assembly output
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: CircAST
  role: Full-length circRNA isoform assembly and quantification via multiple splice
    graphs and the EMPC algorithm; output is benchmarked against simulation ground
    truth
  repo: https://github.com/xiaofengsong/CircAST
- name: TopHat2
  role: Paired-end alignment of simulated FASTQ reads against the UCSC hg19 Bowtie
    index prior to assembly
- name: SAMtools
  role: Sorts accepted_hits.bam by read name and converts it to SAM format required
    by CircAST
- name: CIRCexplorer2
  role: Optional upstream detector supplying the circRNA back-spliced junction list
    input to CircAST
- name: CIRI2
  role: Optional upstream detector supplying the circRNA back-spliced junction list
    input to CircAST
- name: UROBORUS
  role: Optional upstream detector supplying the circRNA back-spliced junction list
    input to CircAST
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:3a3b57f6c14ba8dd5240ff9537e1921288a9a13f8fcb0e44e85d2bd4b2af2321
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CIRCexplorer2
  - CIRI2
  - CircAST
  - SAMtools
  - TopHat2
  - UROBORUS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# simulation-benchmark-comparison

## Summary

Benchmark full-length circRNA transcript assembly (CircAST with the EMPC algorithm) on simulated paired-end RNA-seq datasets, comparing assembled isoforms against simulation ground truth to compute sensitivity, precision, and F1 across sequencing depths and read lengths.

## When to use

Use when evaluating a circRNA full-length assembly/quantification tool against simulated paired-end FASTQ reads with known ground-truth circular isoforms, to characterize how sequencing depth or read length affects assembly sensitivity, precision, and F1.

## When NOT to use

- Input is real experimental RNA-seq data without ground-truth circular isoform sequences — sensitivity/precision cannot be computed as in this simulation benchmark
- Data are not RNase R-treated: the article states CircAST can be applied only to RNase R-treated samples, so simulated/real benchmarks must model rRNA-depleted, RNase R-treated conditions
- The goal is circRNA detection (back-splice site identification) rather than full-length isoform assembly — use upstream detectors like CIRI2 or CIRCexplorer2 instead of benchmarking CircAST assembly output

## Inputs

- Simulated paired-end FASTQ files spanning sequencing depths (e.g., 0.37M–5.92M reads) and read-length series
- UCSC hg19 Bowtie genome index
- UCSC-compatible gene annotation GTF (genes.gtf)
- circRNA back-spliced junction list (e.g., circRNA_list.txt from CIRCexplorer2/CIRI2/UROBORUS)
- Simulation ground-truth circular isoform sequences

## Outputs

- Per-dataset full-length circular transcript assembly results with isoform sequences, junction read counts, FPKM/TPM expression levels
- Sensitivity, precision, and F1 scores per simulated dataset
- Depth-response and read-length response curves showing where assembly performance peaks (best F1 at 1.48M reads)

## How to apply

Align each simulated paired-end FASTQ dataset against the UCSC hg19 Bowtie index with TopHat2 using a gene annotation GTF ('tophat -o results -p 12 -G genes.gtf'), then sort accepted_hits.bam by read name with 'samtools sort -n' and convert to SAM with 'samtools view'. Obtain back-spliced junction lists from an upstream detector (CIRCexplorer2, CIRI2, or UROBORUS), ensuring the GTF matches the one used for mapping, since CircAST requires the same UCSC-compatible GTF for both upstream detection and assembly. Run CircAST with the read length (-L), junction-support threshold (-T, default 10), GTF, name-sorted SAM, and junction list; it builds multiple splice graphs, assembles circular isoforms with the EMPC algorithm, and quantifies expression by maximum-likelihood read assignment. Compare assembled full-length circular transcripts against the simulation ground-truth isoforms to compute sensitivity, precision, and F1 for every dataset in the depth series (0.37M–5.92M reads) and read-length series. Compile metrics across datasets and check them against the published ranges to validate the benchmark reproduction.

## Related tools

- **CircAST** (Full-length circRNA isoform assembly and quantification via multiple splice graphs and the EMPC algorithm; output is benchmarked against simulation ground truth) — https://github.com/xiaofengsong/CircAST
- **TopHat2** (Paired-end alignment of simulated FASTQ reads against the UCSC hg19 Bowtie index prior to assembly)
- **SAMtools** (Sorts accepted_hits.bam by read name and converts it to SAM format required by CircAST)
- **CIRCexplorer2** (Optional upstream detector supplying the circRNA back-spliced junction list input to CircAST)
- **CIRI2** (Optional upstream detector supplying the circRNA back-spliced junction list input to CircAST)
- **UROBORUS** (Optional upstream detector supplying the circRNA back-spliced junction list input to CircAST)

## Examples

```
python CircAST.py -L 100 -T 5 -G genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt
```

## Evaluation signals

- Sensitivity of full-length assembly falls within the reported 85.63%-94.32% range across simulated datasets
- Precision falls within the reported 81.96%-87.55% range across simulated datasets
- Best F1 score occurs at the simulated depth of 1.48 million paired-end reads
- At least 86% of circRNAs are correctly assembled at 0.37M depth, and precision exceeds 81% even at 5.92M depth
- Isoform expression quantification shows correlation coefficients of 0.85-0.99 between theoretical and estimated values

## Limitations

- CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over; remnant linear mRNA can affect assembly performance and generate false transcripts
- The current CircAST pipeline assembles and quantifies only exon-annotated circular transcripts and may miss intronic or intergenic circular transcripts
- Other BSJ detection tools can be used only after converting their output to the CircAST input file format; the GTF annotation must be identical between CircAST and upstream software

## Evidence

- [abstract] Reported sensitivity/precision ranges validating the benchmark: "CircAST correctly assembled the full sequences of circRNAs with a sensitivity of 85.63%-94.32% and a precision of 81.96%-87.55%"
- [methods] TopHat2 alignment command for simulated paired-end reads: "tophat -o results -p 12 -G /home/***/genes_homo.gtf /home/***/index/Homo_sapiens/UCSC/hg19/Sequence/BowtieIndex/genome reads_1.fastq reads_2.fastq"
- [readme] SAM preprocessing required by CircAST: "samtools sort -n -o accepted_hits.sorted.bam accepted_hits.bam"
- [readme] GTF consistency requirement between mapping, detection, and assembly: "The GTF annotation file should be the same one when running CircAST and its upstream softwares."
- [abstract] Quantification accuracy expected in simulation: "CircAST quantified the expression of circRNA isoforms with correlation coefficients of 0.85-0.99 between theoretical and estimated values"
