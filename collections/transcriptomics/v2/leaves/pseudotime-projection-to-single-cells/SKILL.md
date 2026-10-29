---
name: pseudotime-projection-to-single-cells
description: Use when a miloR pseudobulk object carries pseudotime and branch probabilities
  computed in VDJ feature space, and per-cell values are needed for plotting, downstream
  analysis, or benchmarking against Palantir.
when_to_use_negative:
- Trajectory values are already at single-cell resolution (e.g. directly computed
  per cell by Palantir) — no pseudobulk-to-cell projection is needed
- The dataset has a cyclic topology (e.g. B cell receptor editing) that absorbing
  Markov chains cannot represent, so no branch probabilities exist to project
- The pseudobulk object lacks markovProbability/DPT outputs, leaving value_key columns
  undefined
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: dandelionR
  role: Provides projectPseudotimeToCell to transfer pseudobulk pseudotime and branch
    probabilities to individual cells in the SingleCellExperiment colData
  repo: https://github.com/tuonglab/dandelionR
- name: miloR
  role: Builds the pseudobulk neighbourhoods whose aggregated values are projected
    back to cells
- name: Palantir
  role: Python reference implementation whose per-cell pseudotime and fate probabilities
    serve as the comparison target for projected dandelionR values
  repo: https://github.com/dpeerlab/Palantir
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:f215e7cd064a0d1c1f51b5b253a5d15d0ff1e7cf3a9fb6568e260d31af13d993
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
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# pseudotime-projection-to-single-cells

## Summary

Projects pseudobulk-level pseudotime and absorbing-Markov-chain branch probabilities (e.g. CD4+T and CD8+T fates from dandelionR::markovProbability) back onto individual cells in the SingleCellExperiment's colData, enabling per-cell trajectory values comparable to Palantir outputs.

## When to use

Use when a miloR pseudobulk object carries pseudotime and branch probabilities computed in VDJ feature space, and per-cell values are needed for plotting, downstream analysis, or benchmarking against Palantir.

## When NOT to use

- Trajectory values are already at single-cell resolution (e.g. directly computed per cell by Palantir) — no pseudobulk-to-cell projection is needed
- The dataset has a cyclic topology (e.g. B cell receptor editing) that absorbing Markov chains cannot represent, so no branch probabilities exist to project
- The pseudobulk object lacks markovProbability/DPT outputs, leaving value_key columns undefined

## Inputs

- miloR pseudobulk object with markovProbability results (pseudotime and branch probabilities) in its metadata
- SingleCellExperiment (sce_vdj) containing individual cells to receive projected values
- value_key character vector naming the pseudobulk columns to project, e.g. c('pseudotime', 'CD8+T', 'CD4+T')

## Outputs

- SingleCellExperiment whose colData now contains per-cell pseudotime and branch probability columns (pseudotime, CD8+T, CD4+T)
- Per-cell values usable for concordance metrics (e.g. Lin's CCC, Kendall tau, MAE) against Palantir outputs, e.g. saved to benchmark2_concordance.csv

## How to apply

After running dandelionR::markovProbability on the milo pseudobulk object (terminal states set to the CD8+T and CD4+T branch tips, root cell defined, knn = 30), call dandelionR::projectPseudotimeToCell to transfer pseudobulk values to single cells by specifying value_key = c('pseudotime', 'CD8+T', 'CD4+T'); the function writes these columns into the SingleCellExperiment colData. Rationale: the absorbing Markov chain and diffusion pseudotime operate on the ~1516 pseudobulk neighbourhoods in the 160-gene VDJ feature space, so projection is required to attach the trajectory quantities to each of the ~17k cells. Projected values then serve as per-cell inputs for concordance metrics (e.g. Lin's CCC) against original Palantir-derived pseudotime and branch probabilities.

## Related tools

- **dandelionR** (Provides projectPseudotimeToCell to transfer pseudobulk pseudotime and branch probabilities to individual cells in the SingleCellExperiment colData) — https://github.com/tuonglab/dandelionR
- **miloR** (Builds the pseudobulk neighbourhoods whose aggregated values are projected back to cells)
- **Palantir** (Python reference implementation whose per-cell pseudotime and fate probabilities serve as the comparison target for projected dandelionR values) — https://github.com/dpeerlab/Palantir

## Examples

```
sce <- projectPseudotimeToCell(pb.milo, sce_vdj, value_key = c("pseudotime", "CD8+T", "CD4+T"))
```

## Evaluation signals

- colData of the returned SingleCellExperiment contains the requested value_key columns ('pseudotime', 'CD8+T', 'CD4+T') for every cell
- Projected per-cell pseudotime and CD4+ probabilities reach Lin's CCC ~0.85+ against original Palantir values (CD4+ CCC reported as 0.850), with Kendall rank correlation up to 0.925 for branch probabilities at pseudobulk level
- Per-cell values are consistent with their parent pseudobulk neighbourhood's markovProbability output (cells within a neighbourhood share the aggregated value)
- Projected CD8+T and CD4+T probabilities are bounded in [0, 1] and jointly interpretable as fate probabilities toward the two terminal branch tips

## Limitations

- Projection inherits pseudobulk granularity: values derive from ~1516 milo neighbourhoods in a 160-gene VDJ feature space, not from per-cell inference
- destiny-based diffusion maps restrict the underlying trajectory to bifurcation or linear topologies, limiting what can be meaningfully projected
- Fundamental discrepancies in diffusion map and KNN graph construction between R and Python packages hinder exact agreement with Palantir per-cell outputs

## Evidence

- [other] projectPseudotimeToCell with value_key = c("pseudotime", "CD8+T", "CD4+T"): [evidence span withheld — anchor status: not_found]
- [readme] custom absorbing markov chain pseudotime inference: "a custom implementation of absorbing markov chain for pseudotime inference, inspired based on the palantir"
- [methods] dandelionR projects results of the original software: "As an R implementation of Dandelion, it aims to reproduce the preprocessing, feature-space-building and result-projecting functions of the original software."
- [other] Kendall rank correlation increased to 0.925: [evidence span withheld — anchor status: not_found]
- [results] Lin's CCC for CD4+ probability was 0.850: "Lin's CCC for CD4 + probability (0.850) falls below the commonly accepted threshold of 0.9"
