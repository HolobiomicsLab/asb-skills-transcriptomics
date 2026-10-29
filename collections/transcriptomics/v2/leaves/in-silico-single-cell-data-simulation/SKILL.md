---
name: in-silico-single-cell-data-simulation
description: Use when a single-cell computational method needs validation against
  a known ground truth (e.g., a known gene-regulatory network with planted fate-driving
  TFs) before applying it to real scRNA-seq data.
when_to_use_negative:
- The analysis goal is discovery on real biological scRNA-seq data rather than method
  benchmarking — simulated ground-truth systems do not substitute for experimental
  validation
- The method under test requires realistic measurement artifacts (dropout, batch effects)
  that the default Dyngen simulation does not model
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Dyngen
  role: Multi-modal simulation engine used to generate the ground-truth in silico
    single-cell dataset with a known gene-regulatory network for validation
  repo: https://github.com/dynverse/dyngen
- name: FateCompass
  role: Method under test — run in differentiation-potential mode on the simulated
    AnnData to estimate fate probabilities and infer TF activities
  repo: https://github.com/sarajimenez/fatecompass
- name: Python
  role: Implementation language of FateCompass; used to run the validation pipeline
    on the simulated data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:81412f1b48479445ddfe84c4d132eee4a130a97d6f07d1f15d2aac429b078a3f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - Dyngen
  - FateCompass
  - Python
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# in-silico-single-cell-data-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate single-cell transcriptomics data with a known ground-truth gene-regulatory network using the Dyngen engine, in order to benchmark methods (e.g., FateCompass fate-probability and TF-activity inference) against a controlled reference system.

## When to use

Use when a single-cell computational method needs validation against a known ground truth (e.g., a known gene-regulatory network with planted fate-driving TFs) before applying it to real scRNA-seq data.

## When NOT to use

- The analysis goal is discovery on real biological scRNA-seq data rather than method benchmarking — simulated ground-truth systems do not substitute for experimental validation
- The method under test requires realistic measurement artifacts (dropout, batch effects) that the default Dyngen simulation does not model

## Inputs

- Dyngen simulation configuration (backbone/model specification, e.g., 1000 cells, 1035 genes, 130 TFs, ground-truth GRN)
- Pre-processed AnnData object of simulated counts with 'cluster' cell-type annotations
- TF binding-site matrix (columns = TFs/motifs, rows = expressed genes)
- Marker genes defining terminal fates (e.g., Target62, C5-TF1, D6-TF1)

## Outputs

- Simulated single-cell count matrix with known ground-truth gene-regulatory network
- Per-cell fate probability estimates toward terminal states (e.g., sEndC and sEndD)
- Inferred fate-driving TF list (e.g., B11 for sEndC, B14 for sEndD) from differential motif activity analysis
- Results file (e.g., fatecompass_dyngen_results.csv) for comparison against the ground truth

## How to apply

Simulate the ground-truth in silico system with Dyngen, configuring the output to a defined scale — in the FateCompass validation this was 1000 cells, 1035 genes of which 130 are transcription factors, plus planted fate-driving TFs (B11 driving sEndC, B14 driving sEndD). Pre-process the simulated counts into an AnnData object annotated with cell-type ('cluster') labels, and build a TF binding-site matrix with TFs/motifs as columns and expressed genes as rows. Run the method under test (e.g., FateCompass via a conda env created with fatecompass.yml) in the relevant mode — here differentiation-potential mode, using marker genes Target62, C5-TF1 and D6-TF1 to define terminal fates and biasing transition probabilities with the differentiation-potential gradient. Extract per-cell fate probabilities toward the terminal states and infer TF activities via the linear model of gene regulation, then perform differential motif activity analysis to recover the planted drivers. The rationale is that Dyngen's multi-modal simulation provides a controllable dynamic cellular process at single-cell resolution, so recovery of the known network elements directly demonstrates method capability.

## Related tools

- **Dyngen** (Multi-modal simulation engine used to generate the ground-truth in silico single-cell dataset with a known gene-regulatory network for validation) — https://github.com/dynverse/dyngen
- **FateCompass** (Method under test — run in differentiation-potential mode on the simulated AnnData to estimate fate probabilities and infer TF activities) — https://github.com/sarajimenez/fatecompass
- **Python** (Implementation language of FateCompass; used to run the validation pipeline on the simulated data)

## Examples

```
install.packages("dyngen")  # then simulate the 1000-cell, 1035-gene ground-truth dataset and export counts to AnnData for FateCompass
```

## Evaluation signals

- Estimated fate probabilities for the terminal states sEndC and sEndD match the known simulation structure
- Differential motif activity analysis recovers the planted drivers: B11 for sEndC and B14 for sEndD
- Simulated dataset matches the configured dimensions (1000 cells, 1035 genes, 130 TFs)
- Saved results (fatecompass_dyngen_results.csv) reproduce the reported ground-truth validation outcome
- AnnData carries 'cluster' annotations and a TF binding-site matrix aligned to expressed genes before running FateCompass

## Limitations

- Ground-truth simulations are simplified relative to real biological processes; exact reconstruction of interactions underlying a complex biological process is acknowledged to be unattainable
- Reliance on computationally predicted regulatory sites in a binding-site matrix introduces a bias on the gene-regulatory network structure
- Pipeline input parameters must be hand tuned (detailed tutorials are provided to guide parameter setup)

## Evidence

- [readme] Dyngen engine description: [evidence span withheld — anchor status: external_doc]
- [readme] Dyngen installation: [evidence span withheld — anchor status: external_doc]
