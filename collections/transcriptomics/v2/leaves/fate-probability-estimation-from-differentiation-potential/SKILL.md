---
name: fate-probability-estimation-from-differentiation-potential
description: Use when single-cell transcriptomics data (AnnData with 'cluster' annotations)
  covers a differentiation system with known terminal-fate marker genes (e.g. Target62,
  C5-TF1, D6-TF1) and no robust RNA velocity, so transition probabilities must be
  biased by differentiation potential.
when_to_use_negative:
- Terminal fates and marker genes are unknown (e.g. cancer studies) since differentiation-potential
  mode 'depends on the input of prior biological knowledge'
- RNA velocity profiles are robust for the dataset, in which case FateCompass should
  use velocity to direct the Markov chain edges instead
- The goal is quantifying TF activities or drivers only, without estimating transition
  probabilities between fates
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: FateCompass
  role: Python pipeline that biases Markov-chain transition probabilities with the
    differentiation-potential gradient and outputs fate probabilities
  repo: https://github.com/sarajimenez/fatecompass
- name: Dyngen
  role: Generates the ground-truth in silico dataset (known gene-regulatory network)
    used to validate fate probability estimates
  repo: https://github.com/dynverse/dyngen
- name: Scanpy
  role: Suggested for basic pre-processing of the counts into the AnnData object FateCompass
    requires
- name: scVelo
  role: Suggested for pre-processing/spliced-unspliced preparation of the AnnData
    input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:a02aea329d16c8cc5b5a81ebca4dd9c4726391339ee4c6ff42999707182e669a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  repo_url: https://github.com/sarajimenez/fatecompass
  tools:
  - Dyngen
  - FateCompass
  - Python
  - Scanpy
  - scVelo
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# fate-probability-estimation-from-differentiation-potential

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate per-cell fate probabilities in differentiation systems by biasing Markov-chain transition probabilities on a nearest-neighbor graph with the gradient of differentiation potential, as implemented in the FateCompass pipeline, and validate against ground-truth in silico data generated with Dyngen.

## When to use

Use when single-cell transcriptomics data (AnnData with 'cluster' annotations) covers a differentiation system with known terminal-fate marker genes (e.g. Target62, C5-TF1, D6-TF1) and no robust RNA velocity, so transition probabilities must be biased by differentiation potential.

## When NOT to use

- Terminal fates and marker genes are unknown (e.g. cancer studies) since differentiation-potential mode 'depends on the input of prior biological knowledge'
- RNA velocity profiles are robust for the dataset, in which case FateCompass should use velocity to direct the Markov chain edges instead
- The goal is quantifying TF activities or drivers only, without estimating transition probabilities between fates

## Inputs

- Pre-processed AnnData object with cell-type ('cluster') annotations (e.g. from Scanpy/scVelo)
- TF binding-site matrix with TFs/motifs as columns and expressed genes as rows
- Marker genes defining terminal fates (e.g. Target62, C5-TF1, D6-TF1 for the Dyngen dataset)
- Ground-truth in silico dataset simulated with Dyngen (1000 cells, 1035 genes of which 130 are TFs) for validation

## Outputs

- Per-cell fate probability estimates toward each terminal state (e.g. sEndC, sEndD), saved e.g. to fatecompass_dyngen_results.csv
- Differentiation trajectories from progenitor cells to terminal states on the cell-neighborhood network

## How to apply

Pre-process the counts (Scanpy/scVelo) into an AnnData object with cell-type ('cluster') annotations and build a TF binding-site matrix with TFs/motifs as columns and expressed genes as rows. Set up the FateCompass conda environment from fatecompass.yml and run the fatecompass.py module in differentiation-potential mode, using known marker genes to define terminal fates and biasing the transition probabilities of the discrete Markov process on the nearest-neighbor graph with the gradient of differentiation potential. Extract per-cell fate probabilities toward each terminal state (e.g. sEndC and sEndD in the Dyngen in silico benchmark of 1000 cells, 1035 genes, 130 TFs). Because differentiation-potential mode relies on prior biological knowledge to name fates, reserve it for systems where terminal-state markers are established; otherwise use the RNA-velocity mode. Validate the resulting probabilities against ground-truth fate assignments where available before using them to drive downstream differential TF activity analysis.

## Related tools

- **FateCompass** (Python pipeline that biases Markov-chain transition probabilities with the differentiation-potential gradient and outputs fate probabilities) — https://github.com/sarajimenez/fatecompass
- **Dyngen** (Generates the ground-truth in silico dataset (known gene-regulatory network) used to validate fate probability estimates) — https://github.com/dynverse/dyngen
- **Scanpy** (Suggested for basic pre-processing of the counts into the AnnData object FateCompass requires)
- **scVelo** (Suggested for pre-processing/spliced-unspliced preparation of the AnnData input)

## Examples

```
conda env create -f fatecompass.yml; conda install notebook; jupyter notebook tutorials/fatecompass_basics_differentiation_potential.ipynb
```

## Evaluation signals

- On the Dyngen benchmark, per-cell fate probabilities toward sEndC and sEndD should match the known ground-truth fate assignments reported in the validation
- Differential TF activity analysis downstream of the fate probabilities should recover B11 as the driver of sEndC and B14 as the driver of sEndD
- Fate probabilities should be per-cell values bounded in [0, 1] with each cell's probabilities across terminal states forming a coherent partition
- The pipeline runs end-to-end in the fatecompass.yml conda environment using fatecompass.py, following the documented tutorial fatecompass_basics_differentiation_potential.ipynb

## Limitations

- Differentiation-potential mode depends on prior biological knowledge (terminal-fate marker genes), which is difficult to provide in some systems such as cancer studies
- Pipeline input parameters must be hand tuned; detailed tutorials are provided to guide parameter setup
- Trajectories are built on a nearest-neighbor graph in reduced gene expression space; dimensionality reduction methods such as UMAP lack biological interpretability
- Whether putative driver factors are truly involved in differentiation requires further experimental validation of the generated hypotheses

## Evidence
