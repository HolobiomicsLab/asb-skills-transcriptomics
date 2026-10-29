---
name: pancreatic-endocrine-lineage-annotation
description: Use when annotating mouse pancreatic endocrine scRNA-seq (e.g., E15.5
  Bastidas-Ponce AnnData with 'clusters' and an scVelo RNA velocity field) into progenitor
  and terminal hormone-producing fates so downstream fate-probability or trajectory-sampling
  methods have defined terminal states.
when_to_use_negative:
- Input AnnData lacks cell-type 'clusters' annotations, so progenitor and terminal
  hormone populations cannot be defined as Markov-chain start and absorbing states
- The system is not a pancreatic endocrine (or comparable differentiating) lineage
  with biologically defined terminal fates — e.g., cancer studies, where the differentiation-potential
  mode's required prior biological knowledge is hard to provide
- The dataset has no RNA velocity field and velocity-driven trajectory sampling was
  intended; pre-compute velocities with scVelo or switch to the differentiation-potential-driven
  scenario
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FateCompass
  role: Runs the RNA-velocity-driven discrete Markov chain from progenitor states
    to annotated terminal endocrine fates and ranks terminal-fate probabilities (module
    fatecompass.py)
  repo: github.com/sarajimenez/fatecompass
- name: Scanpy
  role: Basic pre-processing of the E15.5 AnnData object (clustering and cell-type
    annotations) before lineage annotation
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scVelo
  role: Computes the RNA velocity field used to direct Markov-chain edges toward the
    annotated terminal fates
  repo: https://scvelo.readthedocs.io/en/stable/
- name: CellRank
  role: Baseline Markov-chain estimator whose terminal-state calls (ε and β found
    terminal; α missed) are compared against FateCompass fate rankings
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:7a8cc9cb9f714618c192b4608362ca86fe90051c82690dbc91592767b903dc89
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  tools:
  - CellRank
  - FateCompass
  - Python
  - Scanpy
  - scVelo
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# pancreatic-endocrine-lineage-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Defines pancreatic endocrine cell lineages on the pre-processed Bastidas-Ponce E15.5 mouse pancreas AnnData object (3,696 cells), anchoring progenitor-to-terminal hormone fates (α, β, ε, and other endocrine states) that FateCompass's RNA-velocity-driven Markov chain and CellRank baselines use for fate-probability estimation.

## When to use

Use when annotating mouse pancreatic endocrine scRNA-seq (e.g., E15.5 Bastidas-Ponce AnnData with 'clusters' and an scVelo RNA velocity field) into progenitor and terminal hormone-producing fates so downstream fate-probability or trajectory-sampling methods have defined terminal states.

## When NOT to use

- Input AnnData lacks cell-type 'clusters' annotations, so progenitor and terminal hormone populations cannot be defined as Markov-chain start and absorbing states
- The system is not a pancreatic endocrine (or comparable differentiating) lineage with biologically defined terminal fates — e.g., cancer studies, where the differentiation-potential mode's required prior biological knowledge is hard to provide
- The dataset has no RNA velocity field and velocity-driven trajectory sampling was intended; pre-compute velocities with scVelo or switch to the differentiation-potential-driven scenario

## Inputs

- Pre-processed Bastidas-Ponce E15.5 mouse pancreas AnnData object (3,696 cells) with 'clusters' cell-type annotations
- scVelo-computed RNA velocity field on the same AnnData
- KNN graph parameters (e.g., k=10 neighbors in UMAP space)
- TF binding-sites matrix (columns = TFs/motifs, rows = expressed genes) for downstream FateCompass TF-activity steps

## Outputs

- Terminal-fate labels for endocrine populations (α, β, ε, and other endocrine states) used as Markov-chain absorbing states
- Per-cell fate probabilities (fate_probabilities_e155.csv)
- Overall end-fate frequency distribution across 2,000 simulated trajectories (alpha ~12.4%, beta ~77%)
- CellRank-vs-FateCompass comparison table (cellrank_fate_probabilities.csv) with per-population fate probability, terminal-state call, and FateCompass terminal-fate ranking

## How to apply

Load the pre-processed E15.5 AnnData object containing cell-type annotations in the 'clusters' slot (including Ductal and Ngn3-low EP populations) plus the scVelo-derived RNA velocity field, following Scanpy/scVelo preprocessing as suggested for FateCompass. Treat terminal hormone-producing populations (α, β, ε and other endocrine states) as candidate terminal fates and progenitor/endocrine-progenitor clusters as start states for the discrete Markov chain. Build the KNN graph (e.g., k=10 neighbors in UMAP space) and derive transition probabilities from RNA velocity to direct the Markov-chain edges; then run FateCompass (fatecompass.py) in RNA-velocity-driven mode over 2,000 iterations, aggregating sampled trajectories into per-cell fate probabilities and overall end-fate frequencies (reported: β ~77%, α ~12.4%). As a sanity baseline, run CellRank's velocity kernel and note that it identified ε and β as terminal but failed to call the glucagon-producing α cells terminal, whereas FateCompass recovered β as the most likely terminal fate across parameters. Record results as fate_probabilities_e155.csv and a CellRank-vs-FateCompass comparison table (cellrank_fate_probabilities.csv).

## Related tools

- **FateCompass** (Runs the RNA-velocity-driven discrete Markov chain from progenitor states to annotated terminal endocrine fates and ranks terminal-fate probabilities (module fatecompass.py)) — github.com/sarajimenez/fatecompass
- **Scanpy** (Basic pre-processing of the E15.5 AnnData object (clustering and cell-type annotations) before lineage annotation) — https://scanpy.readthedocs.io/en/stable/
- **scVelo** (Computes the RNA velocity field used to direct Markov-chain edges toward the annotated terminal fates) — https://scvelo.readthedocs.io/en/stable/
- **CellRank** (Baseline Markov-chain estimator whose terminal-state calls (ε and β found terminal; α missed) are compared against FateCompass fate rankings)

## Examples

```
conda env create -f fatecompass.yml; jupyter notebook tutorials/fatecompass_basics_rna_velocity.ipynb
```

## Evaluation signals

- File fate_probabilities_e155.csv exists and end-fate frequencies match the reported distribution (alpha ~12.4%, beta ~77%) within the tolerance implied by stochastic Markov sampling
- β is recovered as the most likely terminal fate for endocrine progenitors across all tested parameters and kernels
- CellRank baseline reproduces the expected calls: ε and β identified as terminal hormone-producing fates, glucagon-producing α cells not identified as terminal
- cellrank_fate_probabilities.csv has one row per cell population with columns for CellRank fate probability, CellRank terminal-state call, and FateCompass terminal-fate ranking
- Simulations terminated only in final endocrine fates (α, β, ε) — no trajectories ending in non-endocrine clusters such as Ductal or Ngn3-low EP

## Limitations

- FateCompass depends on input parameters that must be hand tuned, although detailed tutorials guide the parameter setup
- Commonly used dimensionality-reduction spaces such as UMAP lack biological interpretability, so neighbor graphs built in that space should be treated carefully
- Results are stochastic: end-fate frequencies from 2,000 Markov-chain iterations carry sampling variability, so validation is against the reported distribution within tolerance

## Evidence
