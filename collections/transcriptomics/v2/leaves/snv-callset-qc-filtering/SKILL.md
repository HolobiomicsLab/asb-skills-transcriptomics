---
name: snv-callset-qc-filtering
description: Use when raw breast cancer (GSE75688, SRP067248) or glioblastoma (EGAS00001001880)
  bulk WES and single-cell/bulk RNA-seq FASTQ files must be converted into QC-filtered
  somatic SNV read count matrices for Canopy2 input.
when_to_use_negative:
- The read count matrices (Rs, Xs, Rb, Xb) already exist, e.g. as the packaged GBM10_preproc/GBM10_postproc
  datasets — proceed directly to Canopy2 inference
- The goal is germline heterozygous locus calling in single-cell data — the authors
  ship a separate germline variant calling pipeline for that purpose
- The data are not matched bulk DNA WES plus scRNA-seq from the same samples, since
  force-calling and 0/0-normal QC depend on paired normal/bulk DNA data
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: BWA
  role: Aligns bulk DNA WES reads to produce sorted, indexed BAMs prior to Mutect2
    somatic variant calling
- name: STAR
  role: Aligns single-cell and bulk RNA-seq reads to produce BAMs for read-count extraction
    at SNV loci
- name: Mutect2
  role: Calls candidate somatic SNVs on aligned bulk DNA and single-cell data; PASS
    calls from FilterMutectCalls are retained by QC
- name: SAMtools
  role: Runs mpileup in force-calling mode at finalized SNV loci to extract alternative
    and total read counts per cell/sample
- name: R
  role: Assembles the Rs/Xs/Rb/Xb matrices and applies the nonzero row/column filters
    before saving read_count_matrices.rda
  repo: https://github.com/annweideman/canopy2
- name: Canopy2
  role: Downstream consumer of the QC-filtered read count matrices; its input requirements
    (Rs, Xs, Rb, Xb) define the output schema of this skill
  repo: https://github.com/annweideman/canopy2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:960d177f2f7ae01ec4e68ce7013f03e2ff0ef5cc04b8ab0be9f2ee05cd7643d6
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# snv-callset-qc-filtering

## Summary

Pre-processing QC pipeline that converts raw bulk WES and scRNA-seq FASTQ data into a finalized somatic SNV callset and the alternative/total read count matrices (Rs, Xs, Rb, Xb) required by Canopy2, via Mutect2 variant calling followed by stringent filtering.

## When to use

Use when raw breast cancer (GSE75688, SRP067248) or glioblastoma (EGAS00001001880) bulk WES and single-cell/bulk RNA-seq FASTQ files must be converted into QC-filtered somatic SNV read count matrices for Canopy2 input.

## When NOT to use

- The read count matrices (Rs, Xs, Rb, Xb) already exist, e.g. as the packaged GBM10_preproc/GBM10_postproc datasets — proceed directly to Canopy2 inference
- The goal is germline heterozygous locus calling in single-cell data — the authors ship a separate germline variant calling pipeline for that purpose
- The data are not matched bulk DNA WES plus scRNA-seq from the same samples, since force-calling and 0/0-normal QC depend on paired normal/bulk DNA data

## Inputs

- Raw FASTQ files: breast cancer single-cell and bulk RNA-seq (GEO GSE75688), breast cancer bulk DNA WES (SRA SRP067248), glioblastoma RNA-seq and bulk DNA WES (EGA EGAS00001001880)
- Sorted, indexed bulk DNA and RNA-seq BAM files from BWA/STAR alignment
- Candidate somatic SNV calls from Mutect2

## Outputs

- Finalized somatic SNV callset passing QC (PASS, 0/0 normal, single alt allele, >=20 total reads in normal)
- Rs: M x N mutation-by-single-cell alternative read count matrix
- Xs: M x N mutation-by-single-cell total read count matrix
- Rb: M x S mutation-by-bulk-sample alternative read count matrix
- Xb: M x S mutation-by-bulk-sample total read count matrix
- read_count_matrices.rda file with identical mutation rows across single-cell and bulk matrices

## How to apply

