---
name: differentiation-potential-trajectory-inference
description: Use when RNA velocity profiles are inconclusive or noisy in a single-cell
  differentiation dataset and prior biological knowledge defines progenitor and terminal
  cell states (e.g. NKX6-1+ progenitors toward SC-β fates), so trajectories can be
  biased by a differentiation-potential gradient.
when_to_use_negative:
- RNA velocity profiles are robust — in that case FateCompass uses velocity to direct
  the edges of the Markov chain instead of differentiation potential
- No prior biological knowledge of progenitor and terminal cell states is available
  (e.g. cancer studies), since the differentiation-potential mode depends on such
  input
- The input is raw sequencing data rather than a pre-processed, cell-type-annotated
  AnnData object
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: FateCompass
  role: Python module (fatecompass.py) that builds the potential-energy landscape,
    biases transition probabilities, and propagates stochastic trajectories
  repo: github.com/sarajimenez/fatecompass
- name: Python
  role: Language in which FateCompass is fully implemented; used with conda environment
    from fatecompass.yml
- name: MATLAB
  role: Hosts the reproducibility scripts (differentiation_trajectories.m) that rebuild
    differentiation-potential-driven Markov-chain trajectories for the Veres dataset
- name: Scanpy
  role: Suggested for basic pre-processing of the adata object before trajectory inference
provenance:
  source_task_ids:
  - task_004
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:b4ffe4164d8c7cfb68682d58f99cd68e7c3a88d6f32de80730af47f1c8f6c4d0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  tools:
  - FateCompass
  - MATLAB
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_3935
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

# differentiation-potential-trajectory-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

FateCompass method for inferring cell-fate differentiation trajectories by biasing Markov-chain transition probabilities with the gradient of a differentiation potential when RNA velocity is inconclusive, then propagating stochastic trajectories from progenitor to terminal states to obtain per-cell fate-probability distributions.

## When to use

Use when RNA velocity profiles are inconclusive or noisy in a single-cell differentiation dataset and prior biological knowledge defines progenitor and terminal cell states (e.g. NKX6-1+ progenitors toward SC-β fates), so trajectories can be biased by a differentiation-potential gradient.

## When NOT to use

- RNA velocity profiles are robust — in that case FateCompass uses velocity to direct the edges of the Markov chain instead of differentiation potential
- No prior biological knowledge of progenitor and terminal cell states is available (e.g. cancer studies), since the differentiation-potential mode depends on such input
- The input is raw sequencing data rather than a pre-processed, cell-type-annotated AnnData object

## Inputs

- Pre-processed AnnData (adata) object with cell-type cluster annotations and differentiation potentials (e.g. Veres et al. stage-5, 25,299 cells)
- TF binding-site matrix (columns = TFs/motifs, rows = expressed genes)
- Progenitor and terminal state definitions (e.g. NKX6-1+ progenitors; SC-α, SC-β, SC-EC terminal fates)
- Parameters: |Q|=100 trajectories per progenitor cluster, exponent n=0.5

## Outputs

- Fate-probability distributions per progenitor cluster (fate_probabilities.csv)
- Discrete Markov-chain differentiation trajectories from progenitors to terminal states
- Fate-probability patterns per lineage (e.g. SC-α predominantly from NEUROG3-early progenitors; SC-β/SC-EC from NEUROG3-late)

## How to apply

Load a pre-processed AnnData object (Scanpy/scVelo pre-processing, with cell-type 'clusters' annotations and differentiation potentials) plus a TF binding-site matrix (columns = TFs/motifs, rows = expressed genes) into the FateCompass Python module. Build a cell-neighborhood graph and define a potential-energy landscape from the differentiation potential, combining repulsive source forces at progenitor states with attractive sink forces at terminal fates to bias transition probabilities. Propagate |Q|=100 stochastic trajectories per progenitor cluster using exponent n=0.5, accepting proposed moves with a Metropolis-style criterion and sampling rejections from the rejection distribution (Equations 5-8). Aggregate trajectory endpoints into fate-probability distributions per progenitor cluster (e.g. fate_probabilities.csv); these downstream feed TF-activity inference via the linear model of gene regulation. In the Veres stage-5 case, reproduce via differentiation_trajectories.m from reproducibility/scripts to build trajectories from NKX6-1+ progenitors toward SC-alpha, SC-beta and SC-EC terminal states.

## Related tools

- **FateCompass** (Python module (fatecompass.py) that builds the potential-energy landscape, biases transition probabilities, and propagates stochastic trajectories) — github.com/sarajimenez/fatecompass
- **Python** (Language in which FateCompass is fully implemented; used with conda environment from fatecompass.yml)
- **MATLAB** (Hosts the reproducibility scripts (differentiation_trajectories.m) that rebuild differentiation-potential-driven Markov-chain trajectories for the Veres dataset)
- **Scanpy** (Suggested for basic pre-processing of the adata object before trajectory inference)

## Examples

```
conda env create -f fatecompass.yml && jupyter notebook tutorials/fatecompass_basics_differentiation_potential.ipynb
```

## Evaluation signals

- Fate-probability distributions reproduce the reported pattern: SC-α fates predominantly from NEUROG3-early progenitors and SC-β/SC-EC fates from NEUROG3-late progenitors (compare against the FateCompass Shiny App)
- SC-α probability strongly decreases for NEUROG3-mid and NEUROG3-late progenitors, indicating SC-α cells arise mainly from early endocrine precursors
- fate_probabilities.csv is produced with one fate-probability distribution per progenitor cluster after aggregating trajectory endpoints
- Reproduction matches the 126 differentially active TFs and lineage-specific sets (e.g. 20 SC-β-specific, 25 SC-EC-specific) downstream in the Shiny App reference

## Limitations

- The differentiation-potential mode depends on input of prior biological knowledge, which is difficult to provide in some cases such as cancer studies
- Pipeline input parameters must be hand tuned, although detailed tutorials are provided
- Trajectories are built on a nearest-neighbor graph in reduced gene expression space; dimensionality reduction methods such as UMAP lack biological interpretability and should be treated carefully

## Evidence

- [other] Propagate |Q|=100 stochastic trajectories from each NEUROG3-early and NEUROG3-late progenitor cluster using exponent n=0.5, accepting proposed moves with a Metropolis-style criterion: [evidence span withheld — anchor status: not_found]
