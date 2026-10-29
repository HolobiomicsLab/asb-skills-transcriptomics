---
name: interactor-function-assessment
description: Use when a completed interactome/GMM analysis has produced a list of
  key interactor genes/proteins and you need to annotate each interactor with protein
  type and subcellular location to interpret the cluster biologically.
when_to_use_negative:
- You want to cluster or score the genes themselves (use OmicInt's Gaussian mixture
  modelling pipeline on expression/LFC data instead)
- The proteins of interest are not key interactors from the interactome assessment
  and you need de novo subcellular localisation prediction rather than retrieval of
  known UniProt annotations
- The input is not a properly formatted CSV data frame of identifiers (the README
  stresses the user must ensure the supplied data is in the right format)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: OmicInt
  role: Provides the classification function that retrieves UniProt-based annotations
    and assigns protein type and cellular location labels to key interactors
  repo: https://github.com/AusteKan/OmicInt
- name: R
  role: Runtime environment in which the OmicInt package functions are called on the
    CSV interactor data frame
- name: UniProt
  role: Annotation resource supplying the protein classification and subcellular location
    terms used for labelling
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ailsci.2021.100025
    title: 'OmicInt package: Exploring omics data and regulatory networks using integrative
      analyses and machine learning'
  source_content_hash: sha256:3066dda8eb81b4f0ab4a0051e8ad09dc1be8351885cb0b3c18187de5561e132e
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
  - UniProt
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# interactor-function-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A classification step of the OmicInt R package that labels key interactor proteins with their protein type and cellular compartment by retrieving known UniProt-based annotations, extending the interactome assessment into a functional annotation table.

## When to use

Use when a completed interactome/GMM analysis has produced a list of key interactor genes/proteins and you need to annotate each interactor with protein type and subcellular location to interpret the cluster biologically.

## When NOT to use

- You want to cluster or score the genes themselves (use OmicInt's Gaussian mixture modelling pipeline on expression/LFC data instead)
- The proteins of interest are not key interactors from the interactome assessment and you need de novo subcellular localisation prediction rather than retrieval of known UniProt annotations
- The input is not a properly formatted CSV data frame of identifiers (the README stresses the user must ensure the supplied data is in the right format)

## Inputs

- CSV data frame of key interactor gene/protein identifiers (output of the OmicInt interactome assessment)
- UniProt protein classification and subcellular location annotations (retrieved by the OmicInt classification function)

## Outputs

- Annotation table (e.g. protein_classification_table.csv) mapping each key interactor to a protein type label and a cellular compartment/location label

## How to apply

Load the key interactor gene/protein list, exported from the OmicInt interactome assessment as a CSV data frame in R. Run the OmicInt classification function, which retrieves known annotations for each interactor from UniProt-based resources (OmicInt also draws on STRINGdb for interactor resources). Assign a protein type label to each interactor from its UniProt protein classification and a cellular compartment label from its UniProt subcellular location annotations. Assemble the per-protein type and cellular location assignments into a single annotation table (e.g. protein_classification_table.csv), keeping one row per input interactor so assignments can be joined back to the interactome results.

## Related tools

- **OmicInt** (Provides the classification function that retrieves UniProt-based annotations and assigns protein type and cellular location labels to key interactors) — https://github.com/AusteKan/OmicInt
- **R** (Runtime environment in which the OmicInt package functions are called on the CSV interactor data frame)
- **UniProt** (Annotation resource supplying the protein classification and subcellular location terms used for labelling)

## Evaluation signals

- Every key interactor present in the input CSV appears as a row in the output annotation table
- Each output row has both a protein type label and a cellular location label populated
- Assigned protein type and location values correspond to valid UniProt classification and subcellular location terms
- The output table can be joined losslessly (one-to-one on identifiers) back to the key interactor list produced by the interactome assessment

## Limitations

- Labels are limited to annotations already known in the retrieved resources (e.g. UniProt, STRINGdb); interactors lacking curated protein type or subcellular location entries cannot be labelled
- Data must be supplied as a correctly formatted CSV data frame in R; the package requires the user to prepare the right input format
- Classification of interactors complements but does not replace the GMM/LFC scoring pipeline; it assesses key interactors rather than the changed genes themselves

## Evidence
