---
name: cell-barcode-calling-and-filtering
description: Use when raw 3' droplet scRNA-seq FASTQ (e.g., GemCode/10x data such
  as SRA SRP073767) must be turned into a cell-filtered gene-barcode UMI matrix, deciding
  which barcodes represent cells versus empty droplets.
when_to_use_negative:
- Input is already a gene-barcode UMI matrix (counting and barcode filtering already
  performed); this skill operates on raw reads
- Data lacks droplet cell barcodes and UMIs (e.g., plate-based full-length or bulk
  RNA-seq), so the barcode-ranking rule and UMI filters cannot apply
- Earlier libraries documented with 5 nt UMI and 5 bp Read2 (Supplementary Table 1
  protocol heterogeneity), where the paper's UMI-quality and read-structure assumptions
  differ
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Performs sample demultiplexing, barcode processing (posterior-probability
    barcode correction, valid-UMI QC) and single-cell 3' gene counting to generate
    the filtered gene-barcode matrix
  repo: https://github.com/10XGenomics/cellranger
- name: STAR
  role: Aligns Read1 (the cDNA insert) to hg19 or mm10 so confidently mapped (MAPQ=255)
    reads can be counted per cell barcode
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:c1e68961445a2394e1e6778c7ecde05204d4630aefca9450aa1c162598a3de46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - STAR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cell-barcode-calling-and-filtering

## Summary

Converts raw 3' droplet scRNA-seq reads into a cell-filtered gene-barcode UMI count matrix by applying the paper's cell-barcode calling rule (order-of-magnitude ranking vs. expected recovered cell count) plus barcode/UMI validity filters (posterior >=0.975 correction, UMI quality >10 non-homopolymer, MAPQ=255 confident mapping) via Cell Ranger.

## When to use

Use when raw 3' droplet scRNA-seq FASTQ (e.g., GemCode/10x data such as SRA SRP073767) must be turned into a cell-filtered gene-barcode UMI matrix, deciding which barcodes represent cells versus empty droplets.

## When NOT to use

- Input is already a gene-barcode UMI matrix (counting and barcode filtering already performed); this skill operates on raw reads
- Data lacks droplet cell barcodes and UMIs (e.g., plate-based full-length or bulk RNA-seq), so the barcode-ranking rule and UMI filters cannot apply
- Earlier libraries documented with 5 nt UMI and 5 bp Read2 (Supplementary Table 1 protocol heterogeneity), where the paper's UMI-quality and read-structure assumptions differ

## Inputs

- FASTQ reads from GemCode/droplet 3' scRNA-seq libraries (Read1 containing the cDNA insert; e.g., SRA SRP073767)
- Reference genome index (hg19 for human, mm10 for mouse)
- Expected recovered cell count used by the cell-barcode order-of-magnitude calling rule

## Outputs

- Gene-barcode UMI count matrix restricted to called cell barcodes
- List of called cell barcodes passing the >10%-of-top-nth-barcode rule
- Per-cell QC statistics (median detected genes, median UMI counts, cell capture efficiency), e.g., written to qc_metrics_mixed_cells.csv

## How to apply

Run the Cell Ranger Single-Cell Software Suite on the demultiplexed FASTQs to perform sample demultiplexing, barcode processing and single-cell 3' gene counting; STAR aligns Read1 (the cDNA insert) to hg19 for human cells or mm10 for mouse cells. Correct a barcode only when the posterior probability for a candidate barcode is at least 0.975, and treat UMIs as valid only if their sequencing quality score is >10 and they are not homopolymers, collapsing 1-Hamming-distance UMIs to the UMI with more reads for the same cell barcode and gene. Call cell barcodes by keeping all top barcodes within the same order of magnitude (>10% of the top nth barcode, where n is 1% of the expected recovered cell count). Generate the gene-barcode matrix only from confidently mapped (MAPQ=255), non-PCR-duplicate reads with valid barcodes and UMIs. Validate by computing per-cell QC (median detected genes and UMI counts); in the mixed 293T/3T3 experiment at ~100k reads per cell this yields ~4,500 genes and ~27,000 transcripts per cell, alongside cell capture efficiency (~50%).

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Performs sample demultiplexing, barcode processing (posterior-probability barcode correction, valid-UMI QC) and single-cell 3' gene counting to generate the filtered gene-barcode matrix) — https://github.com/10XGenomics/cellranger
- **STAR** (Aligns Read1 (the cDNA insert) to hg19 or mm10 so confidently mapped (MAPQ=255) reads can be counted per cell barcode)

## Examples

```
cellranger count --id=mix_293t_3t3 --transcriptome=/refs/hg19 --fastqs=/data/SRP073767 --expect-cells=1000
```

## Evaluation signals

- All reads feeding the gene-barcode matrix are confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs; any MAPQ!=255 or duplicate read is excluded
- Every called cell barcode satisfies the order-of-magnitude rule: >10% of the top nth barcode where n = 1% of the expected recovered cell count
- No barcode corrections applied below posterior probability 0.975, and no invalid UMIs (quality score <=10 or homopolymers) remain in the matrix
- 1-Hamming-distance UMI collapse is done only toward the UMI with more reads for the same cell barcode and gene
- Per-cell QC in the mixed 293T/3T3 run at ~100k reads per cell reproduces the reported medians (~4,500 genes and ~27,000 transcripts per cell) with ~50% cell capture efficiency

## Limitations

- Multiple-cell encapsulation rate was 16% higher than the Poisson expectation, possibly due to subsampling error or cell–cell interactions, so multiplet filtering matters
- There is a 15–20% error in cell counts, which could account for variability in calculated cell capture efficiencies and in the expected-recovered-cell-count used for barcode calling
- Some earlier libraries used 5 nt UMI and 5 bp Read2 (documented in Supplementary Table 1), introducing protocol heterogeneity across libraries

## Evidence

- [discussion] All top barcodes within the same order of magnitude (>10% of the top nth barcode, where n is 1% of the expected recovered cell count) were considered cell barcodes.: "All top barcodes within the same order of magnitude (>10% of the top nth barcode, where n is 1% of the expected recovered cell count) were considered cell barcodes."
- [discussion] If the posterior probability for any candidate barcode is at least 0.975, then the barcode is corrected to the candidate barcode with the highest posterior probability.: "If the posterior probability for any candidate barcode is at least 0.975, then the barcode is corrected to the candidate barcode with the highest posterior probability."
- [discussion] UMIs with sequencing quality score >10 were considered valid if they were not homopolymers.: "UMIs with sequencing quality score >10 were considered valid if they were not homopolymers."
- [discussion] Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix.: "Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix."
- [discussion] The Cell Ranger Single-Cell Software Suite was used to perform sample demultiplexing, barcode processing and single-cell 3′ gene counting: "The Cell Ranger Single-Cell Software Suite was used to perform sample demultiplexing, barcode processing and single-cell 3′ gene counting"
- [readme] Cell Ranger is a set of analysis pipelines that perform sample demultiplexing, barcode processing, single cell 3' and 5' gene counting: "Cell Ranger is a set of analysis pipelines that perform sample demultiplexing, barcode processing, single cell 3' and 5' gene counting"
