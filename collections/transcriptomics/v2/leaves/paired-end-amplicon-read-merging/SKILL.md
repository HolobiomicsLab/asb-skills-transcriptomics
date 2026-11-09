---
name: paired-end-amplicon-read-merging
description: Use when the input is paired-end FASTQ data from a population-level cellular
  barcoding experiment whose overlapping mate pairs cover the barcode, and a single
  merged read per molecule is needed before trimming and quantification.
when_to_use_negative:
- Input is single-end FASTQ data — use the bulk (test_SE) workflow, which skips MERGE_READS
  entirely
- Mate pairs do not overlap (e.g. long inserts with no shared region), so FLASh overlap-based
  merging cannot reconstruct the barcode
- Data are single-cell or spatial barcoding runs where cell barcode-UMI handling (e.g.
  REMOVE_PCR_CHIMERISM, SAW-processed spatial data) rather than bulk amplicon merging
  is required
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: BARtab
  role: Nextflow pipeline whose paired-bulk mode orchestrates the MERGE_READS step
    as part of bulk barcode counts table generation
  repo: https://github.com/DaneVass/BARtab
- name: FLASh
  role: Merges forward and reverse reads by overlap in the paired-bulk workflow
- name: Nextflow
  role: Workflow engine used to run BARtab (e.g. nextflow run DaneVass/bartab with
    conda/docker/singularity profiles)
- name: MultiQC
  role: Aggregates FLASh (and fastp/cutadapt/bowtie) statistics to validate the merge
    step
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:df0d30ac50f0136532179f339daf4ed616d463840200a53fcfa01452e0211b55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/BARtab
  tools:
  - BARtab
  - FLASh
  - MultiQC
  - Nextflow
  - bowtie1
  - cutadapt
  - fastp
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47368421052631576
n_claims: 19
---

# paired-end-amplicon-read-merging

## Summary

Merge forward and reverse amplicon FASTQ reads spanning a synthetic cellular barcode using FLASh within the BARtab paired-bulk Nextflow workflow, producing a single merged read set ready for quality filtering and constant-region trimming.

## When to use

Use when the input is paired-end FASTQ data from a population-level cellular barcoding experiment whose overlapping mate pairs cover the barcode, and a single merged read per molecule is needed before trimming and quantification.

## When NOT to use

- Input is single-end FASTQ data — use the bulk (test_SE) workflow, which skips MERGE_READS entirely
- Mate pairs do not overlap (e.g. long inserts with no shared region), so FLASh overlap-based merging cannot reconstruct the barcode
- Data are single-cell or spatial barcoding runs where cell barcode-UMI handling (e.g. REMOVE_PCR_CHIMERISM, SAW-processed spatial data) rather than bulk amplicon merging is required

## Inputs

- Paired-end FASTQ read files (R1/R2) from barcode amplicon sequencing of population-level cellular barcoding samples
- BARtab parameters (mode=paired-bulk, mergeoverlap)

## Outputs

- Merged (combined) FASTQ reads per sample from FLASh
- Merge statistics surfaced in the pipeline MultiQC report

## How to apply

Run the BARtab pipeline in paired-bulk mode (`nextflow run DaneVass/bartab --mode paired-bulk`) so that the MERGE_READS module invokes FLASh on the forward and reverse FASTQ files of each sample. Set the mergeoverlap parameter (e.g. 10 in the test_PE exemplar run) to control the minimum overlap required for combining mate pairs. Only reads that successfully merge — with consistent overlap between R1 and R2 — proceed to the downstream FILTER_READS step, where fastp quality filtering (minqual 20, pctqual 80) and cutadapt trimming of the flanking constant regions (e.g. upconstant TGACCATGTACGATTGACTA) occur. Merging first improves downstream barcode extraction because the full barcode sequence and flanking constant regions are reconstructed from the consensus of the overlapping mates, reducing per-base error before alignment against the barcode reference library. Validate the merge step by inspecting FLASh/MultiQC merge-rate metrics and confirming subsequent stages (fastp, cutadapt, bowtie) reported in the MultiQC summary behave as expected.

## Related tools

- **BARtab** (Nextflow pipeline whose paired-bulk mode orchestrates the MERGE_READS step as part of bulk barcode counts table generation) — https://github.com/DaneVass/BARtab
- **FLASh** (Merges forward and reverse reads by overlap in the paired-bulk workflow)
- **Nextflow** (Workflow engine used to run BARtab (e.g. nextflow run DaneVass/bartab with conda/docker/singularity profiles))
- **MultiQC** (Aggregates FLASh (and fastp/cutadapt/bowtie) statistics to validate the merge step)

## Examples

```
nextflow run DaneVass/bartab --mode paired-bulk -profile test_PE,docker
```

## Evaluation signals

- Merge-rate metrics from FLASh in the MultiQC report indicate a substantial fraction of R1/R2 pairs combined successfully
- The paired-bulk run uses mode paired-bulk and a defined mergeoverlap value (e.g. 10) rather than silently falling back to single-end processing
- Downstream fastp quality-filter and cutadapt trimming statistics are non-degenerate, showing merged reads carried intact flanking constant regions
- Final barcode-by-sample counts table for the test_PE exemplar matches the deposited output when the full workflow (merge → filter → trim → align → count) is reproduced

## Limitations

- Merging is only performed in paired-bulk mode; single-end datasets cannot benefit and take a different code path
- No accepted gold-standard cellular barcoding dataset existed for benchmarking, so merge settings must be validated per experiment
- low-complexity merged reads (e.g. poly-A reads from empty barcode constructs) survive merging unless complexity_threshold is raised in the downstream fastp filter step

## Evidence

- [methods] Merging paired-end reads with FLASh in the BARtab paired-bulk mode: "If running in mode paired-bulk, forward and reverse reads are merged using [FLASh](http://ccb.jhu.edu/software/FLASH/)."
- [methods] Reads are quality filtered using fastp: "Reads are quality filtered using [fastp](https://github.com/OpenGene/fastp)."
- [abstract] BARtab is a portable and scalable Nextflow pipeline for barcode counts table generation: "we present BARtab, a portable and scalable Nextflow pipeline"
- [readme] BARtab README describes the pipeline's purpose: "A Nextflow pipeline to tabulate synthetic barcode counts from NGS data"
- [readme] fastp supports merging paired-end reads: [evidence span withheld — anchor status: external_doc]
