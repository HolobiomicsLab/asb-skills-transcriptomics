---
name: in-silico-transcription-factor-perturbation-simulation
description: Use when CellOracle in silico TF perturbation simulations must be converted
  into quantitative perturbation scores to rank candidate regulators (e.g., Gata1
  KO on the Paul et al. 2015 hematopoiesis data) for experimental validation.
when_to_use_negative:
- The goal is GRN inference or network-topology analysis from scRNA-seq/scATAC-seq
  data rather than scoring perturbation effects against a reference trajectory
- No pseudotime-derived differentiation gradient or reference trajectory exists for
  the system, so simulated transitions cannot be compared to a reference vector field
- The candidates of interest are not transcription factors; the study scope was mainly
  focused on TFs associated with installing new cell identities (KO or overexpression)
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CellOracle
  role: Runs the in silico TF perturbation simulation, converts simulated transitions
    and the pseudotime gradient into the shared 2D vector-field format, computes inner-product
    perturbation scores, and provides the simulated-value range check function
  repo: https://github.com/morris-lab/CellOracle
- name: Python
  role: Runtime environment for the CellOracle perturbation-scoring workflow (developed
    on Python 3.8, tested with Python 3.10)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:f72575a258a82e60df1fccf5bd8d5e1a1476279029698dcb539b33e50aa01809
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
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# in-silico-transcription-factor-perturbation-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies CellOracle in silico TF perturbation simulations by converting simulated cell-state transitions into the same 2D grid vector-field format as the pseudotime-derived differentiation gradient and taking their per-grid-point inner product. The resulting perturbation scores are summed (negative scores) across TFs to rank candidate regulators for experimental validation.

## When to use

Use when CellOracle in silico TF perturbation simulations must be converted into quantitative perturbation scores to rank candidate regulators (e.g., Gata1 KO on the Paul et al. 2015 hematopoiesis data) for experimental validation.

## When NOT to use

- The goal is GRN inference or network-topology analysis from scRNA-seq/scATAC-seq data rather than scoring perturbation effects against a reference trajectory
- No pseudotime-derived differentiation gradient or reference trajectory exists for the system, so simulated transitions cannot be compared to a reference vector field
- The candidates of interest are not transcription factors; the study scope was mainly focused on TFs associated with installing new cell identities (KO or overexpression)

## Inputs

- scRNA-seq expression data with Monocle pseudotime (e.g., Paul et al. 2015 hematopoiesis dataset)
- Base GRN derived from scATAC-seq data (e.g., the Mouse sci-ATAC-seq Atlas base-GRN)
- CellOracle Oracle object with computed transition probabilities and Markov-simulated cell transitions
- Simulated cell-state transitions from in silico TF KO or overexpression (e.g., Gata1 KO)

## Outputs

- Per-grid-point perturbation scores (inner products of reference differentiation gradient and perturbed transition vectors)
- Ranked TF candidate list based on summed negative scores
- perturbation_scores.csv containing the per-TF ranked perturbation score table

## How to apply

In a Python 3.10 environment, load the Paul et al. (2015) hematopoiesis scRNA-seq dataset and the Mouse sci-ATAC-seq Atlas base-GRN using CellOracle's data-loading functions, then compute oracle transition probabilities along the Monocle pseudotime with the transition-probability and Markov simulation functions. Convert the pseudotime-derived cell transitions into a 2D grid-based differentiation gradient vector field on the low-dimensional embedding. Run the in silico TF perturbation (e.g., Gata1 KO as in the Gata1_KO_simulation_with_Paul_etal_2015_data tutorial) and convert the simulated cell transitions into the same 2D vector-field format so the two fields are directly comparable. Compute the inner product between the reference and perturbed gradient vectors at each grid point to obtain per-point perturbation scores, check the distribution range of simulated values with the package's simulated-value range check function, and rank TFs by summing negative scores. Save the score vectors and TF ranking to perturbation_scores.csv and validate top-ranked regulators experimentally, as was done for Id1, Fosb, Fos, Eno1, and Klf4 in the iEP reprogramming study.

## Related tools

- **CellOracle** (Runs the in silico TF perturbation simulation, converts simulated transitions and the pseudotime gradient into the shared 2D vector-field format, computes inner-product perturbation scores, and provides the simulated-value range check function) — https://github.com/morris-lab/CellOracle
- **Python** (Runtime environment for the CellOracle perturbation-scoring workflow (developed on Python 3.8, tested with Python 3.10))

## Examples

```
pip install celloracle; jupyter nbconvert --execute Gata1_KO_simulation_with_Paul_etal_2015_data.ipynb
```

## Evaluation signals

- perturbation_scores.csv exists and contains a per-TF ranked perturbation score table
- The distribution range of simulated values was checked with the package's simulated-value range check function before scoring
- The reference differentiation gradient and the perturbed transition vectors share the same 2D grid vector-field format, so per-grid-point inner products are well-defined
- TF ranking is produced by summing negative scores across the grid, with top candidates (e.g., Id1, Fosb, Fos, Eno1, Klf4) supported by experimental validation
- The environment matches CellOracle's supported Python (3.8/3.10) and the demo inputs (Paul et al. 2015 scRNA-seq; sci-ATAC-seq Atlas base GRN) load without error

## Limitations

- Requires a trustworthy pseudotime-derived differentiation gradient; without a valid reference trajectory the inner-product score is not meaningful
- Analysis scope centered on TFs that install new cell identities, so scores for non-identity factors may be less informative
- Simulated value ranges must be checked (via the simulated-value range check function) before ranking; out-of-range simulated values can distort the TF prioritization

## Evidence
