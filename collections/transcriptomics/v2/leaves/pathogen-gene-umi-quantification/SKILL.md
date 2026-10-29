---
name: pathogen-gene-umi-quantification
description: Use when 10x cellranger output exists for infected-cell scRNAseq data
  and per-cell UMI counts for individual pathogen genes (not just total pathogen reads)
  are needed, e.g. for Fig. 3J-style aligner consistency or gene-level infection analysis.
when_to_use_negative:
- No pathogen/viral GTF file is available — scPathoQuant will only produce total pathogen
  counts, since pathogen_genes_al_sort_counts.sam 'will not be produced if pathogen/viral
  gtf is not provided'
- Input is not 10x cellranger count output (e.g. bulk RNA-seq FASTQs or already-aggregated
  gene-count matrices) — scPathoQuant expects the cellranger output folder structure
- Goal is viral phylogenetic or clade analysis rather than gene-level quantification
  — use the mapped-read FASTQ (pathogen_al_mapped.fq.gz) instead of per-gene UMI counts
edam_operation: http://edamontology.org/operation_3799
tools:
- name: scPathoQuant
  role: Python package that aligns unmapped host reads to the pathogen genome and
    runs htseq to produce per-gene UMI counts, integrating them into cellranger output
  repo: https://github.com/galelab/scPathoQuant
- name: bowtie2
  role: Alternative aligner invoked via -align bowtie2 for mapping pathogen reads
    before per-gene htseq quantification
- name: htseq
  role: Performs per-gene counting with --intersection-nonempty on reads mapped to
    the pathogen genome
- name: samtools
  role: Used internally by scPathoQuant to process mapped pathogen reads (BAM/SAM
    manipulation) prior to htseq counting
- name: cellranger
  role: Produces the 10x output folder (cellranger count) from which unmapped reads
    are extracted and into which pathogen gene counts are re-integrated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:0d97b9499b40c23af05cb261028102598b0c261bf3ecc41577e56e6bba23806d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - Python
  - bowtie2
  - cellranger
  - htseq
  - samtools
  - scPathoQuant
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pathogen-gene-umi-quantification

## Summary

Quantifies per-cell UMI counts mapping to individual pathogen genes (e.g. the ten annotated SARS-CoV-2 genes on GCF_009858895.2) using scPathoQuant, which runs htseq with --intersection-nonempty on reads aligned to the pathogen genome and integrates per-gene counts back into 10x cellranger outputs. Per-gene resolution also enables aligner-consistency checks such as correlating bbmap vs bowtie2 gene count vectors.

## When to use

Use when 10x cellranger output exists for infected-cell scRNAseq data and per-cell UMI counts for individual pathogen genes (not just total pathogen reads) are needed, e.g. for Fig. 3J-style aligner consistency or gene-level infection analysis.

## When NOT to use

- No pathogen/viral GTF file is available — scPathoQuant will only produce total pathogen counts, since pathogen_genes_al_sort_counts.sam 'will not be produced if pathogen/viral gtf is not provided'
- Input is not 10x cellranger count output (e.g. bulk RNA-seq FASTQs or already-aggregated gene-count matrices) — scPathoQuant expects the cellranger output folder structure
- Goal is viral phylogenetic or clade analysis rather than gene-level quantification — use the mapped-read FASTQ (pathogen_al_mapped.fq.gz) instead of per-gene UMI counts

## Inputs

- 10x cellranger count output folder for each sample (filtered_feature_bc_matrix and raw_feature_bc_matrix)
- Pathogen reference genome FASTA (e.g. SARS-CoV-2 NCBI GenBank assembly GCF_009858895.2)
- Pathogen GTF annotation file (required for per-gene counts)
- Unmapped host reads from possorted_genome_bam.bam extracted by scPathoQuant into _tmp/

## Outputs

- pathogen_al_gene_counts.csv — number of reads mapping to pathogen genes in each cell
- pathogen_genes_al_sort_counts.sam — htseq output of reads mapping to individual pathogen genes
- pathogen_al_counts.csv — total pathogen-mapping reads per cell (context for gene-level counts)
- Updated filtered_feature_bc_matrix / raw_feature_bc_matrix with pathogen gene UMI counts integrated
- Per-gene correlation table across samples (e.g. gene_umi_correlations.csv comparing bbmap vs bowtie2)

