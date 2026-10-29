---
name: immune-repertoire-trajectory-analysis
description: Use when analyzing paired scRNA-seq + single-cell VDJ-seq data (e.g.
  Suo et al. fetal immune atlas T cells) to reconstruct T or B cell differentiation
  trajectories and assign per-cell pseudotime and branch fate probabilities.
when_to_use_negative:
- Data lacks paired single-cell VDJ (BCR/TCR) sequencing — the workflow requires complete
  TCR/BCR information for filtering and pseudobulking
- The trajectory topology is cyclic (e.g. B cell receptor editing), which cannot be
  represented by absorbing Markov chains
- The dataset has too few relevant cells (e.g. only ~972 healthy B cells in Tonglin
  et al.), likely insufficient for miloR neighbourhood construction
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2830
tools:
- name: dandelionR
  role: 'R implementation of the trajectory workflow: setupVdjPseudobulk, vdjPseudobulk,
    and projectPseudotimeToCell for per-cell projection'
  repo: https://github.com/tuonglab/dandelionR
- name: miloR
  role: Neighbourhood construction (buildGraph, makeNhoods, miloUmap) and pseudobulking
    of single cells into the VDJ feature space
- name: scRepertoire
  role: 'Preprocessing of AIRR/contig data: loadContigs, combineTCR, combineExpression'
  repo: https://github.com/ncborcherding/scRepertoire
- name: destiny
  role: Diffusion map (DiffusionMap, DPT) computation over the VDJ feature space for
    pseudotime and branch tip identification
- name: Dandelion
  role: Original Python implementation of the VDJ feature space and trajectory workflow
    that dandelionR replicates
  repo: https://github.com/zktuong/dandelion
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:99141baa27e44d25c0a1c1e94c58d431f6ce4eb7a09344ad5f7f52faab0c3a22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tuonglab/dandelionR
  tools:
  - Dandelion
  - R
  - dandelionR
  - destiny
  - miloR
  - scRepertoire
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2830
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# immune-repertoire-trajectory-analysis

## Summary

A single-cell immune repertoire (scVDJ-seq) trajectory analysis workflow that constructs a VDJ feature space, infers pseudotime and branching probabilities via diffusion maps and absorbing Markov chains, and projects pseudobulk-level trajectory results back to individual cells. Implemented in R via dandelionR, replicating the Python Dandelion workflow with miloR pseudobulking and scRepertoire interoperability.

## When to use

Use when analyzing paired scRNA-seq + single-cell VDJ-seq data (e.g. Suo et al. fetal immune atlas T cells) to reconstruct T or B cell differentiation trajectories and assign per-cell pseudotime and branch fate probabilities.

## When NOT to use

- Data lacks paired single-cell VDJ (BCR/TCR) sequencing — the workflow requires complete TCR/BCR information for filtering and pseudobulking
- The trajectory topology is cyclic (e.g. B cell receptor editing), which cannot be represented by absorbing Markov chains
- The dataset has too few relevant cells (e.g. only ~972 healthy B cells in Tonglin et al.), likely insufficient for miloR neighbourhood construction

## Inputs

- Paired scRNA-seq + scVDJ-seq data (e.g. SingleCellExperiment object with transcriptomics plus AIRR-format contig lists)
- AIRR-format or 10X contig files loaded via loadContigs
- Prebuilt miloR neighbourhood object with per-cell metadata, UMAP, and 'X_scvi' reduced dimension
- Pseudobulk milo object (pb.milo) with pseudotime and branch probability columns in colData

## Outputs

- SingleCellExperiment object with projected per-cell pseudotime and branch probability columns (e.g. 'pseudotime', 'CD8+T', 'CD4+T') in colData
- VDJ feature space built from V/D/J gene usage across pseudobulks (160 V/D/J genes x ~1516 pseudobulks)
- Diffusion map / DPT trajectory with pseudotime and branching probabilities at pseudobulk level
- projected_cell_trajectory.rds with ~17,281 retained cells from the Suo et al. dataset

## How to apply

