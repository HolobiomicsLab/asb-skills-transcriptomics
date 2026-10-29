---
name: grn-based-cell-identity-transition-simulation
description: Use when scRNA-seq data and a CellOracle Links/GRN object are available
  and the goal is to predict, in silico, how knocking out or overexpressing a candidate
  TF (e.g., Fos) redirects cells among known clusters or attractors before committing
  to experiments.
when_to_use_negative:
- The input is raw sequencing data that has not been processed (e.g., no Cell Ranger
  matrix or no inferred CellOracle Links/GRN object yet) — GRN inference must precede
  simulation
- The cell states of interest lack annotated clusters or a reference for identity
  scoring, so simulated transition probabilities cannot be anchored to real identity
  labels
- The question concerns TFs or regulatory edges absent from the inferred base GRN
  (e.g., base GRN derived from the Mouse sci-ATAC-seq Atlas lacks accessible motifs
  for the factor), since edge removal/reinforcement requires existing network edges
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CellOracle
  role: Infers the GRN (Links object) and runs the in silico perturbation module simulating
    TF KO (edge removal) and overexpression (edge reinforcement) to predict cell identity
    transitions
  repo: https://github.com/morris-lab/CellOracle
- name: Capybara
  role: Performs cell type classification to score each cell for fibroblast and iEP
    identity against reference profiles, validating simulated transition predictions
  repo: https://github.com/morris-lab/Capybara
- name: Cell Ranger 6.0.1
  role: Processes 10x Chromium single-cell gene expression data into the count matrices
    used to build Oracle objects
- name: Python
  role: Runtime environment for CellOracle (developed with python 3.8, tested with
    python 3.10)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:923cf795328d906ce63cf09a8efc3eb906ebaca7b505e92d9e6e931bbf7319d9
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
  - Cell Ranger
  - Cell Ranger 6.0.1
  - CellOracle
  - Python
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# grn-based-cell-identity-transition-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulates how transcription factor perturbation (KO via edge removal, overexpression via edge reinforcement) shifts cell identity along a CellOracle-inferred gene regulatory network, projecting predicted fate transitions as vector-field and Sankey-style transition maps over a scRNA-seq embedding.

## When to use

Use when scRNA-seq data and a CellOracle Links/GRN object are available and the goal is to predict, in silico, how knocking out or overexpressing a candidate TF (e.g., Fos) redirects cells among known clusters or attractors before committing to experiments.

## When NOT to use

- The input is raw sequencing data that has not been processed (e.g., no Cell Ranger matrix or no inferred CellOracle Links/GRN object yet) — GRN inference must precede simulation
- The cell states of interest lack annotated clusters or a reference for identity scoring, so simulated transition probabilities cannot be anchored to real identity labels
- The question concerns TFs or regulatory edges absent from the inferred base GRN (e.g., base GRN derived from the Mouse sci-ATAC-seq Atlas lacks accessible motifs for the factor), since edge removal/reinforcement requires existing network edges

## Inputs

- Cell Ranger v6.0.1-processed single-cell RNA-seq count matrix (e.g., day-20 Perturb-seq of MEF-to-iEP reprogramming)
- Sample-specific CellOracle Links (GRN) objects for each network state (reprogramming and long-term-iEP)
- Cluster/identity annotations for the embedding (e.g., early transition, transition, dead-end, reprogrammed clusters)
- Reference single-cell or bulk profiles for Capybara-style identity scoring (e.g., fibroblast and iEP references)

## Outputs

- Vector-field maps of simulated cell identity transitions over the scRNA-seq embedding
- Sankey-style transition-probability plots per simulated perturbation (KO and overexpression)
- Per-cell and per-condition identity scores (identity_scores.csv) with fibroblast/iEP scores aggregated by reprogramming cocktail

## How to apply

Load the Cell Ranger v6.0.1-processed scRNA-seq matrix (e.g., day-20 Perturb-seq from MEF-to-iEP reprogramming) together with sample-specific CellOracle Links (GRN) objects for each state (e.g., reprogramming and long-term-iEP), and build Oracle objects. Run CellOracle's in silico perturbation module, simulating knockout by removing the TF's regulatory edges and overexpression by reinforcing them, on each GRN configuration. Project simulated cell identity transitions as vector-field maps and Sankey-style transition-probability plots over the scRNA-seq embedding. For validation, independently score each cell for fibroblast vs iEP identity with Capybara, aggregate per reprogramming cocktail (e.g., Hnf4α-Foxa1 with/without Fos and/or Yap1), and export per-cell and per-condition summaries (identity_scores.csv). The result is judged correct if simulated KO displaces cells away from the iEP attractor while overexpression predicts shifts from early transition toward transition and from dead-end toward reprogrammed clusters, and if the Hnf4α-Foxa1 + Fos + Yap1 condition scores higher iEP identity than the cocktail lacking Fos/Yap1, consistent with reported reprogramming efficiencies.

## Related tools

- **CellOracle** (Infers the GRN (Links object) and runs the in silico perturbation module simulating TF KO (edge removal) and overexpression (edge reinforcement) to predict cell identity transitions) — https://github.com/morris-lab/CellOracle
- **Capybara** (Performs cell type classification to score each cell for fibroblast and iEP identity against reference profiles, validating simulated transition predictions) — https://github.com/morris-lab/Capybara
- **Cell Ranger 6.0.1** (Processes 10x Chromium single-cell gene expression data into the count matrices used to build Oracle objects)
- **Python** (Runtime environment for CellOracle (developed with python 3.8, tested with python 3.10))

## Examples

```
import celloracle as co; oracle = co.load_hdf5('oracle.object'); oracle.simulate_single_cell_perturbation(perturbation_condition='Fos-KO'); oracle.trajectory.simulate_trajectory()
```

## Evaluation signals

- Simulated Fos KO displaces cells away from the iEP attractor (shift toward dead-end/early transition identity), while simulated Fos overexpression produces the opposite shift (from early transition to transition, and from dead-end to reprogrammed clusters)
- Per-condition Capybara identity scores show Hnf4α-Foxa1 + Fos + Yap1 cells score higher iEP identity than the cocktail lacking Fos/Yap1, matching experimentally reported reprogramming efficiencies
- Per-cell and per-condition outputs are complete: identity_scores.csv contains both per-cell rows and per-condition aggregates for every simulated cocktail
- Vector-field and transition-probability projections remain interpretable over the original scRNA-seq embedding (simulated cells stay within the manifold defined by the loaded Oracle object)

## Limitations

- Predictions are in silico: the simulation identified candidates (Id1, Fosb, Fos, Eno1, Klf4) but required systematic experimental validation of each
- The Base-GRN depends on external epigenome resources (Mouse sci-ATAC-seq Atlas); factors without inferred edges cannot be meaningfully perturbed
- Python version sensitivity: CellOracle was developed with python 3.8 and tested with 3.10; R-side dependencies (igraph) must be installed for the Capybara validation step

## Evidence
