---
name: gene-regulatory-network-ground-truth-validation
description: Use when a new or reimplemented fate-inference or TF-activity method
  needs validation against a synthetic dataset with a known ground-truth GRN before
  application to real scRNA-seq data.
when_to_use_negative:
- When no ground-truth regulatory network or simulated driver set is available, so
  validation cannot be judged against expected drivers
- When prior biological knowledge to define terminal fates (marker genes) is unavailable,
  e.g., cancer studies where FateCompass's differentiation-potential mode is hard
  to apply
- When the goal is analysis of a real differentiation system rather than method validation
  against in silico data
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: FateCompass
  role: Method under validation; run in differentiation-potential mode to estimate
    fate probabilities and infer TF activities via the linear model of gene regulation
  repo: github.com/sarajimenez/fatecompass
- name: Dyngen
  role: Simulation engine used to generate the in silico dataset with a known ground-truth
    gene-regulatory network and fate drivers
  repo: https://github.com/dynverse/dyngen
- name: Python
  role: Implementation language of the FateCompass module (fatecompass.py) used to
    run the validation workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:c39b7365127d6a594b9039f4b4623a43d0a3b417d189a0124a7d35e7165d172a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  tools:
  - Dyngen
  - FateCompass
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# gene-regulatory-network-ground-truth-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates a cell-fate/TF-activity method against a ground-truth gene-regulatory network by running it on an in silico Dyngen dataset with known fate drivers, confirming that fate probabilities and differential TF activity analysis recover the simulated ground truth.

## When to use

Use when a new or reimplemented fate-inference or TF-activity method needs validation against a synthetic dataset with a known ground-truth GRN before application to real scRNA-seq data.

## When NOT to use

- When no ground-truth regulatory network or simulated driver set is available, so validation cannot be judged against expected drivers
- When prior biological knowledge to define terminal fates (marker genes) is unavailable, e.g., cancer studies where FateCompass's differentiation-potential mode is hard to apply
- When the goal is analysis of a real differentiation system rather than method validation against in silico data

## Inputs

- Dyngen-simulated single-cell count matrix with known ground-truth GRN (1000 cells, 1035 genes, 130 TFs)
- AnnData object with cell-type ('cluster') annotations
- TF binding-site matrix (rows = expressed genes, columns = TFs/motifs)
- Marker genes defining terminal fates (e.g., Target62, C5-TF1, D6-TF1)
- fatecompass.yml conda environment specification

## Outputs

- Per-cell fate probability estimates toward terminal states (sEndC, sEndD)
- List of predicted fate-driving TFs from differential motif activity analysis (B11 for sEndC, B14 for sEndD)
- fatecompass_dyngen_results.csv containing fate probabilities and driver list

## How to apply

Simulate the benchmark dataset with Dyngen (here 1000 cells, 1035 genes of which 130 are transcription factors) so the ground-truth regulatory network and fate drivers are known. Pre-process the simulated counts into an AnnData object with 'cluster' cell-type annotations, and build a TF binding-site matrix with TFs/motifs as columns and expressed genes as rows. Set up the FateCompass conda environment (conda env create -f fatecompass.yml) and run the fatecompass.py module in differentiation-potential mode, using marker genes (Target62, C5-TF1, D6-TF1) to define terminal fates and biasing transition probabilities with the differentiation-potential gradient. Extract per-cell fate probabilities toward the terminal states (sEndC and sEndD), then infer TF activities with the linear model of gene regulation and perform differential motif activity analysis. Success is judged by agreement of the extracted probabilities and recovered drivers (B11 for sEndC, B14 for sEndD) with the simulated ground truth, saved as fatecompass_dyngen_results.csv.

## Related tools

- **FateCompass** (Method under validation; run in differentiation-potential mode to estimate fate probabilities and infer TF activities via the linear model of gene regulation) — github.com/sarajimenez/fatecompass
- **Dyngen** (Simulation engine used to generate the in silico dataset with a known ground-truth gene-regulatory network and fate drivers) — https://github.com/dynverse/dyngen
- **Python** (Implementation language of the FateCompass module (fatecompass.py) used to run the validation workflow)

## Examples

```
conda env create -f fatecompass.yml && conda install notebook  # then run the fatecompass.py module on the pre-processed Dyngen AnnData
```

## Evaluation signals

- FateCompass accurately estimates per-cell fate probabilities for the terminal states sEndC and sEndD
- Differential TF activity analysis recovers B11 as the driver of the sEndC fate and B14 as the driver of the sEndD fate, matching the Dyngen ground truth
- Simulated dataset matches the stated benchmark dimensions: 1000 cells, 1035 genes, 130 transcription factors
- Binding-site matrix has TFs/motifs as columns and expressed genes as rows
- Saved results in fatecompass_dyngen_results.csv match the reported ground-truth validation results

## Limitations

- TF activities are estimated using binding sites only in promoter regions; long-range and distal regulatory interactions are not considered
- Reliance on computationally predicted regulatory sites in the binding-site matrix introduces a bias on the gene-regulatory network structure
- Pipeline input parameters must be hand tuned, and differentiation-potential mode depends on prior biological knowledge to define terminal fates

## Evidence

- [readme] dyngen simulator role: [evidence span withheld — anchor status: external_doc]
