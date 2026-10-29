---
name: umi-quality-correction
description: Use when generating or auditing 3' UMI counting matrices from GemCode/Chromium
  scRNA-seq FASTQ data where barcode synthesis/sequencing errors must be corrected
  before counting UMIs per cell barcode.
when_to_use_negative:
- Input is already a filtered gene-barcode UMI matrix (e.g. a Cell Ranger output matrix)
  — re-applying correction risks double-collapsing UMIs
- Data were generated with a non-GemCode/Chromium chemistry whose barcode/UMI structure
  differs (e.g. full-length protocols like Smart-seq) — the Hamming-1 UMI collapse
  and barcode whitelist logic do not transfer
- Earlier 10x libraries with 5 nt UMIs and 5 bp Read2 require protocol-specific handling
  documented in Supplementary Table 1 rather than the default 10 nt UMI rules
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Performs sample demultiplexing, barcode processing (posterior-probability
    correction) and single-cell 3' gene counting that implement the UMI quality rules
  repo: https://github.com/10XGenomics/cellranger
- name: STAR
  role: Aligns Read1 (cDNA insert) to hg19/mm10 prior to UMI/barcode filtering for
    the gene-barcode matrix
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:7b689a33ee64895e7725505df2d47c3dcb5eb7a11843c28be5885465b3eb2410
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - STAR
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# umi-quality-correction

## Summary

Barcode/UMI error-correction scheme from the 10x Genomics GemCode 3' scRNA-seq pipeline: barcodes are probabilistically corrected, UMIs validated by sequencing quality and homopolymer checks, and 1-Hamming-distance UMIs collapsed so that the gene-barcode matrix reflects true molecules rather than sequencing errors.

## When to use

Use when generating or auditing 3' UMI counting matrices from GemCode/Chromium scRNA-seq FASTQ data where barcode synthesis/sequencing errors must be corrected before counting UMIs per cell barcode.

## When NOT to use

- Input is already a filtered gene-barcode UMI matrix (e.g. a Cell Ranger output matrix) — re-applying correction risks double-collapsing UMIs
- Data were generated with a non-GemCode/Chromium chemistry whose barcode/UMI structure differs (e.g. full-length protocols like Smart-seq) — the Hamming-1 UMI collapse and barcode whitelist logic do not transfer
- Earlier 10x libraries with 5 nt UMIs and 5 bp Read2 require protocol-specific handling documented in Supplementary Table 1 rather than the default 10 nt UMI rules

## Inputs

- FASTQ files from GemCode/Chromium 3' scRNA-seq sequencing (e.g. Illumina NextSeq500 2x75 libraries)
- Precomputed whitelist of known cell barcodes and base quality scores per read (BCL/FASTQ quality strings)
- STAR-aligned BAM of Read1 cDNA inserts against hg19 or mm10

## Outputs

- Filtered gene-barcode UMI count matrix containing only confidently mapped (MAPQ=255), non-duplicate reads with valid barcodes/UMIs
- Per-cell QC metrics (genes and UMI counts detected per cell; in the mixed 293T/3T3 experiment at ~100k reads/cell: median ~4,500 genes and ~27,000 transcripts per cell)
- qc_metrics_mixed_cells.csv of per-cell statistics

## How to apply

After aligning Read1 (the cDNA insert) to hg19 or mm10 with STAR, correct cell barcodes only when the posterior probability for a candidate barcode is at least 0.975, assigning the barcode to the highest-probability candidate. Validate UMIs by requiring sequencing quality score >10 and rejecting homopolymer UMIs. Collapse UMIs for the same cell barcode and gene: a UMI 1-Hamming-distance away from another UMI with more reads is corrected to the more-abundant UMI. Build the gene-barcode matrix from only confidently mapped (MAPQ=255), non-PCR-duplicate reads carrying valid barcodes and UMIs; this is what Cell Ranger automates during sample demultiplexing, barcode processing and single-cell 3' gene counting.

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Performs sample demultiplexing, barcode processing (posterior-probability correction) and single-cell 3' gene counting that implement the UMI quality rules) — https://github.com/10XGenomics/cellranger
- **STAR** (Aligns Read1 (cDNA insert) to hg19/mm10 prior to UMI/barcode filtering for the gene-barcode matrix)

## Examples

```
cellranger count --id=293T_3T3_mix --transcriptome=hg19 --fastqs=fastqs/ --sample=mix_sample --expect-cells=6000
```

## Evaluation signals

- Every corrected barcode has a posterior probability >= 0.975 for the assigned candidate barcode
- All retained UMIs have sequencing quality score >10 and none are homopolymers
- Gene-barcode matrix contains only MAPQ=255, non-PCR-duplicate reads with valid barcodes and UMIs
- UHI collapse check: for each cell barcode/gene pair, no two retained UMIs differ by 1 Hamming distance with the lower-count UMI still present
- Sensitivity benchmark: at ~100k reads per cell in the 293T/3T3 mix, median genes per cell ~4,500 and median UMI counts ~27,000 for both human and mouse cells

## Limitations

- Some earlier libraries were built with 5 nt UMIs and 5 bp Read2, introducing protocol heterogeneity that the default rules must be adjusted for
- The 10xGenomics/cellranger source is provided 'only for informational purposes'; 10x does not support modified builds, so corrections should use the official release binaries
- 15–20% error in cell counts can affect downstream efficiency calculations that depend on correctly called cell barcodes

## Evidence

- [methods] Barcode corrected only if posterior probability ≥0.975: "If the posterior probability for any candidate barcode is at least 0.975, then the barcode is corrected to the candidate barcode with the highest posterior probability."
- [methods] UMI validity rules: "UMIs with sequencing quality score >10 were considered valid if they were not homopolymers."
- [methods] 1-Hamming UMI collapse: "A UMI that is 1-Hamming-distance away from another UMI (with more reads) for the same cell barcode and gene is corrected to the UMI with more reads."
- [methods] Matrix-level filter: "Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix."
- [readme] README scope of Cell Ranger: "Cell Ranger is a set of analysis pipelines that perform sample demultiplexing, barcode processing, single cell 3' and 5' gene counting"
