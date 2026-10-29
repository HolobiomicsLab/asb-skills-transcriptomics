---
name: cohort-ecdna-statistics-aggregation
description: Use when per-sample circlehunter outputs exist for many ATAC-Seq tumor
  samples and you need cohort-level counts, ecDNA-positive sample rates, and size/chromosome/segment
  distributions across cancer types.
when_to_use_negative:
- When analyzing a single sample or cell line rather than a cohort — aggregation statistics
  require many samples to be meaningful
- When the per-sample calls have not been generated yet (this skill aggregates circlehunter
  outputs; run the Snakemake pipeline first)
- When samples lack cancer type annotations, preventing meaningful per-cancer-type
  stratification (the article notes the analyzed set was mostly cell lines with skewed
  cancer type proportions)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3518
tools:
- name: circlehunter
  role: Generates the per-sample ecDNA predictions that are aggregated into cohort-level
    statistics; runs as a Snakemake pipeline over the configured ATAC-Seq samples
  repo: https://github.com/suda-huanglab/circlehunter
- name: Snakemake
  role: Workflow engine executing the circlehunter pipeline across all cohort samples
    with -j 16 parallelism before aggregation
- name: GSEApy
  role: Python library referenced alongside circlehunter for downstream gene-level
    analysis (e.g., enrichment of genes carried on ecDNA via enrichr/biomart subcommands)
  repo: https://github.com/zqfang/GSEApy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41389-023-00476-0
    title: 'Circlehunter: a tool to identify extrachromosomal circular DNA from ATAC-Seq
      data'
  source_content_hash: sha256:bbb421fe97f509f1ecd10a1f03c8597d37d4d850dfe02d84f45ce2ea334cb55e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/suda-huanglab/circlehunter
  tools:
  - GSEApy
  - Snakemake
  - circlehunter
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cohort-ecdna-statistics-aggregation

## Summary

Aggregate per-sample circlehunter ecDNA predictions across a curated cohort of public ATAC-Seq tumor samples into cohort-level statistics — total ecDNA count, number and percentage of ecDNA-positive samples, and ecDNA size, chromosome, and segment-count distributions — to quantify the landscape of extrachromosomal circular DNA across cancer types.

## When to use

Use when per-sample circlehunter outputs exist for many ATAC-Seq tumor samples and you need cohort-level counts, ecDNA-positive sample rates, and size/chromosome/segment distributions across cancer types.

## When NOT to use

- When analyzing a single sample or cell line rather than a cohort — aggregation statistics require many samples to be meaningful
- When the per-sample calls have not been generated yet (this skill aggregates circlehunter outputs; run the Snakemake pipeline first)
- When samples lack cancer type annotations, preventing meaningful per-cancer-type stratification (the article notes the analyzed set was mostly cell lines with skewed cancer type proportions)

## Inputs

- Per-sample circlehunter ecDNA prediction outputs (one per ATAC-Seq sample, as configured in config.yaml)
- Cohort sample metadata: curated GEO ATAC-Seq sample list with output paths and cancer type labels
- Gene annotations (e.g., from Enrichr/biomart resources) for intersecting ecDNA intervals with oncogenes

## Outputs

- Cohort-level statistics table: total ecDNA count, number and percentage of ecDNA-positive samples, number of cancer types represented
- ecDNA size distribution, chromosome distribution, and segment-count distribution
- Per-cancer-type ecDNA prevalence breakdown (e.g., Supplementary Table S3 style)
- List of oncogenes contained within predicted ecDNAs

## How to apply

After running the circlehunter Snakemake pipeline (installed via mamba from environment.yml, launched as `snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml`) on all curated samples listed in config.yaml, collect the per-sample ecDNA predictions into a single table. Tally the total number of candidate ecDNA (or microDNA) circular structures across the cohort, count how many samples carry at least one prediction, and compute the ecDNA-positive sample percentage (in the reference run: 1312 ecDNAs in 94 of 547 samples, 17.18%, spanning 17 cancer types). Then stratify predictions by ecDNA size, chromosome, and segment count to build distributions, and intersect ecDNA genomic intervals with gene annotations to identify contained oncogenes (37 oncogenes with amplification characteristics in the reference cohort, including MYC, MYCN, and EGFR). Finally, cross-tabulate ecDNA-positive rates by cancer type to compare prevalence, noting that sample composition (mostly cell lines) affects the proportions.

## Related tools

- **circlehunter** (Generates the per-sample ecDNA predictions that are aggregated into cohort-level statistics; runs as a Snakemake pipeline over the configured ATAC-Seq samples) — https://github.com/suda-huanglab/circlehunter
- **Snakemake** (Workflow engine executing the circlehunter pipeline across all cohort samples with -j 16 parallelism before aggregation)
- **GSEApy** (Python library referenced alongside circlehunter for downstream gene-level analysis (e.g., enrichment of genes carried on ecDNA via enrichr/biomart subcommands)) — https://github.com/zqfang/GSEApy

## Examples

```
snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml
```

## Evaluation signals

- Total ecDNA count should equal the sum of per-sample prediction counts (reference cohort: 1312 candidate ecDNA/microDNA circular structures)
- EcDNA-positive sample percentage should be reproducible: 94 of 547 curated GEO ATAC-Seq tumor samples = 17.18% in the reference run
- Cancer type coverage should match the metadata (17 cancers in the reference cohort); each ecDNA must map to exactly one sample and one cancer type
- Size distribution should reflect ecDNA's expected >1 Mb scale (with microDNA-scale calls labeled separately), and segment-count/chromosome distributions should be internally consistent with per-sample outputs
- Oncogene intersection should recover known ecDNA oncogenes such as MYC, MYCN, and EGFR when present in the cohort

## Limitations

- Accuracy of ecDNA calls for all types of focal amplifications cannot be guaranteed; the current benchmarking strategy has limitations, so aggregated counts may include false positives from other misregulated-chromatin focal amplifications
- Most analyzed samples were cell lines and cancer type proportions deviated significantly from each other, so cohort-level prevalence statistics are composition-biased and additional data are needed
- Short read-length sequencing may lead circlehunter to discover more computational sub-structures (e.g., COLO320DM), which can inflate per-sample and cohort ecDNA counts

## Evidence

- [other] In total, 1312 candidate ecDNA (or microDNA) circular structures were predicted in 94 (17.18%) samples of 17 cancers: "In total, 1312 candidate ecDNA (or microDNA) circular structures were predicted in 94 (17.18%) samples of 17 cancers (Supplementary Table S3) by circlehunter."
- [abstract] Based on 1312 ecDNAs predicted from 94 publicly available datasets of ATAC-Seq assays, we found 37 oncogenes contained in these ecDNAs: "Based on 1312 ecDNAs predicted from 94 publicly available datasets of ATAC-Seq assays, we found 37 oncogenes contained in these ecDNAs"
- [methods] snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml: "snakemake --snakefile ../Snakefile -j 16 --configfile config.yaml"
- [discussion] most of the samples in the dataset we analyzed were cell lines, and the proportion of individual cancer types deviated significantly from each other: "most of the samples in the dataset we analyzed were cell lines, and the proportion of individual cancer types deviated significantly from each other. Additional data are needed"
- [intro] ecDNA is typically found only in cancer samples, typically has a size over 1 million base pairs, and contains both genes and noncoding DNA, including regulatory regions: "ecDNA is typically found only in cancer samples, typically has a size over 1 million base pairs, and contains both genes and noncoding DNA, including regulatory regions"
