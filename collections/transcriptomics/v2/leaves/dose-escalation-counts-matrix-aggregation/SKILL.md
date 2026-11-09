---
name: dose-escalation-counts-matrix-aggregation
description: Use when you have a raw bulk barcode counts table (e.g. from GEO GSE246611)
  and need to replicate the dose-escalation QC pipeline that removes low-quality samples
  and low-abundance barcodes before downstream bartools analysis.
when_to_use_negative:
- Input is single-cell or spatial transcriptomics barcode data — this workflow is
  specific to the population-level (bulk) dose-escalation counts matrix; BARtab has
  separate single-cell and spatial modes
- Input is raw FASTQ sequencing data that still needs barcode extraction/enumeration
  — first run the BARtab Nextflow pipeline to generate the counts table
- The counts table has already passed both QC filters — re-applying would not change
  dimensions and risks double-filtering
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: bartools
  role: R package whose sample-level and per-barcode QC filter workflows are reproduced
    here on the dose-escalation counts matrix
  repo: https://github.com/DaneVass/bartools
- name: R + tidyverse
  role: Environment for loading the GSE246611 counts table, computing per-sample totals
    and per-barcode sample-support counts, and writing the filtered TSV
- name: bartools_manuscript_code
  role: Repository containing the dose-escalation analysis scripts that this skill
    reproduces
  repo: https://github.com/DaneVass/bartools_manuscript_code
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:4cf41ed2d5d8b60f9655a516f36380b9c49f0b8231ce0a099b385fc3055ccb76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/bartools
  tools:
  - BARtab
  - R
  - R + tidyverse
  - bartools
  - bartools_manuscript_code
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# dose-escalation-counts-matrix-aggregation

## Summary

Reproduces the bartools sample-level (5th-percentile read-count outlier) and per-barcode (<10 reads in <4 samples) quality-control filtering of a population-level SPLINTR dose-escalation barcode counts table, yielding the high-confidence 1,680-barcode x 41-sample filtered counts matrix reported by the BARtab/bartools manuscript.

## When to use

Use when you have a raw bulk barcode counts table (e.g. from GEO GSE246611) and need to replicate the dose-escalation QC pipeline that removes low-quality samples and low-abundance barcodes before downstream bartools analysis.

## When NOT to use

- Input is single-cell or spatial transcriptomics barcode data — this workflow is specific to the population-level (bulk) dose-escalation counts matrix; BARtab has separate single-cell and spatial modes
- Input is raw FASTQ sequencing data that still needs barcode extraction/enumeration — first run the BARtab Nextflow pipeline to generate the counts table
- The counts table has already passed both QC filters — re-applying would not change dimensions and risks double-filtering

## Inputs

- Raw barcode counts table from the SPLINTR dose-escalation dataset (GEO accession GSE246611), barcodes x samples
- Sample-level QC parameters: 5th-percentile total read-count outlier threshold
- Per-barcode QC parameters: >=10 reads in >=4 samples

## Outputs

- dose_escalation_filtered_counts.tsv — a TSV counts matrix of 1,680 high-confidence barcodes across 41 samples
- List of 6 samples removed by the 5th-percentile read-count outlier filter
- Set of 110 low-abundance barcodes removed by the <10-reads-in-<4-samples filter

## How to apply

Load the dose-escalation raw barcode counts table (GEO accession GSE246611) into R using the tidyverse, then compute per-sample total read counts and flag samples falling below the 5th percentile as low-quality outliers, removing the 6 samples identified by this sample-level bartools filter. Next apply the low-stringency per-barcode threshold, retaining only barcodes with at least 10 reads in at least 4 samples, which eliminates the 110 low-abundance barcodes reported in the study. Write the resulting filtered counts matrix (1,680 barcodes x 41 samples) to dose_escalation_filtered_counts.tsv as a TSV. The rationale is that bartools sample-level outlier detection and per-barcode abundance thresholds jointly define the 'high-confidence' barcode set used for all downstream population-level lineage analyses in the manuscript.

## Related tools

- **bartools** (R package whose sample-level and per-barcode QC filter workflows are reproduced here on the dose-escalation counts matrix) — https://github.com/DaneVass/bartools
- **R + tidyverse** (Environment for loading the GSE246611 counts table, computing per-sample totals and per-barcode sample-support counts, and writing the filtered TSV)
- **bartools_manuscript_code** (Repository containing the dose-escalation analysis scripts that this skill reproduces) — https://github.com/DaneVass/bartools_manuscript_code

## Examples

```
Rscript -e 'counts <- read.delim("gse246611_counts.tsv"); f <- counts[rowSums(counts >= 10) >= 4, ]; write.table(f, "dose_escalation_filtered_counts.tsv", sep="\t", quote=FALSE)'
```

## Evaluation signals

- Filtered output file dose_escalation_filtered_counts.tsv exists and is a valid TSV
- Dimensions of the filtered matrix are exactly 1,680 barcodes x 41 samples, matching the published result
- Exactly 6 samples are removed by the 5th-percentile total-read-count outlier filter
- Exactly 110 barcodes are removed by the <10 reads in <4 samples threshold
- Read counts for retained barcodes are unchanged relative to the raw table (filtering only removes rows/columns, never modifies values)

## Limitations

- The barcode threshold is the low-stringency option; more stringent thresholds would remove additional barcodes and change the final dimensions
- Benchmarking was hindered by the lack of a gold-standard dataset with known barcode abundance, so absolute filter optimality is not established
- The 5th-percentile sample filter is sensitive to the composition of the input sample set; adding or removing samples shifts the percentile cutoff

## Evidence

- [intro] bartools, a flexible open-source R package that incorporates workflows for population-level cellular barcoding data analysis: "we developed bartools, a flexible open-source R package that incorporates workflows for population-level cellular barcoding data analysis"
- [readme] Analysis of population-level dose escalation dataset using bartools in dose-escalation/ folder: [evidence span withheld — anchor status: external_doc]
- [abstract] BARtab and bartools QC of lineage barcodes across population-level and single-cell data: "BARtab and bartools contain methods to simplify the extraction, quality control, analysis, and visualization of lineage barcodes"
