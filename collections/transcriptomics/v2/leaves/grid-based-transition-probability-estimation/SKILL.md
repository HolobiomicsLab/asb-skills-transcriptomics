---
name: grid-based-transition-probability-estimation
description: 'Use when ranking candidate transcription-factor perturbations from CellOracle
  GRN simulations: simulated KO/overexpression transitions must be scored against
  a reference differentiation trajectory on a low-dimensional embedding.'
when_to_use_negative:
- When the goal is static GRN inference or network topology analysis (degree/connectivity
  of regulators like Mef2a or Klf6) rather than scoring simulated fate-transition
  outcomes
- When no reference differentiation trajectory (pseudotime-derived gradient) is available
  to compare simulated transitions against, since the inner-product score is undefined
  without both vector fields
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellOracle
  role: Provides transition-probability calculation, Markov simulation, in silico
    TF perturbation (KO/overexpression) simulation, and vector-field conversion utilities
    used to compute and rank perturbation scores
  repo: https://github.com/morris-lab/CellOracle
- name: Python
  role: Runtime environment for executing the CellOracle perturbation-scoring workflow
    (developed with python 3.8, tested with python 3.10)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:5fa5108ef730495db5feb350e8922d3d330b362be61092883a6efed792a43d5c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/morris-lab/CellOracle
  repo_url: https://github.com/morris-lab/CellOracle
  tools:
  - CellOracle
  - Python
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# grid-based-transition-probability-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies in silico TF perturbation effects in CellOracle by converting simulated cell-state transitions and a pseudotime-derived differentiation gradient into 2D grid-based vector fields, then computing the inner product of the two vectors at each grid point to obtain a perturbation score per candidate TF.

## When to use

Use when ranking candidate transcription-factor perturbations from CellOracle GRN simulations: simulated KO/overexpression transitions must be scored against a reference differentiation trajectory on a low-dimensional embedding.

## When NOT to use

- When the goal is static GRN inference or network topology analysis (degree/connectivity of regulators like Mef2a or Klf6) rather than scoring simulated fate-transition outcomes
- When no reference differentiation trajectory (pseudotime-derived gradient) is available to compare simulated transitions against, since the inner-product score is undefined without both vector fields

## Inputs

- Processed scRNA-seq dataset with pseudotime (e.g., Paul et al. 2015 hematopoiesis data)
- CellOracle base GRN derived from single-cell epigenome data (e.g., Mouse sci-ATAC-seq Atlas base-GRN)
- CellOracle Oracle object with low-dimensional embedding (UMAP/pseudotime) and inferred cell-type-specific GRNs
- In silico perturbation specification (TF name and KO/overexpression condition)

## Outputs

- Per-grid-point perturbation score vectors (inner product of reference differentiation gradient and simulated transition vector field)
- Ranked TF candidate list (perturbation_scores.csv with per-TF aggregated scores)
- Checked distribution range of simulated values from the package's simulated-value range check function

## How to apply

Load the scRNA-seq dataset (e.g., the Paul et al. 2015 hematopoiesis dataset) together with a base GRN (e.g., the Mouse sci-ATAC-seq Atlas base-GRN) in a Python 3.10 environment. Compute oracle transition probabilities along Monocle pseudotime using CellOracle's transition-probability calculation and Markov simulation functions, then convert these transitions into a 2D grid-based differentiation gradient vector field on the low-dimensional embedding. Run the in silico TF perturbation (e.g., Gata1 KO as in the Gata1_KO_simulation_with_Paul_etal_2015_data tutorial), convert the simulated transitions into the identical 2D vector field format, and compute the inner product between reference and simulated gradient vectors at each grid point. Aggregate per-point scores across the grid and rank TFs by summing negative scores so that perturbations opposing the reference differentiation direction are prioritized for experimental validation.

## Related tools

- **CellOracle** (Provides transition-probability calculation, Markov simulation, in silico TF perturbation (KO/overexpression) simulation, and vector-field conversion utilities used to compute and rank perturbation scores) — https://github.com/morris-lab/CellOracle
- **Python** (Runtime environment for executing the CellOracle perturbation-scoring workflow (developed with python 3.8, tested with python 3.10))

## Examples

```
pip install celloracle  # then run the Gata1_KO_simulation_with_Paul_etal_2015_data tutorial to compute grid-based perturbation scores
```

## Evaluation signals

- perturbation_scores.csv exists and contains a per-TF ranked perturbation score table
- The distribution range of simulated values has been checked with the package's simulated-value range check function
- Simulated transitions were converted into the same 2D grid-based vector field format as the pseudotime-derived differentiation gradient before inner-product computation
- Score aggregation produces a ranked candidate list where negative-score sums prioritize regulators (e.g., the workflow recovered new regulators such as Id1, Fosb, Fos, Eno1, and Klf4 in the paper's systematic in silico KO analysis)

## Limitations

- The score requires a coherent reference trajectory: perturbation effects are only meaningful relative to the pseudotime-derived differentiation gradient vector field on the same low-dimensional embedding
- Base GRN quality depends on the source epigenome atlas (e.g., Mouse sci-ATAC-seq Atlas); network reconfiguration differences between cell states can change which TFs are scored
- CellOracle requires an R igraph dependency and specific Python versions (developed with python 3.8, tested with python 3.10)

## Evidence
