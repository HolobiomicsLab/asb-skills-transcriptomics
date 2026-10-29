---
name: inner-product-perturbation-scoring
description: Use when scoring systematic in silico KO/overexpression simulations (e.g.,
  Gata1 KO on the Paul et al. 2015 hematopoiesis data) against a reference differentiation
  trajectory to prioritize TF regulators for validation.
when_to_use_negative:
- When the perturbation effect is assessed by clonal lineage tracing (CellTag) rather
  than by in silico vector-field simulation
- When no reference differentiation gradient (pseudotime-derived vector field) is
  available to take the inner product against
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3170
tools:
- name: CellOracle
  role: Runs transition-probability/Markov simulation, converts transitions to grid-based
    gradient vector fields, provides the simulated-value range check, and scores perturbations
    by inner product
  repo: https://github.com/morris-lab/CellOracle
- name: Python
  role: Execution environment for CellOracle (tested with Python 3.10)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:7e861d735508366cde94d9a2f56b94ede3e4cc192cc945ceaeb33fdc037bd4a7
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
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# inner-product-perturbation-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies CellOracle in silico TF perturbation effects by converting simulated cell-state transitions and the pseudotime-derived differentiation gradient into a shared 2D grid-based vector field and taking the inner product of the two vectors per grid point, then ranking candidate TFs by summing negative scores.

## When to use

Use when scoring systematic in silico KO/overexpression simulations (e.g., Gata1 KO on the Paul et al. 2015 hematopoiesis data) against a reference differentiation trajectory to prioritize TF regulators for validation.

## When NOT to use

- When the perturbation effect is assessed by clonal lineage tracing (CellTag) rather than by in silico vector-field simulation
- When no reference differentiation gradient (pseudotime-derived vector field) is available to take the inner product against

## Inputs

- scRNA-seq count data with pseudotime (hematopoiesis dataset, Paul et al. 2015)
- Base GRN from the Mouse sci-ATAC-seq Atlas
- Reference differentiation gradient vector field (2D grid on low-dimensional embedding)
- Simulated transition vectors from in silico TF perturbation (e.g., Gata1 KO)

## Outputs

- Per-grid-point perturbation score vectors (inner product of reference and simulated gradients)
- Ranked TF candidate list saved to perturbation_scores.csv

## How to apply

Load the demo hematopoiesis scRNA-seq dataset (Paul et al., 2015) and the mouse sci-ATAC-seq Atlas base GRN in a Python 3.10 CellOracle environment. Compute oracle transition probabilities along the Monocle pseudotime with CellOracle's transition-probability and Markov simulation functions, and convert these transitions into a 2D grid-based differentiation gradient vector field on the low-dimensional embedding. Run the in silico TF perturbation (e.g., Gata1 KO as in the Gata1_KO_simulation_with_Paul_etal_2015_data tutorial) and convert the simulated cell transitions into the same vector field format. At each grid point compute the inner product between the reference and simulated gradient vectors to obtain per-point perturbation scores, then aggregate by summing negative scores across TFs to produce a ranked candidate list. Before saving, check the distribution range of simulated values using the package's simulated-value range check function.

## Related tools

- **CellOracle** (Runs transition-probability/Markov simulation, converts transitions to grid-based gradient vector fields, provides the simulated-value range check, and scores perturbations by inner product) — https://github.com/morris-lab/CellOracle
- **Python** (Execution environment for CellOracle (tested with Python 3.10))

## Examples

```
oracle.simulate_shift(perturb_condition={"Gata1": 0.0}, grand_propagation=True); oracle.calculate_inner_product_and_save()
```

## Evaluation signals

- perturbation_scores.csv exists and contains a per-TF ranked perturbation score table
- The distribution range of simulated values has been checked with the package's simulated-value range check function
- Inner-product scores at grid points where the simulated vector aligns with the differentiation gradient are positive; opposite-alignment points are negative
- Top-ranked TFs (via summed negative scores) match biologically validated regulators (e.g., Id1, Fosb, Fos, Eno1, Klf4 in the iEP study)
- Both vector fields are in the same 2D grid format before the inner product is computed

## Limitations

- Scores depend on the quality of the base GRN (sci-ATAC-seq Atlas) and the inferred GRN; incorrect edges distort the simulated transitions
- The reference differentiation gradient must be derived from pseudotime; without a meaningful pseudotime the inner-product score is not interpretable
- The study mainly focused on TFs associated with installing new cell identities; scoring may not capture non-TF perturbations

## Evidence
