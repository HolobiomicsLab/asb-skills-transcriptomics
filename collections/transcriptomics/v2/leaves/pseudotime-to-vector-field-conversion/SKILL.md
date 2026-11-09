---
name: pseudotime-to-vector-field-conversion
description: Use when in silico TF perturbation simulations (e.g., Gata1 KO on the
  Paul et al. 2015 hematopoiesis data) must be quantified as scores by comparing simulated
  transitions to the pseudotime differentiation gradient.
when_to_use_negative:
- The dataset lacks a pseudotime ordering or a low-dimensional embedding, so no reference
  gradient vector field can be constructed.
- The perturbed TF is not covered by the base GRN (e.g., not present in the Mouse
  sci-ATAC-seq Atlas base GRN), making the simulated transition field meaningless.
- The goal is cell-type classification or identity-continuum measurement (use Capybara
  instead) rather than perturbation-effect scoring via vector-field inner products.
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellOracle
  role: Computes oracle transition probabilities and Markov simulations, converts
    transitions to grid-based vector fields, and supports in silico TF perturbation
    (KO/overexpression) simulation whose scores are derived from vector inner products.
  repo: https://github.com/morris-lab/CellOracle
- name: Python
  role: Execution environment for CellOracle (developed with Python 3.8, tested with
    Python 3.10).
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:342cc0cb2959a83d4df68ce87d5d70c9a3cdeef5f81ea2e2be76e6b3c79357e6
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
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pseudotime-to-vector-field-conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts CellOracle pseudotime-derived cell-state transitions into a 2D grid-based gradient vector field on the low-dimensional embedding so that simulated TF-perturbation transitions can be compared against reference differentiation vectors via per-grid-point inner products (perturbation scores).

## When to use

Use when in silico TF perturbation simulations (e.g., Gata1 KO on the Paul et al. 2015 hematopoiesis data) must be quantified as scores by comparing simulated transitions to the pseudotime differentiation gradient.

## When NOT to use

- The dataset lacks a pseudotime ordering or a low-dimensional embedding, so no reference gradient vector field can be constructed.
- The perturbed TF is not covered by the base GRN (e.g., not present in the Mouse sci-ATAC-seq Atlas base GRN), making the simulated transition field meaningless.
- The goal is cell-type classification or identity-continuum measurement (use Capybara instead) rather than perturbation-effect scoring via vector-field inner products.

## Inputs

- scRNA-seq dataset with Monocle pseudotime (e.g., Paul et al. 2015 hematopoiesis demo data)
- CellOracle base GRN (e.g., from the Mouse sci-ATAC-seq Atlas)
- Oracle object with transition-probability and Markov simulation results for the TF perturbation (e.g., Gata1 KO)
- Low-dimensional embedding (2D grid) on which vector fields are defined

## Outputs

- 2D grid-based differentiation gradient vector field derived from pseudotime transitions
- 2D simulated transition vector field per perturbed TF in the same format
- Per-grid-point perturbation scores (inner products) aggregated into a per-TF ranked perturbation score table
- perturbation_scores.csv with the score vectors and TF ranking

## How to apply

Load the demo hematopoiesis scRNA-seq dataset (Paul et al., 2015) and the Mouse sci-ATAC-seq Atlas base GRN with CellOracle's data-loading functions in Python 3.10. Compute oracle transition probabilities along the Monocle pseudotime using the Oracle transition-probability calculation and Markov simulation functions, then convert these cell transitions into a 2D grid-based differentiation gradient vector field on the low-dimensional embedding. Run the in silico TF perturbation (e.g., Gata1 KO) and convert the simulated transitions into the same vector field format so both fields are directly comparable. Compute the inner product between the reference and simulated gradient vectors at each grid point to obtain per-point perturbation scores, then aggregate (summing negative scores) into a ranked TF candidate list for validation.

## Related tools

- **CellOracle** (Computes oracle transition probabilities and Markov simulations, converts transitions to grid-based vector fields, and supports in silico TF perturbation (KO/overexpression) simulation whose scores are derived from vector inner products.) — https://github.com/morris-lab/CellOracle
- **Python** (Execution environment for CellOracle (developed with Python 3.8, tested with Python 3.10).)

## Evaluation signals

- perturbation_scores.csv exists and contains a per-TF ranked perturbation score table.
- The distribution range of simulated values has been checked with the package's simulated-value range check function.
- Reference (pseudotime) and simulated perturbation vector fields are in the same 2D grid vector field format before the inner product is taken.
- The ranked list prioritizes TFs by summed negative scores, with known regulators (e.g., Gata1 in the hematopoiesis demo) recoverable for validation.
- CellOracle installation check passes, including the R igraph dependency ('checking R library installation: igraph -> OK').

## Limitations

- The perturbation score depends on the quality of the inferred GRN and the base GRN source (Mouse sci-ATAC-seq Atlas); regulatory edges outside the base GRN are invisible to the simulation.
- Scores are computed on a 2D embedding grid, so geometry of the low-dimensional projection influences per-point inner products.
- Study scope note: this analysis mainly targets TFs associated with installing new cell identities; other perturbation effects may not be captured.
- CellOracle requires R library igraph; missing igraph installation causes setup failures.

## Evidence
