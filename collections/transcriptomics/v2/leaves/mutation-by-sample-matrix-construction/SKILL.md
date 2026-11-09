---
name: mutation-by-sample-matrix-construction
description: Use when raw bulk WES and scRNA-seq data must be converted into the M
  x N (mutation-by-cell) and M x T (mutation-by-bulk) alternative/total read count
  matrices (Rs, Xs, Rb, Xb) required as Canopy2 input.
when_to_use_negative:
- The read count matrices (Rs, Xs, Rb, Xb) already exist, e.g. the packaged GBM10_preproc/GBM10_postproc,
  GBM2, GBM9, BC03, or BC07 sample datasets — proceed directly to burstiness estimation
  and get_trees()
- The goal is downstream tumor phylogeny inference or clone assignment — that is performed
  by Canopy2's get_trees() on the assembled matrices, not by this construction step
- Input is germline heterozygous loci for estimating gene activation/deactivation
  rates — use the separate germline variant calling pipeline included in the Zenodo
  pre-processing scripts
edam_operation: http://edamontology.org/operation_3227
tools:
- name: BWA
  role: Aligns bulk DNA WES reads to produce the BAMs on which somatic SNVs are called
    before matrix construction
- name: STAR
  role: Aligns single-cell and bulk RNA-seq reads so per-cell and per-sample read
    counts can be extracted at SNV loci
- name: Mutect2
  role: Calls candidate somatic SNVs on bulk DNA and single-cell data; its output
    is filtered into the finalized callset used as the matrix rows
- name: SAMtools
  role: Runs mpileup in force-calling mode at finalized SNV loci to extract alternative
    and total read counts per cell/sample
- name: Canopy2
  role: Downstream consumer of the assembled Rs, Xs, Rb, Xb matrices as model inputs
  repo: https://github.com/annweideman/canopy2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:b8981d776e4947899249f0a943b78f2e1febba45e29b5e7f2381da315d98ee5f
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# mutation-by-sample-matrix-construction

## Summary

Constructs the alternative and total read count matrices (Rs, Xs for single cells; Rb, Xb for bulk samples) that Canopy2 consumes, by force-calling read counts at a QC-filtered somatic SNV callset with SAMtools mpileup and assembling them into a single read_count_matrices.rda.

## When to use

Use when raw bulk WES and scRNA-seq data must be converted into the M x N (mutation-by-cell) and M x T (mutation-by-bulk) alternative/total read count matrices (Rs, Xs, Rb, Xb) required as Canopy2 input.

## When NOT to use

- The read count matrices (Rs, Xs, Rb, Xb) already exist, e.g. the packaged GBM10_preproc/GBM10_postproc, GBM2, GBM9, BC03, or BC07 sample datasets — proceed directly to burstiness estimation and get_trees()
- The goal is downstream tumor phylogeny inference or clone assignment — that is performed by Canopy2's get_trees() on the assembled matrices, not by this construction step
- Input is germline heterozygous loci for estimating gene activation/deactivation rates — use the separate germline variant calling pipeline included in the Zenodo pre-processing scripts

## Inputs

- Raw FASTQ files: breast cancer single-cell and bulk RNA-seq (GEO GSE75688), breast cancer bulk DNA WES (SRA SRP067248), glioblastoma RNA-seq and bulk DNA WES (EGA EGAS00001001880)
- Sorted, indexed BAM files from BWA (bulk DNA WES) and STAR (RNA-seq) alignment
- Candidate somatic SNV callset from Mutect2 (VCF)
- Finalized QC-passing SNV loci list

## Outputs

- read_count_matrices.rda containing Rs and Xs (M x N mutation-by-single-cell alternative and total read count matrices)
- Rb and Xb (M x S mutation-by-bulk-sample alternative and total read count matrices) with identical mutation rows across single-cell and bulk data

## How to apply

First derive a finalized somatic SNV callset: run Mutect2 on aligned bulk DNA WES (reads aligned with BWA) and single-cell RNA-seq (reads aligned with STAR), then retain only SNVs that are PASS from FilterMutectCalls, have homozygous 0/0 normal genotypes, a single alternative allele, and at least 20 total reads in the normal sample. Next, run SAMtools mpileup in force-calling mode at the finalized SNV loci to extract alternative (mutated-locus) and total (mutated plus benign-locus) read counts for every single cell and bulk sample. Assemble the M x N matrices Rs and Xs (mutations by single-cells) and the M x S matrices Rb and Xb (mutations by bulk samples), ensuring identical mutation rows across the single-cell and bulk data, and save them as read_count_matrices.rda. Before Canopy2 inference, subset rows/columns with non-zero alternative counts across all cells and non-zero total counts across all mutations, and keep only mutations for which gene burstiness can be estimated.

## Related tools

- **BWA** (Aligns bulk DNA WES reads to produce the BAMs on which somatic SNVs are called before matrix construction)
- **STAR** (Aligns single-cell and bulk RNA-seq reads so per-cell and per-sample read counts can be extracted at SNV loci)
- **Mutect2** (Calls candidate somatic SNVs on bulk DNA and single-cell data; its output is filtered into the finalized callset used as the matrix rows)
- **SAMtools** (Runs mpileup in force-calling mode at finalized SNV loci to extract alternative and total read counts per cell/sample)
- **Canopy2** (Downstream consumer of the assembled Rs, Xs, Rb, Xb matrices as model inputs) — https://github.com/annweideman/canopy2

## Examples

```
mut.nonzero<-which(rowSums(Rs0)!=0); cell.nonzero<-which(colSums(Xs0)!=0)
```

## Evaluation signals

- Rs and Xs have identical dimensions (M x N) and Rb and Xb have identical dimensions (M x S), with mutation rows matched one-to-one across the single-cell and bulk matrices
- Every retained SNV satisfies the QC criteria: PASS from FilterMutectCalls, homozygous 0/0 normal genotype, a single alternative allele, and at least 20 total reads in the normal sample
- No mutation row in Rs has zero alternative read counts across all cells, and no cell column in Xs has zero total read counts across all mutations (checked via rowSums(Rs0)!=0 and colSums(Xs0)!=0)
- Alternative counts never exceed total counts at any mutation-by-sample entry (R <= X elementwise)
- Output is a single R data object saved as read_count_matrices.rda loadable with base R and accepted by Canopy2's input schema

## Limitations

- The pipeline requires matched bulk DNA WES and scRNA-seq from the same patient; single-cell data alone cannot produce the Rb/Xb matrices Canopy2 needs
- Stringent QC of the bulk and single-cell read count data via the pre-processing pipeline is recommended to obtain the finalized SNV callset; skipping it risks feeding unreliable counts to the model
- Mutations for which gene burstiness (alpha/beta) cannot be estimated must be subsetted out before Canopy2 inference, shrinking the final matrix

## Evidence

- [readme] README inputs Rs: "`Rs`:   $M \times N$ (mutation by single-cell) matrix of alternative read counts from single-cell RNA sequencing data."
- [readme] README Xs total counts: [evidence span withheld — anchor status: external_doc]
- [discussion] stringent QC recommendation: "We suggest stringent QC of the bulk and single-cell read count data using our pre-processing pipeline (Fig. 6 in appendix) to obtain the finalized SNV callset."
- [methods] Zenodo pre-processing pipeline: "A sample pipeline that can be used to preprocess these data is located in our Zenodo repository at https://zenodo.org/record/7931384"
- [methods] burstiness estimability filter: "Subset alternative and total read counts to include only mutations for which gene burstiness can be estimated."
