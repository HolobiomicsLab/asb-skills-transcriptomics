---
name: htseq-gene-count-parsing
description: Use when scPathoQuant runs have produced htseq SAM output files (pathogen_al_sort_counts.sam
  / pathogen_genes_al_sort_counts.sam) and you need per-gene or total pathogen count
  tables for downstream comparison or correlation.
when_to_use_negative:
- No pathogen GTF was supplied to scPathoQuant, so pathogen_genes_al_sort_counts.sam
  is absent and only total pathogen counts (pathogen_al_sort_counts.sam / pathogen_copy-level
  counts) exist
- The goal is host-gene expression quantification from cellranger count output rather
  than pathogen-gene counts
- Input is already a per-gene count matrix (e.g. pathogen_al_gene_counts.csv) and
  no re-parsing of the htseq SAM is needed
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: htseq
  role: Produces the htseq-count outputs (pathogen_al_sort_counts.sam, pathogen_genes_al_sort_counts.sam)
    that this skill parses; run with --intersection-nonempty
- name: scPathoQuant
  role: Generates the htseq SAM outputs and the per-cell per-gene matrix (pathogen_al_gene_counts.csv)
    as part of its alignment and quantification run
  repo: https://github.com/galelab/scPathoQuant
- name: samtools
  role: External dependency scPathoQuant uses alongside bowtie2 and htseq to quantify
    pathogen reads
- name: Python
  role: Language for loading the count tables, matching gene identifiers, and computing
    Pearson correlations between aligner runs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:ef357007c531e2ac8ec39694dd31bb3a6969f9cb54a5d6362270a29d3cba672f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - Python
  - bowtie2
  - htseq
  - samtools
  - scPathoQuant
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# htseq-gene-count-parsing

## Summary

Parse and interpret the htseq-count SAM outputs produced by scPathoQuant — pathogen_al_sort_counts.sam for total pathogen mapping and pathogen_genes_al_sort_counts.sam for per-gene counts — to obtain cell- or gene-level pathogen UMI quantifications (e.g. the ten SARS-CoV-2 genes in the GCF_009858895.2 annotation).

## When to use

Use when scPathoQuant runs have produced htseq SAM output files (pathogen_al_sort_counts.sam / pathogen_genes_al_sort_counts.sam) and you need per-gene or total pathogen count tables for downstream comparison or correlation.

## When NOT to use

- No pathogen GTF was supplied to scPathoQuant, so pathogen_genes_al_sort_counts.sam is absent and only total pathogen counts (pathogen_al_sort_counts.sam / pathogen_copy-level counts) exist
- The goal is host-gene expression quantification from cellranger count output rather than pathogen-gene counts
- Input is already a per-gene count matrix (e.g. pathogen_al_gene_counts.csv) and no re-parsing of the htseq SAM is needed

## Inputs

- pathogen_genes_al_sort_counts.sam — htseq output of reads mapping to individual pathogen genes (requires pathogen GTF)
- pathogen_al_sort_counts.sam — htseq output of reads mapping to the pathogen
- pathogen_al_gene_counts.csv — per-cell per-gene pathogen UMI counts
- Pathogen GTF annotation (e.g. SARS-CoV-2 GCF_009858895.2 GTF)

## Outputs

- Parsed per-gene pathogen UMI count table restricted to annotated genes (e.g. the ten SARS-CoV-2 genes)
- Per-sample correlation table such as gene_umi_correlations.csv when comparing aligner runs
- Per-gene paired UMI vectors matched by gene identifier across runs

## How to apply

scPathoQuant runs htseq-count (with the non-default --intersection-nonempty parameter) against the pathogen reference to emit pathogen_al_sort_counts.sam (reads mapping to the pathogen) and pathogen_genes_al_sort_counts.sam (reads mapping to individual pathogen genes, produced only when a pathogen GTF is supplied). Load the per-gene output (paired with pathogen_al_gene_counts.csv for the per-cell gene matrix), restrict to the annotated genes of interest — e.g. the ten annotated SARS-CoV-2 genes from GCF_009858895.2 — and match gene identifiers across runs before forming paired count vectors. The gene-level table is the basis for downstream analyses such as correlating per-gene UMI quantifications between bbmap and bowtie2 aligner runs (Pearson correlation per sample, assembled into gene_umi_correlations.csv). Correct parsing requires confirming a GTF was provided, since the gene-level SAM is 'not ... produced if pathogen/viral gtf is not provided'.

## Related tools

- **htseq** (Produces the htseq-count outputs (pathogen_al_sort_counts.sam, pathogen_genes_al_sort_counts.sam) that this skill parses; run with --intersection-nonempty)
- **scPathoQuant** (Generates the htseq SAM outputs and the per-cell per-gene matrix (pathogen_al_gene_counts.csv) as part of its alignment and quantification run) — https://github.com/galelab/scPathoQuant
- **samtools** (External dependency scPathoQuant uses alongside bowtie2 and htseq to quantify pathogen reads)
- **Python** (Language for loading the count tables, matching gene identifiers, and computing Pearson correlations between aligner runs)

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/sars-cov2/GCF_009858895.2 -align bowtie2 --bowtie2_params="--very-sensitive --non-deterministic"
```

## Evaluation signals

- pathogen_genes_al_sort_counts.sam exists only if a pathogen GTF was provided — absence indicates the run was GTF-less and gene-level parsing is impossible
- Parsed gene-level counts restricted to the ten annotated SARS-CoV-2 genes yield high, statistically significant Pearson correlations between bbmap and bowtie2 runs for each HBEC dpi sample
- Per-gene identifier matching between the two aligner count tables must succeed for all ten genes before paired vectors are formed
- htseq counts must reflect the --intersection-nonempty mode used by scPathoQuant, not default htseq-count parameters
- Output table (e.g. gene_umi_correlations.csv) contains one correlation per sample, consistent with the Fig. 3J aligner-consistency pattern

## Limitations

- pathogen_genes_al_sort_counts.sam 'will not be produced if pathogen/viral gtf is not provided', so gene-level parsing depends on supplying the GTF
- scPathoQuant uses a non-default htseq-count setting ('non default htseq-count include using the --intersection-nonempty parameter'), so counts may differ from default htseq-count recomputation
- Gene identifiers must be matched between count tables from different aligner runs before pairing vectors

## Evidence

- [readme] The software uses outside software samtools, bowtie2, and htseq to quantify pathogen reads.: "The software uses outside software samtools, bowtie2, and htseq to quantify pathogen reads."
- [readme] Non-default htseq-count parameter: "and non default htseq-count include using the --intersection-nonempty parameter."
- [readme] Gene-level SAM requires GTF: "pathogen_genes_al_sort_counts.sam - htseq output reads mapping to individual pathogen genes (will not be produced if pathogen/viral gtf is not provided)"
- [results] Per-gene correlation between aligner runs: [evidence span withheld — anchor status: not_found]
- [methods] SARS-CoV-2 reference genome source: "The Sars-CoV-2 reference genome and gtf file can be found [here](https://www.ncbi.nlm.nih.gov/datasets/genome/GCF_009858895.2/)"
