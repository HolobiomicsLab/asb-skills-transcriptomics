---
name: sc-rna-seq-unique-read-fraction-quantification
description: Use when processing raw scRNA-seq FASTQs (e.g. 10X human PBMC, Tabula
  Muris, lemur lung, mole rat spleen, sea urchin embryo) and you need to measure how
  many uniquely mapped reads fall outside current gene annotations (uTARs) to assess
  annotation completeness.
when_to_use_negative:
- Reads are already aligned with cellranger count — use the from_cellranger workflow
  variant instead of from_fastqs
- The goal is differential expression or homology annotation of uTARs rather than
  quantifying annotation-incompleteness read fractions
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Snakemake
  role: Orchestrates the from_fastqs TAR-scRNA-seq pipeline (trimming, barcode tagging,
    STAR alignment, matrix generation, HMM calling)
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: groHMM
  role: Hidden Markov model that divides the genome into transcribed/untranscribed
    states to call transcriptionally active regions strand-specifically
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: rtracklayer
  role: Loads refFlat annotations and groHMM bed outputs used for aTAR/uTAR overlap
    classification
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: STAR
  role: Aligns reads to each genome without gene-annotation indexing
- name: Drop-seq tools
  role: Generates gene and TAR expression matrices from aligned reads
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:d219a293b1d09eeb4cd3fe78f82d21e977ae9f9c251930eb515b78d22377c059
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fw262/TAR-scRNA-seq
  tools:
  - Drop-seq tools
  - STAR
  - Snakemake
  - groHMM
  - rtracklayer
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# sc-rna-seq-unique-read-fraction-quantification

## Summary

Quantifies the fraction of unique scRNA-seq reads assigned to unannotated transcriptionally active regions (uTARs) per organism, using the TAR-scRNA-seq Snakemake workflow to expose incompleteness of genome annotations across human, mouse, gray mouse lemur, naked mole rat, and purple sea urchin datasets.

## When to use

Use when processing raw scRNA-seq FASTQs (e.g. 10X human PBMC, Tabula Muris, lemur lung, mole rat spleen, sea urchin embryo) and you need to measure how many uniquely mapped reads fall outside current gene annotations (uTARs) to assess annotation completeness.

## When NOT to use

- Reads are already aligned with cellranger count — use the from_cellranger workflow variant instead of from_fastqs
- The goal is differential expression or homology annotation of uTARs rather than quantifying annotation-incompleteness read fractions

## Inputs

- Raw scRNA-seq FASTQ files (e.g. 10X human PBMC, Tabula Muris mouse spleen/kidney, gray mouse lemur lung, naked mole rat spleen, purple sea urchin embryo, chicken heart)
- Genome assembly FASTA per organism
- refFlat-format gene annotation files per organism

## Outputs

- Gene and TAR single-cell expression matrices (Drop-seq tools)
- Strand-specific groHMM TAR bed files (chr*_HMM.bed)
- aTAR/uTAR classification per predicted region
- CSV summary table of unique-read fractions assigned to uTARs per cell per dataset

## How to apply

Run the TAR-scRNA-seq Snakemake workflow in from_fastqs mode: trim adapters, tag barcodes, and align reads with STAR without gene-annotation indexing, producing gene and TAR expression matrices via Drop-seq tools. Call strand-specific transcriptionally active regions with groHMM using a 50 bp non-overlapping window and Baum-Welch EM-learned gamma emission probabilities, merging predicted regions within 500 bp via bedtools merge (-s -d 500) and retaining TARs with at least 1/10,000,000 of uniquely mapped reads. For memory-heavy alignments such as Tabula Muris, downsample the combined BAM to 5% of uniquely aligned reads before HMM calling. Classify HMM TARs against refFlat annotations (any overlap, ignoring strand, = aTAR; remainder = uTAR), then compute the relative number of unique reads assigned to uTARs per cell per dataset and aggregate into a summary table. The rationale is that genome annotations are often incomplete, so uTAR read fractions directly index annotation gaps.

## Related tools

- **Snakemake** (Orchestrates the from_fastqs TAR-scRNA-seq pipeline (trimming, barcode tagging, STAR alignment, matrix generation, HMM calling)) — https://github.com/fw262/TAR-scRNA-seq
- **groHMM** (Hidden Markov model that divides the genome into transcribed/untranscribed states to call transcriptionally active regions strand-specifically) — https://github.com/fw262/TAR-scRNA-seq
- **rtracklayer** (Loads refFlat annotations and groHMM bed outputs used for aTAR/uTAR overlap classification) — https://github.com/fw262/TAR-scRNA-seq
- **STAR** (Aligns reads to each genome without gene-annotation indexing)
- **Drop-seq tools** (Generates gene and TAR expression matrices from aligned reads)

## Examples

```
Rscript -e 'library(groHMM); library(rtracklayer)'
```

## Evaluation signals

- Summary table exists in CSV format with a uTAR fraction value for each of the six datasets
- Computed fractions fall within tolerance bands of reported reference means: hg38 2%, gray mouse lemur 7.1%, naked mole rat 4.2%, purple sea urchin 10.7%
- Every retained TAR has at least 1/10,000,000 of uniquely mapped reads
- groHMM completed: chr*_HMM.bed files are non-empty (empty files trigger 'no lines available in input' in rule calcHMMrefFlat)
- aTAR/uTAR classification agrees with refFlat overlap ignoring strandedness

## Limitations

- groHMM memory constraints: large combined alignments (e.g. Tabula Muris) require downsampling to 5% of uniquely aligned reads before HMM calling
- uTAR read fractions reflect annotation incompleteness, so they are inflated for less-studied organisms and strongly dependent on organism, tissue, cell type, and developmental stage
- If the groHMM R script fails (e.g. groHMM or rtracklayer not installed), rule calcHMMrefFlat errors with 'no lines available in input' because chr*_HMM.bed files are missing

## Evidence

- [abstract] We have written Snakemake workflows to perform this analysis.: "We have written Snakemake workflows to perform this analysis."
- [abstract] Current workflows support starting with either raw fastq files (from_fastqs) or if you have already aligned with cellranger count (from_cellranger).: "Current workflows support starting with either raw fastq files (*from_fastqs*) or if you have already aligned with cellranger count (*from_cellranger*)."
- [intro] groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin.: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [abstract] Our tool generates a single-cell expression matrix for all transcriptionally active regions (TARs).: "Our tool generates a single-cell expression matrix for all transcriptionally active regions (TARs)"
- [readme] Please manually install groHMM and rtracklayer and make sure you can load these packages in R.: "Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html) and make sure"
