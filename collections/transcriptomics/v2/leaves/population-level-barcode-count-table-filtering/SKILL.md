---
name: population-level-barcode-count-table-filtering
description: Use when you have a raw barcode counts matrix (e.g. from BARtab output
  on SPLINTR/GEO data) and need to remove low-quality samples and low-abundance barcodes
  before downstream barcoding analysis.
when_to_use_negative:
- Input is still raw FASTQ or per-read barcode extractions rather than a tabulated
  counts matrix — run the BARtab pipeline first
- The analysis is single-cell or spatial barcode data, where per-cell UMI-based filters
  (e.g. umi_count_filter, umi_fraction_filter) rather than bulk per-sample thresholds
  apply
- A stricter barcode filter stringency is required by the study design — the <10 reads
  in <4 samples rule is explicitly low-stringency
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3308
tools:
- name: bartools
  role: Open-source R package providing the population-level barcode filtering/QC
    workflows applied to the dose-escalation counts table
  repo: https://github.com/DaneVass/bartools
- name: R
  role: Language in which the per-sample 5th-percentile outlier flag and the >=10
    reads in >=4 samples barcode filter are implemented with tidyverse
- name: BARtab
  role: Upstream Nextflow pipeline that generates the barcode counts table this filtering
    skill operates on
  repo: https://github.com/DaneVass/BARtab
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:9f8f58b6a9f70ac946d33b18e78e34c6f8e0cdbe47ceaf7b132ce9ba159d37bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/bartools
  tools:
  - BARtab
  - R
  - bartools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# population-level-barcode-count-table-filtering

## Summary

Filters a population-level cellular barcoding counts table (barcodes x samples) with bartools-style QC: a sample-level read-count outlier removal (5th-percentile total reads) followed by a per-barcode low-abundance filter, as demonstrated on the SPLINTR dose-escalation dataset from GSE246611 to yield 1,680 high-confidence barcodes across 41 samples.

## When to use

Use when you have a raw barcode counts matrix (e.g. from BARtab output on SPLINTR/GEO data) and need to remove low-quality samples and low-abundance barcodes before downstream barcoding analysis.

## When NOT to use

- Input is still raw FASTQ or per-read barcode extractions rather than a tabulated counts matrix — run the BARtab pipeline first
- The analysis is single-cell or spatial barcode data, where per-cell UMI-based filters (e.g. umi_count_filter, umi_fraction_filter) rather than bulk per-sample thresholds apply
- A stricter barcode filter stringency is required by the study design — the <10 reads in <4 samples rule is explicitly low-stringency

## Inputs

- Raw population-level barcode counts table (barcodes x samples, TSV) from BARtab, e.g. GEO accession GSE246611 dose-escalation dataset (1,790 barcodes x 47 samples pre-filter)

## Outputs

- Filtered counts matrix of 1,680 high-confidence barcodes x 41 samples written as dose_escalation_filtered_counts.tsv (TSV format)

## How to apply

Load the raw counts table (e.g. GEO accession GSE246611) into R with tidyverse, then compute per-sample total read counts and flag samples below the 5th percentile as low-quality outliers to remove (6 samples in the dose-escalation study). Next apply the low-stringency per-barcode threshold, retaining only barcodes with at least 10 reads in at least 4 samples, which removes the 110 low-abundance barcodes reported in the study. The rationale is that sample-level outlier removal protects downstream comparisons from failed sequencing, while the per-barcode rule removes barcodes too sparse to distinguish true lineages from noise. Write the filtered matrix (1,680 barcodes x 41 samples for the reference dataset) to dose_escalation_filtered_counts.tsv in TSV format. Use bartools, the open-source R package from the BARtab/bartools workflow, as the reference implementation of these population-level filtering steps.

## Related tools

- **bartools** (Open-source R package providing the population-level barcode filtering/QC workflows applied to the dose-escalation counts table) — https://github.com/DaneVass/bartools
- **R** (Language in which the per-sample 5th-percentile outlier flag and the >=10 reads in >=4 samples barcode filter are implemented with tidyverse)
- **BARtab** (Upstream Nextflow pipeline that generates the barcode counts table this filtering skill operates on) — https://github.com/DaneVass/BARtab

## Examples

```
Rscript -e "library(tidyverse); c <- read.delim('GSE246611_counts.tsv', row.names=1); keep <- rowSums(c >= 10) >= 4; f <- c[keep, colSums(c) >= quantile(colSums(c), 0.05)]; write.table(f, 'dose_escalation_filtered_counts.tsv', sep='\t')"
```

## Evaluation signals

- Output file dose_escalation_filtered_counts.tsv exists and parses as a valid TSV
- Dimensions of the filtered matrix equal 1,680 barcodes x 41 samples for the GSE246611 dose-escalation dataset
- Exactly 6 samples are removed by the 5th-percentile total-read-count outlier filter
- No retained barcode has fewer than 10 reads in 4 or more of the 41 retained samples
- Retained barcodes plus the 110 removed low-abundance barcodes reconcile with the pre-filter barcode count

## Limitations

- The <10 reads in <4 samples cutoff is a low-stringency choice; stricter thresholds would discard more barcodes and change dataset dimensions
- The 5th-percentile sample cutoff is dataset-dependent — sample counts and percentile outcomes will differ on other datasets
- The bartools/BARtab authors note there is no accepted gold-standard pipeline for population-level barcoding analysis, so filter parameters remain a researcher decision rather than a validated standard

## Evidence

- [intro] bartools package supports population-level filtering: "we developed bartools, a flexible open-source R package that incorporates workflows for population-level cellular barcoding data analysis"
- [readme] readme points to dose-escalation analysis code: [evidence span withheld — anchor status: external_doc]
- [abstract] workflow covers QC of lineage barcodes: "BARtab and bartools contain methods to simplify the extraction, quality control, analysis, and visualization of lineage barcodes"
- [results] per-barcode filter yields reference dataset dimensions: [evidence span withheld — anchor status: not_found]
