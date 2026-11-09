---
name: potential-energy-landscape-construction-cell-graph
description: Use when inferring differentiation trajectories without reliable RNA
  velocity, e.g. from progenitor clusters (NEUROG3-early/mid/late) to terminal fates
  (SC-α, SC-β, SC-EC) in a preprocessed AnnData object.
when_to_use_negative:
- RNA velocity profiles are robust — FateCompass should then use velocity to direct
  the Markov chain edges instead of the differentiation-potential gradient
- No prior biological knowledge of progenitor and terminal states is available (e.g.
  cancer studies), since the potential-energy definition requires labeling sources
  and sinks
- The goal is TF activity estimation rather than fate-probability inference — use
  the linear model of gene regulation workflow instead
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: FateCompass
  role: Python module (fatecompass.py) implementing graph construction, potential-energy
    landscape definition, and the Markov propagator biased by differentiation potential
  repo: github.com/sarajimenez/fatecompass
- name: Python
  role: Implementation language; FateCompass is fully implemented in python with Scanpy/scVelo
    suggested for preprocessing
- name: MATLAB
  role: Runs the reproducibility scripts (differentiation_trajectories.m) for differentiation-potential-driven
    trajectories
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:62beb3aa69d6ae2565c183f1cf6a1c8bc2412135179e970b0713a95023738442
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Potential-energy-landscape construction on a cell-neighborhood graph

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a potential-energy landscape from differentiation potential over a cell-neighborhood graph, combining repulsive source forces at progenitor states with attractive sink forces at terminal fates to bias Markov transition probabilities in FateCompass.

## When to use

Use when inferring differentiation trajectories without reliable RNA velocity, e.g. from progenitor clusters (NEUROG3-early/mid/late) to terminal fates (SC-α, SC-β, SC-EC) in a preprocessed AnnData object.

## When NOT to use

- RNA velocity profiles are robust — FateCompass should then use velocity to direct the Markov chain edges instead of the differentiation-potential gradient
- No prior biological knowledge of progenitor and terminal states is available (e.g. cancer studies), since the potential-energy definition requires labeling sources and sinks
- The goal is TF activity estimation rather than fate-probability inference — use the linear model of gene regulation workflow instead

## Inputs

- Pre-processed AnnData object with cell-type cluster annotations and differentiation potentials (e.g. Veres pancreatic endocrine dataset)
- TF binding-site matrix (rows = expressed genes, columns = TFs or motifs)
- Definition of progenitor (source) and terminal-fate (sink) cell clusters

## Outputs

- Biased Markov transition-probability matrix over the cell-neighborhood graph
- Stochastic trajectory endpoints per progenitor cluster (|Q|=100 trajectories each)
- fate_probabilities.csv with fate-probability distributions per progenitor cluster

## How to apply

Load the preprocessed AnnData (e.g. the Veres pancreatic endocrine dataset with 'clusters' annotations and per-cell differentiation potentials) plus a TF binding-site matrix (rows = expressed genes, columns = TFs/motifs) into the FateCompass Python module (fatecompass.py). Build the cell-neighborhood graph in the reduced gene-expression space and define the potential-energy landscape from the differentiation-potential gradient, assigning repulsive source forces at progenitor states and attractive sink forces at terminal fates. Use this landscape to bias transition probabilities of the discrete Markov chain on the graph (Equations 5-8), with exponent n=0.5. Propagate |Q|=100 stochastic trajectories per progenitor cluster, accepting proposed moves with a Metropolis-style criterion and sampling rejected moves from the rejection distribution. Aggregate trajectory endpoints into fate-probability distributions per progenitor cluster (fate_probabilities.csv) and compare against the reference results in the FateCompass Shiny App. This mode is chosen when RNA velocity profiles are not robust; otherwise use the RNA-velocity-driven variant of the same propagator.

## Related tools

- **FateCompass** (Python module (fatecompass.py) implementing graph construction, potential-energy landscape definition, and the Markov propagator biased by differentiation potential) — github.com/sarajimenez/fatecompass
- **Python** (Implementation language; FateCompass is fully implemented in python with Scanpy/scVelo suggested for preprocessing)
- **MATLAB** (Runs the reproducibility scripts (differentiation_trajectories.m) for differentiation-potential-driven trajectories)

## Examples

```
import fatecompass; fc = fatecompass.FateCompass(adata, bnd_mtx); fc.build_graph(); fc.compute_transition_probabilities(mode='differentiation_potential', exponent=0.5)
```

## Evaluation signals

- Fate-probability distributions reproduce the reported pattern: SC-alpha fates predominantly from NEUROG3-early progenitors, while SC-beta/SC-EC fates come from NEUROG3-late progenitors
- SC-alpha as a final fate strongly decreases for NEUROG3-mid and NEUROG3-late progenitors, matching reference results in the FateCompass Shiny App
- Each progenitor cluster yields exactly |Q|=100 trajectory endpoints that all terminate in defined sink (terminal fate) states
- Transition probabilities respect the potential-energy bias (moves toward lower potential-energy / sink states accepted under the Metropolis criterion)
- Output fate_probabilities.csv contains one distribution per progenitor cluster summing to 1 over terminal fates

## Limitations

- Differentiation-potential mode depends on prior biological knowledge to define sources and sinks, which is difficult to provide in some cases such as cancer studies
- Trajectories are built on a nearest-neighbor graph in reduced gene-expression space; common dimensionality-reduction methods such as UMAP lack biological interpretability and should be treated carefully
- Pipeline input parameters must be hand tuned, although detailed tutorials are provided

## Evidence
