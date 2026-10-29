---
name: pseudotime-concordance-metric-computation
description: Use when you have per-cell pseudotime and fate-probability values from
  two trajectory inference implementations run on the same single-cell VDJ dataset
  and need quantitative concordance metrics to benchmark them.
when_to_use_negative:
- Only one trajectory implementation's output is available, so no cross-implementation
  comparison can be computed
- The dataset has a cyclic trajectory (e.g. B cell receptor editing) that cannot be
  represented by absorbing Markov chains, making branch-probability comparison ill-defined
- The two implementations were run on differently filtered cell sets, so shared-cell
  subsetting is impossible without cell identity reconciliation
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: dandelionR
  role: R implementation producing per-cell pseudotime and CD4+ branch probabilities
    via destiny diffusion maps, DPT, and absorbing Markov chains
  repo: github.com/tuonglab/dandelionR
- name: Dandelion
  role: Original Python (Palantir-based) implementation producing the reference per-cell
    pseudotime and branch probabilities for comparison
  repo: github.com/zktuong/dandelion
- name: scRepertoire
  role: Interfaces AIRR contig data (combineTCR, combineExpression) into the SingleCellExperiment
    object prior to pseudobulking and trajectory inference
  repo: github.com/ncborcherding/scRepertoire
- name: Palantir
  role: Algorithm underlying the original Dandelion pseudotime and fate-probability
    computation in Python
  repo: https://github.com/dpeerlab/Palantir
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:30b9096c45c45d3ff57be89ec9bfceaa66f1dfc979d314e21567a13a2b190f98
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Dandelion
  - Palantir
  - R
  - dandelionR
  - scRepertoire
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# pseudotime-concordance-metric-computation

## Summary

Benchmarking skill that quantifies agreement between per-cell pseudotime and branch-probability outputs of two trajectory-analysis implementations (R dandelionR vs Python Dandelion) using cell counts, shared-cell overlap, Pearson/Kendall correlations, Lin's CCC, and mean absolute error.

## When to use

Use when you have per-cell pseudotime and fate-probability values from two trajectory inference implementations run on the same single-cell VDJ dataset and need quantitative concordance metrics to benchmark them.

## When NOT to use

- Only one trajectory implementation's output is available, so no cross-implementation comparison can be computed
- The dataset has a cyclic trajectory (e.g. B cell receptor editing) that cannot be represented by absorbing Markov chains, making branch-probability comparison ill-defined
- The two implementations were run on differently filtered cell sets, so shared-cell subsetting is impossible without cell identity reconciliation

## Inputs

- Per-cell pseudotime values from the dandelionR workflow (destiny DiffusionMap + DPT, w_width=0.1)
- Per-cell CD4+ branch probability values from dandelionR markovProbability/projectPseudotimeToCell
- Per-cell pseudotime and CD4+ branch probability values from the Python Dandelion (Palantir-based) workflow
- Suo et al. fetal immune atlas T-cell SingleCellExperiment object with TCR information (17,308 cells retained after filtering from 65,102)

## Outputs

- benchmark1_comparison.csv with output cell counts, shared-cell count, Pearson correlation (pseudotime and CD4+ probability), Kendall rank correlation, Lin's CCC, and mean absolute error
- A documented note on seed-handling discrepancies due to diffusion map and KNN graph construction differences

## How to apply

Run both implementations on the same input (the Suo et al. fetal immune T-cell dataset: dandelionR with setupVdjPseudobulk mode_option='abT', miloR buildGraph k=30/d=20 on X_scvi, makeNhoods prop=0.3, vdjPseudobulk, destiny DiffusionMap n_pcs=50/n_eigs=10 with DPT w_width=0.1, then markovProbability and projectPseudotimeToCell; and the Python Dandelion Palantir-based workflow). First compare output cell counts and their intersection (17,281 vs 17,234 cells with 17,208 shared in Benchmark 1), restricting correlation metrics to the shared-cell subset. Then compute Pearson correlation of pseudotime and CD4+ branch probability, Kendall rank correlation (expected ~0.68 in Benchmark 1), Lin's concordance correlation coefficient for CD4+ probability (expected ~0.850, below the commonly accepted 0.9 threshold), and mean absolute error. Finally, document residual discrepancies as arising from fundamental differences in diffusion map and KNN graph construction between the R and Python packages rather than as implementation errors.

## Related tools

- **dandelionR** (R implementation producing per-cell pseudotime and CD4+ branch probabilities via destiny diffusion maps, DPT, and absorbing Markov chains) — github.com/tuonglab/dandelionR
- **Dandelion** (Original Python (Palantir-based) implementation producing the reference per-cell pseudotime and branch probabilities for comparison) — github.com/zktuong/dandelion
- **scRepertoire** (Interfaces AIRR contig data (combineTCR, combineExpression) into the SingleCellExperiment object prior to pseudobulking and trajectory inference) — github.com/ncborcherding/scRepertoire
- **Palantir** (Algorithm underlying the original Dandelion pseudotime and fate-probability computation in Python) — https://github.com/dpeerlab/Palantir

## Examples

```
cor.test(dandelionR_out$pseudotime, dandelion_out$pseudotime, method = "kendall", subset = shared_cells)
```

## Evaluation signals

- Cell-count invariant: dandelionR output 17,281 cells and Dandelion 17,234 cells with 17,208 shared — deviations beyond ~a few dozen cells signal a filtering or seed-handling mismatch
- Kendall rank correlation of pseudotime between implementations should be near 0.68
- Lin's CCC for CD4+ probability should be near 0.850; values far below indicate divergence beyond the documented R/Python diffusion map and KNN graph differences
- Pearson correlation of pseudotime and of CD4+ probability must be computed on the 17,208 shared cells only
- The metrics table (benchmark1_comparison.csv) must include a discrepancy note attributing residual differences to diffusion map and KNN graph construction, not to pipeline bugs

## Limitations

- Lin's CCC for CD4+ probability (0.850) falls below the commonly accepted threshold of 0.9, so outputs are concordant but not identical
- Fundamental discrepancies in diffusion map construction and KNN graph construction between R and Python packages prevent full reimplementation and cause seed-handling differences
- The destiny-based diffusion map limits trajectory inference to bifurcation or linear topologies, restricting the scope of concordance testing
- It remains unclear whether brute-force or KD-tree KNN methods more accurately capture the underlying biological structure, affecting graph-based pseudobulk comparability

## Evidence

- [results] The dandelionR output includes 17,281 cells, while the original Dandelion output contains 17,234 cells, with 17,208 shared between the two.: "The dandelionR output includes 17,281 cells, while the original Dandelion output contains 17,234 cells, with 17,208 shared between the two."
- [results] Lin's CCC for CD4+ probability was 0.850, below the commonly accepted threshold of 0.9: "Lin's CCC for CD4 + probability (0.850) falls below the commonly accepted threshold of 0.9"
- [discussion] fundamental discrepancies in diffusion map construction and KNN graph construction between R and Python packages generally, which hinder our ability to fully implement the original Dandelion and: "fundamental discrepancies in diffusion map construction and KNN graph construction between R and Python packages generally, which hinder our ability to fully implement the original Dandelion and"
- [methods] dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10): "dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10)"
- [readme] dandelionR is an R package for performing single-cell immune repertoire trajectory analysis, based on the original python implementation in dandelion: "`dandelionR` is an R package for performing single-cell immune repertoire trajectory analysis, based on the original python implementation in [`dandelion`]"
