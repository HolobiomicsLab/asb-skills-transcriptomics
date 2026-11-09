---
name: single-cell-viral-count-comparison
description: Use when scPathoQuant has been run twice on the same cellranger output
  with -align bbmap and -align bowtie2, and you need to test whether per-gene pathogen
  UMI quantifications are aligner-independent per sample.
when_to_use_negative:
- The pathogen GTF was not supplied to scPathoQuant — pathogen_genes_al_sort_counts.sam
  (per-gene htseq output) is not produced in that case, so only whole-pathogen counts
  (pathogen_al_counts.csv) exist and gene-level comparison is impossible
- The goal is benchmarking scPathoQuant against external tools (ViralTrack, Venus,
  PathogenTrack) rather than internal bbmap-vs-bowtie2 consistency
- Input is a single pathogen-level count table (pathogen_al_counts.csv) without gene
  resolution — correlate whole-pathogen copy numbers instead
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: scPathoQuant
  role: Produces the paired per-gene pathogen UMI outputs (bbmap default vs '-align
    bowtie2') that are correlated; run once per aligner on the same 10x input
  repo: https://github.com/galelab/scPathoQuant
- name: bowtie2
  role: Alternate aligner invoked via '-align bowtie2' to generate the comparison
    arm of the per-gene UMI counts
- name: bbmap
  role: Default aligner whose per-gene UMI output serves as the baseline for the correlation
- name: htseq
  role: Generates the per-gene counts (pathogen_genes_al_sort_counts.sam) using non-default
    --intersection-nonempty mode
- name: Python
  role: Used to load both per-gene count tables, match gene identifiers, compute Pearson
    correlations per sample, and write gene_umi_correlations.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:0a5adb3761d15691cb9b0d2449ee520553593367fe0c4d522e1cc660f8938222
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - Python
  - bbmap
  - bowtie2
  - htseq
  - scPathoQuant
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-viral-count-comparison

## Summary

Quantifies consistency of per-gene viral UMI counts between the two scPathoQuant aligner options (bbmap default vs bowtie2) by correlating per-gene counts — e.g. the ten annotated SARS-CoV-2 genes from the GCF_009858895.2 reference — across single-cell samples.

## When to use

Use when scPathoQuant has been run twice on the same cellranger output with -align bbmap and -align bowtie2, and you need to test whether per-gene pathogen UMI quantifications are aligner-independent per sample.

## When NOT to use

- The pathogen GTF was not supplied to scPathoQuant — pathogen_genes_al_sort_counts.sam (per-gene htseq output) is not produced in that case, so only whole-pathogen counts (pathogen_al_counts.csv) exist and gene-level comparison is impossible
- The goal is benchmarking scPathoQuant against external tools (ViralTrack, Venus, PathogenTrack) rather than internal bbmap-vs-bowtie2 consistency
- Input is a single pathogen-level count table (pathogen_al_counts.csv) without gene resolution — correlate whole-pathogen copy numbers instead

## Inputs

- 10x cellranger count output folder (filtered_feature_bc_matrix / raw_feature_bc_matrix) for each sample
- SARS-CoV-2 reference genome fasta and GTF from NCBI GenBank assembly GCF_009858895.2
- pathogen_al_gene_counts.csv / pathogen_genes_al_sort_counts.sam from the scPathoQuant bbmap (default) run
- pathogen_al_gene_counts.csv / pathogen_genes_al_sort_counts.sam from the scPathoQuant '-align bowtie2' run

## Outputs

- gene_umi_correlations.csv — per-sample Pearson correlations of per-gene viral UMI counts between the two aligner runs
- paired per-gene UMI count tables restricted to the ten SARS-CoV-2 genes

## How to apply

Run scPathoQuant twice on the same cellranger count output with the same pathogen reference folder (SARS-CoV-2 GCF_009858895.2 fasta + GTF), once with default bbmap and once with '-align bowtie2 --bowtie2_params="--very-sensitive --non-deterministic"'. From each run, load the per-gene UMI outputs (pathogen_al_gene_counts.csv / pathogen_genes_al_sort_counts.sam produced by htseq with --intersection-nonempty) per HBEC dpi sample. Restrict both tables to the ten annotated SARS-CoV-2 genes, match gene identifiers between the bbmap and bowtie2 tables to form paired per-gene UMI vectors per sample, then compute Pearson correlation per sample. Assemble per-sample correlations into gene_umi_correlations.csv; validation requires each sample's correlation across the ten genes to be high and statistically significant, matching the aligner-consistency pattern of Fig. 3J.

## Related tools

- **scPathoQuant** (Produces the paired per-gene pathogen UMI outputs (bbmap default vs '-align bowtie2') that are correlated; run once per aligner on the same 10x input) — https://github.com/galelab/scPathoQuant
- **bowtie2** (Alternate aligner invoked via '-align bowtie2' to generate the comparison arm of the per-gene UMI counts)
- **bbmap** (Default aligner whose per-gene UMI output serves as the baseline for the correlation)
- **htseq** (Generates the per-gene counts (pathogen_genes_al_sort_counts.sam) using non-default --intersection-nonempty mode)
- **Python** (Used to load both per-gene count tables, match gene identifiers, compute Pearson correlations per sample, and write gene_umi_correlations.csv)

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/viral/fastafilefolder -align bowtie2 --bowtie2_params="--very-sensitive --non-deterministic"
```

## Evaluation signals

- Each sample's Pearson correlation across the ten SARS-CoV-2 genes is high and statistically significant, matching the Fig. 3J aligner-consistency pattern
- Gene identifiers match one-to-one between bbmap and bowtie2 tables after restriction to the ten annotated genes
- Both runs used the same GCF_009858895.2 fasta + GTF and the same 10x input, so differences are attributable to the aligner
- pathogen_genes_al_sort_counts.sam exists for both runs (GTF was provided) before attempting the comparison
- Output gene_umi_correlations.csv contains one correlation value per HBEC dpi sample

## Limitations

- Per-gene htseq output (pathogen_genes_al_sort_counts.sam) is only produced when a pathogen/viral GTF is provided; with fasta alone the comparison cannot be done
- Default alignment parameters are used for bbmap, and htseq-count uses the non-default --intersection-nonempty parameter, so results reflect those settings
- The _tmp/ directory files used by scPathoQuant can be large; --tmp_removal should be considered when space is an issue

## Evidence

- [readme] The software uses outside software samtools, bowtie2, and htseq to quantify pathogen reads. Default parameters are used when aligning reads to the pathogen genome and and non default htseq-count include using the --intersection-nonempty parameter.: "The software uses outside software samtools, bowtie2, and htseq to quantify pathogen reads. Default parameters are used when aligning reads to the pathogen genome and and non default htseq-count"
- [readme] * -align = alignment tool bbmap (default) or bowtie2: "* -align = alignment tool bbmap (default) or bowtie2"
- [readme] pathogen_al_gene_counts.csv - number of reads mapping to pathoen genes in each cell: "pathogen_al_gene_counts.csv - number of reads mapping to pathoen genes in each cell"
- [readme] pathogen_genes_al_sort_counts.sam - htseq output reads mapping to individual pathogen genes (will not be produced if pathogen/viral gtf is not provided): "pathogen_genes_al_sort_counts.sam - htseq output reads mapping to individual pathogen genes (will not be produced if pathogen/viral gtf is not provided)"
- [methods] -10x = Path/to/10x/sample/ (this the path to the output folder generated by first running cellranger count ): "-10x = Path/to/10x/sample/ (this the path to the output folder generated by first running cellranger count )"
- [methods] The Sars-CoV-2 reference genome and gtf file can be found [here](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_009858895.2/): "The Sars-CoV-2 reference genome and gtf file can be found [here](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_009858895.2/)"