Preprocess single-cell VDJ data with scRepertoire (loadContigs, combineTCR with removeNA=TRUE/removeMulti=FALSE/filterMulti=TRUE, combineExpression) and run dandelionR::setupVdjPseudobulk to filter non-productive/ambiguous VDJ chains. Build a miloR neighbourhood structure (buildGraph k=30 d=20 on 'X_scvi', makeNhoods prop=0.3, miloUmap n_neighbors=30) and form pseudobulks with vdjPseudobulk; with 160 V/D/J genes this yields ~1516 pseudobulks forming the VDJ feature space. Infer trajectories with destiny's DiffusionMap (n_pcs=50, n_eigs=10) and DPT (tips set to root and branch tips, w_width=0.1), then use absorbing Markov chain-based pseudotime inference inspired by Palantir. Finally project pseudobulk pseudotime and branch probabilities (e.g. 'pseudotime', 'CD8+T', 'CD4+T') back to single cells with dandelionR::projectPseudotimeToCell, weighting each projected value by the inverse pseudobulk size and removing cells not assigned to any neighbourhood.

## Related tools

- **dandelionR** (R implementation of the trajectory workflow: setupVdjPseudobulk, vdjPseudobulk, and projectPseudotimeToCell for per-cell projection) — https://github.com/tuonglab/dandelionR
- **miloR** (Neighbourhood construction (buildGraph, makeNhoods, miloUmap) and pseudobulking of single cells into the VDJ feature space)
- **scRepertoire** (Preprocessing of AIRR/contig data: loadContigs, combineTCR, combineExpression) — https://github.com/ncborcherding/scRepertoire
- **destiny** (Diffusion map (DiffusionMap, DPT) computation over the VDJ feature space for pseudotime and branch tip identification)
- **Dandelion** (Original Python implementation of the VDJ feature space and trajectory workflow that dandelionR replicates) — https://github.com/zktuong/dandelion

## Examples

```
sce <- projectPseudotimeToCell(milo_object, pb.milo, value_key = c('pseudotime', 'CD8+T', 'CD4+T'))
```

## Evaluation signals

- Output SingleCellExperiment contains projected value columns for every requested value_key entry (e.g. 'pseudotime', 'CD8+T', 'CD4+T') in colData
- Retained cell count after projectPseudotimeToCell is consistent with the paper's reported scale (~17,281 cells; 39 cells removed for not belonging to any neighbourhood)
- VDJ feature space dimensions match expectation: 160 V/D/J genes and 1516 pseudobulks for the Suo et al. abT data
- Projected values for cells in multiple neighbourhoods are inverse-pseudobulk-size weighted averages of the pseudobulk values
- Cross-check against original Dandelion Python output: 17,208 of ~17,281 cells shared, and Lin's CCC for CD4+ probability ~0.850

## Limitations

- destiny-based diffusion map limits trajectory inference to bifurcation or linear topologies, restricting versatility for more complex trajectories
- dandelionR has not yet been applied to B cell development data, which involves more complex (cyclic) processes that absorbing Markov chains cannot represent
- Fundamental discrepancies in diffusion map and KNN graph construction between R and Python packages prevent full reimplementation of original Dandelion results
- Cells not assigned to any neighbourhood (39 cells in the paper) are removed during projectPseudotimeToCell
- Filtering and network generation functionalities from original Dandelion are not yet implemented in dandelionR

## Evidence

- [abstract] we present dandelionR , an R implementation of Dandelion 's trajectory analysis workflow, bringing the VDJ feature space construction and trajectory analysis using diffusion maps and absorbing Markov: "we present dandelionR , an R implementation of Dandelion 's trajectory analysis workflow, bringing the VDJ feature space construction and trajectory analysis using diffusion maps and absorbing Markov"
- [methods] milo_object <- buildGraph(milo_object, k = 30, d = 20, reduced.dim = "X_scvi"): "milo_object <- buildGraph(milo_object, k = 30, d = 20, reduced.dim = "X_scvi")"
- [results] With 160 V/D/J genes and 1516 pseudobulks, the VDJ feature space captured features: "With 160 V/D/J genes and 1516 pseudobulks, the VDJ feature space captured features"
- [results] The dandelionR output includes 17,281 cells, while the original Dandelion output contains 17,234 cells, with 17,208 shared between the two.: "The dandelionR output includes 17,281 cells, while the original Dandelion output contains 17,234 cells, with 17,208 shared between the two."
- [discussion] destiny, is only suitable for a dataset with a bifurcation or linear topology, which limits the versatility of dandelionR’s trajectory inference function: "destiny, is only suitable for a dataset with a bifurcation or linear topology, which limits the versatility of dandelionR’s trajectory inference function"
