---
name: samtools-unmapped-read-extraction
description: Use when starting from 10x scRNAseq cellranger count output (possorted_genome_bam.bam)
  and you need the host-unaligned reads, with barcodes and UMIs, to realign against
  a viral or bacterial pathogen reference.
when_to_use_negative:
- The 10x sample was already processed with a combined host+pathogen reference genome
  (e.g. human genome with HIV integrated) during cellranger count, so no pathogen-aligned
  reads remain in the unmapped pool to recover
- You need host-aligned reads or host gene expression counts — this skill only captures
  reads that failed to align to the primary host genome
- Input is not 10x scRNAseq cellranger output (e.g. bulk RNA-seq BAMs or non-10x single-cell
  formats with no cellranger BAM structure)
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: samtools
  role: Extracts all host-unmapped reads from the cellranger possorted_genome_bam.bam
    into the _tmp/ working directory and converts to SAM so barcodes and UMIs can
    be parsed
  repo: https://github.com/galelab/scPathoQuant
- name: cellranger
  role: Upstream step whose count output (possorted_genome_bam.bam, filtered/raw_feature_bc_matrix)
    supplies the unaligned reads and the matrices that pathogen counts are later integrated
    back into
- name: scPathoQuant
  role: Python package orchestrating the extraction; consumes the unmapped-read table
    and FASTQ to align to the pathogen genome and rebuild 10x matrix files
  repo: https://github.com/galelab/scPathoQuant
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:e5ab9e15b3de1149406044d007e14d199a9902ceb549122fbdebf13bac65bb93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - Python
  - bbmap
  - bowtie2
  - cellranger
  - docker
  - samtools
  - scPathoQuant
  - seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# samtools-unmapped-read-extraction

## Summary

Extracts reads left unaligned by cellranger count from the 10x possorted_genome_bam.bam using samtools, recording each unmapped read with its cell barcode and UMI so they can be realigned to a pathogen genome in scPathoQuant.

## When to use

Use when starting from 10x scRNAseq cellranger count output (possorted_genome_bam.bam) and you need the host-unaligned reads, with barcodes and UMIs, to realign against a viral or bacterial pathogen reference.

## When NOT to use

- The 10x sample was already processed with a combined host+pathogen reference genome (e.g. human genome with HIV integrated) during cellranger count, so no pathogen-aligned reads remain in the unmapped pool to recover
- You need host-aligned reads or host gene expression counts — this skill only captures reads that failed to align to the primary host genome
- Input is not 10x scRNAseq cellranger output (e.g. bulk RNA-seq BAMs or non-10x single-cell formats with no cellranger BAM structure)

## Inputs

- 10x cellranger count output folder (containing possorted_genome_bam.bam and filtered_feature_bc_matrix / raw_feature_bc_matrix)
- samtools (used to pull unmapped reads from possorted_genome_bam.bam)

## Outputs

- _tmp/unmmaped.bam - all unmapped reads from CellRanger
- _tmp/unmmaped.sam - SAM file used to extract barcodes and UMIs for unmapped reads
- _tmp/barcodes_umi_read_table.csv - table of unmapped reads with corresponding barcodes and UMIs
- _tmp/unmapped.fq.gz - all unmapped reads in FASTQ format, usable for downstream phylogenetic analysis

## How to apply

After cellranger count has aligned reads to the host genome, use samtools to pull all reads that did not align from the sample's possorted_genome_bam.bam into the scPathoQuant _tmp/ working directory as unmmaped.bam, then convert to unmmaped.sam so that cell barcodes and UMIs can be parsed into barcodes_umi_read_table.csv; the reads are also written as unmapped.fq.gz, which can be reused for downstream phylogenetic analysis. Each unmapped read must keep its barcode and UMI association, since downstream pathogen UMI counting and reintegration into the filtered_feature_bc_matrix and raw_feature_bc_matrix matrices depend on it. The extraction runs before alignment with the selected aligner (bbmap default, e.g. --bbmap_params="--semiperfectmode", or bowtie2 with --bowtie2_params="--very-sensitive --non-deterministic") and is a prerequisite for quantifying per-cell total pathogen reads with htseq. The large _tmp/ directory can be deleted afterwards with the --tmp_removal flag.

## Related tools

- **samtools** (Extracts all host-unmapped reads from the cellranger possorted_genome_bam.bam into the _tmp/ working directory and converts to SAM so barcodes and UMIs can be parsed) — https://github.com/galelab/scPathoQuant
- **cellranger** (Upstream step whose count output (possorted_genome_bam.bam, filtered/raw_feature_bc_matrix) supplies the unaligned reads and the matrices that pathogen counts are later integrated back into)
- **scPathoQuant** (Python package orchestrating the extraction; consumes the unmapped-read table and FASTQ to align to the pathogen genome and rebuild 10x matrix files) — https://github.com/galelab/scPathoQuant

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/pathogen/fastafilefolder --tmp_removal
```

## Evaluation signals

- _tmp/unmmaped.bam exists and contains only reads flagged as unmapped relative to the host genome in possorted_genome_bam.bam
- barcodes_umi_read_table.csv has one row per unmapped read with valid 10x cell barcode and UMI sequences recovered from unmmaped.sam
- Read count in unmapped.fq.gz equals the unmapped read count in barcodes_umi_read_table.csv
- Downstream pathogen_al_counts.csv contains per-cell pathogen read counts, which requires barcode/UMI linkage from this extraction step
- The final filtered_feature_bc_matrix loads via Seurat Read10X with pathogen counts appended, confirming barcodes were preserved through extraction

## Limitations

- Files in _tmp/ (unmmaped.bam, unmmaped.sam, unmapped.fq.gz, barcodes_umi_read_table.csv) can be large; delete with --tmp_removal if disk space is an issue
- Only reads unaligned to the primary host genome are extracted — reads misassigned to the host genome are lost, which is why the test sets also compare against running cellranger on a host+pathogen combined reference
- Default aligner parameters are used when aligning extracted reads to the pathogen genome; recall depends on the chosen aligner settings (e.g. --semiperfectmode for bbmap, --very-sensitive --non-deterministic for bowtie2)

## Evidence

- [readme] unmmaped.bam - all unmapped reads from CellRanger (pulled from possorted_genome_bam.bam): "unmmaped.bam - all unmapped reads from CellRanger (pulled from possorted_genome_bam.bam)"
- [readme] barcodes_umi_read_table.csv: "barcodes_umi_read_table.csv - table of unmapped reads and corresponding barcodes and UMIs"
- [intro] samtools usage: "The software uses outside software samtools, bowtie2, and htseq to quantify pathogen reads"
- [abstract] extraction of host-unaligned reads: "scPathoQuant extracts sequences that were not aligned to the primary host genome"
- [intro] integration back into cellranger files: "scPathoQuant then takes the unaligned reads from the host genome and maps them to your pathogen genome of interest and then integrates them back into the cellranger files"
