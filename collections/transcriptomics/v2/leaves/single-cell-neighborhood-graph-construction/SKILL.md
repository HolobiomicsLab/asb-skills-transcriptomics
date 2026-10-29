---
name: single-cell-neighborhood-graph-construction
description: Use when scRNA-seq data with a pre-computed RNA velocity field (e.g.,
  scVelo on an AnnData object with cluster annotations) must be converted into a probabilistic
  cell-to-cell transition graph for fate/trajectory simulation.
when_to_use_negative:
- RNA velocity profiles are not robust or unavailable — in that case FateCompass instead
  biases transition probabilities with the gradient of a differentiation potential,
  which requires prior biological knowledge of terminal states
- The goal is TF activity estimation rather than trajectory construction — use the
  subsequent tf_activities / differential_motif_activity_analysis steps instead of
  the graph-building step
- The embedding space must be biologically interpretable on its own — the article
  cautions that dimensionality reduction methods such as UMAP lack biological interpretability
  and should be treated carefully
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: FateCompass
  role: Consumes the k-NN velocity-directed graph to run the discrete Markov process
    sampling differentiation trajectories from progenitors to terminal fates
  repo: https://github.com/sarajimenez/fatecompass
- name: scVelo
  role: Computes the RNA velocity field whose vectors direct the edges of the neighborhood
    graph
  repo: https://scvelo.readthedocs.io/en/stable/
- name: Scanpy
  role: Performs the basic pre-processing (clustering, UMAP embedding) of the AnnData
    object before graph construction
  repo: https://scanpy.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:cabc495754f66017177e51de73cb72eb5a5c55aeb3d5033bdb9cbf522655b0c1
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
  - FateCompass
  - Python
  - Scanpy
  - scVelo
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# single-cell-neighborhood-graph-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a nearest-neighbor graph over single cells in a reduced (UMAP) embedding and directs its edges with RNA velocity vectors, providing the transition network on which FateCompass's discrete Markov chain simulates differentiation trajectories.

## When to use

Use when scRNA-seq data with a pre-computed RNA velocity field (e.g., scVelo on an AnnData object with cluster annotations) must be converted into a probabilistic cell-to-cell transition graph for fate/trajectory simulation.

## When NOT to use

- RNA velocity profiles are not robust or unavailable — in that case FateCompass instead biases transition probabilities with the gradient of a differentiation potential, which requires prior biological knowledge of terminal states
- The goal is TF activity estimation rather than trajectory construction — use the subsequent tf_activities / differential_motif_activity_analysis steps instead of the graph-building step
- The embedding space must be biologically interpretable on its own — the article cautions that dimensionality reduction methods such as UMAP lack biological interpretability and should be treated carefully

## Inputs

- Pre-processed AnnData object (adata) with 'clusters' cell-type annotations and an optional RNA velocity field (scVelo), e.g., 3,696-cell mouse pancreas E15.5 dataset
- Number of neighbors parameter (k=10 in the E15.5 workflow)
- UMAP embedding of the cells (reduced expression space)

## Outputs

- Nearest-neighbor cell-cell graph (k=10) in UMAP space with RNA-velocity-directed edges
- Cell-to-cell transition probability matrix feeding the discrete Markov chain
- Aggregated fate probabilities per starting cell (e.g., fate_probabilities_e155.csv) and end-fate frequency distribution across terminal states

## How to apply

Start from a pre-processed AnnData object (e.g., the 3,696-cell Bastidas-Ponce E15.5 mouse pancreas dataset) that already carries cluster annotations and an RNA velocity field computed with scVelo after Scanpy-based preprocessing. Build the nearest-neighbor network in UMAP space with k=10 neighbors, then derive per-cell transition probabilities from the RNA velocity vectors so that velocity directs the edges of the discrete Markov chain — FateCompass uses this graph when RNA velocity profiles are robust; otherwise a differentiation-potential gradient can bias transitions instead. Run the trajectory sampling from progenitor states to terminal fates using the FateCompass module (fatecompass.py) via the differentiation_trajectories step of the reproducibility pipeline (implemented in Matlab for the published datasets), aggregating sampled trajectories into fate probabilities per starting cell. Correctness is judged by checking that end-fate frequencies are reproducible within the tolerance implied by stochastic Markov sampling (e.g., the reported ~12.4% alpha and ~77% beta over 2,000 iterations).

## Related tools

- **FateCompass** (Consumes the k-NN velocity-directed graph to run the discrete Markov process sampling differentiation trajectories from progenitors to terminal fates) — https://github.com/sarajimenez/fatecompass
- **scVelo** (Computes the RNA velocity field whose vectors direct the edges of the neighborhood graph) — https://scvelo.readthedocs.io/en/stable/
- **Scanpy** (Performs the basic pre-processing (clustering, UMAP embedding) of the AnnData object before graph construction) — https://scanpy.readthedocs.io/en/stable/

## Examples

```
conda env create -f fatecompass.yml; jupyter notebook tutorials/fatecompass_basics_rna_velocity.ipynb
```

## Evaluation signals

- Verify the output file fate_probabilities_e155.csv exists after running the differentiation_trajectories step
- Reproduced end-fate frequencies match the reported distribution (alpha ~12.4%, beta ~77%) within the tolerance implied by stochastic Markov sampling over 2,000 iterations
- Simulations mainly terminate in three final endocrine fates (alpha, beta, and epsilon), consistent with the reported result
- Every cell in the graph has a defined transition probability row derived from its k=10 nearest neighbors and velocity vectors, with terminal endocrine states acting as absorbing states

## Limitations

- Graphs built in UMAP space inherit that method's lack of biological interpretability, so dimensionality-reduced coordinates should be treated carefully
- Pipeline input parameters (e.g., number of neighbors) must be hand tuned, although detailed tutorials are provided for parameter setup
- Differentiation-potential mode of edge direction depends on prior biological knowledge, which is difficult to provide for systems such as cancer

## Evidence

- [other] Build the nearest-neighbor network in UMAP space with k=10 neighbors and derive transition probabilities from the RNA velocity vectors, which FateCompass uses to direct the edges of the discrete Markov chain.: [evidence span withheld — anchor status: not_found]
