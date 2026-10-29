---
name: bulk-barcode-counts-normalization
description: Use when a bulk barcode counts table (e.g., counts/all_counts_combined.tsv
  from BARtab) must be normalized across samples and treatments compared by clonal
  composition, diversity, or replicate concordance.
when_to_use_negative:
- Input is raw sequencing FASTQ or unprocessed barcode reads — pre-processing belongs
  in the BARtab Nextflow pipeline, not this downstream R skill
- Data are single-cell or spatial barcoding datasets — this skill targets population-level
  (bulk) counts tables
- The goal is lineage reconstruction with heritable-editing barcodes (e.g., Cassiopeia
  or LinRace workflows), which is explicitly outside the synthetic-barcode counts
  analysis scope
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: bartools
  role: R package providing normalization workflows and Shannon/Simpson/Gini diversity
    functions for population-level barcode counts tables
  repo: https://github.com/DaneVass/bartools/
- name: R
  role: Execution environment for tidyverse-based loading, log-normalization, prcomp
    PCA, and replicate correlation analysis
- name: tidyverse
  role: Used to load and manipulate the counts/all_counts_combined.tsv counts table
    prior to analysis
- name: BARtab
  role: Upstream Nextflow pipeline that generates the bulk barcode counts table consumed
    by this normalization/analysis skill
  repo: https://github.com/DaneVass/BARtab/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:dc1c5909056bb07b7abf277acb042b1b6c127b0ad65dba44b46d2e5d7a813038
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/bartools
  tools:
  - BARtab
  - R
  - bartools
  - tidyverse
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# bulk-barcode-counts-normalization

## Summary

Normalization and downstream global clonal analysis of population-level synthetic barcode counts tables using bartools and tidyverse in R, enabling PCA, diversity-index, and replicate-correlation comparisons across treatments (e.g., DMSO, IBET-151, AraC).

## When to use

Use when a bulk barcode counts table (e.g., counts/all_counts_combined.tsv from BARtab) must be normalized across samples and treatments compared by clonal composition, diversity, or replicate concordance.

## When NOT to use

- Input is raw sequencing FASTQ or unprocessed barcode reads — pre-processing belongs in the BARtab Nextflow pipeline, not this downstream R skill
- Data are single-cell or spatial barcoding datasets — this skill targets population-level (bulk) counts tables
- The goal is lineage reconstruction with heritable-editing barcodes (e.g., Cassiopeia or LinRace workflows), which is explicitly outside the synthetic-barcode counts analysis scope

## Inputs

- Normalized or raw combined barcode counts table (counts/all_counts_combined.tsv) produced by the BARtab pipeline
- Sample metadata mapping samples to treatment (DMSO, IBET-151, AraC), dose, and time point
- R environment with bartools and tidyverse packages

## Outputs

- PCA scores/plots (PC1 vs PC2) colored by treatment
- Per-sample Shannon, Simpson, and Gini diversity indices with time-course plots
- Pairwise Pearson correlation matrix between biological replicates
- Hierarchical clustering dendrogram of samples by barcode composition

## How to apply

Load the combined barcode counts table (counts/all_counts_combined.tsv) in R with tidyverse and normalize counts across samples before any global analysis. Perform PCA on log-normalized barcode counts with prcomp and plot PC1 vs PC2 colored by treatment (DMSO, IBET-151, AraC); valid runs show treatments separated along PC1 and the dose-escalation time course along PC2. Compute per-sample Shannon, Simpson, and Gini diversity indices with bartools functions and plot diversity trends across time points. Compute pairwise Pearson correlations between biological replicates for DMSO and IBET-151 samples, requiring all correlations to exceed 0.95. Finally, hierarchically cluster samples on correlation dissimilarity of barcode composition to confirm that AraC IC90 samples form a distinct cluster from gradual dose-escalation samples.

## Related tools

- **bartools** (R package providing normalization workflows and Shannon/Simpson/Gini diversity functions for population-level barcode counts tables) — https://github.com/DaneVass/bartools/
- **R** (Execution environment for tidyverse-based loading, log-normalization, prcomp PCA, and replicate correlation analysis)
- **tidyverse** (Used to load and manipulate the counts/all_counts_combined.tsv counts table prior to analysis)
- **BARtab** (Upstream Nextflow pipeline that generates the bulk barcode counts table consumed by this normalization/analysis skill) — https://github.com/DaneVass/BARtab/

## Examples

```
library(tidyverse); counts <- read_tsv("counts/all_counts_combined.tsv")
```

## Evaluation signals

- PCA on log-normalized counts shows treatments clearly separated along PC1 and the dose-escalation time course separated along PC2
- All pairwise Pearson correlations between biological replicates for DMSO and IBET-151 samples exceed 0.95
- Hierarchical clustering dendrogram places AraC IC90 samples in a cluster distinct from gradual dose-escalation samples
- Diversity metrics (Shannon, Simpson, Gini) computed per sample yield finite values for every sample and coherent trends across time points
- Input table resolves correctly (barcodes × samples) from counts/all_counts_combined.tsv before normalization is applied

## Limitations

- Validation thresholds such as the > 0.95 replicate correlation are dataset-specific; different experiments may legitimately show lower replicate concordance
- The paper notes there is no accepted gold-standard pipeline for population-level barcoding analysis, so parameter choices (e.g., log-normalization scheme) lack external benchmarking
- Benchmarking on simulated data to guide accuracy and parameter selection was not performed in the manuscript

## Evidence

- [results] By PCA, treatments were clearly separated along PC1, with the dose-escalation time course separated along PC2, and replicate samples displayed similar overall clonal composition with DMSO and IBET-151 correlations all > 0.95.: "By PCA, treatments were clearly separated along PC1, with the dose-escalation time course separated along PC2"
- [methods] Load the normalized combined barcode counts table (counts/all_counts_combined.tsv) in R with the tidyverse and normalize counts across samples.: [evidence span withheld — anchor status: not_found]
- [intro] we developed bartools, a flexible open-source R package that incorporates workflows for population-level cellular barcoding data analysis: "we developed bartools, a flexible open-source R package that incorporates workflows for population-level"
- [readme] Analysis of population-level dose escalation dataset using bartools in dose-escalation/.: [evidence span withheld — anchor status: external_doc]
- [abstract] BARtab and bartools contain methods to simplify the extraction, quality control, analysis, and visualization of lineage barcodes from population-level, single-cell, and spatial transcriptomics: "quality control, analysis, and visualization of lineage barcodes from population-level"
