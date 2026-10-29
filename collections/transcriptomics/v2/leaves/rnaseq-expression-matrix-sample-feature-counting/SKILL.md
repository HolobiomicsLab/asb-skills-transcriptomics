---
name: rnaseq-expression-matrix-sample-feature-counting
description: Use when reproducing or auditing a published multi-dataset compendium
  table (dataset, type, sample description, n samples, n features, Used flag) from
  saved .rds expression objects rather than from publication text alone.
when_to_use_negative:
- The input is already a summarized count/sample table — no matrix dimension extraction
  is needed
- You need to merge or batch-correct datasets across platforms — the article states
  aggregation 'introduced more problems than it solved' across bulk RNA-seq and microarray
  platforms
- You are working with single-cell atlas data (rmoffitt/scOh Seurat .rds objects)
  — the article notes its computational burden is beyond a responsive GUI scope and
  it lacks a web interface
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: pdacR
  role: Source of the parsed dataset objects ($ex, $sampInfo, $featInfo, $metadata)
    whose matrix dimensions yield sample and feature counts
  repo: https://github.com/rmoffitt/pdacR.git
- name: R
  role: Runtime for loading the package data and computing nrow/ncol of expression
    matrices
- name: devtools
  role: Installation pathway for the pdacR package from GitHub
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:e18446cbb619824c0ea5871ca85fe41384b45cbdb47c64c60d883ab243852e98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rmoffitt/pdacR.git
  tools:
  - R
  - devtools
  - pdacR
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# rnaseq-expression-matrix-sample-feature-counting

## Summary

Extracts sample counts and feature counts directly from parsed gene expression matrix objects (e.g. pdacR's $ex matrices, where columns are samples and rows are features) to reproduce a curated dataset inventory such as pdacR's Table 1 across thirteen PDAC datasets.

## When to use

Use when reproducing or auditing a published multi-dataset compendium table (dataset, type, sample description, n samples, n features, Used flag) from saved .rds expression objects rather than from publication text alone.

## When NOT to use

- The input is already a summarized count/sample table — no matrix dimension extraction is needed
- You need to merge or batch-correct datasets across platforms — the article states aggregation 'introduced more problems than it solved' across bulk RNA-seq and microarray platforms
- You are working with single-cell atlas data (rmoffitt/scOh Seurat .rds objects) — the article notes its computational burden is beyond a responsive GUI scope and it lacks a web interface

## Inputs

- pdacR data_set_list object mapping dataset labels to .rds variable names
- Parsed per-dataset list objects with $ex expression matrix, $sampInfo sample metadata, $featInfo feature metadata, and $metadata
- Known TCGA-PAAD whitelist/Decision sample annotation (150 histologically confirmed PDAC of 182 total)

## Outputs

- pdacR_dataset_inventory.csv with 13 rows and columns: dataset, type (MA/RNAseq/scRNA), sample description, sample count, feature count, Used
- Per-dataset sample/feature counts (e.g. TCGA-PAAD, 2017: RNAseq, 181 macrodissected primary tumors, 181 samples, 55 features, Used = yes)

## How to apply

Install the pdacR package from GitHub and load the data_set_list object, which maps human-readable dataset labels to saved .rds variable names. For each of the thirteen datasets, load the parsed list object and read the platform type from $metadata, the sample count as the number of columns of $ex, and the feature count as the number of rows of $ex; pull the sample description from $sampInfo. Record the Used flag per dataset and apply the known TCGA-PAAD correction: of 182 TCGA-PAAD samples only 150 are histologically confirmed PDAC, selected via the Decision = whitelist filter. Assemble all thirteen rows into a single inventory table (columns: dataset, type MA/RNAseq/scRNA, sample description, sample count, feature count, Used) and write it to CSV for comparison against the published Table 1.

## Related tools

- **pdacR** (Source of the parsed dataset objects ($ex, $sampInfo, $featInfo, $metadata) whose matrix dimensions yield sample and feature counts) — https://github.com/rmoffitt/pdacR.git
- **R** (Runtime for loading the package data and computing nrow/ncol of expression matrices)
- **devtools** (Installation pathway for the pdacR package from GitHub)

## Examples

```
devtools::install_github("rmoffitt/pdacR"); Rscript -e 'library(pdacR); data(data_set_list); d <- get(data_set_list[[1]]); c(samples = ncol(d$ex), features = nrow(d$ex))'
```

## Evaluation signals

- Output table contains exactly 13 dataset rows, one per curated PDAC dataset
- Every row has non-missing type, sample count, feature count, and Used fields
- TCGA-PAAD entry reports 182 total samples with 150 PDAC whitelist samples (Decision = whitelist), matching the published Table 1
- TCGA-PAAD, 2017 row matches the recorded inventory: RNAseq, 181 macrodissected primary tumors, 181 samples, 55 features, Used = yes
- Sample counts equal ncol($ex) and feature counts equal nrow($ex) for every loaded dataset object

## Limitations

- Dataset curation distinguishes total deposited samples from whitelist PDAC samples (182 vs 150 for TCGA-PAAD); counts without the Decision filter will disagree with the published inventory
- Datasets span microarray and bulk RNA-seq platforms, so feature counts are platform-specific and datasets cannot be robustly merged
- The scOh single-cell atlas is distributed as large Seurat .rds objects (1.4–6 Gb) with no web interface, and is not amenable to this lightweight matrix-dimension counting workflow

## Evidence

- [full_text] In the pdacR Table 1 inventory, the TCGA-PAAD, 2017 dataset is recorded as RNAseq with 181 macrodissected primary tumors: [evidence span withheld — anchor status: not_found]
- [readme] TCGA-PAAD has 182 samples, but only 150 are PDAC: "For example, TCGA-PAAD has 182 samples, but only 150 are PDAC. The remainder have been identified to be non-PDAC tumors in previous publications."
- [methods] devtools install command: "To install the package, simply run devtools::install_github("rmoffitt/pdacR")"
- [readme] whitelist filtering decision: "To filter to the correct 150, select "Decision", and then remove all variables except "whitelist"."
