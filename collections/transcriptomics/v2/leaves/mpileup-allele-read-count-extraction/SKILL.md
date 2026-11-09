---
name: mpileup-allele-read-count-extraction
description: Use when you have sorted, indexed BAM files (bulk DNA WES and single-cell
  or bulk RNA-seq) plus a finalized, QC-passed SNV callset and need per-locus allele
  read counts to feed Canopy2 or a similar joint bulk/scDNA-scRNA clonal-inference
  model.
when_to_use_negative:
- The SNV callset has not yet passed the pipeline QC filters (variants failing FilterMutectCalls
  PASS, with non-0/0 normal genotypes, multiple alternative alleles, or < 20 total
  reads in the normal) — counts should be extracted only at finalized loci
- The task is germline heterozygous locus calling in single-cell data; the paper provides
  a separate germline variant calling pipeline for that purpose
- You already have Rs/Xs/Rb/Xb matrices — this skill is the pre-processing step, not
  Canopy2 inference itself
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: SAMtools
  role: mpileup run in force-calling mode at finalized SNV loci to extract alternative
    and total read counts per cell/sample
- name: Mutect2
  role: upstream somatic SNV caller whose PASS calls, after QC filtering, define the
    loci at which mpileup counts are extracted
- name: BWA
  role: aligns bulk DNA WES reads to produce the BAM files pileup is run against
- name: STAR
  role: aligns single-cell and bulk RNA-seq reads to produce the BAM files pileup
    is run against
- name: R
  role: assembles Rs/Xs/Rb/Xb matrices and saves read_count_matrices.rda consumed
    by Canopy2
  repo: https://github.com/annweideman/canopy2
- name: Canopy2
  role: downstream consumer of the Rs, Xs, Rb, Xb read count matrices
  repo: https://github.com/annweideman/canopy2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:70e8f1172367ecebcb250e01133462486b99e5ffa2ed36c4979178775321f36e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/annweideman/canopy2
  tools:
  - BWA
  - Canopy2
  - Mutect2
  - R
  - SAMtools
  - STAR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mpileup-allele-read-count-extraction

## Summary

Extracts alternative (mutated-locus) and total (mutated plus benign-locus) read counts at finalized somatic SNV loci using SAMtools mpileup in force-calling mode, and assembles them into the R^s/X^s (mutation-by-single-cell) and R^b/X^b (mutation-by-bulk) matrices required by Canopy2, saved as read_count_matrices.rda.

## When to use

Use when you have sorted, indexed BAM files (bulk DNA WES and single-cell or bulk RNA-seq) plus a finalized, QC-passed SNV callset and need per-locus allele read counts to feed Canopy2 or a similar joint bulk/scDNA-scRNA clonal-inference model.

## When NOT to use

- The SNV callset has not yet passed the pipeline QC filters (variants failing FilterMutectCalls PASS, with non-0/0 normal genotypes, multiple alternative alleles, or < 20 total reads in the normal) — counts should be extracted only at finalized loci
- The task is germline heterozygous locus calling in single-cell data; the paper provides a separate germline variant calling pipeline for that purpose
- You already have Rs/Xs/Rb/Xb matrices — this skill is the pre-processing step, not Canopy2 inference itself

## Inputs

- Sorted, indexed BAM files of bulk DNA WES reads (BWA-aligned) and single-cell/bulk RNA-seq reads (STAR-aligned)
- Finalized somatic SNV callset after QC filtering (PASS from FilterMutectCalls, 0/0 normal genotype, single alternative allele, >= 20 total reads in normal samples)
- Reference genome FASTA used for alignment
- List of finalized SNV loci for force-calling

## Outputs

- Rs: M x N matrix of alternative read counts from single-cell RNA-seq (mutations by single-cells)
- Xs: M x N matrix of total read counts (benign + mutated) from single-cell RNA-seq
- Rb: M x S matrix of alternative read counts from bulk DNA WES (mutations by bulk samples)
- Xb: M x S matrix of total read counts (benign + mutated) from bulk DNA WES
- read_count_matrices.rda combining the four matrices with identical mutation rows

## How to apply

