---
name: umi-counting-and-error-correction
description: Use when processing droplet-based scRNA-seq FASTQ data with cell barcodes
  and UMIs (e.g. 10x GemCode/Chromium), to deduplicate reads into molecule counts
  or to quantify conversion efficiency against known ERCC spike-in inputs.
when_to_use_negative:
- Input data lack cell barcodes and UMIs (e.g. full-length bulk RNA-seq), so the UMI
  validity rules and 1-Hamming-distance correction cannot apply.
- A Cell Ranger count matrix is already available and the goal is downstream analysis
  (e.g. Seurat clustering), not re-derivation or validation of molecule counting.
- Assessing cell-level outcomes such as multiplet detection or chimerism from SNVs,
  which rely on genotype assignment rather than UMI counting.
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: STAR
  role: Aligns Read1 (the cDNA insert) to the reference genome or ERCC reference prior
    to UMI counting
- name: Cell Ranger Single-Cell Software Suite
  role: Performs sample demultiplexing, barcode processing and single-cell 3' gene
    counting that implements the UMI validity and correction rules
  repo: 10XGenomics/cellranger
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:32f1fc1d94096345465bcf3c12f5ff6c2c5bf564e1c2e6a0d4f675c55e2e376f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - STAR
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# umi-counting-and-error-correction

## Summary

Validates and error-corrects cell barcodes and UMIs from 10x GemCode 3' scRNA-seq data, then collapses valid UMI-gene-barcode combinations into digital molecule counts. It underpins accurate 3' mRNA counting by removing PCR duplicates, low-quality UMIs and barcode errors before building the gene-by-cell-barcode matrix.

## When to use

Use when processing droplet-based scRNA-seq FASTQ data with cell barcodes and UMIs (e.g. 10x GemCode/Chromium), to deduplicate reads into molecule counts or to quantify conversion efficiency against known ERCC spike-in inputs.

## When NOT to use

- Input data lack cell barcodes and UMIs (e.g. full-length bulk RNA-seq), so the UMI validity rules and 1-Hamming-distance correction cannot apply.
- A Cell Ranger count matrix is already available and the goal is downstream analysis (e.g. Seurat clustering), not re-derivation or validation of molecule counting.
- Assessing cell-level outcomes such as multiplet detection or chimerism from SNVs, which rely on genotype assignment rather than UMI counting.

## Inputs

- GEM sequencing FASTQ files (e.g. ERCC Spike-in Mix1/Mix2 1:50 dilution runs from SRA SRP073767)
- Reference transcript/genome sequences (hg19, mm10, or ERCC reference from tools.thermofisher.com cms_095047.txt)
- STAR genome index built on the chosen reference

## Outputs

- Gene-by-cell-barcode UMI count matrix built from MAPQ=255, non-PCR-duplicate reads with valid barcodes/UMIs
- Per-transcript UMI counts over ERCC references
- Inferred conversion efficiency per ERCC experiment (reported range 6.7–8.1%), Pearson r between mean UMI counts and expected ERCC molecules (~0.96), and CV2 across ERCC experiments (~7%)

## How to apply

Align Read1 (the cDNA insert) to the appropriate reference (hg19, mm10, or the ERCC reference built with STAR) using STAR. Validate UMIs by keeping only those with sequencing quality score >10 that are not homopolymers, then correct each UMI that is 1-Hamming-distance from another UMI (with more reads) for the same cell barcode and gene to the more-abundant UMI. Correct cell barcodes to the candidate with the highest posterior probability when that posterior probability is at least 0.975. Use only confidently mapped (MAPQ=255), non-PCR-duplicate reads with valid barcodes and UMIs to generate the gene-barcode matrix. For ERCC sensitivity checks, compute inferred cDNA conversion efficiency as the fraction of expected ERCC molecules represented by counted UMIs, and correlate mean UMI counts with expected ERCC molecule counts.

## Related tools

- **STAR** (Aligns Read1 (the cDNA insert) to the reference genome or ERCC reference prior to UMI counting)
- **Cell Ranger Single-Cell Software Suite** (Performs sample demultiplexing, barcode processing and single-cell 3' gene counting that implements the UMI validity and correction rules) — 10XGenomics/cellranger

## Examples

```
STAR --genomeDir ercc_index --readFilesIn ERCC_mix1_R1.fastq --outSAMtype BAM Unsorted --outFileNamePrefix ercc_mix1_
```

## Evaluation signals

- Inferred cDNA conversion efficiency per ERCC experiment falls within the reported 6.7–8.1% range for both ERCC Spike-in Mix1 and Mix2 at 1:50 dilution.
- Pearson correlation between sequenced mean UMI counts and expected ERCC molecule counts is ~0.96.
- CV2 across all ERCC experiments is ~7%.
- Only reads with MAPQ=255, non-PCR-duplicate status, and valid (quality >10, non-homopolymer, Hamming-corrected) barcodes/UMIs appear in the gene-barcode matrix.
- All 1-Hamming-distance UMI collisions resolve toward the UMI with more reads for the same cell barcode and gene.

## Limitations

- Cell Ranger source code is provided for informational purposes only; 10x does not support interpreting, modifying, building, or running it — use official release binaries.
- Some earlier libraries used a 5 nt UMI and 5 bp Read2 instead of the standard configuration, introducing protocol heterogeneity that the counting rules must accommodate (documented in Supplementary Table 1).
- UMI counts reflect conversion efficiency of only ~6.7–8.1%, so counted molecules are a substantial underestimate of input molecules and should not be treated as absolute abundance.

## Evidence

- [discussion] UMI correction rule: "A UMI that is 1-Hamming-distance away from another UMI (with more reads) for the same cell barcode and gene is corrected to the UMI with more reads."
- [discussion] UMI validity filters: "UMIs with sequencing quality score >10 were considered valid if they were not homopolymers."
- [discussion] Gene-barcode matrix construction: "Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix."
- [discussion] Barcode posterior-probability correction: "If the posterior probability for any candidate barcode is at least 0.975, then the barcode is corrected to the candidate barcode with the highest posterior probability."
- [discussion] Read1 alignment with STAR: "Read1, which contains the cDNA insert, was aligned to an appropriate reference genome using STAR35."
- [readme] Cell Ranger role: "Cell Ranger is a set of analysis pipelines that perform sample demultiplexing, barcode processing, single cell 3' and 5' gene counting"
