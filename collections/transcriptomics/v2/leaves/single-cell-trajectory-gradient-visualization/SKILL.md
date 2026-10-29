---
name: single-cell-trajectory-gradient-visualization
description: Use when you need to quantify how an in silico TF perturbation (e.g.,
  Gata1 KO) shifts simulated cell-state transitions relative to the reference differentiation
  gradient in a scRNA-seq trajectory dataset.
when_to_use_negative:
- The goal is discrete cell-type classification rather than vector-field-based perturbation
  scoring — Capybara measures cell identity as a continuum and is a different approach
- No pseudotime or reference differentiation trajectory is available to serve as the
  reference gradient for inner-product scoring
- The GRN has not been inferred/loaded yet; perturbation simulation requires an Oracle
  object built from a base GRN first
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellOracle
  role: Infers GRNs, simulates TF perturbation transitions, and converts reference
    and simulated transitions into 2D gradient vector fields for inner-product perturbation
    scoring
  repo: https://github.com/morris-lab/CellOracle
- name: Python
  role: Runtime environment for CellOracle (developed with Python 3.8, tested with
    3.10) used to run the simulation and scoring workflow
- name: igraph
  role: R dependency required for CellOracle installation (checked during R library
    installation)
  repo: https://github.com/igraph/rigraph
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:3cec364a7e6a09877d89294f1e09ab0a408652c2220d4579e083877b9ddd4baf
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
  - igraph
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# single-cell-trajectory-gradient-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualizes single-cell trajectory dynamics by converting cell transitions into 2D grid-based gradient vector fields on a low-dimensional embedding, and scores TF perturbations by taking the inner product of simulated perturbation vectors against the pseudotime-derived differentiation gradient in CellOracle.

## When to use

Use when you need to quantify how an in silico TF perturbation (e.g., Gata1 KO) shifts simulated cell-state transitions relative to the reference differentiation gradient in a scRNA-seq trajectory dataset.

## When NOT to use

- The goal is discrete cell-type classification rather than vector-field-based perturbation scoring — Capybara measures cell identity as a continuum and is a different approach
- No pseudotime or reference differentiation trajectory is available to serve as the reference gradient for inner-product scoring
- The GRN has not been inferred/loaded yet; perturbation simulation requires an Oracle object built from a base GRN first

## Inputs

- scRNA-seq dataset with pseudotime (e.g., Paul et al. 2015 hematopoiesis dataset)
- Base GRN from scATAC-seq (e.g., Mouse sci-ATAC-seq Atlas base GRN)
- Oracle object with transition probabilities along pseudotime
- In silico perturbation simulation results (e.g., Gata1 KO simulation)

## Outputs

- 2D grid-based differentiation gradient vector field on the low-dimensional embedding
- 2D simulated perturbation vector field in the same format
- Per-grid-point perturbation scores (inner products)
- Ranked TF candidate list saved to perturbation_scores.csv

## How to apply

Load a scRNA-seq dataset (e.g., the Paul et al. 2015 hematopoiesis data) together with a base GRN (e.g., the Mouse sci-ATAC-seq Atlas base GRN) via celloracle's data-loading functions in Python 3.8/3.10, then compute transition probabilities along Monocle pseudotime using the Oracle transition-probability and Markov simulation functions. Convert the pseudotime-derived transitions into a 2D grid-based differentiation gradient vector field on the low-dimensional embedding. Run in silico TF perturbation (KO or overexpression) and convert the simulated transitions into the same 2D vector field format. Compute the inner product between the reference and simulated gradient vectors at each grid point to yield per-point perturbation scores, aggregate them (summing negative scores) to rank candidate TFs, and check the distribution range of simulated values with the package's simulated-value range check before exporting the ranking.

## Related tools

- **CellOracle** (Infers GRNs, simulates TF perturbation transitions, and converts reference and simulated transitions into 2D gradient vector fields for inner-product perturbation scoring) — https://github.com/morris-lab/CellOracle
- **Python** (Runtime environment for CellOracle (developed with Python 3.8, tested with 3.10) used to run the simulation and scoring workflow)
- **igraph** (R dependency required for CellOracle installation (checked during R library installation)) — https://github.com/igraph/rigraph

## Examples

```
jupyter nbconvert --to notebook --execute Gata1_KO_simulation_with_Paul_etal_2015_data.ipynb
```

## Evaluation signals

- perturbation_scores.csv exists and contains a per-TF ranked perturbation score table
- The distribution range of simulated values has been checked with the package's simulated-value range check function
- Reference and simulated cell transitions are both represented as 2D grid-based vector fields on the same low-dimensional embedding before inner-product computation
- Perturbation scores aggregate negative inner products across grid points to produce a ranked TF candidate list
- Reproducing the Gata1 KO on the Paul et al. 2015 hematopoiesis data yields scores consistent with the published Gata1_KO_simulation tutorial

## Limitations

- Scores are computed on a 2D low-dimensional embedding grid, so vector-field directionality is only meaningful relative to the reference pseudotime gradient in that projection
- Study mainly focused on TFs associated with installing new cell identities; perturbation scores prioritize regulators but do not by themselves establish mechanism
- CellOracle requires an R igraph installation as a dependency, which can complicate setup across platforms

## Evidence
