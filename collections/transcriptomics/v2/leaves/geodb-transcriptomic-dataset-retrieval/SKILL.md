---
name: geodb-transcriptomic-dataset-retrieval
description: Use when you need a reproducible annotation/inventory table (dataset,
  type, sample description, sample count, feature count, Used flag) for the thirteen
  human PDAC expression datasets housed in the pdacR R package, or when verifying
  curated dataset metadata against a published compendium table.
when_to_use_negative:
- You need raw sequencing data or the original GEO/TCGA downloads — pdacR houses parsed,
  annotated datasets rather than raw files
- You need to merge or aggregate datasets across platforms — the article notes the
  collection cannot robustly merge datasets and aggregation was removed because it
  introduced more problems than it solved
- Your goal is single-cell analysis — use the separate rmoffitt/scOh single-cell atlas
  repository; the pdacR GUI/app scope excludes scRNA due to computational burden
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: pdacR
  role: R package housing the thirteen parsed and annotated public PDAC transcriptomics
    datasets and the data_set_list index used to build the inventory table
  repo: github.com/rmoffitt/pdacR
- name: devtools
  role: Used to install the pdacR package from GitHub via devtools::install_github('rmoffitt/pdacR')
  repo: github.com/rmoffitt/pdacR
- name: R
  role: Runtime language for loading the parsed dataset objects and extracting matrix
    dimensions and metadata fields
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:708927bdd2db885c173abc93a8cb3aa3749a3b56a91e75160161199e43388d1e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - devtools
  - pdacR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# geodb-transcriptomic-dataset-retrieval

## Summary

Reconstruct the pdacR curated 13-dataset PDAC transcriptomic compendium inventory (Table 1) by installing the pdacR R package, loading its parsed dataset objects, and extracting per-dataset measurement type, sample description, sample count, feature count, and Used flag.

## When to use

Use when you need a reproducible annotation/inventory table (dataset, type, sample description, sample count, feature count, Used flag) for the thirteen human PDAC expression datasets housed in the pdacR R package, or when verifying curated dataset metadata against a published compendium table.

## When NOT to use

- You need raw sequencing data or the original GEO/TCGA downloads — pdacR houses parsed, annotated datasets rather than raw files
- You need to merge or aggregate datasets across platforms — the article notes the collection cannot robustly merge datasets and aggregation was removed because it introduced more problems than it solved
- Your goal is single-cell analysis — use the separate rmoffitt/scOh single-cell atlas repository; the pdacR GUI/app scope excludes scRNA due to computational burden

## Inputs

- pdacR R package installed from github.com/rmoffitt/pdacR (bundles parsed .rds datasets, gene signatures, and PurIST classifier in Data/)
- data_set_list object mapping human-readable dataset labels to .rds variable names
- Per-dataset parsed list objects: $ex (expression matrix), $sampInfo (sample metadata), $featInfo (feature metadata), $metadata (dataset metadata)
- Known TCGA-PAAD whitelist correction (Decision = whitelist; 150 of 182 samples are histologically confirmed PDAC)

## Outputs

- pdacR_dataset_inventory.csv with exactly 13 rows and columns: dataset, type (MA/RNAseq/scRNA), sample description, sample count, feature count, Used
- TCGA-PAAD row recording RNAseq, macrodissected primary tumors (150 whitelisted), 181 samples, 55 features, Used = yes

## How to apply

Install pdacR from GitHub with devtools::install_github('rmoffitt/pdacR') (installation is slow because the package bundles substantial parsed data), then load the data_set_list object from the package data directory, which maps human-readable dataset labels to saved .rds variable names. For each of the thirteen datasets, load its parsed list object and extract the platform type (microarray, bulk RNA-seq, or single-cell) from $metadata, the sample description from $sampInfo, the sample count as the number of columns of $ex, and the feature count as the number of rows of $ex; $featInfo provides feature-level metadata. Record the Used flag per dataset and apply the known TCGA-PAAD correction: of 182 TCGA-PAAD samples only 150 are histologically confirmed PDAC, selected via the 'Decision' metadata field with 'whitelist' retained (macrodissected primary tumors whitelisted by histologic identification of neoplastic cellularity and cancer type). Assemble all thirteen rows into a table with columns dataset, type (MA/RNAseq/scRNA), sample description, sample count, feature count, and Used, and write it to pdacR_dataset_inventory.csv. The rationale is that the article's curation addresses decentralized storage and insufficiently annotated samples that cause inappropriate use of public PDAC data, so the inventory documents exactly which datasets and filtered samples underpin downstream two-subtype (basal-like/classical) analyses.

## Related tools

- **pdacR** (R package housing the thirteen parsed and annotated public PDAC transcriptomics datasets and the data_set_list index used to build the inventory table) — github.com/rmoffitt/pdacR
- **devtools** (Used to install the pdacR package from GitHub via devtools::install_github('rmoffitt/pdacR')) — github.com/rmoffitt/pdacR
- **R** (Runtime language for loading the parsed dataset objects and extracting matrix dimensions and metadata fields)

## Examples

```
devtools::install_github("rmoffitt/pdacR"); data(data_set_list); dim(pdacR_data$ex)
```

## Evaluation signals

- The output table contains exactly 13 dataset rows, one per curated human PDAC dataset
- Every row has non-missing type, sample description, sample count, feature count, and Used fields
- The TCGA-PAAD entry reports 182 total samples with 150 PDAC whitelist samples (Decision = whitelist), matching the published Table 1 correction
- TCGA-PAAD row records type = RNAseq, 181 samples (macrodissected primary tumors, 150 whitelisted), 55 features, Used = yes
- Sample counts equal ncol($ex) and feature counts equal nrow($ex) for each loaded parsed dataset object

## Limitations

- Installation may take considerable time because the package bundles many datasets and substantial raw data
- Of 182 TCGA-PAAD samples only 150 are histologically confirmed PDAC; the remaining 32 are non-PDAC tumors and must be removed via Decision = whitelist to avoid misleading analyses
- The datasets are all bulk analyses (microarray or bulk RNA-seq), limiting the ability to fully separate cellular populations; single-cell data reside in the separate scOh atlas which lacks a web interface

## Evidence

- [other] TCGA-PAAD inventory record: [evidence span withheld — anchor status: not_found]
- [methods] TCGA-PAAD correction: "For example, TCGA-PAAD has 182 samples, but only 150 are PDAC."
- [methods] Filter steps: "To filter to the correct 150, select "Decision", and then remove all variables except "whitelist"."
- [discussion] Package purpose: "we developed an R package termed pdacR that houses carefully organized and annotated public PDAC transcriptomics data for ease of access and heightened transparency"
- [readme] Installation: "To install the package, simply run
```
devtools::install_github("rmoffitt/pdacR")
```"
