---
name: cell-cycle-marker-expression-analysis
description: Use when a velocity-informed circular trajectory (e.g., Cytopath on the
  U2OS SMART-Seq2 cell cycle dataset) has been inferred and G1-phase cells must be
  split into early vs late stages and validated with marker expression tests.
when_to_use_negative:
- When the underlying RNA velocity estimation itself is faulty — the authors do not
  expect trajectory inference (and hence G1 partitioning) to be meaningful in that
  case
- When root and terminal cell states are not clearly identifiable — the authors do
  not recommend performing trajectory inference with Cytopath then
- When the analysis must rely on expression similarity alone — methods like Angle,
  ReCAT, and PAGA + velocity pseudotime fail to correctly model the repeating cell-cycle
  pattern
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cytopath
  role: Infers the circular trajectory and provides the pseudotime used to partition
    G1 cells into early- and late-stage groups
  repo: github.com/aron0093/cytopath
- name: scvelo
  role: Estimates the RNA velocity fields on the U2OS cells that supply the velocity
    transition probabilities for Cytopath's Markov chain model
  repo: https://github.com/aron0093/cytopath-notebooks
- name: Angle
  role: Cell-cycle-specific baseline trajectory method used for comparison; fails
    to correctly model the cycle
  repo: https://github.com/dynverse/ti_angle
- name: PAGA
  role: Baseline run with directionality enabled by velocity pseudotime; fails to
    correctly model the cycle
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:fffb43821e3f5fe2dfb0b5fa8ded7ca9d5e7fea17123f5b8546cad61a1864498
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Angle
  - Cytopath
  - PAGA
  - scvelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# cell-cycle-marker-expression-analysis

## Summary

Partitions G1-phase cells into early- and late-stage groups using Cytopath pseudotime from an RNA velocity-based circular trajectory, then statistically tests cell cycle marker expression between the groups. This validates that the inferred circular trajectory captures real transcriptional progression through the U2OS cell cycle.

## When to use

Use when a velocity-informed circular trajectory (e.g., Cytopath on the U2OS SMART-Seq2 cell cycle dataset) has been inferred and G1-phase cells must be split into early vs late stages and validated with marker expression tests.

## When NOT to use

- When the underlying RNA velocity estimation itself is faulty — the authors do not expect trajectory inference (and hence G1 partitioning) to be meaningful in that case
- When root and terminal cell states are not clearly identifiable — the authors do not recommend performing trajectory inference with Cytopath then
- When the analysis must rely on expression similarity alone — methods like Angle, ReCAT, and PAGA + velocity pseudotime fail to correctly model the repeating cell-cycle pattern

## Inputs

- U2OS cell cycle scRNA-seq count matrix from GEO GSE146773 (Mahdessian et al., 1,067 SMART-Seq2 cells) as an AnnData (.h5ad) object
- RNA velocity fields estimated by scvelo on the preprocessed U2OS counts
- Cell phase annotations (e.g., G1 labels) and cell cycle marker gene expression matrix

## Outputs

- Cytopath pseudotime assignment per cell along the uninterrupted circular trajectory
- Binary partition of G1-phase cells into early-stage and late-stage groups
- Statistical test result comparing cell cycle marker expression between early- and late-G1 groups

## How to apply

Load the U2OS cell cycle dataset (Mahdessian et al., 1,067 cells, SMART-Seq2, GEO GSE146773) into an AnnData object, preprocess it, and estimate RNA velocity with scvelo. Run Cytopath: build a Markov chain model from velocity transition probabilities, simulate an ensemble of trajectories, align simulations terminating at a common terminal state with dynamic time warping, average cell states per step into consensus states, and assign cells to trajectory steps via an alignment score of static and velocity profiles. Confirm the trajectory forms a single uninterrupted circle without break points. Use the Cytopath pseudotime to partition G1-phase cells into early- and late-stage groups, then statistically test cell cycle marker expression between the two groups; the paper reports a significant difference, consistent with Cytopath's ability to partition G1 into late- and early-stage cells.

## Related tools

- **Cytopath** (Infers the circular trajectory and provides the pseudotime used to partition G1 cells into early- and late-stage groups) — github.com/aron0093/cytopath
- **scvelo** (Estimates the RNA velocity fields on the U2OS cells that supply the velocity transition probabilities for Cytopath's Markov chain model) — https://github.com/aron0093/cytopath-notebooks
- **Angle** (Cell-cycle-specific baseline trajectory method used for comparison; fails to correctly model the cycle) — https://github.com/dynverse/ti_angle
- **PAGA** (Baseline run with directionality enabled by velocity pseudotime; fails to correctly model the cycle)

## Examples

```
import cytopath; cytopath.cytocurve(adata, cluster_key='phase'); from scipy.stats import mannwhitneyu; mannwhitneyu(adata[adata.obs.g1_stage=='early', :][:, markers].X, adata[adata.obs.g1_stage=='late', :][:, markers].X)
```

## Evaluation signals

- The Cytopath trajectory forms a single uninterrupted circular path from G1 through intermediate cell cycle phases back to G1 and into the G1 checkpoint stage, with no spurious endpoints or break points
- The early- vs late-G1 cell cycle marker expression comparison is statistically significant, matching the paper's reported result
- Cell-to-trajectory-step assignment is driven by the alignment score combining static expression and velocity profile similarity
- Pseudotime is robust across runs and to root/terminal probability thresholds within the reported [0.85, 1] range (reported as 'data not shown')

## Limitations

- RNA velocity estimation can be unrepresentative of true transcriptional dynamics due to simplifying assumptions (time-invariant transcription/splicing/degradation rates, single regulatory regime per gene)
- Correctness of RNA velocity estimation cannot be verified in every scenario, even when independent marker-based information is available
- Robustness of Cytopath pseudotime to root and terminal probability thresholds was only reported as 'data not shown' for a [0.85, 1] threshold range

## Evidence

- [discussion] The first is the ability to partition cells in the G1 phase of the cell cycling dataset into late- and early-stage cells.: "The first is the ability to partition cells in the G1 phase of the cell cycling dataset into late- and early-stage cells."
- [results] U2OS SMART-Seq2 dataset description: "we compared the reconstruction of the cell cycle in a dataset comprising 1,067 U2OS cells generated using the SMART-Seq2 protocol"
- [results] Cell assignment via alignment score: "Assignment is based on an alignment score that evaluates for each cell the similarity of its static as well as the velocity profile with each trajectory step."
- [readme] Cytopath method overview: "Cytopath can model complex behaviours like cycling and convergence as well as cooccurring combinations of multiple"
