---
name: sars-cov-2-gene-annotation-mapping
description: Use when scPathoQuant per-gene pathogen UMI outputs exist for the same
  10x samples under two alignment options (bbmap default and -align bowtie2) and you
  need to verify that per-gene SARS-CoV-2 UMI quantifications are consistent across
  aligners.
when_to_use_negative:
- The scPathoQuant run was performed without a pathogen GTF — per-gene htseq output
  (pathogen_genes_al_sort_counts.sam) will not exist, so only total pathogen counts
  can be compared
- You need to quantify host-cell gene expression; this skill covers pathogen (SARS-CoV-2)
  gene counts only
- Comparing scPathoQuant against other tools such as ViralTrack, Venus, or PathogenTrack
  — that benchmarking workflow uses different per-tool scripts
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_3308
tools:
- name: scPathoQuant
  role: Extracts unaligned host reads, maps them to the SARS-CoV-2 genome with bbmap
    or bowtie2, and produces per-gene UMI count outputs
  repo: https://github.com/galelab/scPathoQuant
- name: bowtie2
  role: Alternative aligner invoked via -align bowtie2 (e.g. with --very-sensitive
    --non-deterministic) to generate the comparison per-gene counts
- name: samtools
  role: Used by scPathoQuant to process alignments and produce pathogen_al_mapped_sort.bam
    inputs for htseq counting
- name: htseq
  role: htseq-count with --intersection-nonempty assigns mapped reads to individual
    SARS-CoV-2 genes when the GTF is provided
- name: cellranger
  role: Generates the 10x host-alignment output whose unaligned reads are the substrate
    for pathogen mapping
- name: Python
  role: Used to load the per-gene count tables, restrict to the ten SARS-CoV-2 genes,
    and compute per-sample Pearson correlations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:e09b13e9db6b838394a4902335f39671fef8cc53cbbd69d4c120bb74fa869583
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# sars-cov-2-gene-annotation-mapping

## Summary

Maps scPathoQuant-extracted SARS-CoV-2 reads to the annotated SARS-CoV-2 genome (NCBI GCF_009858895.2) and quantifies per-gene UMI counts, enabling aligner-consistency checks such as Pearson correlation of per-gene counts between bbmap and bowtie2 runs across the ten annotated SARS-CoV-2 genes in HBEC dpi samples.

## When to use

Use when scPathoQuant per-gene pathogen UMI outputs exist for the same 10x samples under two alignment options (bbmap default and -align bowtie2) and you need to verify that per-gene SARS-CoV-2 UMI quantifications are consistent across aligners.

## When NOT to use

- The scPathoQuant run was performed without a pathogen GTF — per-gene htseq output (pathogen_genes_al_sort_counts.sam) will not exist, so only total pathogen counts can be compared
- You need to quantify host-cell gene expression; this skill covers pathogen (SARS-CoV-2) gene counts only
- Comparing scPathoQuant against other tools such as ViralTrack, Venus, or PathogenTrack — that benchmarking workflow uses different per-tool scripts

## Inputs

- Per-gene pathogen UMI outputs from scPathoQuant bbmap run per HBEC dpi sample (pathogen_al_gene_counts.csv / pathogen_genes_al_sort_counts.sam)
- Per-gene pathogen UMI outputs from scPathoQuant bowtie2 run (-align bowtie2) per HBEC dpi sample
- SARS-CoV-2 reference genome FASTA and GTF from NCBI GenBank assembly GCF_009858895.2
- 10x cellranger count output folders (filtered_feature_bc_matrix / raw_feature_bc_matrix) for each sample

## Outputs

- gene_umi_correlations.csv — table of per-sample Pearson correlations of the ten SARS-CoV-2 gene UMI counts between bbmap and bowtie2
- Paired per-gene UMI vectors per sample restricted to the ten annotated SARS-CoV-2 genes

## How to apply

Run scPathoQuant twice per HBEC dpi sample against the SARS-CoV-2 GCF_009858895.2 genome plus its GTF so that htseq produces per-gene counts (pathogen_genes_al_sort_counts.sam and pathogen_al_gene_counts.csv); the GTF is required because per-gene htseq counts are not produced without it. Load the per-gene UMI count tables from the bbmap and bowtie2 runs, restrict both to the ten annotated SARS-CoV-2 genes, and match gene identifiers between the two tables to form paired per-gene UMI vectors per sample. Compute the Pearson correlation of the paired per-gene UMI vectors for each sample and assemble the per-sample correlations into gene_umi_correlations.csv. The rationale is that alignment-based quantification should be robust to the choice of aligner, so high, significant per-gene correlations (matching the Fig. 3J aligner-consistency pattern) validate the quantification step.

## Related tools

- **scPathoQuant** (Extracts unaligned host reads, maps them to the SARS-CoV-2 genome with bbmap or bowtie2, and produces per-gene UMI count outputs) — https://github.com/galelab/scPathoQuant
- **bowtie2** (Alternative aligner invoked via -align bowtie2 (e.g. with --very-sensitive --non-deterministic) to generate the comparison per-gene counts)
- **samtools** (Used by scPathoQuant to process alignments and produce pathogen_al_mapped_sort.bam inputs for htseq counting)
- **htseq** (htseq-count with --intersection-nonempty assigns mapped reads to individual SARS-CoV-2 genes when the GTF is provided)
- **cellranger** (Generates the 10x host-alignment output whose unaligned reads are the substrate for pathogen mapping)
- **Python** (Used to load the per-gene count tables, restrict to the ten SARS-CoV-2 genes, and compute per-sample Pearson correlations)

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/viral/fastafilefolder -align bowtie2 --bowtie2_params="--very-sensitive --non-deterministic"
```

## Evaluation signals

- Each HBEC dpi sample's Pearson correlation across the ten SARS-CoV-2 genes is high and statistically significant, matching the aligner-consistency pattern reported in Fig. 3J
- pathogen_genes_al_sort_counts.sam exists for both the bbmap and bowtie2 runs — it is only produced when the SARS-CoV-2 GTF is supplied
- Gene identifiers from the bbmap and bowtie2 per-gene tables match one-to-one after identifier harmonization, yielding ten paired values per sample
- gene_umi_correlations.csv contains one correlation row per dpi sample with no missing samples
- Violin plots pathogen_copy.png and pathogen_genes.png confirm cells carry non-zero pathogen gene UMI counts before correlation is computed

## Limitations

- Per-gene htseq counts require the pathogen GTF; without it scPathoQuant only reports total pathogen counts, so gene-level correlation is impossible
- The temporary _tmp/ directory of files used by scpathoquant can be large and must be removed with --tmp_removal if space is an issue
- The fasta header is used to quantify total pathogen reads, so complicated headers should be simplified (e.g. > HIV_virus) before quantification

## Evidence

- [readme] GTF requirement: "pathogen_genes_al_sort_counts.sam - htseq output reads mapping to individual pathogen genes (will not be produced if pathogen/viral gtf is not provided)"
- [readme] aligner options: "-align = alignment tool bbmap (default) or bowtie2"
- [readme] htseq parameters: "The software uses outside software samtools, bowtie2, and htseq to quantify pathogen reads.  Default parameters are used when aligning reads to the pathogen genome and and non default htseq-count"
- [methods] SARS-CoV-2 reference: "The Sars-CoV-2 reference genome and gtf file can be found [here](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_009858895.2/)"
- [abstract] per-gene quantification workflow: "quantifies total reads mapping to the entire pathogen"
- [readme] bowtie2 parameters: "-p2genome path/to/viral/fastafilefolder -align bowtie2 --bowtie2_params="--very-sensitive --non-deterministic""
