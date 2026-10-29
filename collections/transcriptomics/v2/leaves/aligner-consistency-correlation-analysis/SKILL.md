---
name: aligner-consistency-correlation-analysis
description: Use when two aligner runs of scPathoQuant (-align bbmap vs -align bowtie2)
  have produced per-gene pathogen UMI outputs for the same samples and you need to
  quantify their per-gene count consistency.
when_to_use_negative:
- The scPathoQuant run was performed without a pathogen GTF, so per-gene outputs (pathogen_genes_al_sort_counts.sam)
  were not produced and only whole-pathogen counts exist
- The two runs to compare were not performed on the same samples with the same pathogen
  reference genome, so per-gene vectors are not paired
- The goal is benchmarking against external tools (ViralTrack, Venus, PathogenTrack)
  rather than internal aligner consistency
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0769
tools:
- name: scPathoQuant
  role: Generates the per-gene pathogen UMI count outputs (pathogen_al_gene_counts.csv
    / pathogen_genes_al_sort_counts.sam) under both -align bbmap and -align bowtie2
    for correlation
  repo: https://github.com/galelab/scPathoQuant
- name: bowtie2
  role: One of the two aligners being compared, invoked via scPathoQuant -align bowtie2
- name: htseq
  role: Produces the per-gene htseq-count output reads mapping to individual pathogen
    genes that supply the count vectors being correlated
- name: cellranger
  role: Upstream host-genome alignment whose unaligned reads are fed to both scPathoQuant
    aligner runs
- name: Python
  role: Used to load both per-gene UMI count tables, restrict to the ten SARS-CoV-2
    genes, match gene identifiers, compute Pearson correlations, and write gene_umi_correlations.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:3d28d96d259662620d789de57759307c34a2fab2957aa386ee4823acdaf8f9f4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - Python
  - bowtie2
  - cellranger
  - htseq
  - scPathoQuant
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# aligner-consistency-correlation-analysis

## Summary

Validates that scPathoQuant's two alignment backends (bbmap and bowtie2) yield concordant per-gene pathogen UMI quantifications by computing per-sample Pearson correlations of gene-level UMI count vectors across the ten annotated SARS-CoV-2 genes.

## When to use

Use when two aligner runs of scPathoQuant (-align bbmap vs -align bowtie2) have produced per-gene pathogen UMI outputs for the same samples and you need to quantify their per-gene count consistency.

## When NOT to use

- The scPathoQuant run was performed without a pathogen GTF, so per-gene outputs (pathogen_genes_al_sort_counts.sam) were not produced and only whole-pathogen counts exist
- The two runs to compare were not performed on the same samples with the same pathogen reference genome, so per-gene vectors are not paired
- The goal is benchmarking against external tools (ViralTrack, Venus, PathogenTrack) rather than internal aligner consistency

## Inputs

- pathogen_al_gene_counts.csv from the scPathoQuant bbmap run per HBEC dpi sample
- pathogen_al_gene_counts.csv from the scPathoQuant bowtie2 run (-align bowtie2) per HBEC dpi sample
- pathogen_genes_al_sort_counts.sam (htseq per-gene output) from each aligner run
- SARS-CoV-2 GTF annotation (GCF_009858895.2) defining the ten annotated SARS-CoV-2 genes

## Outputs

- gene_umi_correlations.csv — per-sample Pearson correlations of per-gene UMI quantifications between bbmap and bowtie2 runs
- Paired per-gene UMI count vectors restricted to the ten SARS-CoV-2 genes per sample

## How to apply

Run scPathoQuant twice per sample with the SARS-CoV-2 GCF_009858895.2 genome and GTF so that per-gene htseq outputs are produced (pathogen_al_gene_counts.csv / pathogen_genes_al_sort_counts.sam), once with the default bbmap aligner and once with -align bowtie2. Load both per-gene UMI count tables per sample, restrict them to the ten annotated SARS-CoV-2 genes, and match gene identifiers between the bbmap and bowtie2 tables to form paired per-gene UMI vectors. Compute the Pearson correlation of per-gene UMI quantifications between the two aligner runs for each sample. Assemble the per-sample correlations into a table saved as gene_umi_correlations.csv; a high, statistically significant correlation per sample indicates aligner consistency.

## Related tools

- **scPathoQuant** (Generates the per-gene pathogen UMI count outputs (pathogen_al_gene_counts.csv / pathogen_genes_al_sort_counts.sam) under both -align bbmap and -align bowtie2 for correlation) — https://github.com/galelab/scPathoQuant
- **bowtie2** (One of the two aligners being compared, invoked via scPathoQuant -align bowtie2)
- **htseq** (Produces the per-gene htseq-count output reads mapping to individual pathogen genes that supply the count vectors being correlated)
- **cellranger** (Upstream host-genome alignment whose unaligned reads are fed to both scPathoQuant aligner runs)
- **Python** (Used to load both per-gene UMI count tables, restrict to the ten SARS-CoV-2 genes, match gene identifiers, compute Pearson correlations, and write gene_umi_correlations.csv)

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/sarscov2/genomefiles -align bowtie2 --bowtie2_params="--very-sensitive --non-deterministic"
```

## Evaluation signals

- Each sample's Pearson correlation across the ten SARS-CoV-2 genes between bbmap and bowtie2 counts is high and statistically significant
- The per-sample correlation pattern matches the aligner-consistency result reported in Fig. 3J of the paper
- Gene identifiers successfully matched between the bbmap and bowtie2 per-gene tables for all ten annotated genes in every sample
- gene_umi_correlations.csv contains one correlation value per HBEC dpi sample with no missing samples

## Limitations

- Per-gene htseq output (pathogen_genes_al_sort_counts.sam) is only produced if a pathogen/viral GTF is provided alongside the FASTA; without it only whole-pathogen counts exist and gene-level correlation is impossible
- Correlation across only ten genes gives limited statistical power compared with host-gene-level comparisons
- Correlation is computed on default alignment parameters (bbmap default; non-default htseq-count --intersection-nonempty); results may differ if --bbmap_params or --bowtie2_params (e.g. --very-sensitive --non-deterministic) are changed

## Evidence

- [methods] bbmap and bowtie2 aligner consistency: "-align = alignment tool bbmap (default) or bowtie2"
- [readme] per-gene output availability: "pathogen_genes_al_sort_counts.sam - htseq output reads mapping to individual pathogen genes (will not be produced if pathogen/viral gtf is not provided)"
- [readme] per-gene counts definition: "pathogen_al_gene_counts.csv - number of reads mapping to pathoen genes in each cell"
- [methods] SARS-CoV-2 reference with GTF: "The Sars-CoV-2 reference genome and gtf file can be found [here](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_009858895.2/)"
- [readme] htseq-count non-default parameter: "Default parameters are used when aligning reads to the pathogen genome and and non default htseq-count include using the --intersection-nonempty parameter."
