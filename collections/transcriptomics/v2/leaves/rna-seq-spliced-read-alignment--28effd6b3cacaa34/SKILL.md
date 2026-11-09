---
name: rna-seq-spliced-read-alignment
description: Use when paired-end RNA-seq FASTQ files must be spliced-aligned before
  circRNA full-length assembly, e.g. SRA/GSA reads destined for CircAST, CIRCexplorer2/CIRI2/UROBORUS
  junction detection.
when_to_use_negative:
- The task is single-end or non-polyA/linear-transcript-only expression quantification
  — CircAST and this pipeline expect paired-end reads and, ideally, RNase R-treated
  samples
- The reads target intronic/intergenic circular RNAs outside annotated genes — the
  CircAST pipeline is exon-annotation-dependent and would miss them
- A splice-aware aligner other than TopHat2 is required (CircAST's documented input
  is the TopHat2-generated accepted_hits alignment name-sorted by samtools)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: TopHat
  role: Spliced-aligns paired-end reads to the UCSC Bowtie genome index using the
    gene GTF, producing accepted_hits.bam
- name: samtools
  role: Name-sorts accepted_hits.bam (samtools sort -n) and converts it to accepted_hits.sorted.sam
    for CircAST input
- name: CircAST
  role: Downstream consumer of the name-sorted SAM (-F), GTF (-G), and junction list
    (-J) for full-length circRNA assembly and EM-based quantification
  repo: https://github.com/xiaofengsong/CircAST
- name: CIRCexplorer2
  role: Alternative upstream BSJ detector run on the same alignments to produce the
    CircAST junction list (annotated_circ.txt)
- name: CIRI2
  role: Alternative upstream BSJ detector run on the same alignments to produce the
    CircAST junction list (result.txt)
- name: UROBORUS
  role: Alternative upstream BSJ detector run on the same alignments to produce the
    CircAST junction list (circRNA_list.txt)
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:f7b60f7d15c9601db7fc0d9cafd191c61532b2999fe7b928d8faaaaead144a8b
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
  - TopHat
  - UROBORUS
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# rna-seq-spliced-read-alignment

## Summary

Spliced alignment of paired-end RNA-seq FASTQ reads against a UCSC Bowtie genome index using TopHat2 with a gene GTF, followed by name-sorting with samtools, to prepare the accepted_hits alignments that CircAST needs for full-length circRNA isoform assembly and quantification.

## When to use

Use when paired-end RNA-seq FASTQ files must be spliced-aligned before circRNA full-length assembly, e.g. SRA/GSA reads destined for CircAST, CIRCexplorer2/CIRI2/UROBORUS junction detection.

## When NOT to use

