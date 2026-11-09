---
name: sam-bam-name-sorting-and-conversion
description: Use when a CircAST-style circular isoform assembly pipeline has produced
  accepted_hits.bam from TopHat2 paired-end alignment and the downstream assembly
  tool requires a read-name-sorted SAM file.
when_to_use_negative:
- 'Do not use position-sorted BAMs directly: CircAST requires the SAM sorted by read
  name, not coordinate.'
- Not needed for workflows that consume TopHat2 BAM directly without name-sorted SAM
  input (e.g. the upstream BSJ detectors alone).
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0080
tools:
- name: samtools
  role: name-sorts the TopHat2 accepted_hits.bam ('samtools sort -n') and converts
    the sorted BAM to SAM ('samtools view') for CircAST input
- name: TopHat
  role: produces the accepted_hits.bam paired-end alignment that is subsequently name-sorted
    and converted
- name: CircAST
  role: downstream consumer of the name-sorted SAM file supplied via the -F option
  repo: https://github.com/xiaofengsong/CircAST
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2019.03.004
    title: 'CircAST: Full-Length Assembly and Quantification of Alternatively Spliced
      Isoforms in Circular RNAs'
  source_content_hash: sha256:d8b8fdac79f08a7e3d7e4ac19474f99ab3e49ff7fde7d2044ea65310b43a8ff1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaofengsong/CircAST
  tools:
  - CIRCexplorer2
  - CIRI2
  - CircAST
  - TopHat
  - UROBORUS
  - samtools
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# sam-bam-name-sorting-and-conversion

## Summary

Prepare TopHat2 alignments for CircAST by sorting accepted_hits.bam by read name with samtools and converting it to a SAM file, since CircAST requires name-sorted paired-end SAM input.

## When to use

Use when a CircAST-style circular isoform assembly pipeline has produced accepted_hits.bam from TopHat2 paired-end alignment and the downstream assembly tool requires a read-name-sorted SAM file.

## When NOT to use

- Do not use position-sorted BAMs directly: CircAST requires the SAM sorted by read name, not coordinate.
- Not needed for workflows that consume TopHat2 BAM directly without name-sorted SAM input (e.g. the upstream BSJ detectors alone).

## Inputs

- accepted_hits.bam produced by TopHat2 paired-end alignment
- samtools for name-sorting and BAM-to-SAM conversion

## Outputs

- accepted_hits.sorted.bam (name-sorted BAM)
- accepted_hits.sorted.sam (name-sorted SAM passed to CircAST via -F)

## How to apply

After aligning paired-end FASTQ reads with TopHat2 against a UCSC Bowtie index (e.g. 'tophat -o results -p 12 -G genes_homo.gtf genome reads_1.fastq reads_2.fastq'), take the resulting accepted_hits.bam and sort it by read name with 'samtools sort -n -o accepted_hits.sorted.bam accepted_hits.bam'. Name-sorting is required because CircAST's EM-based read assignment relies on paired-end reads of the same pair appearing together. Then convert the sorted BAM to plain text with 'samtools view accepted_hits.sorted.bam > accepted_hits.sorted.sam' and pass this file via CircAST's -F option together with the same GTF used for alignment and the upstream BSJ junction list.

## Related tools

- **samtools** (name-sorts the TopHat2 accepted_hits.bam ('samtools sort -n') and converts the sorted BAM to SAM ('samtools view') for CircAST input)
- **TopHat** (produces the accepted_hits.bam paired-end alignment that is subsequently name-sorted and converted)
- **CircAST** (downstream consumer of the name-sorted SAM file supplied via the -F option) — https://github.com/xiaofengsong/CircAST

## Examples

```
tophat -o results -p 12 -G genes_homo.gtf genome reads_1.fastq reads_2.fastq; samtools sort -n -o accepted_hits.sorted.bam accepted_hits.bam; samtools view accepted_hits.sorted.bam > accepted_hits.sorted.sam
```

## Evaluation signals

- The output file is named accepted_hits.sorted.sam and is a valid SAM (plain-text, tab-separated) rather than BAM.
- Reads from each paired-end pair are adjacent in the sorted file (name-sorted, not coordinate-sorted).
- The SAM file is accepted by 'python CircAST.py -L <read_length> -T <threshold> -G genes.gtf -F accepted_hits.sorted.sam -J junction.txt' without format errors.
- The same UCSC-compatible GTF used in the TopHat2 alignment step is used downstream, keeping annotations consistent.

## Limitations

- The sorting order is by read name ('-n'); coordinate sorting will not satisfy CircAST's paired-end read pairing assumption.
- CircAST works with TopHat2-generated alignments; alignment BAMs from other spliced aligners are not described as supported.

## Evidence

- [methods] samtools sort -n -o accepted_hits.sorted.bam accepted_hits.bam: "samtools sort -n -o accepted_hits.sorted.bam accepted_hits.bam"
- [methods] samtools view sorted bam: "samtools view accepted_hits.sorted.bam > accepted_hits.sorted.sam"
- [readme] README: name-sorted SAM requirement: "SAM file sotsorted by name can be obtained by the following commands"
- [readme] README: accepted_hits.bam must be name-sorted first: "The accepted_hits.bam file generated by tophat2 should be sorted by name first, then be converted to accepted_hits.sorted.sam file."
