---
name: mixed-species-alignment
description: Use when scRNA-seq FASTQ data from a mixed-species (e.g. human 293T +
  mouse 3T3) droplet experiment at ~100k reads per cell must be aligned to separate
  reference genomes and per-cell gene/UMI QC metrics computed.
when_to_use_negative:
- Input is already a gene-barcode count matrix — this skill operates on raw FASTQ
  mixed-species data requiring species-specific alignment
- Single-species samples with no cross-species contamination — separate hg19/mm10
  alignment and SNV-based multiplet detection are unnecessary
- Full-length or non-3′ scRNA-seq protocols, since Cell Ranger here performs single-cell
  3′ gene counting with 3′-oriented Read1 alignment
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Performs sample demultiplexing, barcode processing (posterior-probability
    correction, UMI validation) and single-cell 3′ gene counting to generate the gene-barcode
    matrix
  repo: https://github.com/10XGenomics/cellranger
- name: STAR
  role: Aligns Read1 (the cDNA insert) to hg19 for human cells and mm10 for mouse
    cells in the mixed-species experiment
- name: Freebayes 1.0.2
  role: Calls SNVs on the Cell Ranger genome BAM to compute per-species SNV fractions
    used to flag multiplets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:e200522f8b870335522f73d561c1a72710d07bdb7e2fe0bef77b089aa29a87eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - Freebayes 1.0.2
  - STAR
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mixed-species-alignment

## Summary

Quantify droplet-based 3′ scRNA-seq sensitivity in a mixed human/mouse cell-line experiment (293T + 3T3 at ~100k reads per cell) by aligning Read1 (cDNA insert) separately to hg19 and mm10 with STAR and counting UMIs with the Cell Ranger Single-Cell Software Suite, reporting median genes and transcripts per cell.

## When to use

Use when scRNA-seq FASTQ data from a mixed-species (e.g. human 293T + mouse 3T3) droplet experiment at ~100k reads per cell must be aligned to separate reference genomes and per-cell gene/UMI QC metrics computed.

## When NOT to use

- Input is already a gene-barcode count matrix — this skill operates on raw FASTQ mixed-species data requiring species-specific alignment
- Single-species samples with no cross-species contamination — separate hg19/mm10 alignment and SNV-based multiplet detection are unnecessary
- Full-length or non-3′ scRNA-seq protocols, since Cell Ranger here performs single-cell 3′ gene counting with 3′-oriented Read1 alignment

## Inputs

- Mixed-species (293T + 3T3) droplet scRNA-seq FASTQ data from SRA accession SRP073767 or the 10x Genomics single-cell datasets site
- Human reference genome hg19
- Mouse reference genome mm10
- Expected recovered cell count for barcode calling

## Outputs

- Gene-barcode count matrix per species (hg19 and mm10)
- Per-cell QC metrics: median detected genes, median UMI counts, exonic mapping fraction per species, cell capture efficiency, multiplet rate
- qc_metrics_mixed_cells.csv

## How to apply

Download the mixed-species data from SRA SRP073767 (also mirrored at the 10x Genomics single-cell datasets site). Align Read1, which contains the cDNA insert, to hg19 with STAR for human-cell quantification and to mm10 with STAR for mouse-cell quantification. Run the Cell Ranger Single-Cell Software Suite for sample demultiplexing, barcode processing and single-cell 3′ gene counting, applying the article's filters: barcodes corrected only if posterior probability >= 0.975, UMIs valid if quality score > 10 and not homopolymers, and only confidently mapped (MAPQ=255), non-PCR-duplicate reads with valid barcodes/UMIs used to generate the gene-barcode matrix. Call cell barcodes using the rule that top barcodes within the same order of magnitude (>10% of the top nth barcode, n = 1% of the expected recovered cell count) are cell barcodes. Compute per-cell medians of detected genes and UMI counts, the fraction of reads mapping exonic to each species' genome, cell capture efficiency, and classify multiplets as cells whose SNV fraction from either species lies between 0.2 and 0.8 (inferred rate = 2 × observed). At 100k reads per cell the system achieved a median of ~4,500 genes and ~27,000 transcripts per cell in both species; write all metrics to qc_metrics_mixed_cells.csv.

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Performs sample demultiplexing, barcode processing (posterior-probability correction, UMI validation) and single-cell 3′ gene counting to generate the gene-barcode matrix) — https://github.com/10XGenomics/cellranger
- **STAR** (Aligns Read1 (the cDNA insert) to hg19 for human cells and mm10 for mouse cells in the mixed-species experiment)
- **Freebayes 1.0.2** (Calls SNVs on the Cell Ranger genome BAM to compute per-species SNV fractions used to flag multiplets)

## Examples

```
cellranger count --id=293T_3T3_mix --transcriptome=hg19 --fastqs=SRP073767_fastqs --sample=sample1; STAR --readFilesIn Read1.fq.gz --genomeDir hg19
```

## Evaluation signals

- At ~100k reads per cell, median detected genes should be ~4,500 and median UMI counts ~27,000 per human and per mouse cell in the mixed 293T/3T3 experiment
- Cell capture efficiency (recovered cell barcodes relative to cells loaded) should be near ~50%
- Gene-barcode matrix should contain only confidently mapped (MAPQ=255), non-PCR-duplicate reads with valid barcodes and UMIs (quality score > 10, non-homopolymer)
- Multiplets classified as cells with SNV fraction from either species between 0.2 and 0.8; inferred multiplet rate equals 2 × observed
- All computed metrics written to qc_metrics_mixed_cells.csv with expected schema (medians, exonic fractions, capture efficiency)

## Limitations

- Cell counts carry a 15–20% error, which could account for variability in calculated cell capture efficiencies
- The percentage of multiple cell encapsulations was 16% higher than the Poisson-expected value, possibly due to subsampling error or cell–cell interactions
- Some earlier libraries used 5 nt UMI and 5 bp Read2 instead of the standard configuration, introducing protocol heterogeneity across libraries (documented in Supplementary Table 1)

## Evidence

- [methods] Read1 aligned separately to hg19 and mm10: "Read1, which contains the cDNA insert, was aligned to an appropriate reference genome using STAR35."
- [methods] For mouse cells, mm10 was used and for human cells, hg19 was used.: "For mouse cells, mm10 was used and for human cells, hg19 was used."
- [methods] Only confidently mapped (MAPQ=255), non-PCR-duplicate reads with valid barcodes/UMIs used for gene-barcode matrix: "Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix."
- [results] Median genes and transcripts per cell in the mixed experiment: [evidence span withheld — anchor status: not_found]
- [discussion] Data deposited in SRA and mirrored on 10x datasets site: "Single-cell RNA-seq data have been deposited in the Short Read Archive under accession number SRP073767."
- [readme] Data also available on the 10x single-cell datasets site: [evidence span withheld — anchor status: not_found]