## How to apply

Run scPathoQuant on the cellranger output folder with -p2genome pointing to a folder containing both the pathogen reference FASTA and the pathogen GTF file — without the GTF, per-gene counts (pathogen_genes_al_sort_counts.sam) are not produced. Use either the default bbmap aligner or -align bowtie2 (e.g. with --bowtie2_params="--very-sensitive --non-deterministic"); scPathoQuant uses default alignment parameters and non-default htseq-count with --intersection-nonempty. Collect the per-gene output pathogen_al_gene_counts.csv (UMI counts per cell per gene) alongside the htseq SAM pathogen_genes_al_sort_counts.sam, then optionally restrict to annotated genes (e.g. the ten SARS-CoV-2 genes) and match gene identifiers across aligner runs to compute per-sample Pearson correlations. scPathoQuant also overwrites the 10x filtered_feature_bc_matrix/raw_feature_bc_matrix so pathogen gene counts load directly into Seurat via Read10X.

## Related tools

- **scPathoQuant** (Python package that aligns unmapped host reads to the pathogen genome and runs htseq to produce per-gene UMI counts, integrating them into cellranger output) — https://github.com/galelab/scPathoQuant
- **bowtie2** (Alternative aligner invoked via -align bowtie2 for mapping pathogen reads before per-gene htseq quantification)
- **htseq** (Performs per-gene counting with --intersection-nonempty on reads mapped to the pathogen genome)
- **samtools** (Used internally by scPathoQuant to process mapped pathogen reads (BAM/SAM manipulation) prior to htseq counting)
- **cellranger** (Produces the 10x output folder (cellranger count) from which unmapped reads are extracted and into which pathogen gene counts are re-integrated)

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/viral/fastafilefolder -align bowtie2 --bowtie2_params="--very-sensitive --non-deterministic"
```

## Evaluation signals

- pathogen_genes_al_sort_counts.sam exists for each sample, confirming the pathogen GTF was supplied and htseq per-gene counting ran
- Per-gene correlation between bbmap and bowtie2 runs across the ten SARS-CoV-2 genes is high and statistically significant for each HBEC dpi sample, matching the aligner-consistency pattern in Fig. 3J
- pathogen_al_gene_counts.csv rows/columns are restricted to the annotated SARS-CoV-2 genes and gene identifiers match between aligner runs before pairing
- pathogen_genes.png violin plot shows cells with pathogen gene UMI counts, confirming gene-level quantification propagated into downstream summaries
- Overwritten filtered_feature_bc_matrix loads in Seurat via Read10X and includes pathogen gene UMI features

## Limitations

- Per-gene UMI counts require a pathogen GTF; with only a FASTA, scPathoQuant quantifies total reads mapped to the pathogen (header of the FASTA is used to quantify total reads, so complicated headers should be simplified, e.g. > HIV_virus)
- Temporary _tmp/ files (e.g. unmmaped.bam, barcodes_umi_read_table.csv) can be large; use --tmp_removal if space is an issue
- Default alignment parameters are used when aligning reads to the pathogen genome unless overridden via --bbmap_params or --bowtie2_params

## Evidence

- [readme] htseq per-gene output availability: "pathogen_genes_al_sort_counts.sam - htseq output reads mapping to individual pathogen genes (will not be produced if pathogen/viral gtf is not provided)"
- [readme] per-gene count file: "pathogen_al_gene_counts.csv - number of reads mapping to pathoen genes in each cell"
- [readme] htseq non-default parameter: "non default htseq-count include using the --intersection-nonempty parameter"
- [methods] SARS-CoV-2 reference genome source: "The Sars-CoV-2 reference genome and gtf file can be found [here](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_009858895.2/)"
- [readme] aligner choice parameter: "-align = alignment tool bbmap (default) or bowtie2"
- [readme] bowtie2 sensitive parameters: "-align bowtie2 --bowtie2_params="--very-sensitive --non-deterministic""
- [intro] workflow of unaligned reads: "scPathoQuant then takes the unaligned reads from the host genome and maps them to your pathogen genome of interest and then integrates them back into the cellranger files"
