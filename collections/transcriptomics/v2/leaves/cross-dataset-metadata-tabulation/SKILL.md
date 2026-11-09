---
name: cross-dataset-metadata-tabulation
description: Use when a collection of parsed expression datasets must be inventoried
  into a single annotation table with consistent columns (measurement type, sample
  description, n samples, n features, Used flag), e.g. to audit a compendium or reproduce
  a published dataset table.
when_to_use_negative:
- The goal is analyzing expression values or deriving subtypes rather than producing
  a metadata inventory of the datasets themselves
- Datasets are not present as pdacR parsed list objects (with $ex/$sampInfo/$featInfo/$metadata),
  e.g. raw FASTQ or raw count files requiring first-level processing
- You need to merge or aggregate data across platforms — the article explicitly states
  aggregation was removed because 'aggregation introduced more problems than it solved'
  across mixed bulk RNA-seq and microarray platforms
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3518
tools:
- name: pdacR
  role: Source of the parsed dataset objects (expression matrices, sample/feature
    metadata) and the data_set_list index from which per-dataset counts and descriptions
    are extracted
  repo: https://github.com/rmoffitt/pdacR.git
- name: R
  role: Language in which the thirteen dataset objects are loaded and the inventory
    table is assembled and exported
- name: devtools
  role: Used to install the pdacR package from GitHub before metadata extraction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:ccf2df0ff23f7b527dd70ac9ae25230ceb05d43d8002eb9e7fba402c1c2d7ecc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rmoffitt/pdacR.git
  tools:
  - R
  - devtools
  - pdacR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cross-dataset-metadata-tabulation

## Summary

Systematically tabulate per-dataset metadata (platform type, sample description, sample count, feature count, Used flag) across a curated multi-dataset compendium, using the pdacR package's parsed dataset objects to reproduce a publication-style inventory table (Table 1) covering thirteen human PDAC datasets.

## When to use

Use when a collection of parsed expression datasets must be inventoried into a single annotation table with consistent columns (measurement type, sample description, n samples, n features, Used flag), e.g. to audit a compendium or reproduce a published dataset table.

## When NOT to use

- The goal is analyzing expression values or deriving subtypes rather than producing a metadata inventory of the datasets themselves
- Datasets are not present as pdacR parsed list objects (with $ex/$sampInfo/$featInfo/$metadata), e.g. raw FASTQ or raw count files requiring first-level processing
- You need to merge or aggregate data across platforms — the article explicitly states aggregation was removed because 'aggregation introduced more problems than it solved' across mixed bulk RNA-seq and microarray platforms

## Inputs

- pdacR R package installed from GitHub (rmoffitt/pdacR)
- data_set_list object mapping dataset labels to .rds variable names
- Per-dataset parsed list objects containing $ex (expression matrix), $sampInfo (sample metadata), $featInfo (feature metadata), and $metadata
- Known per-dataset corrections (e.g. TCGA-PAAD Decision=whitelist flag)

## Outputs

- pdacR_dataset_inventory.csv with exactly 13 rows and columns: dataset, type (MA/RNAseq/scRNA), sample description, sample count, feature count, Used
- Per-dataset metadata records (platform type, sample description, sample and feature counts)

## How to apply

Install pdacR via devtools::install_github('rmoffitt/pdacR') and load the data_set_list object, which maps human-readable dataset labels to saved .rds variables. For each of the thirteen datasets, load its parsed list object and extract the platform type (microarray, bulk RNA-seq, or single-cell), the sample description from $sampInfo/$metadata, the sample count as ncol($ex), and the feature count as nrow($ex). Record each dataset's Used flag, and apply the known TCGA-PAAD correction: of 182 TCGA-PAAD samples only 150 are histologically confirmed PDAC, selected via the 'Decision' field with 'whitelist'. Assemble all rows into a single table with columns dataset, type (MA/RNAseq/scRNA), sample description, sample count, feature count, and Used, written to pdacR_dataset_inventory.csv.

## Related tools

- **pdacR** (Source of the parsed dataset objects (expression matrices, sample/feature metadata) and the data_set_list index from which per-dataset counts and descriptions are extracted) — https://github.com/rmoffitt/pdacR.git
- **R** (Language in which the thirteen dataset objects are loaded and the inventory table is assembled and exported)
- **devtools** (Used to install the pdacR package from GitHub before metadata extraction)

## Examples

```
devtools::install_github('rmoffitt/pdacR'); data(data_set_list); ex <- get(data_set_list[['TCGA-PAAD, 2017']])$ex; c(ncol(ex), nrow(ex))
```

## Evaluation signals

- Output table contains exactly 13 dataset rows
- Every row has non-missing type, sample count, feature count, and Used fields
- TCGA-PAAD entry reports 182 total samples with 150 PDAC whitelist samples (Decision = whitelist), matching published Table 1
- Sample counts equal ncol($ex) and feature counts equal nrow($ex) for each loaded .rds dataset object
- Type values are drawn only from the allowed vocabulary {MA, RNAseq, scRNA}

## Limitations

- The TCGA-PAAD dataset requires manual correction: 182 samples exist but only 150 are histologically confirmed PDAC; including the other 32 non-PDAC samples misleads signature derivation or validation
- Datasets span multiple platforms (bulk RNA-seq and microarrays), so counts are per-dataset and datasets cannot be robustly merged or batch-corrected into one matrix
- The compendium is limited to bulk analyses plus one single-cell atlas (scOh), and the single-cell atlas lacks a web interface and is computationally too heavy for the responsive GUI

## Evidence

- [methods] TCGA-PAAD sample count and whitelist correction: "For example, TCGA-PAAD has 182 samples, but only 150 are PDAC."
- [methods] whitelist filtering decision: "To filter to the correct 150, select "Decision", and then remove all variables except "whitelist"."
- [intro] thirteen curated datasets: "we have gathered and carefully annotated thirteen human PDAC datasets from the past decade for a thorough comparison of the performance of subtyping schemas"
- [readme] data_set_list and parsed dataset structure: "Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object. These datasets are used both in the app and in the [associated publication Torre-Healy et al. 2023]"
- [discussion] multi-platform aggregation limitation: "our datasets come from a variety of platforms, including bulk RNA sequencing and microarrays, adding another layer of complexity to aggregation and batch correction"
