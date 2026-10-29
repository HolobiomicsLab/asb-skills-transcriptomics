---
name: hmm-transcribed-region-calling-from-sc-rna-seq
description: Use when raw scRNA-seq FASTQs must be analyzed without relying on a complete
  gene annotation — e.g., to quantify reads falling outside GENCODE/refFlat gene bodies
  in less-studied organisms (lemur, mole rat, sea urchin).
when_to_use_negative:
- Input is already aligned — use the from_cellranger workflow (cellranger count output)
  instead of re-running from FASTQs
- The organism has a high-quality, complete annotation and the goal is standard gene-level
  quantification rather than discovery of transcription outside annotation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Snakemake
  role: Workflow engine orchestrating the from_fastqs/from_cellranger TAR-calling
    pipeline (trimming, STAR alignment, matrix generation)
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: groHMM
  role: Hidden Markov model that divides the genome into transcribed/untranscribed
    states from per-bin read coverage to call TARs strand-specifically
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: rtracklayer
  role: R package used to load the groHMM-generated bed files during TAR/annotation
    overlap classification
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:1fdf99fa85f19111f3f5147056c439565a187f65578239b345dbb4bf52c352b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fw262/TAR-scRNA-seq
  tools:
  - Snakemake
  - groHMM
  - rtracklayer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# hmm-transcribed-region-calling-from-sc-rna-seq

## Summary

Call transcriptionally active regions (TARs) genome-wide from scRNA-seq data using a groHMM hidden Markov model that classifies the genome into transcribed and untranscribed states, then split TARs into annotated (aTARs) and unannotated (uTARs) against refFlat annotations to quantify transcriptional dark matter.

## When to use

Use when raw scRNA-seq FASTQs must be analyzed without relying on a complete gene annotation — e.g., to quantify reads falling outside GENCODE/refFlat gene bodies in less-studied organisms (lemur, mole rat, sea urchin).

## When NOT to use

- Input is already aligned — use the from_cellranger workflow (cellranger count output) instead of re-running from FASTQs
- The organism has a high-quality, complete annotation and the goal is standard gene-level quantification rather than discovery of transcription outside annotation

## Inputs

- Raw scRNA-seq FASTQ files (e.g., Tabula Muris mouse, 10X human PBMC, gray mouse lemur, naked mole rat, sea urchin, chicken)
- Genome assembly FASTA for the target organism
- refFlat-format gene annotations (e.g., GENCODE v30 hg38)

## Outputs

- Per-chromosome groHMM bed files (chr*_HMM.bed) of predicted TARs
- Single-cell gene expression and TAR expression matrices (Drop-seq tools format)
- Per-dataset summary table (CSV) of the fraction of unique reads assigned to uTARs
- aTAR/uTAR classification of predicted regions

## How to apply

Launch the Snakemake TAR-scRNA-seq workflow in from_fastqs mode to trim adapters, tag cell barcodes/UMIs, and align reads to the genome with STAR run without gene-annotation indexing, then build gene and TAR expression matrices with Drop-seq tools. Call TARs strand-specifically with groHMM using a 50 bp non-overlapping window and gamma emission probabilities learned via the Baum-Welch EM algorithm, merging predicted regions within 500 bp with bedtools merge (-s -d 500) and retaining TARs with at least 1/10,000,000 of uniquely mapped reads. For memory-heavy datasets (Tabula Muris), downsample the combined alignment to 5% of uniquely aligned reads before groHMM. Classify each HMM-derived TAR as an aTAR if it overlaps any refFlat transcript (ignoring strandedness), otherwise a uTAR, and compute the fraction of unique scRNA-seq reads per cell assigned to uTARs per dataset.

## Related tools

- **Snakemake** (Workflow engine orchestrating the from_fastqs/from_cellranger TAR-calling pipeline (trimming, STAR alignment, matrix generation)) — https://github.com/fw262/TAR-scRNA-seq
- **groHMM** (Hidden Markov model that divides the genome into transcribed/untranscribed states from per-bin read coverage to call TARs strand-specifically) — https://github.com/fw262/TAR-scRNA-seq
- **rtracklayer** (R package used to load the groHMM-generated bed files during TAR/annotation overlap classification) — https://github.com/fw262/TAR-scRNA-seq

## Evaluation signals

- chr*_HMM.bed groHMM output files exist on every chromosome; a missing-file error means the HMM step failed
- SingleCellHMM_Run_combined_bam_HMM_features.log completes without 'no lines available in input' / 'cannot read: chr*_HMM.bed' errors
- Summary CSV exists with a uTAR fraction value for each of the six datasets
- Computed uTAR read fractions fall within a tolerance band of reported references: hg38 ~2%, gray mouse lemur 7.1%, naked mole rat 4.2%, purple sea urchin 10.7%
- Every retained TAR has at least 1/10,000,000 of uniquely mapped reads

## Limitations

- groHMM has high memory demands: for Tabula Muris the combined alignment must be downsampled to 5% of uniquely aligned reads
- groHMM may fail silently if not manually installed/loaded in R, producing empty bed files and downstream 'no lines available in input' errors
- The information content of uTARs is strongly dependent on organism, tissue, cell type, and developmental stage, so uTAR fractions are not comparable without caution

## Evidence

- [intro] groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [abstract] Our tool generates a single-cell expression matrix for all transcriptionally active regions (TARs): "Our tool generates a single-cell expression matrix for all transcriptionally active regions (TARs)"
- [readme] Please manually install groHMM and rtracklayer and make sure you can load these packages in R.: "Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html) and make sure"
- [other] Call transcriptionally active regions strand-specifically with groHMM using a 50 bp non-overlapping window, gamma emission probabilities learned by the Baum-Welch EM algorithm: [evidence span withheld — anchor status: not_found]
