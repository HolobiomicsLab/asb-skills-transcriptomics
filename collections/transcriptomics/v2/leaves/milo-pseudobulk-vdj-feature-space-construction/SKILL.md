---
name: milo-pseudobulk-vdj-feature-space-construction
description: Use when you have preprocessed scVDJ-seq data (e.g. SingleCellExperiment
  with X_scvi embeddings and complete TCR information) and need to build a VDJ feature
  space for trajectory analysis, as in the dandelionR abT workflow.
when_to_use_negative:
- The dataset has too few relevant cells for miloR neighbourhood construction (e.g.
  the Tonglin et al. dataset with only 972 healthy B cells was judged likely insufficient)
- The trajectory of interest is cyclic (e.g. B cell receptor editing), which cannot
  be represented by the downstream absorbing Markov chain framework
- Input is raw AIRR/contig files that have not yet been preprocessed and integrated
  into a SingleCellExperiment with an X_scvi embedding
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2830
tools:
- name: dandelionR
  role: Provides setupVdjPseudobulk and vdjPseudobulk to filter VDJ chains and build
    the VDJ feature space from milo neighbourhoods
  repo: https://github.com/tuonglab/dandelionR
- name: miloR
  role: buildGraph (k = 30, d = 20 on X_scvi) and makeNhoods (prop = 0.3) define the
    neighbourhoods used for pseudobulking
- name: scRepertoire
  role: combineTCR and combineExpression prepare and attach TCR contig information
    to the SingleCellExperiment prior to pseudobulking
  repo: https://github.com/ncborcherding/scRepertoire
- name: Dandelion
  role: Original Python implementation whose VDJ feature space strategy this R workflow
    replicates
  repo: https://github.com/zktuong/dandelion
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:9c29a2afc1a39a988d8507a2b60721680f7ca2ce66312d7f608717dc6a70a08f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tuonglab/dandelionR
  tools:
  - Dandelion
  - R
  - dandelionR
  - miloR
  - scRepertoire
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# milo-pseudobulk-vdj-feature-space-construction

## Summary

Constructs a VDJ feature space by pseudobulking single-cell VDJ data with miloR neighbourhoods: buildGraph (k = 30, d = 20 on X_scvi), makeNhoods (prop = 0.3), and vdjPseudobulk summarize V/D/J gene usage per neighbourhood, replicating Dandelion's VDJ-feature-space strategy in R.

## When to use

Use when you have preprocessed scVDJ-seq data (e.g. SingleCellExperiment with X_scvi embeddings and complete TCR information) and need to build a VDJ feature space for trajectory analysis, as in the dandelionR abT workflow.

## When NOT to use

- The dataset has too few relevant cells for miloR neighbourhood construction (e.g. the Tonglin et al. dataset with only 972 healthy B cells was judged likely insufficient)
- The trajectory of interest is cyclic (e.g. B cell receptor editing), which cannot be represented by the downstream absorbing Markov chain framework
- Input is raw AIRR/contig files that have not yet been preprocessed and integrated into a SingleCellExperiment with an X_scvi embedding

## Inputs

- SingleCellExperiment object with X_scvi reduced dimension and TCR contig information (e.g. from Suo et al. fetal immune atlas, 65102 cells of which 17308 retained after filtering)
- Combined TCR contig list from scRepertoire combineTCR, integrated via combineExpression

## Outputs

- VDJ feature space matrix of 160 V/D/J genes x 1516 pseudobulks (milo neighbourhoods)
- miloR object with neighbourhood assignments for downstream pseudotime projection (markovProbability / projectPseudotimeToCell)

## How to apply

First run setupVdjPseudobulk (mode_option = 'abT') to filter non-productive and ambiguous VDJ chains and retain cells with complete TCR information. Then create a miloR object and run buildGraph with k = 30, d = 20 on the 'X_scvi' reduced dimension, followed by makeNhoods with d = 20 and prop = 0.3 to define pseudobulk neighbourhoods. Run vdjPseudobulk with mode_option = 'abT' to aggregate V/D/J gene usage within each neighbourhood, yielding the VDJ feature space matrix (pseudobulks x V/D/J genes) that serves as input to downstream diffusion-map trajectory analysis. The rationale is that Dandelion's VDJ feature space strategy improved prediction accuracy of trajectories from double-positive T cells to CD4/CD8 T cells; on the Suo et al. data the resulting feature space contained 160 V/D/J genes and 1516 pseudobulks.

## Related tools

- **dandelionR** (Provides setupVdjPseudobulk and vdjPseudobulk to filter VDJ chains and build the VDJ feature space from milo neighbourhoods) — https://github.com/tuonglab/dandelionR
- **miloR** (buildGraph (k = 30, d = 20 on X_scvi) and makeNhoods (prop = 0.3) define the neighbourhoods used for pseudobulking)
- **scRepertoire** (combineTCR and combineExpression prepare and attach TCR contig information to the SingleCellExperiment prior to pseudobulking) — https://github.com/ncborcherding/scRepertoire
- **Dandelion** (Original Python implementation whose VDJ feature space strategy this R workflow replicates) — https://github.com/zktuong/dandelion

## Examples

```
milo_object <- buildGraph(milo_object, k = 30, d = 20, reduced.dim = "X_scvi"); milo_object <- makeNhoods(milo_object, reduced_dims = "X_scvi", d = 20, prop = 0.3); pb.milo <- vdjPseudobulk(milo_object, mode_option = "abT")
```

## Evaluation signals

- On the Suo et al. abT data, the VDJ feature space should contain 160 V/D/J genes and 1516 pseudobulks
- Cell filtering via setupVdjPseudobulk on the Suo et al. data (65102 input cells) should retain approximately 17308 cells with complete TCR information
- The full dandelionR workflow downstream of this step should yield 17281 cells, comparable to the original Dandelion output of 17234 cells with 17208 shared
- All neighbourhood graph parameters must match: buildGraph k = 30, d = 20 on reduced.dim 'X_scvi'; makeNhoods prop = 0.3, d = 20
- milo object must contain valid neighbourhood assignments before vdjPseudobulk is called; check that reduced.dim 'X_scvi' exists in the input object

## Limitations

- Fundamental discrepancies in diffusion map construction and KNN graph construction between R and Python packages hinder exact reproduction of the original Dandelion output, causing seed-handling discrepancies
- It remains unclear whether brute-force or KD-tree KNN methods better capture the underlying biological structure
- Pseudobulk-based VDJ feature space construction and the downstream destiny diffusion map limit trajectory inference to bifurcation or linear topologies

## Evidence

- [methods] miloR buildGraph k=30 d=20: "milo_object <- buildGraph(milo_object, k = 30, d = 20, reduced.dim = "X_scvi")"
- [methods] makeNhoods prop=0.3: "milo_object <- makeNhoods(milo_object,
    reduced_dims = "X_scvi", d = 20,
    prop = 0.3
)"
- [results] 160 V/D/J genes, 1516 pseudobulks: "With 160 V/D/J genes and 1516 pseudobulks, the VDJ feature space captured features"
- [results] 17308 cells retained after filtering: "Out of 65102 cells in the data, 17308 cells were retained after filtering due to having complete TCR information"
- [methods] Pseudobulking via miloR: "Pseudobulking can be achieved through the miloR (v2.0.0) package [7]."
- [intro] Dandelion VDJ feature space improved trajectory prediction: "which improved the prediction accuracy of trajectories from double-positive T cells to CD4/CD8 T cells, demonstrating significant potential for future applications"
