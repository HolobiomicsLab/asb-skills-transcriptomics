---
name: circrna-full-length-isoform-assembly
description: Use when paired-end RNA-seq alignments and a back-splice junction list
  exist and the goal is to reconstruct full-length circular transcript sequences (including
  isoforms) and estimate FPKM/TPM expression per circRNA locus.
when_to_use_negative:
- 'Samples without RNase R treatment: CircAST can be applied only to RNase R-treated
  samples, and remnant linear mRNA can generate false assembled transcripts'
- 'Goal is intronic or intergenic circRNAs: the current pipeline assembles and quantifies
  only exon-annotated circular transcripts'
- 'Only a list of back-splice sites is needed: BSJ detectors alone suffice; CircAST
  is for full-length isoform assembly and quantification, not BSJ discovery'
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: CircAST
  role: 'Core tool: builds multiple splice graphs from BSJ signatures, assembles circular
    isoforms with EMPC, quantifies by EM maximum likelihood'
  repo: https://github.com/xiaofengsong/CircAST
- name: CircFullSeq.py
  role: Companion script to extract full sequences of circular transcripts from CircAST_result.txt
    given a UCSC fasta folder
  repo: https://github.com/xiaofengsong/CircAST
- name: TopHat2
  role: Paired-end alignment against UCSC Bowtie index producing accepted_hits.bam
    for name-sorted SAM input
