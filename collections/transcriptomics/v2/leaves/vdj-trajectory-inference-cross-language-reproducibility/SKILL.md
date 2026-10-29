---
name: vdj-trajectory-inference-cross-language-reproducibility
description: Use when porting a Python-based scVDJ-seq trajectory pipeline (Dandelion/Palantir)
  to R (dandelionR) and you need to quantify cross-language output concordance via
  cell counts, Pearson/Kendall correlations, and Lin's CCC.
when_to_use_negative:
- Input data has a cyclic trajectory (e.g. B cell receptor editing) that cannot be
  represented by absorbing Markov chains, or a topology more complex than bifurcation/linear,
  since destiny-based diffusion map limits trajectory inference to bifurcation or
  linear topologies
- The dataset lacks paired BCR/TCR sequencing or has too few cells (e.g. only 972
  healthy B cells in Tonglin et al.), which is insufficient for miloR neighbourhood
  construction
- You only need to run a single-language workflow and have no cross-implementation
  outputs to compare — concordance benchmarking requires both R and Python results
  on the same input data
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2830
tools:
- name: dandelionR
  role: 'R implementation of the Dandelion trajectory analysis workflow: preprocessing
    (setupVdjPseudobulk), pseudobulking via miloR, VDJ feature space construction,
    diffusion map/DPT pseudotime, and absorbing Markov chain branch probabilities'
  repo: https://github.com/tuonglab/dandelionR
- name: Dandelion
  role: Original Python implementation whose per-cell pseudotime and CD4+ probability
    outputs serve as the reference for the Benchmark 1 concordance comparison
  repo: https://www.github.com/zktuong/dandelion
- name: scRepertoire
  role: R toolkit used for TCR contig processing (loadContigs, combineTCR) and combineExpression
    to interface AIRR data with the SingleCellExperiment object in the dandelionR
    workflow
  repo: https://github.com/ncborcherding/scRepertoire
- name: Palantir
  role: Python package that inspired the absorbing Markov chain pseudotime inference;
    its workflow underlies the original Dandelion reference outputs
  repo: https://github.com/dpeerlab/Palantir
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:1a52b856383f5d42230146baf68caadca859917f6521608ee06e965de0de6289
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tuonglab/dandelionR
  tools:
  - Dandelion
  - Palantir
  - R
  - dandelionR
  - scRepertoire
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# vdj-trajectory-inference-cross-language-reproducibility

## Summary

Reproduce and benchmark a VDJ feature-space trajectory analysis workflow (dandelionR in R) against its original Python implementation (Dandelion/Palantir) on the Suo et al. fetal immune T-cell dataset, quantifying concordance of per-cell pseudotime and branch probabilities.

## When to use

Use when porting a Python-based scVDJ-seq trajectory pipeline (Dandelion/Palantir) to R (dandelionR) and you need to quantify cross-language output concordance via cell counts, Pearson/Kendall correlations, and Lin's CCC.

## When NOT to use

- Input data has a cyclic trajectory (e.g. B cell receptor editing) that cannot be represented by absorbing Markov chains, or a topology more complex than bifurcation/linear, since destiny-based diffusion map limits trajectory inference to bifurcation or linear topologies
- The dataset lacks paired BCR/TCR sequencing or has too few cells (e.g. only 972 healthy B cells in Tonglin et al.), which is insufficient for miloR neighbourhood construction
- You only need to run a single-language workflow and have no cross-implementation outputs to compare — concordance benchmarking requires both R and Python results on the same input data

## Inputs

- Suo et al. fetal immune atlas single-cell VDJ data (65102 cells, GEX Lymphoid Cells from developmental.cellatlas.io/fetal-immune)
- AIRR-format contig list (demo_airr) and SingleCellExperiment object (demo_sce / sce_vdj)
- Python Dandelion (Palantir-based) per-cell pseudotime and CD4+ branch probability outputs on the same dataset

## Outputs

- Per-cell pseudotime and CD4+ branch probability values from both R and Python workflows
- Benchmark 1 concordance table (e.g. benchmark1_comparison.csv) with cell counts, shared-cell count, Pearson and Kendall correlations, Lin's CCC, and mean absolute error
- Discrepancy note on seed handling and R/Python diffusion map and KNN graph construction differences

## How to apply

