---
name: omics-gene-cluster-interactor-analysis
description: Use when you have a list of key interactor genes/proteins produced by
  an interactome assessment and need to summarise their functional classes and typical
  cellular compartments.
when_to_use_negative:
- The goal is clustering or scoring of gene expression patterns (e.g. Gaussian mixture
  modelling of LFC values) rather than annotating an existing interactor list
- The input is raw single cell RNA-seq or proteomics data that has not yet been reduced
  to a key interactor list via the interactome assessment
- The input data is not a CSV data frame in the format OmicInt expects
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: OmicInt
  role: R package whose classification functions retrieve UniProt-based protein type
    and cellular location annotations for key interactors
  repo: https://github.com/AusteKan/OmicInt
- name: R
  role: Runtime environment in which the OmicInt classification functions are executed
    on CSV data frames
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ailsci.2021.100025
    title: 'OmicInt package: Exploring omics data and regulatory networks using integrative
      analyses and machine learning'
  source_content_hash: sha256:744f23dafe4e6d3e492ee58eb7a3a102bbf23ebe4b18fc1393c4a9e501bceb43
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/AusteKan/OmicInt
  repo_url: https://github.com/AusteKan/OmicInt
  tools:
  - OmicInt
  - R
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# omics-gene-cluster-interactor-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An OmicInt (R) workflow that classifies key interactor proteins from an interactome assessment by assigning each a protein type label and a cellular compartment/location label based on UniProt-based annotations. It turns a bare gene/protein hit list into an interpretable annotation table.

## When to use

Use when you have a list of key interactor genes/proteins produced by an interactome assessment and need to summarise their functional classes and typical cellular compartments.

## When NOT to use

- The goal is clustering or scoring of gene expression patterns (e.g. Gaussian mixture modelling of LFC values) rather than annotating an existing interactor list
- The input is raw single cell RNA-seq or proteomics data that has not yet been reduced to a key interactor list via the interactome assessment
- The input data is not a CSV data frame in the format OmicInt expects

## Inputs

- Key interactor gene/protein list produced by the interactome assessment, as a CSV data frame loadable in R
- UniProt-based annotation resources (optionally STRINGdb for known interactor assessment)

## Outputs

- protein_classification_table.csv — a single annotation table with a protein type label and a cellular location/compartment label for every key interactor

## How to apply

Load the key interactor gene/protein list as a CSV data frame in R, since OmicInt requires the supplied data to be in the right format (CSV data frame with relevant experimental information). Call the OmicInt classification function to retrieve known annotations for each key interactor from UniProt-based resources (the package also supports STRINGdb for proteome-level interactor assessment). For each interactor, assign a protein type label from its UniProt protein classification and a cellular compartment/location label from its UniProt subcellular location annotations. Assemble the per-protein type and cellular location assignments into a single annotation table and write it to protein_classification_table.csv. The rationale is that protein type and cellular location classification provide useful assessments of the key interactors, helping to assess gene clusters based on their known interactors.

## Related tools

- **OmicInt** (R package whose classification functions retrieve UniProt-based protein type and cellular location annotations for key interactors) — https://github.com/AusteKan/OmicInt
- **R** (Runtime environment in which the OmicInt classification functions are executed on CSV data frames)

## Examples

```
Rscript -e 'install.packages("OmicInt"); library(OmicInt)'
```

## Evaluation signals

- Every key interactor in the input list appears in protein_classification_table.csv
- Each row of the output table has both a protein type label and a cellular location label populated
- Output is a single CSV annotation table in R data frame format, per the workflow validation rule
- Assigned labels are consistent with the UniProt protein classification and subcellular location annotations retrieved by OmicInt

## Limitations

- Labels depend entirely on known annotations from resources such as UniProt (and STRINGdb); interactors lacking curated annotations may be unclassified
- OmicInt requires the input data to be prepared in the right format (CSV data frame); incorrect formatting is a documented failure mode
- The README documents package-version drift (V1.1.5 badge vs v1.1.7 in comments) and no changelog, so function availability should be checked against the installed version

## Evidence
