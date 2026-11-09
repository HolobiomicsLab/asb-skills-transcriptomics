---
name: bulk-dna-wes-read-alignment
description: Use when raw bulk DNA WES FASTQ files (e.g., SRA SRP067248 or EGA EGAS00001001880)
  must be aligned to a reference genome to enable downstream somatic SNV calling and
  read count matrix construction.
when_to_use_negative:
- Input data are single-cell or bulk RNA-seq FASTQ files — these are aligned with
  STAR, not BWA
- Input is already an aligned BAM or a finalized SNV callset passing QC (PASS from
  FilterMutectCalls, 0/0 normal genotypes, single alternative allele, >=20 total reads
  in normal samples)
- Goal is germline heterozygous locus calling in single-cell data — the pipeline provides
  separate pre-processing scripts for that purpose
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: BWA
  role: Aligns raw bulk DNA WES FASTQ reads to the reference genome, producing sorted,
    indexed BAM files
  repo: https://zenodo.org/record/7931384
- name: Mutect2
  role: Calls candidate somatic SNVs from the BWA-aligned bulk DNA BAMs via the deposited
    variant calling script
  repo: https://zenodo.org/record/7931384
- name: SAMtools
  role: Runs mpileup in force-calling mode at finalized SNV loci to extract alternative
    and total read counts for the bulk samples
- name: Canopy2
  role: Consumes the resulting Rb and Xb bulk read count matrices for joint tumor
    phylogeny inference
  repo: https://github.com/annweideman/canopy2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:2f2cbb471c8c9aa1b29210b1485f690cbdada0a98547a43365cc762c8dbabd84
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA
  - Canopy2
  - Mutect2
  - R
  - SAMtools
  - STAR
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bulk-dna-wes-read-alignment

## Summary

Align bulk DNA whole-exome sequencing (WES) reads with BWA to produce sorted, indexed BAM files as the substrate for Mutect2 somatic SNV calling in the Canopy2 pre-processing pipeline (breast cancer SRP067248 and glioblastoma EGAS00001001880 data).

## When to use

Use when raw bulk DNA WES FASTQ files (e.g., SRA SRP067248 or EGA EGAS00001001880) must be aligned to a reference genome to enable downstream somatic SNV calling and read count matrix construction.

## When NOT to use

- Input data are single-cell or bulk RNA-seq FASTQ files — these are aligned with STAR, not BWA
- Input is already an aligned BAM or a finalized SNV callset passing QC (PASS from FilterMutectCalls, 0/0 normal genotypes, single alternative allele, >=20 total reads in normal samples)
- Goal is germline heterozygous locus calling in single-cell data — the pipeline provides separate pre-processing scripts for that purpose

## Inputs

- Raw bulk DNA WES FASTQ files (breast cancer from SRA SRP067248; glioblastoma from EGA EGAS00001001880)
- Deposited shell scripts for alignment and variant calling from Zenodo record 7931384
- Reference genome for BWA alignment

## Outputs

- Sorted, indexed BAM files of aligned bulk DNA WES reads
- Candidate somatic SNV callset from Mutect2
- Alternative (Rb) and total (Xb) read count matrices (M x S, mutations by bulk samples) saved in read_count_matrices.rda

## How to apply

Download the raw bulk DNA WES FASTQ data (breast cancer WES from SRA under SRP067248; glioblastoma WES from EGA under EGAS00001001880) and execute the deposited shell scripts from the Zenodo repository (zenodo.org/record/7931384) to align reads with BWA. The alignment step produces sorted, indexed BAM files, which are then passed to Mutect2 for candidate somatic SNV calling. After QC filtering of the SNV callset, SAMtools mpileup in force-calling mode extracts alternative (mutated-locus) and total (mutated plus benign-locus) read counts at the finalized SNV loci, which are assembled into the M x S matrices Rb and Xb (mutations by bulk samples) with mutation rows identical to those in the single-cell matrices. Rationale: consistent, sorted/indexed BAMs and identical mutation rows across bulk and single-cell data are required so Canopy2 can perform joint phylogenetic inference on matched read count inputs.

## Related tools

- **BWA** (Aligns raw bulk DNA WES FASTQ reads to the reference genome, producing sorted, indexed BAM files) — https://zenodo.org/record/7931384
- **Mutect2** (Calls candidate somatic SNVs from the BWA-aligned bulk DNA BAMs via the deposited variant calling script) — https://zenodo.org/record/7931384
- **SAMtools** (Runs mpileup in force-calling mode at finalized SNV loci to extract alternative and total read counts for the bulk samples)
- **Canopy2** (Consumes the resulting Rb and Xb bulk read count matrices for joint tumor phylogeny inference) — https://github.com/annweideman/canopy2

## Examples

```
bash run_alignment_mutect2.sh  # BWA alignment of bulk WES FASTQ + Mutect2 somatic calling (scripts from https://zenodo.org/record/7931384)
```

## Evaluation signals

- BAM files are both sorted and indexed after BWA execution
- Retained somatic SNVs meet the QC criteria: PASS from FilterMutectCalls, homozygous 0/0 normal genotypes, a single alternative allele, and at least 20 total reads in normal samples
- Rb and Xb are M x S matrices whose mutation rows are identical to those of the single-cell matrices Rs and Xs
- Read count matrices are saved as read_count_matrices.rda and loadable in R
- mpileup is run in force-calling mode at the finalized SNV loci only

## Limitations

- Stringent QC of the bulk and single-cell read count data using the pre-processing pipeline is explicitly recommended to obtain the finalized SNV callset; skipping QC risks feeding noisy SNVs to Canopy2
- No software version is stated for Canopy2, BWA, or R in the article; pipeline reproducibility depends on the deposited Zenodo scripts
- Mutect2 calls must come from matched bulk DNA; RNA-derived SNVs alone cannot substitute for the bulk WES input required by Canopy2

## Evidence

- [other] Download the raw FASTQ data: breast cancer single-cell and bulk RNA-seq from GEO under accession GSE75688, breast cancer bulk DNA WES from SRA under accession SRP067248: [evidence span withheld — anchor status: not_found]
- [other] Execute the deposited shell scripts to align the bulk DNA WES reads with BWA and the single-cell and bulk RNA-seq reads with STAR, producing sorted, indexed BAM files.: [evidence span withheld — anchor status: not_found]
- [other] During pre-processing, only somatic SNVs passing stringent QC were retained: PASS from FilterMutectCalls, homozygous 0/0 normal genotypes, a single alternative allele, and at least 20 total reads in normal samples.: [evidence span withheld — anchor status: not_found]
- [other] Run SAMtools mpileup in force-calling mode at the finalized SNV loci to extract alternative (mutated-locus) and total (mutated plus benign-locus) read counts for every single cell and bulk sample.: [evidence span withheld — anchor status: not_found]
- [discussion] Master shell scripts for the somatic and germline variant calling pipeline are available at the open-access Zenodo repository https://zenodo.org/record/7931384.: "Master shell scripts for the somatic and germline variant calling pipeline are available at the open-access Zenodo repository"
- [readme] Rb: M x T (mutation by bulk sample) matrix Alternative read counts from bulk DNA whole exome sequencing (WES).: [evidence span withheld — anchor status: not_found]
