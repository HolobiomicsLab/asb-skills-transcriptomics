---
name: umi-validity-filtering-and-hamming-correction
description: Use when processing droplet scRNA-seq FASTQ reads (e.g. 10x Genomics
  GemCode/Chromium data) into a UMI count matrix and you need to remove invalid UMIs
  and collapse sequencing-error UMIs to avoid inflating molecule counts.
when_to_use_negative:
- Input is already a filtered gene-cell count matrix (e.g. .h5ad or Cell Ranger matrix
  output) — UMI-level correction has already been applied
- Full-length or plate-based scRNA-seq protocols without UMIs (e.g. Smart-seq2), where
  molecule counting by UMI collapse does not apply
- Bulk RNA-seq data where cell barcodes and UMIs are absent
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Performs sample demultiplexing, barcode processing, UMI validity filtering
    and correction, PCR-duplicate marking and single-cell 3' gene counting
  repo: https://github.com/10XGenomics/cellranger
- name: STAR
  role: Aligns Read1 (cDNA insert) to hg19/mm10 upstream of UMI filtering so only
    confidently mapped reads contribute counts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:5f4984340afde23053fdf601e32c041f041573afb107ed8db329ec66285e0235
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - SRA SRP073767
  - STAR
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# umi-validity-filtering-and-hamming-correction

## Summary

Validates and corrects UMIs during single-cell 3' gene counting so that PCR duplicates are collapsed accurately: low-quality or homopolymer UMIs are discarded, and UMIs within 1-Hamming-distance of a more abundant UMI for the same cell barcode and gene are merged before the gene-barcode count matrix is built.

## When to use

Use when processing droplet scRNA-seq FASTQ reads (e.g. 10x Genomics GemCode/Chromium data) into a UMI count matrix and you need to remove invalid UMIs and collapse sequencing-error UMIs to avoid inflating molecule counts.

## When NOT to use

- Input is already a filtered gene-cell count matrix (e.g. .h5ad or Cell Ranger matrix output) — UMI-level correction has already been applied
- Full-length or plate-based scRNA-seq protocols without UMIs (e.g. Smart-seq2), where molecule counting by UMI collapse does not apply
- Bulk RNA-seq data where cell barcodes and UMIs are absent

## Inputs

- Aligned BAM/SAM from STAR mapping of Read1 (cDNA insert) to hg19 or mm10
- FASTQ Read2 carrying cell barcode and UMI sequences with base quality scores
- Whitelist of valid cell barcodes (top barcodes >10% of the top nth barcode, n = 1% of expected recovered cell count)

## Outputs

- Set of deduplicated (cell barcode, corrected UMI, gene ID) tuples per cell
- Gene-by-cell-barcode UMI count matrix (e.g. gene_barcode_matrix.csv)

## How to apply

After aligning Read1 (the cDNA insert) to the reference genome with STAR, extract each read's cell barcode and UMI from Read2. First apply barcode correction: if the posterior probability for a candidate barcode is at least 0.975, correct the barcode to the highest-posterior candidate; otherwise discard the read. Then check UMI validity: keep only UMIs with sequencing quality score >10 that are not homopolymers. Finally collapse UMIs that are 1-Hamming-distance away from another UMI (with more reads) for the same cell barcode and gene into the more abundant UMI. This rationale is grounded in the Zheng et al. Cell Ranger pipeline; downstream, reads sharing the same (cell barcode, UMI, gene ID) tuple are marked as PCR duplicates and only one read per tuple is retained for the gene-barcode matrix.

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Performs sample demultiplexing, barcode processing, UMI validity filtering and correction, PCR-duplicate marking and single-cell 3' gene counting) — https://github.com/10XGenomics/cellranger
- **STAR** (Aligns Read1 (cDNA insert) to hg19/mm10 upstream of UMI filtering so only confidently mapped reads contribute counts)

## Examples

```
cellranger count --id=sample1 --transcriptome=/refs/hg19 --fastqs=fastqs/ --sample=sample1 --expect-cells=8000
```

## Evaluation signals

- All retained UMIs have sequencing quality score >10 and none are homopolymers
- No two retained UMIs for the same (cell barcode, gene) pair remain within 1-Hamming-distance, since the lower-abundance UMI must have been collapsed into the more abundant one
- Every read contributing to the matrix is confidently mapped (MAPQ=255) and non-PCR-duplicate, with at most one read retained per (cell barcode, UMI, gene ID) tuple
- Reads whose cell barcode could not be corrected with posterior probability >=0.975 are excluded from the matrix

## Limitations

- UMI collapse by 1-Hamming-distance can over-merge genuinely distinct UMIs when UMI density is high for a gene in a cell
- Some earlier libraries used a 5 nt UMI and 5 bp Read2 instead, introducing protocol heterogeneity across libraries
- Reads with ambiguous cell barcodes (posterior probability <0.975) are discarded entirely, losing data at low sequencing quality

## Evidence

- [discussion] UMIs with sequencing quality score >10 were considered valid if they were not homopolymers.: "UMIs with sequencing quality score >10 were considered valid if they were not homopolymers."
- [discussion] A UMI that is 1-Hamming-distance away from another UMI (with more reads) for the same cell barcode and gene is corrected to the UMI with more reads.: "A UMI that is 1-Hamming-distance away from another UMI (with more reads) for the same cell barcode and gene is corrected to the UMI with more reads."
- [discussion] Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix.: "Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix."
- [discussion] If the posterior probability for any candidate barcode is at least 0.975, then the barcode is corrected to the candidate barcode with the highest posterior probability.: "If the posterior probability for any candidate barcode is at least 0.975, then the barcode is corrected to the candidate barcode with the highest posterior probability."
- [discussion] The Cell Ranger Single-Cell Software Suite was used to perform sample demultiplexing, barcode processing and single-cell 3′ gene counting: "The Cell Ranger Single-Cell Software Suite was used to perform sample demultiplexing, barcode processing and single-cell 3′ gene counting"
- [discussion] Some earlier libraries were made with 5 nt UMI, and 5 bp Read2 was obtained instead.: "Some earlier libraries were made with 5 nt UMI, and 5 bp Read2 was obtained instead."
