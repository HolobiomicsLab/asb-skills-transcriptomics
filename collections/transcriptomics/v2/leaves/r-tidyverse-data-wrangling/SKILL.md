---
name: r-tidyverse-data-wrangling
description: Use when an R-readable barcode counts table (e.g. a SPLINTR/GSE246611
  dose-escalation matrix) needs sample-level and per-barcode QC filtering or reshaping
  prior to bartools downstream analysis.
when_to_use_negative:
- Input is raw FASTQ/BAM data still needing barcode extraction — use the BARtab Nextflow
  pipeline instead of direct R wrangling
- Barcodes require clustering or error correction (Levenshtein distance, UMI collapsing)
  — these are handled upstream by BARtab's starcode/cutadapt modules
- Single-cell or spatial transcriptomics barcode matrices requiring cell-level annotation
  rather than sample-level filtering
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Runtime language for loading, filtering, and writing the barcode counts tables
- name: tidyverse
  role: readr/dplyr verbs used to compute per-sample totals, apply the 5th-percentile
    and per-barcode thresholds, and export TSV output
- name: bartools
  role: R package whose sample-level and per-barcode QC filtering conventions the
    tidyverse wrangling reproduces for downstream population-level analysis
  repo: https://github.com/DaneVass/bartools/
- name: BARtab
  role: Upstream Nextflow pipeline that generates the barcode counts tables that this
    R wrangling step filters
  repo: https://github.com/DaneVass/BARtab/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:e4ef4038612f466a2264121ecd581983875865f94ae61a65550ffe2570c9664e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/bartools
  tools:
  - BARtab
  - R
  - bartools
  - tidyverse
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# r-tidyverse-data-wrangling

## Summary

R/tidyverse-based filtering and reshaping of synthetic barcode counts tables, exemplified by reproducing the SPLINTR dose-escalation QC (5th-percentile sample filter plus per-barcode <10 reads in <4 samples filter) from the BARtab/bartools workflow.

## When to use

Use when an R-readable barcode counts table (e.g. a SPLINTR/GSE246611 dose-escalation matrix) needs sample-level and per-barcode QC filtering or reshaping prior to bartools downstream analysis.

## When NOT to use

- Input is raw FASTQ/BAM data still needing barcode extraction — use the BARtab Nextflow pipeline instead of direct R wrangling
- Barcodes require clustering or error correction (Levenshtein distance, UMI collapsing) — these are handled upstream by BARtab's starcode/cutadapt modules
- Single-cell or spatial transcriptomics barcode matrices requiring cell-level annotation rather than sample-level filtering

## Inputs

- Raw synthetic barcode counts table (TSV/CSV matrix, barcodes x samples), e.g. SPLINTR dose-escalation counts from GEO GSE246611
- Sample metadata mapping sample IDs to dose/time-point conditions (optional, for grouped filtering)

## Outputs

- Filtered counts matrix of 1,680 high-confidence barcodes across 41 samples (dose_escalation_filtered_counts.tsv)
- Per-sample total read counts with 5th-percentile low-quality outlier flags

## How to apply

Load the raw barcode counts table (e.g. GEO accession GSE246611) into R with readr from the tidyverse. Compute per-sample total read counts, flag samples below the 5th percentile as low-quality outliers, and remove them (6 samples in the reference dataset). Apply the low-stringency per-barcode threshold — retain only barcodes with at least 10 reads in at least 4 samples — which drops the 110 low-abundance barcodes reported in the study. Write the filtered matrix (1,680 barcodes x 41 samples) as a TSV (dose_escalation_filtered_counts.tsv) and validate its dimensions against the published result. The rationale is that bartools is an open-source R package designed for population-level cellular barcoding analysis, so tidyverse data frames are the native input format for its QC and visualization workflows.

## Related tools

- **R** (Runtime language for loading, filtering, and writing the barcode counts tables)
- **tidyverse** (readr/dplyr verbs used to compute per-sample totals, apply the 5th-percentile and per-barcode thresholds, and export TSV output)
- **bartools** (R package whose sample-level and per-barcode QC filtering conventions the tidyverse wrangling reproduces for downstream population-level analysis) — https://github.com/DaneVass/bartools/
- **BARtab** (Upstream Nextflow pipeline that generates the barcode counts tables that this R wrangling step filters) — https://github.com/DaneVass/BARtab/

## Examples

```
library(tidyverse); counts <- read_tsv("GSE246611_counts.tsv"); write_tsv(filtered_counts, "dose_escalation_filtered_counts.tsv")
```

## Evaluation signals

- Output file dose_escalation_filtered_counts.tsv exists and parses as a valid TSV
- Filtered matrix dimensions equal exactly 1,680 barcodes x 41 samples, matching the published result
- Exactly 6 samples are removed by the 5th-percentile total-read-count outlier filter
- Exactly 110 low-abundance barcodes are removed by the <10 reads in <4 samples threshold
- Every retained barcode has at least 10 reads in at least 4 samples (invariant check on the output matrix)

## Limitations

- The 5th-percentile sample cutoff and <10-reads-in-<4-samples barcode threshold are dataset-specific choices; other experiments may need different stringency
- This wrangling performs no barcode error correction or clustering — collapsing of similar sequences must occur upstream in BARtab (default Levenshtein distance 3)
- Results depend on the upstream counts table quality; per the article, no gold-standard dataset with known barcode abundance exists for validating absolute counts

## Evidence

- [results] Sample-level and per-barcode QC filtering of the SPLINTR dose-escalation dataset: [evidence span withheld — anchor status: not_found]
- [intro] bartools is the R package whose workflow this wrangling supports: "we developed bartools, a flexible open-source R package that incorporates workflows for population-level cellular barcoding data analysis"
- [readme] The dose-escalation analysis is performed in R with bartools: [evidence span withheld — anchor status: external_doc]
- [abstract] BARtab produces the counts tables that the R wrangling step filters: "we present BARtab, a portable and scalable Nextflow pipeline"
