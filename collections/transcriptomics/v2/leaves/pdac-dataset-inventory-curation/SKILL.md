---
name: pdac-dataset-inventory-curation
description: Use when building or validating a dataset inventory table from pdacR's
  parsed .rds objects (e.g. verifying Table 1's 13-dataset compendium), or when auditing
  sample/feature counts and Used flags for PDAC transcriptomic datasets before analysis.
when_to_use_negative:
- You intend to merge or aggregate datasets across the microarray and bulk RNA-seq
  platforms — the article states aggregation was removed because 'aggregation introduced
  more problems than it solved' and cross-platform merging is not robust.
- You need single-cell analyses — the scRNA atlas (rmoffitt/scOh) is distributed separately
  and its computational burden is 'beyond the scope of a responsive GUI'.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0203
tools:
- name: pdacR
  role: Source of the parsed, annotated PDAC datasets (expression matrices, sample
    metadata, feature metadata) that populate the inventory table
  repo: https://github.com/rmoffitt/pdacR.git
- name: R
  role: Language in which pdacR is distributed and where the inventory extraction
    script runs
- name: devtools
  role: Used to install pdacR from GitHub via devtools::install_github('rmoffitt/pdacR')
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:ea79e14aac3c1b77d985223af26b1db8382fc82ad0a48a66807a307513d483c8
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
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pdac-dataset-inventory-curation

## Summary

Reproduces the pdacR Table 1 compendium inventory by programmatically extracting platform type, sample description, sample/feature counts, and Used flags from the thirteen parsed PDAC datasets stored in the pdacR R package, with the known TCGA-PAAD whitelist correction (150 of 182 samples histologically confirmed PDAC).

## When to use

Use when building or validating a dataset inventory table from pdacR's parsed .rds objects (e.g. verifying Table 1's 13-dataset compendium), or when auditing sample/feature counts and Used flags for PDAC transcriptomic datasets before analysis.

## When NOT to use

- You intend to merge or aggregate datasets across the microarray and bulk RNA-seq platforms — the article states aggregation was removed because 'aggregation introduced more problems than it solved' and cross-platform merging is not robust.
- You need single-cell analyses — the scRNA atlas (rmoffitt/scOh) is distributed separately and its computational burden is 'beyond the scope of a responsive GUI'.

## Inputs

- pdacR package data_set_list object (maps dataset labels to .rds variable names)
- Thirteen parsed dataset list objects ($ex expression matrix, $sampInfo, $featInfo, $metadata) from pdacR Data/
- Known TCGA-PAAD Decision whitelist annotation (150 of 182 samples are confirmed PDAC)

## Outputs

- pdacR_dataset_inventory.csv with exactly 13 rows and columns: dataset, type (MA/RNAseq/scRNA), sample description, sample count, feature count, Used
- TCGA-PAAD entry reporting 182 total samples with 150 PDAC whitelist samples

## How to apply

Install pdacR from GitHub (devtools::install_github('rmoffitt/pdacR')) — installation is slow because the package bundles all parsed datasets and raw data — and load the data_set_list object from the package data directory, which maps human-readable dataset labels to saved .rds variable names. For each of the thirteen datasets, load the parsed list object and extract the $ex expression matrix, $sampInfo sample metadata, $featInfo feature metadata, and $metadata; classify platform as microarray (MA), bulk RNA-seq (RNAseq), or single-cell (scRNA); record the sample description, sample count as ncol($ex), and feature count as nrow($ex). Record the Used flag per dataset and apply the TCGA-PAAD correction: the dataset has 182 samples but only 150 are histologically confirmed PDAC, so the remaining 32 are excluded by filtering the 'Decision' variable to 'whitelist' only. Assemble the thirteen rows into a table with columns dataset, type, sample description, sample count, feature count, and Used, and write it to pdacR_dataset_inventory.csv.

## Related tools

- **pdacR** (Source of the parsed, annotated PDAC datasets (expression matrices, sample metadata, feature metadata) that populate the inventory table) — https://github.com/rmoffitt/pdacR.git
- **R** (Language in which pdacR is distributed and where the inventory extraction script runs)
- **devtools** (Used to install pdacR from GitHub via devtools::install_github('rmoffitt/pdacR'))

## Examples

```
devtools::install_github("rmoffitt/pdacR"); data(data_set_list); sapply(data_set_list, function(n) { d <- get(n); c(type=d$metadata$type, n_samples=ncol(d$ex), n_features=nrow(d$ex)) })
```

## Evaluation signals

- Output table contains exactly 13 dataset rows, one per curated human PDAC dataset.
- Every row has non-missing type, sample count, feature count, and Used fields.
- Sample count per dataset equals ncol($ex) and feature count equals nrow($ex) of the corresponding pdacR parsed object.
- TCGA-PAAD entry reports 182 total samples with 150 PDAC whitelist samples (Decision = whitelist), matching the published Table 1.
- Assembled table matches the published Table 1 inventory values, e.g. TCGA-PAAD 2017 recorded as RNAseq, 181 macrodissected primary tumors, 181 samples, 55 features, Used = yes.

## Limitations

- The pdacR package bundles many datasets and substantial raw data, so installation via devtools may take considerable time.
- The compendium's datasets span bulk RNA-seq and microarray platforms and cannot be robustly merged or batch-corrected; aggregation was removed from the tool.
- For TCGA-PAAD, failing to filter the 'Decision' variable to 'whitelist' includes 32 non-PDAC samples that mislead signature derivation or validation.

## Evidence

- [methods] TCGA-PAAD has 182 samples, but only 150 are PDAC: "For example, TCGA-PAAD has 182 samples, but only 150 are PDAC."
- [methods] Decision whitelist filtering: "To filter to the correct 150, select "Decision", and then remove all variables except "whitelist"."
- [readme] package installation and bundled data: "there are quite a few datasets and substantial raw data within this package, so installation may take some time"
- [readme] data_set_list location: "Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object."