Run the dandelionR full workflow on the Suo et al. fetal immune atlas T-cell data: setupVdjPseudobulk (mode_option = 'abT'), miloR buildGraph (k = 30, d = 20 on X_scvi) and makeNhoods (prop = 0.3) for pseudobulking, vdjPseudobulk to build the VDJ feature space (160 V/D/J genes x 1516 pseudobulks), destiny DiffusionMap (n_pcs = 50, n_eigs = 10) with DPT (w_width = 0.1), then markovProbability with projectPseudotimeToCell for per-cell pseudotime and CD4+ branch probability. Run the original Python Dandelion (Palantir-based) workflow on the same dataset to obtain its per-cell pseudotime and CD4+ probability. Compute Benchmark 1 concordance metrics: output cell counts (17,281 dandelionR vs 17,234 Dandelion, 17,208 shared), Pearson correlation of pseudotime and CD4+ probability, Kendall rank correlation (0.68), Lin's CCC for CD4+ probability (0.850), and mean absolute error. Document the seed-handling discrepancy arising from fundamental differences in diffusion map and KNN graph construction between the R and Python packages, then compile all metrics into a comparison table (e.g. benchmark1_comparison.csv).

## Related tools

- **dandelionR** (R implementation of the Dandelion trajectory analysis workflow: preprocessing (setupVdjPseudobulk), pseudobulking via miloR, VDJ feature space construction, diffusion map/DPT pseudotime, and absorbing Markov chain branch probabilities) — https://github.com/tuonglab/dandelionR
- **Dandelion** (Original Python implementation whose per-cell pseudotime and CD4+ probability outputs serve as the reference for the Benchmark 1 concordance comparison) — https://www.github.com/zktuong/dandelion
- **scRepertoire** (R toolkit used for TCR contig processing (loadContigs, combineTCR) and combineExpression to interface AIRR data with the SingleCellExperiment object in the dandelionR workflow) — https://github.com/ncborcherding/scRepertoire
- **Palantir** (Python package that inspired the absorbing Markov chain pseudotime inference; its workflow underlies the original Dandelion reference outputs) — https://github.com/dpeerlab/Palantir

## Examples

```
Rscript -e 'sce <- setupVdjPseudobulk(sce, mode_option="abT"); milo_object <- buildGraph(milo_object, k=30, d=20, reduced.dim="X_scvi"); dm <- DiffusionMap(as.matrix(feature_space), n_pcs=50, n_eigs=10)'
```

## Evaluation signals

- Output cell counts: dandelionR should yield 17,281 cells and original Dandelion 17,234 cells on the Suo et al. abT data, with 17,208 cells shared between the two implementations
- Kendall rank correlation between the two implementations' outputs should be approximately 0.68
- Lin's concordance correlation coefficient for CD4+ probability should be approximately 0.850 (notably below the commonly accepted 0.9 threshold)
- Preprocessing retains 17,308 of 65,102 cells with complete TCR information, and the VDJ feature space should contain 160 V/D/J genes and 1,516 pseudobulks
- The comparison table must include a documented seed-handling discrepancy note attributing differences to diffusion map and KNN graph construction differences between R and Python

## Limitations

- Fundamental discrepancies in diffusion map construction and KNN graph construction between R and Python packages hinder exact reimplementation, causing seed-handling differences and imperfect concordance
- Lin's CCC for CD4+ probability (0.850) falls below the commonly accepted threshold of 0.9
- The destiny-based diffusion map approach is only suitable for datasets with bifurcation or linear topology, limiting versatility
- Palantir >= 1.4.5 uses numpy.random.Generator (PCG64) instead of the legacy RandomState, so numerical outputs differ from previous versions unless a prior version is pinned

## Evidence

- [results] The dandelionR output includes 17,281 cells, while the original Dandelion output contains 17,234 cells, with 17,208 shared between the two.: "The dandelionR output includes 17,281 cells, while the original Dandelion output contains 17,234 cells, with 17,208 shared between the two."
- [results] Lin's CCC for CD4 + probability (0.850) falls below the commonly accepted threshold of 0.9: "Lin's CCC for CD4 + probability (0.850) falls below the commonly accepted threshold of 0.9"
- [methods] sce <- setupVdjPseudobulk(sce,
    mode_option = "abT",: "sce <- setupVdjPseudobulk(sce,
    mode_option = "abT","
- [methods] dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10): "dm <- DiffusionMap(as.matrix(feature_space), n_pcs = 50, n_eigs = 10)"
- [discussion] fundamental discrepancies in diffusion map construction and KNN graph construction between R and Python packages generally, which hinder our ability to fully implement the original Dandelion and: "fundamental discrepancies in diffusion map construction and KNN graph construction between R and Python packages generally, which hinder our ability to fully implement the original Dandelion and"
- [results] Out of 65102 cells in the data, 17308 cells were retained after filtering due to having complete TCR information: "Out of 65102 cells in the data, 17308 cells were retained after filtering due to having complete TCR information"
- [readme] It provides all the necessary tools to interface with [`scRepertoire`](https://github.com/ncborcherding/scRepertoire) and a custom implementation of absorbing markov chain for pseudotime inference: "It provides all the necessary tools to interface with [`scRepertoire`](https://github.com/ncborcherding/scRepertoire) and a custom implementation of absorbing markov chain for pseudotime inference"
