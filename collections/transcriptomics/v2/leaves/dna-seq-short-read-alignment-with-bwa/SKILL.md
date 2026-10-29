---
name: dna-seq-short-read-alignment-with-bwa
description: Use when raw DNA-seq/WES FASTQ read groups from (sample, seqtype, group)
  rows of a Snakemake sample sheet must be mapped to a reference genome before SNV/indel
  calling or neoantigen variant detection.
when_to_use_negative:
- 'The input is RNA-seq requiring chimeric-aware spliced alignment (STAR with chimSegmentMin:
  20 is used for RNA-seq read groups in this workflow, with a BWA ''realign'' step
  only to convert CIGARs)'
- The input is already an aligned BAM or CRAM — skip to the postproc_bam_index/variant-calling
  stages instead of re-aligning
- The goal is HLA genotyping rather than whole-genome variant discovery — HLA reads
  are filtered with yara and aligned to HLA alleles with Optitype, not aligned to
  the reference genome
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BWA
  role: 'BWA-MEM aligns each DNA-seq read group directly to the reference genome under
    the global mapq: 30 setting; its CIGAR convention is also the target of the RNA
    ''realign'' rule because transIndel and ScanExitron require BWA-style CIGARs'
- name: Snakemake
  role: Drives the fan-out of independent per-group alignment branches for every (sample,
    seqtype, group) row and the postproc_bam_index rule that writes .bai files
- name: ScanNeo2
  role: Host workflow containing the DNA-seq alignment and BAM-indexing rules
  repo: https://github.com/ylab-hi/ScanNeo2/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad659
    title: 'ScanNeo2: a comprehensive workflow for neoantigen detection and immunogenicity
      prediction from diverse genomic and transcriptomic alterations'
  source_content_hash: sha256:eef2f2ad9fe0df662b5cccf4450ef818c440b3c5e0cc0974c338b415bf48d0ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ylab-hi/ScanNeo2
  tools:
  - BWA
  - STAR
  - ScanNeo2
  - Snakemake
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# DNA-seq short-read alignment with BWA

## Summary

Align DNA-seq (e.g., WES) read groups against the ENSEMBL reference genome with BWA-MEM, applying a global mapping-quality setting of mapq: 30, so the sorted BAMs feed downstream somatic variant calling (GATK Mutect2/haplotypecaller, transIndel) in the ScanNeo2 neoantigen workflow.

## When to use

Use when raw DNA-seq/WES FASTQ read groups from (sample, seqtype, group) rows of a Snakemake sample sheet must be mapped to a reference genome before SNV/indel calling or neoantigen variant detection.

## When NOT to use

- The input is RNA-seq requiring chimeric-aware spliced alignment (STAR with chimSegmentMin: 20 is used for RNA-seq read groups in this workflow, with a BWA 'realign' step only to convert CIGARs)
- The input is already an aligned BAM or CRAM — skip to the postproc_bam_index/variant-calling stages instead of re-aligning
- The goal is HLA genotyping rather than whole-genome variant discovery — HLA reads are filtered with yara and aligned to HLA alleles with Optitype, not aligned to the reference genome

## Inputs

- Pre-processed (fastp-trimmed) DNA-seq/WES FASTQ read groups per (sample, seqtype, group) row of the sample sheet
- ENSEMBL reference genome build selected via the workflow's 'release' option

## Outputs

- Aligned BAM files per DNA-seq sample in results/<sample>/dnaseq/align/
- Companion .bai index files written by the postproc_bam_index rule
- Sorted, MAPQ-filtered alignments ready for GATK mutect2/haplotypecaller and transIndel variant calling

## How to apply

Launch the Snakemake workflow so independent per-group alignment branches fan out across cores/nodes, then align each DNA-seq read group directly to the reference genome with BWA-MEM, applying the global mapping-quality setting of mapq: 30. Write aligned BAMs to results/<sample>/dnaseq/align/ and have the postproc_bam_index rule produce .bai index files alongside every BAM. The aligned DNA BAMs are consumed by GATK for detection of short indels and SNVs (mutect2 and haplotypecaller) and combined with transIndel to detect long variants. Keep both alignment intermediates and final BAMs so callers can re-run without re-alignment; ScanExitron and transIndel require a BWA-style CIGAR when the same pattern is applied to realigned RNA BAMs via the 'realign' rule.

## Related tools

- **BWA** (BWA-MEM aligns each DNA-seq read group directly to the reference genome under the global mapq: 30 setting; its CIGAR convention is also the target of the RNA 'realign' rule because transIndel and ScanExitron require BWA-style CIGARs)
- **Snakemake** (Drives the fan-out of independent per-group alignment branches for every (sample, seqtype, group) row and the postproc_bam_index rule that writes .bai files)
- **ScanNeo2** (Host workflow containing the DNA-seq alignment and BAM-indexing rules) — https://github.com/ylab-hi/ScanNeo2/

## Examples

```
git clone --recurse-submodules https://github.com/ylab-hi/ScanNeo2
```

## Evaluation signals

- Every DNA-seq sample yields an aligned BAM in results/<sample>/dnaseq/align/ with a paired .bai index written by the postproc_bam_index rule
- Alignments carry the global mapping-quality setting of mapq: 30, so low-MAPQ reads are excluded downstream
- The reference used matches the ENSEMBL release set by the workflow's 'release' option, with no leftover alternative-genome rules
- Downstream GATK mutect2/haplotypecaller and transIndel run successfully against the BAMs without re-alignment
- Both alignment intermediates and final BAMs are retained so variant-calling rules can be re-executed independently

## Limitations

- The article's results text describes BWA aligning the RNA-seq data against the reference genome, while the reconstructed workflow assigns BWA-MEM to DNA-seq and uses a STAR 'realign' rule for RNA — check which branch matches your ScanNeo2 version before reusing parameters
- transIndel and ScanExitron require a BWA-style CIGAR, so a BAM produced by another aligner (e.g., STAR) must pass through the 'realign' rule before variant calling
- No specific Snakemake version is pinned in the article text; the repository badge indicates snakemake >= 9.0.0

## Evidence

- [results] Alignment of RNA-seq data against the reference genome using BWA: "the RNA-seq data are aligned against the reference genome using BWA (Li and Durbin 2009)"
- [results] Downstream variant detection consuming the alignments: [evidence span withheld — anchor status: not_found]
- [methods] Obtaining the ScanNeo2 workflow code: "git clone --recurse-submodules https://github.com/ylab-hi/ScanNeo2"
- [results] Snakemake-based workflow foundation: "We took advantage of those tools and developed a snakemake-based workflow for the detection of neoantigens from multiple sources, termed ScanNeo2"
