---
name: perturbation-transition-visualization-vector-field
description: Use when you have CellOracle Oracle objects with simulated KO or overexpression
  perturbations and need to visualize predicted cell-state transitions (e.g., early-transition
  vs. reprogrammed clusters) on an embedding.
when_to_use_negative:
- When no inferred GRN (CellOracle Links object) exists for the cells being simulated
  — the vector field is derived from the simulated GRN, not raw expression alone
- When the goal is descriptive trajectory inference (e.g., pseudotime/RNA velocity)
  rather than predicting the effect of a specific TF perturbation (KO or overexpression)
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CellOracle
  role: Runs the in silico perturbation simulation (KO as edge removal, overexpression
    as edge reinforcement) and generates vector-field and transition-probability visualizations
  repo: https://github.com/morris-lab/CellOracle
- name: Capybara
  role: Scores each cell for fibroblast and iEP identity against reference profiles
    to quantify the visualized identity shifts
  repo: https://github.com/morris-lab/Capybara
- name: Cell Ranger 6.0.1
  role: Produces the processed scRNA-seq matrix that supplies the embedding and clusters
    over which transition vectors are projected
- name: Python
  role: Language in which CellOracle (python 3.8, tested with 3.10) is run for the
    simulation and plotting
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:f44f888e1953ea000f63f0b815b472fd57f1a8cc6f658e86420145a028dac47a
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
  - Capybara
  - Cell Ranger 6.0.1
  - CellOracle
  - Python
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# perturbation-transition-visualization-vector-field

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualize predicted cell identity shifts from CellOracle in silico TF perturbation by projecting simulated transition probabilities as vector-field maps and Sankey-style plots over the scRNA-seq embedding, as applied to Fos KO/overexpression during MEF-to-iEP reprogramming.

## When to use

Use when you have CellOracle Oracle objects with simulated KO or overexpression perturbations and need to visualize predicted cell-state transitions (e.g., early-transition vs. reprogrammed clusters) on an embedding.

## When NOT to use

- When no inferred GRN (CellOracle Links object) exists for the cells being simulated — the vector field is derived from the simulated GRN, not raw expression alone
- When the goal is descriptive trajectory inference (e.g., pseudotime/RNA velocity) rather than predicting the effect of a specific TF perturbation (KO or overexpression)

## Inputs

- Cell Ranger v6.0.1-processed scRNA-seq count matrix (e.g., day-20 Perturb-seq)
- Sample-specific CellOracle Links (GRN) objects for reprogramming and long-term-iEP states
- Cell clustering/annotation labels (e.g., early-transition, transition, dead-end, reprogrammed clusters) on the scRNA-seq embedding

## Outputs

- Vector-field maps of simulated cell identity transitions overlaid on the scRNA-seq embedding
- Sankey-style transition-probability plots per perturbation (KO vs. overexpression)
- Per-cell and per-condition identity scores (e.g., identity_scores.csv) from Capybara-style classification

## How to apply

Load the Cell Ranger v6.0.1-processed scRNA-seq matrix (e.g., day-20 Perturb-seq data) together with sample-specific CellOracle Links (GRN) objects for each state (reprogramming and long-term-iEP), and build Oracle objects. Run CellOracle's in silico perturbation module, simulating knockout as edge removal and overexpression as edge reinforcement on each GRN configuration. Project the simulated cell identity transitions as vector-field maps and Sankey-style transition-probability plots over the scRNA-seq embedding so that shifts between clusters (early transition, transition, dead-end, reprogrammed) are visible per perturbation condition. Optionally pair the vector fields with identity scoring (Capybara-style cell type classification) to quantify shifts, e.g., confirming Fos KO displaces cells away from the iEP attractor while Fos overexpression predicts movement from early-transition to transition clusters and from dead-end to reprogrammed clusters.

## Related tools

- **CellOracle** (Runs the in silico perturbation simulation (KO as edge removal, overexpression as edge reinforcement) and generates vector-field and transition-probability visualizations) — https://github.com/morris-lab/CellOracle
- **Capybara** (Scores each cell for fibroblast and iEP identity against reference profiles to quantify the visualized identity shifts) — https://github.com/morris-lab/Capybara
- **Cell Ranger 6.0.1** (Produces the processed scRNA-seq matrix that supplies the embedding and clusters over which transition vectors are projected)
- **Python** (Language in which CellOracle (python 3.8, tested with 3.10) is run for the simulation and plotting)

## Examples

```
import celloracle as co; oracle = co.load_hdf5('oracle.object'); oracle.simulate_perturbation(perturb_condition={'Fos': 0.0}, n_propagation=3)
```

## Evaluation signals

- Fos KO simulation should displace cells away from the iEP attractor in the vector-field map (opposite direction to overexpression)
- Fos overexpression simulation should predict a major shift from early-transition to transition clusters plus dead-end to reprogrammed shifts
- Hnf4α-Foxa1 + Fos + Yap1 condition should show higher iEP identity scores than the cocktail lacking Fos/Yap1, consistent with reported reprogramming efficiencies
- KO and overexpression vector fields should be directionally opposite to one another on the same embedding
- Per-cell identity_scores.csv should aggregate consistently per reprogramming cocktail condition

## Limitations

- Predictions are simulation-based: the vector field reflects GRN-model extrapolation (edge removal/reinforcement) and must be validated experimentally (e.g., via reprogramming efficiency assays)
- Requires multiple GRN configurations per cell state; results depend on the quality and state-specificity of the inferred Links objects
- Simulation models TF perturbation effects only through inferred regulatory edges, so indirect or non-transcriptional effects are not captured

## Evidence
