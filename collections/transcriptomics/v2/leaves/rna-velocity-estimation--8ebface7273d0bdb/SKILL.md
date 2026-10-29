---
name: rna-velocity-estimation
description: Use when scRNA-seq data with spliced and unspliced counts (e.g. SMART-Seq2
  U2OS cell cycle data) must be converted into velocity vectors and a cell-to-cell
  transition matrix to support directionality-aware trajectory reconstruction.
when_to_use_negative:
- When the underlying RNA velocity estimation itself is faulty or unrepresentative
  of true transcriptional dynamics (e.g. time-dependent degradation rates), the authors
  state trajectory inference with Cytopath is not meaningful
- When root and terminal cell states are not clearly identifiable — the authors do
  not recommend performing trajectory inference with Cytopath in this setting (e.g.
  fuzzy root cell probability in the hepatocyte zonation dataset)
- When the data lack spliced/unspliced count information required for velocity estimation
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scvelo
  role: Estimates the RNA velocity field and transition probabilities from spliced/unspliced
    counts; also provides latent time as a velocity-informed reference ordering
  repo: https://cytopath.readthedocs.io/en/latest/
- name: Cytopath
  role: Consumes the velocity-derived Markov chain model to simulate trajectories,
    align them with dynamic time warping, and build consensus trajectories
  repo: https://github.com/aron0093/cytopath
- name: velocyto
  role: Original RNA velocity method whose stationary-state formulation (La Manno
    et al.) underlies the Pr/Pt root and terminal state estimation
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:e951580b461e207e5271a2ad13d311f1ca1feacd335c6934eecd37e1f151dbf9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Angle
  - Cytopath
  - MultiVelo
  - PAGA
  - TFvelo
  - UniTVelo
  - VeloAE
  - scvelo
  - velocyto
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# rna-velocity-estimation

## Summary

Estimate RNA velocity fields (spliced/unspliced dynamics) from single-cell RNA-seq count matrices with scvelo, producing the velocity-informed transition probability matrix that Cytopath requires for simulation-based trajectory inference.

## When to use

Use when scRNA-seq data with spliced and unspliced counts (e.g. SMART-Seq2 U2OS cell cycle data) must be converted into velocity vectors and a cell-to-cell transition matrix to support directionality-aware trajectory reconstruction.

## When NOT to use

- When the underlying RNA velocity estimation itself is faulty or unrepresentative of true transcriptional dynamics (e.g. time-dependent degradation rates), the authors state trajectory inference with Cytopath is not meaningful
- When root and terminal cell states are not clearly identifiable — the authors do not recommend performing trajectory inference with Cytopath in this setting (e.g. fuzzy root cell probability in the hepatocyte zonation dataset)
- When the data lack spliced/unspliced count information required for velocity estimation

## Inputs

- AnnData object with spliced and unspliced count layers (e.g. U2OS cell cycle SMART-Seq2 data, 1,067 cells from GSE146773)
- Optional user-supplied root/terminal cell state annotations for supervised Cytopath mode
- Optional pre-computed cell clustering for defining terminal regions (else Louvain clustering is computed internally)

## Outputs

- RNA velocity field per cell from scvelo (velocity matrix and latent time)
- Cell-to-cell transition probability matrix (Markov random-walk model)
- Root cell probability (Pr) and terminal cell probability (Pt) per cell, thresholded at 0.99 (stationary_states.csv with cell identifier, Pr, Pt, assigned state)
- Terminal regions from Louvain clustering of selected terminal cells

## How to apply

Start from an AnnData object with spliced and unspliced count layers; for the U2OS cell cycle benchmark this is 1,067 cells (Mahdessian et al., GSE146773) generated with SMART-Seq2. Preprocess the count matrix (filter and normalize) and run scvelo to estimate per-gene velocities and the velocity graph, yielding the transition probability matrix consumed downstream. This matrix feeds two decisions: (1) absorbing-state estimation following the La Manno et al. stationary-state formulation to compute root probability (Pr) and terminal probability (Pt), thresholded at 0.99 to select root and terminal cells (or overridden by user-supplied annotations in supervised mode); and (2) Markov-chain simulation of cell state trajectories whose alignment (dynamic time warping) and averaging produce consensus states. The rationale is that static expression profiles are ambiguous with respect to directionality, so velocity is what disambiguates root vs. terminal states and cyclical topologies like the cell cycle. Verify the result before trusting downstream inference: velocity should be coherent with known marker dynamics (e.g. G1-phase markers separating early vs late G1) and Pr/Pt values must lie in [0, 1].

## Related tools

- **scvelo** (Estimates the RNA velocity field and transition probabilities from spliced/unspliced counts; also provides latent time as a velocity-informed reference ordering) — https://cytopath.readthedocs.io/en/latest/
- **Cytopath** (Consumes the velocity-derived Markov chain model to simulate trajectories, align them with dynamic time warping, and build consensus trajectories) — https://github.com/aron0093/cytopath
- **velocyto** (Original RNA velocity method whose stationary-state formulation (La Manno et al.) underlies the Pr/Pt root and terminal state estimation)

## Examples

```
import scvelo as scv; adata = scv.read('u2os_gse146773.h5ad'); scv.pp.filter_and_normalize(adata); scv.tl.velocity(adata); scv.tl.velocity_graph(adata)
```

## Evaluation signals

- stationary_states.csv exists with fields for cell identifier, Pr, Pt, and assigned state; all probability values lie within [0, 1] and selected cells satisfy the 0.99 threshold
- Pseudotime inferred by Cytopath is robust to root and terminal probability thresholds within the range [0.85, 1]
- On the U2OS dataset, the downstream trajectory forms a single uninterrupted circular trajectory (G1 through intermediate phases back to G1 and the G1 checkpoint) with no spurious endpoints
- Spearman correlation between Cytopath pseudotime and known cell type ordering (e.g. hippocampus dataset) is high and robust across multiple runs
- Early- vs late-G1 partitioning from Cytopath pseudotime yields statistically significant differences in cell cycle marker expression

## Limitations

- RNA velocity estimation rests on simplifying assumptions (time-invariant transcription/splicing/degradation rates, single regulatory regime per gene) and can be unrepresentative of true dynamics
- Faulty velocity estimates cause spurious root and terminal state inference, as seen in the erythroid gastrulation dataset and in a simulated scvelo dataset with time-dependent degradation rates
- The hepatocyte zonation dataset showed fuzzy, patternless root cell probability — no usable directionality signal
- Robustness to probability thresholds within [0.85, 1] is reported only as 'data not shown'

## Evidence

- [readme] Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model: "Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model; simulation of this model yields an ensemble of possible differentiation trajectories"
- [results] Root and terminal states from absorbing-state estimation: [evidence span withheld — anchor status: not_found]
- [results] U2OS dataset details: "we compared the reconstruction of the cell cycle in a dataset comprising 1,067 U2OS cells generated using the SMART-Seq2 protocol."
- [discussion] Velocity limitation: "We do not expect that trajectory inference with Cytopath is meaningful when the underlying RNA velocity estimation itself is faulty."