Download raw FASTQ data (GSE75688 for breast cancer scRNA-seq/bulk RNA-seq, SRP067248 for breast cancer bulk DNA WES, EGAS00001001880 for glioblastoma), then run the deposited Zenodo (record 7931384) shell scripts: align bulk DNA WES reads with BWA and RNA-seq reads with STAR to sorted, indexed BAMs. Call candidate somatic SNVs with Mutect2 on the aligned bulk DNA and single-cell data, then apply the QC filter: retain only PASS calls from FilterMutectCalls, homozygous 0/0 normal genotypes, a single alternative allele, and loci with at least 20 total reads in normal samples. At the finalized SNV loci, run SAMtools mpileup in force-calling mode to extract alternative (R) and total (X) read counts per cell/sample, drop mutations with zero alternative counts across all cells (rowSums(Rs0)!=0) and cells with zero total counts across all mutations (colSums(Xs0)!=0), and assemble the M x N matrices Rs and Xs and M x S matrices Rb and Xb with identical mutation rows, saving them as read_count_matrices.rda.

## Related tools

- **BWA** (Aligns bulk DNA WES reads to produce sorted, indexed BAMs prior to Mutect2 somatic variant calling)
- **STAR** (Aligns single-cell and bulk RNA-seq reads to produce BAMs for read-count extraction at SNV loci)
- **Mutect2** (Calls candidate somatic SNVs on aligned bulk DNA and single-cell data; PASS calls from FilterMutectCalls are retained by QC)
- **SAMtools** (Runs mpileup in force-calling mode at finalized SNV loci to extract alternative and total read counts per cell/sample)
- **R** (Assembles the Rs/Xs/Rb/Xb matrices and applies the nonzero row/column filters before saving read_count_matrices.rda) — https://github.com/annweideman/canopy2
- **Canopy2** (Downstream consumer of the QC-filtered read count matrices; its input requirements (Rs, Xs, Rb, Xb) define the output schema of this skill) — https://github.com/annweideman/canopy2

## Examples

```
mut.nonzero<-which(rowSums(Rs0)!=0); cell.nonzero<-which(colSums(Xs0)!=0); Rs<-Rs0[mut.nonzero,cell.nonzero]; Xs<-Xs0[mut.nonzero,cell.nonzero]
```

## Evaluation signals

- Every retained SNV is PASS by FilterMutectCalls, has a homozygous 0/0 normal genotype, a single alternative allele, and at least 20 total reads in the normal sample
- Rs and Xs have dimensions M x N and Rb and Xb have dimensions M x S with identical mutation rows across the single-cell and bulk matrices
- No mutation row has zero alternative reads across all cells (rowSums(Rs0)!=0) and no cell column has zero total reads across all mutations (colSums(Xs0)!=0)
- All four matrices are saved together as read_count_matrices.rda in the format Canopy2 expects
- The pipeline was run end-to-end via the deposited Zenodo scripts (zenodo.org/record/7931384) rather than ad hoc commands

## Limitations

- No software versions are stated for the tools (BWA, STAR, Mutect2, SAMtools, R, Canopy2), so exact reproduction of the environment is not guaranteed
- The QC criteria (e.g. the 20-total-read threshold in normals) were tuned for breast cancer and glioblastoma WES/scRNA-seq datasets and may need re-evaluation for other platforms
- Canopy2 relies on Metropolis-within-Gibbs sampling downstream, so convergence of the final inference is sensitive to proposal distributions even when the callset is correctly filtered

## Evidence

- [results] QC criteria retained PASS calls: [evidence span withheld — anchor status: not_found]
- [discussion] Stringent QC recommendation: "We suggest stringent QC of the bulk and single-cell read count data using our pre-processing pipeline (Fig. 6 in appendix) to obtain the finalized SNV callset."
- [methods] Zero-row/column filtering in R: "# identify mutations (rows) with non-zero alternative read counts across all cells
mut.nonzero<-which(rowSums(Rs0)!=0)"
- [readme] Pre-processing pipeline scripts location: [evidence span withheld — anchor status: external_doc]