First run the upstream pipeline (BWA alignment of bulk DNA WES, STAR alignment of single-cell/bulk RNA-seq, then Mutect2 somatic calling) and apply the QC filtering scripts so that only PASS variants from FilterMutectCalls with homozygous 0/0 normal genotypes, a single alternative allele, and at least 20 total reads in normal samples remain. Then run SAMtools mpileup in force-calling mode restricted to the finalized SNV loci, extracting alternative read counts and total (benign + mutated) read counts for every single cell and bulk sample. Assemble the M x N matrices Rs and Xs (mutations by single-cells) and the M x S matrices Rb and Xb (mutations by bulk samples) with identical mutation rows across the single-cell and bulk data, and save them as read_count_matrices.rda. Identical row ordering across the four matrices is essential because Canopy2 performs joint inference assuming mutations are matched between the single-cell and bulk inputs.

## Related tools

- **SAMtools** (mpileup run in force-calling mode at finalized SNV loci to extract alternative and total read counts per cell/sample)
- **Mutect2** (upstream somatic SNV caller whose PASS calls, after QC filtering, define the loci at which mpileup counts are extracted)
- **BWA** (aligns bulk DNA WES reads to produce the BAM files pileup is run against)
- **STAR** (aligns single-cell and bulk RNA-seq reads to produce the BAM files pileup is run against)
- **R** (assembles Rs/Xs/Rb/Xb matrices and saves read_count_matrices.rda consumed by Canopy2) — https://github.com/annweideman/canopy2
- **Canopy2** (downstream consumer of the Rs, Xs, Rb, Xb read count matrices) — https://github.com/annweideman/canopy2

## Examples

```
samtools mpileup -f ref.fa -l finalized_snvs.bed sample.bam
```

## Evaluation signals

- All four matrices share identical mutation rows (same M mutations in the same order across Rs/Xs/Rb/Xb)
- Rs and Xs are M x N (mutations by single-cells); Rb and Xb are M x S (mutations by bulk samples)
- For every mutation-cell/sample pair, alternative count <= total count
- Every retained locus satisfies the QC criteria: PASS from FilterMutectCalls, homozygous 0/0 normal genotype, single alternative allele, >= 20 total reads in normal samples
- Output saved as read_count_matrices.rda and loads cleanly into the Canopy2 R workflow as Rs, Xs, Rb, Xb

## Limitations

- Read counts are only as good as the upstream alignment and QC; the authors explicitly suggest 'stringent QC of the bulk and single-cell read count data using our pre-processing pipeline' before use
- The force-calling extraction is limited to loci with estimable counts — downstream Canopy2 filtering drops mutations with zero alternative counts across all cells and cells with zero total counts across all mutations
- The published pipeline scripts (Zenodo 7931384) cover somatic variant calling; germline heterozygous locus calling in single-cell data requires the separate germline pipeline

## Evidence

- [other] Run SAMtools mpileup in force-calling mode at the finalized SNV loci to extract alternative (mutated-locus) and total (mutated plus benign-locus) read counts for every single cell and bulk sample.: [evidence span withheld — anchor status: not_found]
- [other] Assemble the M x N matrices Rs and Xs (mutations by single-cells) and the M x S matrices Rb and Xb (mutations by bulk samples) with identical mutation rows across the single-cell and bulk data, and save them as read_count_matrices.rda.: [evidence span withheld — anchor status: not_found]
- [readme] Rs:   $M \times N$ (mutation by single-cell) matrix of alternative read counts from single-cell RNA sequencing data.: "`Rs`:   $M \times N$ (mutation by single-cell) matrix of alternative read counts from single-cell RNA sequencing data"
- [discussion] We suggest stringent QC of the bulk and single-cell read count data using our pre-processing pipeline (Fig. 6 in appendix) to obtain the finalized SNV callset.: "We suggest stringent QC of the bulk and single-cell read count data using our pre-processing pipeline (Fig. 6 in appendix) to obtain the finalized SNV callset"
- [other] only somatic SNVs passing stringent QC were retained: PASS from FilterMutectCalls, homozygous 0/0 normal genotypes, a single alternative allele, and at least 20 total reads in normal samples.: [evidence span withheld — anchor status: not_found]