- The task is single-end or non-polyA/linear-transcript-only expression quantification — CircAST and this pipeline expect paired-end reads and, ideally, RNase R-treated samples
- The reads target intronic/intergenic circular RNAs outside annotated genes — the CircAST pipeline is exon-annotation-dependent and would miss them
- A splice-aware aligner other than TopHat2 is required (CircAST's documented input is the TopHat2-generated accepted_hits alignment name-sorted by samtools)

## Inputs

- Paired-end RNA-seq FASTQ files (reads_1.fastq, reads_2.fastq), e.g. from NCBI SRA or GSA CRA002302
- UCSC Bowtie genome index for the species (hg19 Homo_sapiens, mouse, or chicken)
- UCSC-compatible gene annotation GTF (e.g. genes_homo.gtf from hgdownload.soe.ucsc.edu)
- tophat2 and samtools executables

## Outputs

- accepted_hits.bam from TopHat2 (tophat -o results directory)
- Name-sorted accepted_hits.sorted.bam (samtools sort -n)
- accepted_hits.sorted.sam (name-sorted SAM) consumed by CircAST (-F) and BSJ detectors

## How to apply

Align each paired-end FASTQ sample with TopHat2 against the species-specific UCSC Bowtie index (e.g. /index/Homo_sapiens/UCSC/hg19/Sequence/BowtieIndex/genome), supplying the same UCSC-compatible gene GTF with -p 12 threads (e.g. 'tophat -o results -p 12 -G genes_homo.gtf genome reads_1.fastq reads_2.fastq'). Then name-sort the accepted_hits.bam with 'samtools sort -n -o accepted_hits.sorted.bam accepted_hits.bam' and convert it to text with 'samtools view accepted_hits.sorted.bam > accepted_hits.sorted.sam', because CircAST requires a name-sorted SAM as its -F input. Critical decision point: the GTF used here must be identical to the one used by CircAST and the upstream BSJ detectors (CIRCexplorer2, CIRI2, UROBORUS), since splice-graph construction depends on consistent exon annotation. Repeat per sample (e.g. HeLa SRR1636985/SRR1636986/SRR3476956, HEK293 SRR3479244, Hs68 SRR444974, chicken SRR7350933) and per read length, which is later passed to CircAST via -L.

## Related tools

- **TopHat** (Spliced-aligns paired-end reads to the UCSC Bowtie genome index using the gene GTF, producing accepted_hits.bam)
- **samtools** (Name-sorts accepted_hits.bam (samtools sort -n) and converts it to accepted_hits.sorted.sam for CircAST input)
- **CircAST** (Downstream consumer of the name-sorted SAM (-F), GTF (-G), and junction list (-J) for full-length circRNA assembly and EM-based quantification) — https://github.com/xiaofengsong/CircAST
- **CIRCexplorer2** (Alternative upstream BSJ detector run on the same alignments to produce the CircAST junction list (annotated_circ.txt))
- **CIRI2** (Alternative upstream BSJ detector run on the same alignments to produce the CircAST junction list (result.txt))
- **UROBORUS** (Alternative upstream BSJ detector run on the same alignments to produce the CircAST junction list (circRNA_list.txt))

## Examples

```
tophat -o results -p 12 -G genes_homo.gtf /home/***/index/Homo_sapiens/UCSC/hg19/Sequence/BowtieIndex/genome reads_1.fastq reads_2.fastq; samtools sort -n -o accepted_hits.sorted.bam accepted_hits.bam; samtools view accepted_hits.sorted.bam > accepted_hits.sorted.sam
```

## Evaluation signals

- accepted_hits.bam exists in the tophat -o results directory and is non-empty for every sample processed
- The SAM header/order confirms reads are sorted by name (QNAME), not coordinate, since CircAST requires 'SAM of pair-end reads ... sorted by name'
- The same GTF file (identical path/content) was used for tophat2, the BSJ detector, and CircAST, per the README note that 'The GTF annotation file should be the same one when running CircAST and its upstream softwares'
- Downstream CIRI2/CIRCexplorer2/UROBORUS returns a parseable back-spliced junction list in CircAST format, and CircAST run completes and emits the 13-column bed-like output (chromosome, start, end, transcript name, parental gene, strand, exon number/sizes/offsets, junction read counts, FPKM, TPM, assigned reads)
- Per-sample CircAST results reproduce reported scales, e.g. 5253/4533/11412 exonic circular transcripts in the three human cell line datasets and multi-isoform BSJ fractions within the reported 7%-22% range

## Limitations

- CircAST is best applied to RNase R-treated samples; remnant linear mRNA can affect circular transcript assembly and generate false transcripts
- The pipeline may miss intronic or intergenic circular transcripts because it assembles exon-annotated circular transcripts only
- GTF format must be UCSC-compatible, and mixing GTF versions between alignment and assembly steps invalidates splice-graph construction
- Output junction lists from BSJ detectors other than CIRCexplorer2/CIRI2/UROBORUS must be converted into CircAST input format before use

## Evidence

- [methods] paired-end alignment with tophat2: "tophat -o results -p 12 -G /home/***/genes_homo.gtf /home/***/index/Homo_sapiens/UCSC/hg19/Sequence/BowtieIndex/genome reads_1.fastq reads_2.fastq"
- [methods] sort and convert BAM to SAM: "samtools sort -n -o accepted_hits.sorted.bam accepted_hits.bam"
- [readme] CircAST input requirement from README: "a sorted Sequence Alignment/Map (SAM) of pair-end reads generated by tophat2 and samtools"
- [readme] GTF consistency note: "The GTF annotation file should be the same one when running CircAST and its upstream softwares."
- [discussion] downstream circRNA assembly scale: "thousands of exonic circular transcripts were successfully reconstructed. Interestingly they were found to be highly diverse among different cell lines or tissues"
