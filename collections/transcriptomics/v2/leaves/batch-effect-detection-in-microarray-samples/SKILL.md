---
name: batch-effect-detection-in-microarray-samples
description: Use when normalized microarray intensity data (e.g. quantile- or LOESS-normalized
  samples from example_unnorm.txt) must be screened for systematic processing artifacts
  such as date, operator, or reagent lot effects before downstream analysis.
when_to_use_negative:
- Input data is raw and has not been preprocessed/normalized with PIMENTo's Preprocess
  Data command — the dendrogram should be judged on normalized data (quantile or LOESS)
- Samples come from a single processing batch with no technical metadata to compare
  clusters against, so clustering cannot distinguish batch from biological structure
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: PIMENTo
  role: Generates the dendrogram of all chosen samples using Euclidean distances and
    Ward's clustering; also supports batch-effect removal via limma
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: limma
  role: R/Bioconductor package used within PIMENTo for quantile normalization as well
    as removing batch effects, if necessary
- name: RStudio
  role: IDE for running the PIMENTo commands with autocomplete capability
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:2c01fd40693e78e9243fe804defbf3995e948c7ec8ff0dc2b56c49831724d71d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PIMENTo
  - R
  - RStudio
  - affy
  - limma
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Batch-effect detection in microarray samples

## Summary

Detects batch effects in microarray expression data by using PIMENTo to generate a dendrogram of all chosen samples based on Euclidean distances with Ward's method, then inspecting whether clusters align with technical variables (date, operator, reagent lot) rather than biological response classes.

## When to use

Use when normalized microarray intensity data (e.g. quantile- or LOESS-normalized samples from example_unnorm.txt) must be screened for systematic processing artifacts such as date, operator, or reagent lot effects before downstream analysis.

## When NOT to use

- Input data is raw and has not been preprocessed/normalized with PIMENTo's Preprocess Data command — the dendrogram should be judged on normalized data (quantile or LOESS)
- Samples come from a single processing batch with no technical metadata to compare clusters against, so clustering cannot distinguish batch from biological structure

## Inputs

- Normalized microarray intensity matrix (e.g. tab-separated example_unnorm.txt with gene IDs, gene symbols, and sample intensity columns)
- Sample metadata needed to interpret clustering (response class, processing date, operator, reagent lot)

## Outputs

- Dendrogram plot of all chosen samples clustered by Euclidean distance with Ward's method
- Qualitative verdict on whether a batch effect is present in the samples

## How to apply

After preprocessing data via PIMENTo's Preprocess Data command (e.g. ntext=5, data.col=6:11, symbol.index=2, id.index=1 for a tab-separated file like example_unnorm.txt) and normalizing with quantile (limma) or LOESS (affy), generate a dendrogram of all chosen samples. PIMENTo calculates Euclidean distances between samples and clusters them with Ward's method. Inspect the dendrogram: if clusters align with sample response classes (e.g. WT vs villin-gp130Act), no batch effect is present; if clusters group by date, operator, or reagent lot, a batch effect is indicated. If necessary, use limma to remove the detected batch effects. In the article's example dataset (Array Express E-MEXP-E-MTAB-2400), this inspection showed no batch effect, as clusters aligned with sample response classes.

## Related tools

- **PIMENTo** (Generates the dendrogram of all chosen samples using Euclidean distances and Ward's clustering; also supports batch-effect removal via limma) — https://www.github.com/MUSC-CGM/PIMENTo
- **limma** (R/Bioconductor package used within PIMENTo for quantile normalization as well as removing batch effects, if necessary)
- **RStudio** (IDE for running the PIMENTo commands with autocomplete capability)

## Examples

```
install.packages("devtools"); devtools::install_github("MUSC-CGM/PIMENTo"); library(PIMENTo)
```

## Evaluation signals

- The dendrogram clusters align with the sample response classes (biological groups), not with technical variables such as date, operator, or reagent lot
- The dendrogram is computed from Euclidean distances between all chosen samples and clustered with Ward's method
- The input passed through Preprocess Data with the correct column indices (gene IDs in column 1, symbols in column 2, intensities in columns 6:11, ntext=5)
- If a batch effect is detected, downstream data should be corrected using limma's batch-effect removal before further analysis

## Limitations

- Batch-effect detection is visual: the user must inspect the dendrogram (e.g. Fig. 3) and judge alignment of clusters with response classes vs. technical factors; no automated statistical test is provided
- No version numbers are stated for limma, affy, R, or Bioconductor, so reproducibility of the exact clustering behavior is not guaranteed
- The analysis dataset (wild-type and villin-gp130Act small intestinal crypt expression profiles) has no accession or hosting location given beyond the Array Express entry E-MEXP-E-MTAB-2400

## Evidence

- [methods] It is evident upon inspection of Fig. 3 that there is no batch effect present in the samples, as the clusters align with the sample response classes.: "It is evident upon inspection of Fig. 3 that there is no batch effect present in the samples, as the clusters align with the sample response classes."
- [methods] PIMENTo generates a dendrogram plot of all chosen samples.: [evidence span withheld — anchor status: not_found]
- [methods] "limma" is used for quantile normalization as well as removing batch effects, if necessary: "“limma” is used for quantile normalization as well as removing batch effects, if necessary"
- [intro] the data produced by the thousands of microarray studies published annually are confounded by "batch effects," i.e. the systematic error introduced when samples are processed in different: [evidence span withheld — anchor status: not_found]
- [readme] PIMENTo is an R package to normalize, analyze, and visualize microarray data: "PIMENTo is an R package to normalize, analyze, and visualize microarray data"
