---
name: markov-chain-trajectory-sampling
description: Use when single-cell transcriptomics data (e.g., a Scanpy/scVelo-preprocessed
  AnnData with cluster annotations and an RNA velocity field) must be converted into
  stochastic trajectories and fate probabilities toward defined terminal states.
when_to_use_negative:
- RNA velocity profiles are unreliable or absent and no prior biological knowledge
  is available to define a differentiation potential (the alternative mode requires
  prior knowledge, which is difficult to provide e.g. in cancer studies)
- The goal is transcription factor activity estimation rather than trajectory sampling
  — that is handled by the downstream tf_activities/linear gene regulation step
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: FateCompass
  role: Implements the RNA-velocity-driven discrete Markov chain and stochastic trajectory
    sampling in fatecompass.py
  repo: https://github.com/sarajimenez/fatecompass
- name: scVelo
  role: Computes the RNA velocity field that directs the edges of the Markov chain
- name: Scanpy
  role: Performs basic pre-processing of the single-cell data before trajectory sampling
- name: Python
  role: Language in which FateCompass is fully implemented; the reproducibility scenario
    is executed as differentiation_trajectories
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:79234f4e2e84002d169f85ae0642d250306e32d60d7e22e894536ed15c4efb2b
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# markov-chain-trajectory-sampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Samples stochastic cell differentiation trajectories by running a discrete Markov chain on a nearest-neighbor graph whose edges are directed by RNA velocity vectors, yielding fate probabilities and end-fate frequencies from progenitor to terminal cell states.

## When to use

Use when single-cell transcriptomics data (e.g., a Scanpy/scVelo-preprocessed AnnData with cluster annotations and an RNA velocity field) must be converted into stochastic trajectories and fate probabilities toward defined terminal states.

## When NOT to use

- RNA velocity profiles are unreliable or absent and no prior biological knowledge is available to define a differentiation potential (the alternative mode requires prior knowledge, which is difficult to provide e.g. in cancer studies)
- The goal is transcription factor activity estimation rather than trajectory sampling — that is handled by the downstream tf_activities/linear gene regulation step

## Inputs

- Pre-processed AnnData (.h5ad) object with cell-type cluster annotations and an RNA velocity field (Scanpy/scVelo preprocessed)
- Nearest-neighbor graph parameters (e.g., k=10 neighbors in UMAP space)
- Optionally, a differentiation potential annotation as an alternative driver of transition probabilities

## Outputs

- Sampled differentiation trajectories from progenitor to terminal states
- Per-cell fate probability table (e.g., fate_probabilities_e155.csv)
- Overall end-fate frequency distribution across terminal states (e.g., α, β, ε)

## How to apply

Start from a pre-processed AnnData object (e.g., the 3,696-cell Bastidas-Ponce E15.5 mouse pancreas dataset) containing 'clusters' annotations and an RNA velocity field computed with scVelo. Build a nearest-neighbor network (k=10 neighbors) in the reduced (UMAP) space and derive transition probabilities from the RNA velocity vectors, which direct the edges of the discrete Markov chain. Run stochastic trajectory sampling (e.g., 2,000 iterations) from progenitor states to terminal states using the FateCompass module in fatecompass.py. Aggregate sampled trajectories to compute per-cell fate probabilities and overall end-fate frequencies across terminal states (e.g., α ~12.4%, β ~77%, and ε for the pancreas E15.5 data). If velocity profiles are not robust, an alternative mode biases transition probabilities with the gradient of a differentiation potential derived from prior biological knowledge.

## Related tools

- **FateCompass** (Implements the RNA-velocity-driven discrete Markov chain and stochastic trajectory sampling in fatecompass.py) — https://github.com/sarajimenez/fatecompass
- **scVelo** (Computes the RNA velocity field that directs the edges of the Markov chain)
- **Scanpy** (Performs basic pre-processing of the single-cell data before trajectory sampling)
- **Python** (Language in which FateCompass is fully implemented; the reproducibility scenario is executed as differentiation_trajectories)

## Examples

```
conda env create -f fatecompass.yml; jupyter notebook tutorials/fatecompass_basics_rna_velocity.ipynb
```

## Evaluation signals

- fate_probabilities_e155.csv exists after the run
- End-fate frequencies across 2,000 iterations match the reported distribution (alpha ~12.4%, beta ~77%) within the tolerance implied by stochastic Markov sampling
- Simulated trajectories terminate in the annotated final endocrine fates (α, β, and ε for the E15.5 pancreas dataset)
- Transition probabilities derived from RNA velocity vectors are consistent with the k=10 nearest-neighbor network built in UMAP space

## Limitations

- Trajectories are built on a nearest-neighbor graph in a reduced gene expression space; dimensionality reduction methods such as UMAP lack biological interpretability
- The differentiation-potential mode depends on prior biological knowledge, which is difficult to provide in some cases such as cancer studies
- Pipeline input parameters must be hand tuned
- Stochastic Markov sampling introduces variability, so reproduced end-fate frequencies match reported values only within stochastic tolerance
