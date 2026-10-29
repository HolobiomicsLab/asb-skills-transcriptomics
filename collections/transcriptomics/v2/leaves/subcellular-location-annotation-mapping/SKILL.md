---
name: subcellular-location-annotation-mapping
description: Use when you have a list of key interactor genes/proteins from an interactome
  or GMM-cluster assessment and need to label each with a protein type and a UniProt
  subcellular compartment.
when_to_use_negative:
- Input is raw omics data not yet processed into a key interactor list — OmicInt first
  requires significantly changed genes and interactome assessment (GMM pipeline, LFC-based
  scoring) before interactor classification
- The goal is expression clustering or interactome complexity scoring (LFC-based scoring,
  Gaussian mixture modelling) rather than assigning protein types and subcellular
  compartments
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: OmicInt
  role: R package providing the classification function that retrieves UniProt-based
    protein type and subcellular location annotations for key interactors
  repo: https://github.com/AusteKan/OmicInt
- name: R
  role: runtime environment in which the OmicInt classification functions are executed
    on CSV data frames
- name: UniProt
  role: annotation resource underlying the protein type and subcellular location labels
    assigned to each interactor
  repo: https://www.uniprot.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ailsci.2021.100025
    title: 'OmicInt package: Exploring omics data and regulatory networks using integrative
      analyses and machine learning'
  source_content_hash: sha256:31f53f2a34d0ae906f6a453165ca474b53e3f39ba124ab22c169beb5e0d75507
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
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# subcellular-location-annotation-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classification of key interactor proteins from an interactome analysis into protein types and cellular compartments using OmicInt's annotation functions in R, which retrieve known UniProt-based protein and subcellular location annotations for each interactor.

## When to use

Use when you have a list of key interactor genes/proteins from an interactome or GMM-cluster assessment and need to label each with a protein type and a UniProt subcellular compartment.

## When NOT to use

- Input is raw omics data not yet processed into a key interactor list — OmicInt first requires significantly changed genes and interactome assessment (GMM pipeline, LFC-based scoring) before interactor classification
- The goal is expression clustering or interactome complexity scoring (LFC-based scoring, Gaussian mixture modelling) rather than assigning protein types and subcellular compartments

## Inputs

- CSV data frame of key interactor gene/protein identifiers from an OmicInt interactome assessment
- UniProt annotation resources (protein type and subcellular location)

## Outputs

- protein_classification_table.csv with per-interactor protein type labels and cellular compartment/location labels

## How to apply

Load the key interactor gene/protein list produced by the interactome assessment as a CSV data frame in R (OmicInt requires the supplied data in the right format as a CSV data frame). Call the OmicInt classification function to retrieve known annotations for each key interactor from UniProt-based resources, which the package consults alongside resources such as STRINGdb to assess interactors at the proteome level. Assign a protein type label to each interactor from the UniProt protein classification, and a cellular compartment/location label from the UniProt subcellular location annotations. Assemble the per-protein type and cellular location assignments into a single annotation table (protein_classification_table.csv). Validate that every input interactor appears in the output with both a protein type and a cellular location label populated.

## Related tools

- **OmicInt** (R package providing the classification function that retrieves UniProt-based protein type and subcellular location annotations for key interactors) — https://github.com/AusteKan/OmicInt
- **R** (runtime environment in which the OmicInt classification functions are executed on CSV data frames)
- **UniProt** (annotation resource underlying the protein type and subcellular location labels assigned to each interactor) — https://www.uniprot.org/

## Examples

```
install.packages("OmicInt"); library(OmicInt); key_interactors <- read.csv("key_interactors.csv")
```

## Evaluation signals

- Every key interactor in the input CSV appears as a row in protein_classification_table.csv (no dropped identifiers)
- Each output row has both a non-empty protein type label and a non-empty cellular location label
- Retrieved subcellular location annotations are consistent with UniProt entries for the corresponding gene/protein
- Annotation table columns follow the per-protein type and cellular location assignment schema assembled from the classification function output

## Limitations

- OmicInt depends on external resources (e.g. UniProt, STRINGdb); interactors lacking known annotations may not receive populated labels
- The user must ensure the supplied data is in the right format (CSV data frame with relevant experimental information) or classification cannot proceed
- The article/README do not document a changelog or detailed discussion of this function's failure modes

## Evidence
