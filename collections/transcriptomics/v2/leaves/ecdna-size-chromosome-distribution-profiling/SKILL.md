---
name: ecdna-size-chromosome-distribution-profiling
description: Use when you have per-sample ecDNA calls from tumor ATAC-Seq (e.g., circlehunter
  predictions over hundreds of GEO samples) and need cohort-level counts, ecDNA-positive
  sample percentages, and size/chromosome/segment distributions.
when_to_use_negative:
- Input is raw ATAC-Seq FASTQ rather than per-sample ecDNA predictions — first run
  the circlehunter pipeline; this skill covers only the cohort-level aggregation step
- You need ecDNA detection accuracy benchmarking or oncogene/gene-set analysis (e.g.,
  GSEApy) rather than cohort counts and distributions
- Samples are not tumor ATAC-Seq (e.g., normal controls or single-cell ATAC-Seq),
  which the article's dataset was not designed for
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: circlehunter
  role: Produces the per-sample ecDNA (or microDNA) circular structure predictions
    that are aggregated into cohort-level statistics
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Orchestrates the per-sample circlehunter runs via the project Snakefile with
    the -j 16 parallelism option before aggregation
  repo: https://snakemake.readthedocs.io/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:7b8db6fed7f3938ae7a70fa9c507bd9411599a6b0a43dc6ca421dfbd3cc0fa49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - Snakemake
  - circlehunter
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ecdna-size-chromosome-distribution-profiling

## Summary

Cohort-level aggregation of per-sample ecDNA predictions (e.g., from circlehunter on tumor ATAC-Seq) into summary statistics: total ecDNA count, number and percentage of ecDNA-positive samples, and ecDNA size, chromosome, and segment-count distributions across cancer types.

## When to use

Use when you have per-sample ecDNA calls from tumor ATAC-Seq (e.g., circlehunter predictions over hundreds of GEO samples) and need cohort-level counts, ecDNA-positive sample percentages, and size/chromosome/segment distributions.

## When NOT to use

- Input is raw ATAC-Seq FASTQ rather than per-sample ecDNA predictions — first run the circlehunter pipeline; this skill covers only the cohort-level aggregation step
- You need ecDNA detection accuracy benchmarking or oncogene/gene-set analysis (e.g., GSEApy) rather than cohort counts and distributions
- Samples are not tumor ATAC-Seq (e.g., normal controls or single-cell ATAC-Seq), which the article's dataset was not designed for

## Inputs

- Curated list of public ATAC-Seq tumor samples (e.g., 547 GEO samples) with sample identifiers and output paths
- config.yaml derived from config-template.yaml listing samples and outputs
- Per-sample circlehunter ecDNA prediction outputs (coordinates, sizes, segment counts)

## Outputs

- Cohort-level ecDNA statistics table (Supplementary-Table-S3-style): total ecDNA count, ecDNA-positive sample count and percentage, cancer types involved
- ecDNA size distribution across the cohort
- ecDNA chromosome distribution
- ecDNA segment-count distribution

## How to apply

Set up a workspace, copy config-template.yaml to config.yaml, and list all curated ATAC-Seq samples and their output paths in the YAML config. Run the circlehunter Snakemake pipeline (snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml) to generate per-sample ecDNA calls, then aggregate them into cohort-level statistics: total ecDNA count, number and percentage of ecDNA-positive samples, and distributions of ecDNA size, chromosome, and segment count, stratified by cancer type. Use -j 16 to bound parallelism across the sample set. In the reference reproduction, 547 curated GEO ATAC-Seq tumor samples yielded 1312 candidate ecDNA (or microDNA) circular structures in 94 (17.18%) samples spanning 17 cancers, which serves as the expected scale and reporting format (Supplementary Table S3).

## Related tools

- **circlehunter** (Produces the per-sample ecDNA (or microDNA) circular structure predictions that are aggregated into cohort-level statistics) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Orchestrates the per-sample circlehunter runs via the project Snakefile with the -j 16 parallelism option before aggregation) — https://snakemake.readthedocs.io/

## Examples

```
snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- Total predicted ecDNA count is reported exactly (reference cohort: 1312 candidate ecDNA/microDNA structures)
- Number and percentage of ecDNA-positive samples are reported (reference: 94 of 547 samples, 17.18%)
- Number of distinct cancer types covered matches the sample panel (reference: 17 cancers)
- Size, chromosome, and segment-count distributions are all computed over the pooled ecDNA set, not left as per-sample outputs
- Pipeline completes via the Snakemake Snakefile with --configfile config.yaml and -j 16

## Limitations

- Most analyzed samples were cell lines, and cancer type proportions deviated significantly from each other, so cohort-level distributions may be skewed; additional data are needed
- circlehunter cannot guarantee accuracy of ecDNA calls for all types of focal amplifications, and other focal amplification classes with misregulated chromatin may appear as false-positive ecDNA in the aggregated counts
- Short read-length sequencing may lead circlehunter to discover more computational sub-structures (e.g., COLO320DM), affecting segment-count distributions

## Evidence

- [results] 1312 candidate ecDNA (or microDNA) circular structures were predicted in 94 (17.18%) samples of 17 cancers (Supplementary Table S3) by circlehunter.: "In total, 1312 candidate ecDNA (or microDNA) circular structures were predicted in 94 (17.18%) samples of 17 cancers (Supplementary Table S3) by circlehunter."
- [abstract] Based on 1312 ecDNAs predicted from 94 publicly available datasets of ATAC-Seq assays, we found 37 oncogenes contained in these ecDNAs: "Based on 1312 ecDNAs predicted from 94 publicly available datasets of ATAC-Seq assays, we found 37 oncogenes contained in these ecDNAs"
- [other] Aggregate per-sample ecDNA predictions into cohort-level statistics including total ecDNA count, number and percentage of ecDNA-positive samples, and ecDNA size, chromosome, and segment-count distributions.: [evidence span withheld — anchor status: not_found]
- [readme] snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [discussion] most of the samples in the dataset we analyzed were cell lines, and the proportion of individual cancer types deviated significantly from each other.: "most of the samples in the dataset we analyzed were cell lines, and the proportion of individual cancer types deviated significantly from each other. Additional data are needed"