- name: samtools
  role: Name-sorts accepted_hits.bam ('samtools sort -n') and converts it to SAM ('samtools
    view') as required by CircAST
- name: CIRCexplorer2
  role: Upstream BSJ detector; its junction output is converted to circRNA_list.txt
    format; also used as a Cufflinks-based assembly comparator
- name: CIRI2
  role: Alternative upstream BSJ detector producing CircAST-compatible junction lists
    (result.txt)
- name: UROBORUS
  role: Alternative upstream BSJ detector (circRNA_list.txt) usable directly with
    CircAST
provenance:
  source_task_ids:
  - task_006
  - task_008
  - task_005
  - task_007
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:e5544f99e20032295f6396e19d9a52629b99e89bd21c0a2442cda8adef969d57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CIRCexplorer2
  - CIRI
  - CIRI-AS
  - CIRI2
  - CircAST
  - CircFullSeq.py
  - Cufflinks
  - TopHat2
  - UROBORUS
  - samtools
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# circrna-full-length-isoform-assembly

## Summary

Assemble full-length alternatively spliced circular RNA isoform sequences and quantify their expression from RNA-seq using CircAST's multiple splice graphs with EM-based maximum-likelihood read allocation, downstream of BSJ detection tools (CIRCexplorer2, CIRI2, UROBORUS).

## When to use

Use when paired-end RNA-seq alignments and a back-splice junction list exist and the goal is to reconstruct full-length circular transcript sequences (including isoforms) and estimate FPKM/TPM expression per circRNA locus.

## When NOT to use

- Samples without RNase R treatment: CircAST can be applied only to RNase R-treated samples, and remnant linear mRNA can generate false assembled transcripts
- Goal is intronic or intergenic circRNAs: the current pipeline assembles and quantifies only exon-annotated circular transcripts
- Only a list of back-splice sites is needed: BSJ detectors alone suffice; CircAST is for full-length isoform assembly and quantification, not BSJ discovery

## Inputs

- Name-sorted SAM alignment of paired-end reads (accepted_hits.sorted.sam) produced by tophat2 + samtools sort -n / samtools view
- UCSC-compatible GTF gene annotation file (must match the GTF used by the upstream BSJ detector)
- circRNA back-spliced junction list (circRNA_list.txt) from CIRCexplorer2/CIRI2/UROBORUS or converted from other tools
- Read length (-L) and junction supporting-read threshold (-T, default 10)
- Decompressed UCSC chromFa.tar.gz fasta folder (only for CircFullSeq.py sequence extraction)

## Outputs

- CircAST_result.txt: 13-column tab-delimited table (chromosome, start, end, transcript name, parental gene, strand, exon number, exon sizes, exon offsets, junction read counts, FPKM, TPM, assigned reads)
- Full-length circular transcript sequences (via CircFullSeq.py) for comparison against ground-truth isoforms
- Per-locus isoform relative abundances usable for sensitivity (TP/(TP+FN)), precision (TP/(TP+FP)), F1, and Pearson correlation against theoretical expression

## How to apply

First obtain a name-sorted SAM: align paired-end reads with tophat2 against a UCSC Bowtie index using a UCSC-compatible GTF (e.g. 'tophat -o results -p 12 -G genes_homo.gtf genome reads_1.fastq reads_2.fastq'), then 'samtools sort -n' and 'samtools view' to produce accepted_hits.sorted.sam. Second, generate a back-spliced junction list with CIRCexplorer2, CIRI2, or UROBORUS (or convert other detectors' output to the CircAST circRNA_list.txt format with chromosome/start/end columns), ensuring the same GTF is used for junction detection and CircAST. Third, run 'python CircAST.py -L <read_length> -T <threshold, default 10> -G genes.gtf -F accepted_hits.sorted.sam -J junction.txt'; CircAST builds multiple splice graphs at each gene locus from BSJ signatures without prior assumptions, assembles circular isoforms with the EMPC algorithm, and allocates reads via an expectation-maximization maximum-likelihood estimation. Fourth, optionally extract full-length sequences with CircFullSeq.py against the decompressed UCSC chromFa.tar.gz fasta folder. Validate by checking the 13-column tab-delimited CircAST_result.txt (bed-like chr/start/end, transcript name, parental gene, strand, exon count/sizes/offsets, junction read counts, FPKM, TPM, assigned reads).

## Related tools

- **CircAST** (Core tool: builds multiple splice graphs from BSJ signatures, assembles circular isoforms with EMPC, quantifies by EM maximum likelihood) — https://github.com/xiaofengsong/CircAST
- **CircFullSeq.py** (Companion script to extract full sequences of circular transcripts from CircAST_result.txt given a UCSC fasta folder) — https://github.com/xiaofengsong/CircAST
- **TopHat2** (Paired-end alignment against UCSC Bowtie index producing accepted_hits.bam for name-sorted SAM input)
- **samtools** (Name-sorts accepted_hits.bam ('samtools sort -n') and converts it to SAM ('samtools view') as required by CircAST)
- **CIRCexplorer2** (Upstream BSJ detector; its junction output is converted to circRNA_list.txt format; also used as a Cufflinks-based assembly comparator)
- **CIRI2** (Alternative upstream BSJ detector producing CircAST-compatible junction lists (result.txt))
- **UROBORUS** (Alternative upstream BSJ detector (circRNA_list.txt) usable directly with CircAST)

## Examples

```
python CircAST.py -L 100 -T 5 -G /home/***/circRNA/Gene/genes.gtf -F accepted_hits.sorted.sam -J circRNA_list.txt
```

## Evaluation signals

- CircAST_result.txt exists as a 13-column tab-delimited file with columns matching the documented bed-like schema (chromosome, start, end, transcript name, parental gene, strand, exon number/sizes/offsets, junction reads, FPKM, TPM, assigned reads)
- On simulated datasets, full-length assembly sensitivity falls in the reported 85.63%-94.32% range and precision in the 81.96%-87.55% range
- Isoform quantification: per-locus Pearson correlations between theoretical and estimated relative abundances fall within the reported 0.849-0.993 range and exceed junction-read-count-based correlation
- On the RNase R-treated mouse testis dataset, assembly of 3464 full-length transcripts from 2883 BSJ circRNAs (with >=10 supporting reads via -T thresholding), including 593 transcripts with AS events distinct from linear cognates and 380 novel AS events
- Same GTF file was used for alignment, BSJ detection, and CircAST runs (README requirement); SAM is sorted by read name, not coordinate

## Limitations

- CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over; remnant linear mRNA can affect assembly performance and generate false transcripts
- The current pipeline may miss intronic or intergenic circular transcripts since it assembles only exon-annotated circular transcripts
- Other BSJ detection tools can be used only if their output is converted to the CircAST input format; Cufflinks-style linear assembly discards back-spliced reads and cannot be substituted
- Version ambiguity: README introduces v1.0.0 while pointing to a v1.0.2 archive download; the article does not state a version number

## Evidence

- [intro] CircAST uses multiple splice graphs as a circRNA model to reconstruct the full-length sequence and estimate expression levels by allocating reads to different isoforms: "we developed CircAST. This tool uses multiple splice graphs as a circRNA model to reconstruct the full-length sequence and estimate expression levels by allocating reads to different isoforms with an"
- [abstract] Sensitivity and precision ranges from simulation studies: "CircAST correctly assembled the full sequences of circRNAs with a sensitivity of 85.63%-94.32% and a precision of 81.96%-87.55%"
- [abstract] Quantification correlation range: "CircAST quantified the expression of circRNA isoforms with correlation coefficients of 0.85-0.99 between theoretical and estimated values"
- [readme] Threshold parameter documentation: "-T, --threshold
		  set threshold for back-splicing junction supporting reads(optional, default: 10)"
- [readme] Output schema specification: "The first 3 columns are the same with bed file format.
	1) Chromosome
	2) Start of circular RNA
	3) End of circular RNA"
- [discussion] RNase R and linear-mRNA limitation: "CircAST can be applied only to RNase R-treated samples, in which linear mRNA may be left over. The remnant linear mRNA in the samples can affect the performance of circular transcript assembly"
- [discussion] EMPC assembly and MLE quantification: "used the EMPC algorithm on these multiple splice graphs to assemble the multiple circular transcripts"
- [discussion] Intronic/intergenic gap: "The current version of the CircAST pipeline can assemble full-length sequences and quantify exon-annotated circular transcripts, but may miss intronic or intergenic circular transcripts."
