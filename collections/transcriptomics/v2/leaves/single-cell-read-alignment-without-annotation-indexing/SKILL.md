---
name: single-cell-read-alignment-without-annotation-indexing
description: Use when starting from raw scRNA-seq FASTQs and the goal is to quantify
  reads mapping outside gene annotations (uTARs) to assess annotation incompleteness,
  e.g. across human, mouse, lemur, mole rat, or sea urchin datasets.
when_to_use_negative:
- Input data are already aligned by cellranger count — use the from_cellranger workflow
  instead of re-aligning FASTQs
- The goal is standard annotation-restricted gene expression quantification with a
  trusted, complete annotation and no interest in uTARs
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: Snakemake
  role: Orchestrates the from_fastqs workflow that trims, tags barcodes, and aligns
    reads without gene-annotation indexing
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: STAR
  role: Aligner run without gene-annotation indexing to keep annotation-external reads
    mappable
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: groHMM
  role: Hidden Markov model used downstream to call transcriptionally active regions
    from the annotation-free alignment
  repo: https://github.com/fw262/TAR-scRNA-seq
- name: rtracklayer
  role: R package used to load/compare refFlat annotations and groHMM bed outputs
    when classifying aTARs vs uTARs
  repo: https://github.com/fw262/TAR-scRNA-seq
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:483306ec62c310b0120df492f07ef0157bb78b46b27a451bc14f34812c9ad7d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fw262/TAR-scRNA-seq
  tools:
  - STAR
  - Snakemake
  - groHMM
  - rtracklayer
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# single-cell-read-alignment-without-annotation-indexing

## Summary

Align raw scRNA-seq FASTQs to a genome with STAR without gene-annotation indexing, so that reads falling outside known gene annotations (uTARs) can be quantified; this is the alignment backbone of the TAR-scRNA-seq annotation-independent workflow.

## When to use

Use when starting from raw scRNA-seq FASTQs and the goal is to quantify reads mapping outside gene annotations (uTARs) to assess annotation incompleteness, e.g. across human, mouse, lemur, mole rat, or sea urchin datasets.

## When NOT to use

- Input data are already aligned by cellranger count — use the from_cellranger workflow instead of re-aligning FASTQs
- The goal is standard annotation-restricted gene expression quantification with a trusted, complete annotation and no interest in uTARs

## Inputs

- Raw scRNA-seq FASTQ files (e.g. 10X human PBMC, Tabula Muris mouse spleen/kidney, gray mouse lemur lung, naked mole rat spleen, sea urchin embryo, chicken heart)
- Genome assembly FASTA for the organism (e.g. GENCODE v30 hg38)
- refFlat-format gene annotations for the organism
- groHMM (Bioconductor), rtracklayer, Drop-seq tools, STAR, bedtools, Snakemake

## Outputs

- STAR BAM alignment produced without gene-annotation indexing
- Gene and TAR single-cell expression matrices generated with Drop-seq tools
- Per-cell unique read counts assigned to uTARs and per-dataset uTAR read fractions (summary CSV)

## How to apply

Run the TAR-scRNA-seq Snakemake workflow in from_fastqs mode: trim adapters, tag cell barcodes and UMIs, then align reads to the organism's genome assembly with STAR run without gene-annotation indexing, so that transcribed loci outside the annotation are not discarded. Generate gene and TAR expression matrices with Drop-seq tools from the annotation-free alignment. After alignment, call strand-specific TARs with groHMM (50 bp non-overlapping bins, Baum-Welch EM-learned gamma emission probabilities, merging regions within 500 bp via bedtools merge -s -d 500, retaining TARs with at least 1/10,000,000 of uniquely mapped reads), and classify them as aTARs or uTARs by overlap against refFlat annotations ignoring strandedness. The rationale is that standard annotation-restricted quantification hides 'transcriptional dark matter'; alignment without annotation indexing preserves reads outside annotations for downstream uTAR fraction estimation. For very deep datasets such as Tabula Muris, downsample the combined alignment to 5% of uniquely aligned reads to fit groHMM memory constraints.

## Related tools

- **Snakemake** (Orchestrates the from_fastqs workflow that trims, tags barcodes, and aligns reads without gene-annotation indexing) — https://github.com/fw262/TAR-scRNA-seq
- **STAR** (Aligner run without gene-annotation indexing to keep annotation-external reads mappable) — https://github.com/fw262/TAR-scRNA-seq
- **groHMM** (Hidden Markov model used downstream to call transcriptionally active regions from the annotation-free alignment) — https://github.com/fw262/TAR-scRNA-seq
- **rtracklayer** (R package used to load/compare refFlat annotations and groHMM bed outputs when classifying aTARs vs uTARs) — https://github.com/fw262/TAR-scRNA-seq

## Examples

```
snakemake --directory workflow/from_fastqs/ --configfile config/organism.yaml --cores 32  # from_fastqs mode: trim, tag barcodes, STAR align without gene-annotation indexing
```

## Evaluation signals

- Summary table exists as CSV with a uTAR read-fraction value for each of the six datasets
- Computed fractions fall within a tolerance band of reported reference means: hg38 2%, gray mouse lemur 7.1%, naked mole rat 4.2%, purple sea urchin 10.7%
- groHMM bed files are produced per chromosome; absence of chr*_HMM.bed files with 'no lines available in input' errors indicates the R HMM step failed
- TAR classification yields both aTARs (overlapping any refFlat transcript, ignoring strandedness) and uTARs (no overlap); a workflow missing either class signals incorrect annotation comparison
- Retained TARs each contain at least 1/10,000,000 of uniquely mapped reads

## Limitations

- groHMM memory constraints require downsampling very large combined alignments (e.g. Tabula Muris to 5% of uniquely aligned reads)
- groHMM/rtracklayer must be manually installed and loadable in R, or the HMM step silently fails leaving missing chr*_HMM.bed files
- Fractions of reads outside annotations are strongly dependent on organism, tissue, cell type, and developmental stage, so reference means are not transferable across datasets

## Evidence

- [results] Only 2% of reads aligned outside a recent human annotation (GENCODE v30 hg38), whereas far more fell outside current assemblies for gray mouse lemur (7.1%), naked mole rat (4.2%), and purple sea urchin (10.7%): [evidence span withheld — anchor status: not_found]
- [intro] groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [readme] Current workflows support starting with either raw fastq files (*from_fastqs*) or if you have already aligned with cellranger count (*from_cellranger*): "Current workflows support starting with either raw fastq files (*from_fastqs*) or if you have already aligned with cellranger count (*from_cellranger*)"
- [readme] Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer] ... and make sure you can load these packages in R.: "Please manually install [groHMM](https://bioconductor.org/packages/release/bioc/html/groHMM.html) and [rtracklayer](https://bioconductor.org/packages/release/bioc/html/rtracklayer.html) and make sure"
