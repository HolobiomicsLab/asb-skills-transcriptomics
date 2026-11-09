---
name: single-cell-qc-metrics-computation
description: Use when raw droplet scRNA-seq FASTQ data (e.g. 10x GemCode, SRA SRP073767)
  must be turned into per-cell QC statistics such as median genes/UMIs per cell, capture
  efficiency, and multiplet rates.
when_to_use_negative:
- Input is already a filtered gene-barcode matrix or feature table rather than raw
  FASTQ needing alignment and counting
- The experiment does not mix two species (or lacks SNV genotype information), since
  the multiplet definition relies on per-cell SNV species fractions
- Full-length transcript quantification or isoform analysis is needed — this skill
  is specific to 3' mRNA counting (UMI-based digital counting)
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: Cell Ranger Single-Cell Software Suite
  role: Performs sample demultiplexing, barcode processing (correction at posterior
    probability ≥0.975, UMI quality >10 and non-homopolymer) and single-cell 3' gene
    counting to build the gene-barcode matrix used for QC metrics
  repo: https://github.com/10XGenomics/cellranger
- name: STAR
  role: Aligns Read1 (the cDNA insert) to hg19 for human cells and mm10 for mouse
    cells prior to counting
- name: GemCode Single-Cell Instrument (10x Genomics)
  role: Generated the single-cell GEMs from which the QC-metric data originate
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms14049
    title: Massively parallel digital transcriptional profiling of single cells
  source_content_hash: sha256:dfc90d2a1dcaebe978f2e90c12967f71fe2e0ac55a5839e5a4549356c3dcdb5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/10XGenomics/cellranger
  tools:
  - Cell Ranger Single-Cell Software Suite
  - GemCode Single-Cell Instrument (10x Genomics)
  - STAR
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-cell-qc-metrics-computation

## Summary

Computes per-cell quality-control metrics (median detected genes, UMI/transcript counts, exonic mapping fraction, cell capture efficiency, and multiplet rate) for droplet-based 3' scRNA-seq data, as demonstrated on the mixed 293T/3T3 cell-line experiment from Zheng et al. 2017.

## When to use

Use when raw droplet scRNA-seq FASTQ data (e.g. 10x GemCode, SRA SRP073767) must be turned into per-cell QC statistics such as median genes/UMIs per cell, capture efficiency, and multiplet rates.

## When NOT to use

- Input is already a filtered gene-barcode matrix or feature table rather than raw FASTQ needing alignment and counting
- The experiment does not mix two species (or lacks SNV genotype information), since the multiplet definition relies on per-cell SNV species fractions
- Full-length transcript quantification or isoform analysis is needed — this skill is specific to 3' mRNA counting (UMI-based digital counting)

## Inputs

- Droplet scRNA-seq FASTQ files (e.g. mixed 293T/3T3 experiment, SRA accession SRP073767)
- Reference genomes hg19 (human) and mm10 (mouse) for STAR alignment
- Expected recovered cell count (for the cell-barcode calling rule)

## Outputs

- qc_metrics_mixed_cells.csv containing per-cell and summary QC metrics
- Per-cell gene and UMI count medians
- Fraction of exonic reads mapping to each species' genome
- Cell capture efficiency (recovered cell barcodes / cells loaded)
- Observed and inferred (2× observed) multiplet rate

## How to apply

Download the mixed-species sequencing data (SRA SRP073767 or the 10x Genomics single-cell datasets site) and align Read1 (the cDNA insert) to hg19 with STAR for human cells and to mm10 for mouse cells. Run the Cell Ranger Single-Cell Software Suite for sample demultiplexing, barcode processing, and single-cell 3' gene counting, keeping only confidently mapped (MAPQ=255), non-PCR-duplicate reads with valid barcodes/UMIs when building the gene-barcode matrix. Apply the filtering rules: barcodes corrected only if posterior probability ≥0.975; UMIs valid only if quality score >10 and non-homopolymer; call cell barcodes as those within the same order of magnitude (>10% of the top nth barcode, n = 1% of the expected recovered cell count). Compute per-cell QC statistics — number of detected genes, UMI counts (report medians), fraction of reads mapping exonic to each species' genome, and cell capture efficiency (recovered barcodes relative to cells loaded). Classify multiplets as cells whose SNV fraction from either species lies between 0.2 and 0.8, and apply the inferred rate = 2 × observed multiplet rate. Write all metrics to qc_metrics_mixed_cells.csv.

## Related tools

- **Cell Ranger Single-Cell Software Suite** (Performs sample demultiplexing, barcode processing (correction at posterior probability ≥0.975, UMI quality >10 and non-homopolymer) and single-cell 3' gene counting to build the gene-barcode matrix used for QC metrics) — https://github.com/10XGenomics/cellranger
- **STAR** (Aligns Read1 (the cDNA insert) to hg19 for human cells and mm10 for mouse cells prior to counting)
- **GemCode Single-Cell Instrument (10x Genomics)** (Generated the single-cell GEMs from which the QC-metric data originate)

## Examples

```
cellranger count --id=293T_3T3_mix --transcriptome=hg19 --fastqs=SRP073767_fastqs --expect-cells=1000
```

## Evaluation signals

- At ~100k reads per cell, medians of ∼4,500 genes and ∼27,000 transcripts (UMI counts) should be detected per human and per mouse cell in the mixed 293T/3T3 experiment
- qc_metrics_mixed_cells.csv exists and contains per-cell gene counts, UMI counts, exonic mapping fractions per species, capture efficiency, and multiplet rate
- Gene-barcode matrix was built only from confidently mapped (MAPQ=255), non-PCR-duplicate reads with valid barcodes/UMIs
- Cell barcodes satisfy the order-of-magnitude rule (>10% of the top nth barcode, n = 1% of expected recovered cell count)
- Inferred multiplet rate equals exactly 2× the observed rate, with multiplets defined as SNV fraction between 0.2 and 0.8

## Limitations

- There is a 15–20% error in cell counts, which could account for variability in calculated cell capture efficiencies
- Multiple-cell encapsulation rate exceeded the Poisson expectation by 16%, possibly due to subsampling error or cell–cell interactions
- Some earlier libraries used a 5 nt UMI and 5 bp Read2, introducing protocol heterogeneity across libraries (documented in Supplementary Table 1)

## Evidence

- [methods] The Cell Ranger Single-Cell Software Suite was used to perform sample demultiplexing, barcode processing and single-cell 3′ gene counting: "The Cell Ranger Single-Cell Software Suite was used to perform sample demultiplexing, barcode processing and single-cell 3′ gene counting"
- [methods] Only confidently mapped, non-duplicate reads: "Only confidently mapped (MAPQ=255), non-PCR duplicates with valid barcodes and UMIs were used to generate gene-barcode matrix."
- [methods] Cell barcode calling rule: "All top barcodes within the same order of magnitude (>10% of the top nth barcode, where n is 1% of the expected recovered cell count) were considered cell barcodes."
- [methods] Multiplet definition and rate: "Cells with a fraction of SNV from either species between 0.2 and 0.8 are considered multiplets. The inferred multiplet rate is 2* observed multiplet rate"
- [discussion] Cell count error caveat: "It is worth noting that there is a 15–20% error in cell counts, which could account for at least some of the variability in the calculated efficiencies."
