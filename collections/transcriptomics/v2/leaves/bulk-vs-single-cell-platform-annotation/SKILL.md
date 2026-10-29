---
name: bulk-vs-single-cell-platform-annotation
description: Use when building or auditing a multi-dataset transcriptomic compendium
  where each dataset must be labeled by measurement type (MA/RNAseq/scRNA) and checked
  for sample/feature counts before analysis, e.g. when deciding whether TCGA-PAAD
  should be whitelisted to histologically confirmed PDAC samples.
when_to_use_negative:
- The input is already a curated feature/sample count table that only needs reformatting
  — no dataset re-parsing is required.
- 'You need to merge or aggregate datasets across platforms: the paper explicitly
  states data aggregation was removed because it ''introduced more problems than it
  solved'', and multi-platform (bulk RNA-seq + microarray) merging is not robustly
  supported.'
- 'You are working with single-cell data needing a web GUI: the scOh single-cell atlas
  lacks a web interface and its computational burden is beyond the scope of the responsive
  pdacR GUI.'
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: pdacR
  role: R package housing the parsed, annotated PDAC datasets whose $ex/$sampInfo/$featInfo/$metadata
    structures are inventoried to build Table 1
  repo: https://github.com/rmoffitt/pdacR.git
- name: R
  role: language in which the package is loaded and per-dataset platform, sample,
    and feature counts are extracted
- name: devtools
  role: used to install the pdacR package from GitHub via install_github
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:4df99a9e92ea17237ba9de60f88d40e4040a99aaa410b2b49915f3f3d8dfc38c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rmoffitt/pdacR.git
  tools:
  - R
  - devtools
  - pdacR
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bulk-vs-single-cell-platform-annotation

## Summary

Annotate each curated PDAC expression dataset with its measurement platform (microarray, bulk RNA-seq, or single-cell), sample description, sample and feature counts, and a Used flag, reproducing the 13-dataset inventory of pdacR Table 1 from the packaged .rds data.

## When to use

Use when building or auditing a multi-dataset transcriptomic compendium where each dataset must be labeled by measurement type (MA/RNAseq/scRNA) and checked for sample/feature counts before analysis, e.g. when deciding whether TCGA-PAAD should be whitelisted to histologically confirmed PDAC samples.

## When NOT to use

- The input is already a curated feature/sample count table that only needs reformatting — no dataset re-parsing is required.
- You need to merge or aggregate datasets across platforms: the paper explicitly states data aggregation was removed because it 'introduced more problems than it solved', and multi-platform (bulk RNA-seq + microarray) merging is not robustly supported.
- You are working with single-cell data needing a web GUI: the scOh single-cell atlas lacks a web interface and its computational burden is beyond the scope of the responsive pdacR GUI.

## Inputs

- pdacR R package (installed from github.com/rmoffitt/pdacR)
- data_set_list object mapping dataset labels to .rds variable names
- Per-dataset parsed list objects containing $ex (expression matrix), $sampInfo (sample metadata), $featInfo (feature metadata), and $metadata

## Outputs

- pdacR_dataset_inventory.csv with 13 rows and columns: dataset, type (MA/RNAseq/scRNA), sample description, sample count, feature count, Used
- Per-dataset whitelist correction notes (e.g. TCGA-PAAD: 150 whitelisted of ~182 samples)

## How to apply

Install pdacR from GitHub and load the data_set_list object, which maps human-readable dataset labels to saved .rds variables in the package data directory. For each of the thirteen listed datasets, load its parsed list object and extract the platform type from metadata ($metadata), the sample description from $sampInfo, the sample count as ncol($ex), and the feature count as nrow($ex). Record the Used flag per dataset and apply the known TCGA-PAAD correction: of the ~182 TCGA-PAAD samples, only 150 are histologically confirmed PDAC, so filter via the 'Decision' sample track keeping only 'whitelist'. Assemble all thirteen rows into a table with columns dataset, type (MA/RNAseq/scRNA), sample description, sample count, feature count, and Used, and write it to pdacR_dataset_inventory.csv.

## Related tools

- **pdacR** (R package housing the parsed, annotated PDAC datasets whose $ex/$sampInfo/$featInfo/$metadata structures are inventoried to build Table 1) — https://github.com/rmoffitt/pdacR.git
- **R** (language in which the package is loaded and per-dataset platform, sample, and feature counts are extracted)
- **devtools** (used to install the pdacR package from GitHub via install_github)

## Examples

```
devtools::install_github('rmoffitt/pdacR'); data(data_set_list); ex <- pdacR::tcga_paad$ex; c(ncol(ex), nrow(ex))
```

## Evaluation signals

- Output table contains exactly 13 dataset rows, one per curated PDAC dataset in data_set_list.
- Each row has non-missing type, sample count, feature count, and Used fields.
- The TCGA-PAAD entry reports ~182 total samples with 150 PDAC whitelist samples (Decision = whitelist), matching the published Table 1.
- Platform type for every dataset falls into one of the three allowed classes: microarray (MA), bulk RNA-seq (RNAseq), or single-cell (scRNA).
- Sample count equals ncol($ex) and feature count equals nrow($ex) for each dataset's expression matrix.

## Limitations

- TCGA-PAAD contains non-PDAC tumors: only 150 of ~182 mRNA samples are histologically confirmed PDAC, so unfiltered use misleads signature derivation or validation.
- All bulk datasets limit the ability to fully separate cellular populations, and the single-cell atlas cannot be merged into this compendium or served through the web GUI.
- Multi-platform composition (bulk RNA-seq and microarrays) prevents robust cross-dataset aggregation and batch correction.

## Evidence

- [readme] TCGA-PAAD whitelisting: "For example, TCGA-PAAD has 182 samples, but only 150 are PDAC. The remainder have been identified to be non-PDAC tumors in previous publications."
- [readme] filter procedure: "To filter to the correct 150, select "Decision", and then remove all variables except "whitelist"."
- [intro] TCGA-PAAD sample composition: "The mRNA dataset includes 182 samples, but only 150 have been confirmed histologically as PDAC"
- [intro] compendium scope: "we have gathered and carefully annotated thirteen human PDAC datasets from the past decade for a thorough comparison of the performance of subtyping schemas"
- [methods] package install: "devtools::install_github("rmoffitt/pdacR")"
