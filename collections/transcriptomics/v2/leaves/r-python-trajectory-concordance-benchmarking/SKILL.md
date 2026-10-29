---
name: r-python-trajectory-concordance-benchmarking
description: Use when reimplementing a Python trajectory tool (e.g. Palantir) in R
  and you must demonstrate numerical concordance of branch probabilities or pseudotime
  against the original implementation on identical inputs.
when_to_use_negative:
- The trajectory is cyclic (e.g. B cell receptor editing) — absorbing Markov chains
  cannot represent cyclic trajectories
- You need multi-branch topologies beyond bifurcation/linear — the destiny-based diffusion
  map restricts dandelionR's trajectory inference to bifurcation or linear topology
- You lack the original tool's outputs for identical inputs — concordance cannot be
  computed and you are doing method development, not benchmarking
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: dandelionR
  role: R implementation of absorbing Markov chain branch probability inference (markovProbability)
    being benchmarked against Palantir
  repo: https://github.com/tuonglab/dandelionR
- name: destiny
  role: Computes the diffusion map and DPT pseudotime that are held identical across
    the R and Python comparisons
- name: miloR
  role: Provides the pseudobulk neighbourhoods (milo object) on which markovProbability
    operates
- name: Palantir
  role: Python reference implementation whose branching probability outputs serve
    as the concordance target
  repo: https://github.com/dpeerlab/Palantir
- name: Dandelion
  role: Original Python package whose tutorial dataset (sce_vdj) and workflow are
    the shared benchmark substrate
  repo: https://github.com/zktuong/dandelion
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:2d3de0110c854c75afb11e438e00e9e664ba2d897d25cd338cf008b19aca1681
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tuonglab/dandelionR
  tools:
  - Dandelion
  - Palantir
  - R
  - dandelionR
  - destiny
  - miloR
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# r-python-trajectory-concordance-benchmarking

## Summary

A benchmarking protocol that quantifies whether dandelionR's absorbing-Markov-chain branch probabilities reproduce Palantir's outputs when both are given identical diffusion map representations and pseudotime, using Lin's CCC, Kendall rank correlation, and MAE at the pseudobulk level.

## When to use

Use when reimplementing a Python trajectory tool (e.g. Palantir) in R and you must demonstrate numerical concordance of branch probabilities or pseudotime against the original implementation on identical inputs.

## When NOT to use

- The trajectory is cyclic (e.g. B cell receptor editing) — absorbing Markov chains cannot represent cyclic trajectories
- You need multi-branch topologies beyond bifurcation/linear — the destiny-based diffusion map restricts dandelionR's trajectory inference to bifurcation or linear topology
- You lack the original tool's outputs for identical inputs — concordance cannot be computed and you are doing method development, not benchmarking

## Inputs

- sce_vdj: SingleCellExperiment with pseudobulked VDJ feature space (Dandelion tutorial data, originally from Suo et al.)
- DiffusionMap object from destiny::DiffusionMap (n_pcs = 50, n_eigs = 10)
- Diffusion pseudotime from destiny::DPT (w_width = 0.1) with root and CD8+/CD4+ branch tips
- Reference Palantir pseudotime and branch probability outputs for the same cells

## Outputs

- Branch probabilities (CD8+T, CD4+T) and pseudotime stored in milo pseudobulk colData
- Cell-level projections via projectPseudotimeToCell
- Concordance metrics table (Lin's CCC, Kendall rank correlation, MAE) e.g. benchmark2_concordance.csv

## How to apply

Reconstruct the reference tool's inputs exactly: compute a destiny::DiffusionMap on the pseudobulk VDJ feature space with n_pcs = 50 and n_eigs = 10, then derive diffusion pseudotime with destiny::DPT (w_width = 0.1) using tips defined from pseudobulk PCA extremes (CD8+T top-most and CD4+T bottom-most on PC2, root right-most on PC1). Run dandelionR::markovProbability on the milo pseudobulk object with terminal_state set to the CD8+/CD4+ branch tips, root_cell, and knn = 30, then project values to single cells with projectPseudotimeToCell(value_key = c('pseudotime', 'CD8+T', 'CD4+T')). Compute Lin's concordance correlation coefficient and Kendall's rank correlation between R-derived and Palantir-derived values, and report metric-level deltas (e.g. Kendall tau rose from 0.790 to 0.925 while MAE worsened). Writing metrics to a CSV (e.g. benchmark2_concordance.csv) makes the comparison auditable. Note that agreement improves when inputs — not implementations — are matched, so isolate input parity from algorithmic differences.

## Related tools

- **dandelionR** (R implementation of absorbing Markov chain branch probability inference (markovProbability) being benchmarked against Palantir) — https://github.com/tuonglab/dandelionR
- **destiny** (Computes the diffusion map and DPT pseudotime that are held identical across the R and Python comparisons)
- **miloR** (Provides the pseudobulk neighbourhoods (milo object) on which markovProbability operates)
- **Palantir** (Python reference implementation whose branching probability outputs serve as the concordance target) — https://github.com/dpeerlab/Palantir
- **Dandelion** (Original Python package whose tutorial dataset (sce_vdj) and workflow are the shared benchmark substrate) — https://github.com/zktuong/dandelion

## Examples

```
markovProbability(pb.milo, terminal_state = c("CD8+T", "CD4+T"), root_cell = root, knn = 30); projectPseudotimeToCell(pb.milo, sce_vdj, value_key = c("pseudotime", "CD8+T", "CD4+T"))
```

## Evaluation signals

- Kendall rank correlation between R-derived and Palantir branch probabilities should approach the reported 0.925 (vs 0.790 with non-identical inputs)
- Lin's CCC for CD4+ probability should be reported and interpreted against the 0.9 threshold (article reported 0.850, below it)
- MAE is expected to move in the opposite direction from rank correlation when only inputs are matched — report it explicitly rather than cherry-picking
- projectPseudotimeToCell must return values for all three value_keys ('pseudotime', 'CD8+T', 'CD4+T') in cell-level colData
- Metrics must be written to a machine-readable file (benchmark2_concordance.csv) for reproducible cross-run comparison

## Limitations

- Fundamental discrepancies in diffusion map construction and KNN graph construction between R and Python packages hinder a full reimplementation of the original Dandelion and Palantir
- Even with identical inputs, some metrics (MAE) can worsen while rank correlation improves — concordance is metric-dependent
- Lin's CCC for CD4+ probability (0.850) falls below the commonly accepted threshold of 0.9, so 'reproduction' is partial, not exact

## Evidence

- [full_text] Using an identical diffusion map representation and pseudotime as input, dandelionR's absorbing Markov chain showed improved agreement with Palantir in all metrics except MAE, with Kendall rank correlation increasing from 0.790 to 0.925.: [evidence span withheld — anchor status: not_found]
- [methods] Diffusion map computation on pseudobulk feature space.: "dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10)"
- [methods] DPT pseudotime with tip and width parameters.: "dif.pse <- DPT(dm, tips = c(root, branch.tips), w_width = 0.1)"
- [results] Concordance threshold interpretation.: "Lin's CCC for CD4 + probability (0.850) falls below the commonly accepted threshold of 0.9"
- [discussion] Cross-language implementation gap.: "fundamental discrepancies in diffusion map construction and KNN graph construction between R and Python packages generally, which hinder our ability to fully implement the original Dandelion and"
- [readme] README: Palantir-inspired markov chain in dandelionR.: "a custom implementation of absorbing markov chain for pseudotime inference, inspired based on the [palantir](https://github.com/dpeerlab/Palantir) python package"
