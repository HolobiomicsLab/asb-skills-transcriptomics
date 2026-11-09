---
name: anndata-single-cell-data-handling
description: Use when an RNA velocity trajectory analysis must load, validate, and
  pass scvelo-processed AnnData (spliced/unspliced counts, velocity field) into Cytopath's
  Markov-chain simulation or between its workflow stages.
when_to_use_negative:
- The underlying RNA velocity estimation itself is faulty (e.g. time-dependent degradation
  rates, multiple regulatory regimes per gene), which produces spurious root and terminal
  states
- Root and terminal cell states are not clearly identifiable — the authors do not
  recommend performing Cytopath trajectory inference in this situation
- Input is raw counts without an estimated velocity field, since Markov random-walk
  stationary-state estimation depends on the RNA velocity-derived transition probability
  matrix
tools:
- name: scvelo
  role: Produces the processed AnnData object (spliced/unspliced counts, velocity
    field) that is loaded and passed between Cytopath workflow steps
- name: Cytopath
  role: Consumes the AnnData object to build the Markov random-walk transition matrix,
    estimate stationary states (Pr/Pt), and write stationary_states.csv
  repo: https://github.com/aron0093/cytopath
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:982d1891724fb244549d35a57ebcbcc339f175901ffe803cfa3d052342f50a29
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aron0093/cytopath
  tools:
  - Cytopath
  - scvelo
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# anndata-single-cell-data-handling

## Summary

Handles the scvelo-processed AnnData object (spliced/unspliced counts plus the RNA velocity field) as the data backbone of a Cytopath trajectory-inference workflow, feeding it into Markov-chain stationary-state estimation and trajectory simulation, and serializing intermediate results such as stationary_states.csv.

## When to use

Use when an RNA velocity trajectory analysis must load, validate, and pass scvelo-processed AnnData (spliced/unspliced counts, velocity field) into Cytopath's Markov-chain simulation or between its workflow stages.

## When NOT to use

- The underlying RNA velocity estimation itself is faulty (e.g. time-dependent degradation rates, multiple regulatory regimes per gene), which produces spurious root and terminal states
- Root and terminal cell states are not clearly identifiable — the authors do not recommend performing Cytopath trajectory inference in this situation
- Input is raw counts without an estimated velocity field, since Markov random-walk stationary-state estimation depends on the RNA velocity-derived transition probability matrix

## Inputs

- scvelo-processed AnnData object (.h5ad) with spliced/unspliced counts and an RNA velocity field
- optional user-supplied root/terminal state annotations for supervised mode
- optional pre-computed cell clustering used to define terminal regions

## Outputs

- stationary_states.csv with fields for cell identifier, root probability (Pr), terminal probability (Pt), and assigned state
- root cell set and terminal cell set (cells with Pr/Pt >= 0.99)
- terminal regions derived from the supplied or internal Louvain clustering

## How to apply

Load the scvelo-processed AnnData object containing spliced/unspliced counts and the velocity field as the single in-memory container passed between workflow steps. From this AnnData, Cytopath builds a Markov random-walk transition probability matrix and computes root (Pr) and terminal (Pt) cell probabilities via absorbing-state estimation following the La Manno et al. stationary-state formulation; Pr and Pt are thresholded at 0.99 to select root and terminal cells unless user-supplied annotations are used (supervised mode). Terminal cells are grouped into terminal regions using a supplied clustering or an internal Louvain clustering when none is provided. Write the root cell set, terminal cell set, terminal regions, and raw Pr/Pt values to stationary_states.csv, keeping the AnnData object as the source of static and velocity profiles for downstream alignment-score-based cell assignment to trajectory steps.

## Related tools

- **scvelo** (Produces the processed AnnData object (spliced/unspliced counts, velocity field) that is loaded and passed between Cytopath workflow steps)
- **Cytopath** (Consumes the AnnData object to build the Markov random-walk transition matrix, estimate stationary states (Pr/Pt), and write stationary_states.csv) — https://github.com/aron0093/cytopath

## Evaluation signals

- stationary_states.csv exists and contains fields for cell identifier, Pr, Pt, and assigned state
- All probability values in stationary_states.csv lie within [0, 1]
- Every selected root and terminal cell satisfies the 0.99 probability threshold on Pr or Pt respectively
- Terminal cells are partitioned into terminal regions using either the supplied clustering or the internal Louvain clustering fallback
- In supervised mode, root/terminal sets match the user-supplied annotations exactly rather than the 0.99-threshold selection

## Limitations

- Pseudotime robustness to root and terminal probability thresholds was only reported as 'data not shown' (stated as robust within [0.85, 1])
- Incorrect RNA velocity estimation leads to spurious inference of root and terminal cell states, as observed on the erythroid gastrulation dataset and a simulated dataset with time-dependent degradation rates
- The correctness of RNA velocity estimation cannot be verified in every scenario, even when independent marker-based information is available

## Evidence

- [results] Root and terminal states can be derived from a Markov random-walk model utilizing the transition probability matrix itself: "Root and terminal states can be derived from a Markov random-walk model utilizing the transition probability matrix itself"
- [results] otherwise, a clustering is computed internally using Louvain.: "otherwise, a clustering is computed internally using Louvain."
- [readme] Cytopath can infer trajectories with or without root/terminal state supervision.: "Cytopath can infer trajectories with or without root/terminal state supervision."
- [readme] Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model: "Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model"
- [discussion] We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable.: "We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable."
