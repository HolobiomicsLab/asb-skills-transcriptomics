---
name: cancer-gene-expression-data-loading
description: Use when reproducing the GNIAP gene co-expression network workflow and
  the study's breast or prostate cancer expression data files must be located, loaded,
  and fed into the GCN inference step in R.
when_to_use_negative:
- If the goal is GCN performance assessment (overlap analysis, topological or biological
  assessment) rather than acquiring and loading the expression data inputs
- If the input data are not the study's breast/prostate cancer expression files enumerated
  in the GNIAP reproducibility document, since the loading procedure is tied to that
  file inventory
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GNIAP
  role: R package whose reproducibility document enumerates the study's data files
    and whose installation document governs package setup before data loading
  repo: https://github.com/ozgurcingiz/GNIAP
- name: R
  role: Runtime environment in which GNIAP and its dependent packages are installed
    and the expression data files are loaded
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s12539-021-00440-9
    title: The Performance Comparison of Gene Co-expression Networks of Breast and
      Prostate Cancer using Different Selection Criteria
  source_content_hash: sha256:fc1e2c85a189893cc0d006d9d9b18b4aca605d273342ec8f37c29cf502b517ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ozgurcingiz/GNIAP
  tools:
  - GNIAP
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# cancer-gene-expression-data-loading

## Summary

Locates and loads the breast and prostate cancer expression data files listed in the GNIAP R Package Reproducibility Document (ozgurcingiz/GNIAP) so the study's GCN inference and assessment workflow can be run reproducibly in R.

## When to use

Use when reproducing the GNIAP gene co-expression network workflow and the study's breast or prostate cancer expression data files must be located, loaded, and fed into the GCN inference step in R.

## When NOT to use

- If the goal is GCN performance assessment (overlap analysis, topological or biological assessment) rather than acquiring and loading the expression data inputs
- If the input data are not the study's breast/prostate cancer expression files enumerated in the GNIAP reproducibility document, since the loading procedure is tied to that file inventory

## Inputs

- Breast cancer gene expression data file as listed in the GNIAP R Package Reproducibility Document
- Prostate cancer gene expression data file as listed in the GNIAP R Package Reproducibility Document
- GNIAP R Package Reproducibility Document (data file inventory and loading instructions)

## Outputs

- Loaded breast/prostate cancer expression data objects in the R session, ready for the GCN inference step
- Exported inferred gene co-expression network (nodes and weighted edges) written to inferred_gcn.csv

## How to apply

Follow the GNIAP Installation Document from the ozgurcingiz/GNIAP GitHub repository to install the GNIAP package and its dependent packages in R. Then consult the GNIAP R Package Reproducibility Document, which lists all data files the authors used in the study, and locate the breast and prostate cancer expression data files it names. Load those expression data files into the R session as instructed by the reproducibility document so that the subsequent GCN inference step can execute on the loaded data. Correct application is verified by confirming the loaded data matches the files enumerated in the reproducibility document and that the GNIAP inference functions run without errors on the resulting data objects.

## Related tools

- **GNIAP** (R package whose reproducibility document enumerates the study's data files and whose installation document governs package setup before data loading) — https://github.com/ozgurcingiz/GNIAP
- **R** (Runtime environment in which GNIAP and its dependent packages are installed and the expression data files are loaded)

## Evaluation signals

- The installed GNIAP package and its dependent packages load in R per the GNIAP Installation Document
- Every data file named in the GNIAP R Package Reproducibility Document for the breast and prostate cancer study is located and loaded
- The GCN inference step of the GNIAP workflow runs on the loaded data without errors
- The inferred network is exported to inferred_gcn.csv containing nodes and weighted edges

## Limitations

- The data-file inventory and loading instructions live in a .docx document (GNIAP R Package Reproducibility Document.docx), so data provenance depends on that file being present and up to date in the repository
- No changelog or discussion text was available for the repository, so changes to the data files or loading procedure may not be documented

## Evidence

- [readme] This document display all data files we used in our study.: "This document display all data files we used in our study."
- [other] Locate and load the study's data files as listed in the GNIAP R Package Reproducibility Document.: [evidence span withheld — anchor status: not_found]
- [intro] This document presents how users install GNIAP and its dependent packages.: "This document presents how users install GNIAP and its dependent packages."
