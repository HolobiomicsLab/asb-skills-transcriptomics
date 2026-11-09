---
name: lisi-mixing-metric-computation
description: Use when per-cell UMAP embeddings from one or more integration methods
  (e.g. Supervised Harmony, Harmony, Seurat RPCA, Scanorama, Limma) must be benchmarked
  for batch/donor mixing versus AUCell-derived cell-type conservation.
when_to_use_negative:
- When the inputs are raw count matrices or gene expression tables rather than integrated
  UMAP embeddings plus batch/cell-type metadata — run integration and annotation first.
- When the goal is to measure clustering quality of a single method without batch
  covariates (batch and donor annotations are required for iLISI/kBET mixing assessment).
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SingCellaR
  role: R package hosting the runLISI and runKBET wrappers used to compute iLISI/cLISI
    and kBET acceptance rates on the loaded integration embeddings
  repo: https://github.com/supatt-lab/SingCellaR
- name: kBET
  role: Underlying batch-effects test whose acceptance rates are computed by runKBET
    with a fixed seed and fixed n.sample subsample size
- name: R
  role: Runtime environment in which SingCellaR, runLISI, and runKBET are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:0791263104376338c1edaaf4b0777ca2d54455c1570a46df580657cdfb32a946
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - R
  - SingCellaR
  - kBET
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lisi-mixing-metric-computation

## Summary

Quantify batch mixing and cell-type conservation after single-cell data integration by computing per-cell iLISI/cLISI scores and kBET acceptance rates on each method's UMAP embeddings, as done for five integration methods on HSPC data within SingCellaR.

## When to use

Use when per-cell UMAP embeddings from one or more integration methods (e.g. Supervised Harmony, Harmony, Seurat RPCA, Scanorama, Limma) must be benchmarked for batch/donor mixing versus AUCell-derived cell-type conservation.

## When NOT to use

- When the inputs are raw count matrices or gene expression tables rather than integrated UMAP embeddings plus batch/cell-type metadata — run integration and annotation first.
- When the goal is to measure clustering quality of a single method without batch covariates (batch and donor annotations are required for iLISI/kBET mixing assessment).

## Inputs

- UMAP embeddings (one per integration method: Supervised Harmony, Harmony, Seurat RPCA, Scanorama, Limma)
- Per-cell batch/donor annotation metadata
- Per-cell AUCell-derived cell-type annotations
- SingCellaR object (Human HSPC dataset)

## Outputs

- Per-cell iLISI and cLISI score distributions per integration method
- Per-cell kBET acceptance-rate distributions per integration method
- Figure 3B iLISI/cLISI comparison plot
- Figure 3C kBET acceptance-rate boxplot

## How to apply

Load the deposited UMAP embeddings for each integration method together with per-cell batch (donor) and AUCell-derived cell-type metadata into the SingCellaR object. For each integration embedding, run the runLISI wrapper to obtain per-cell iLISI (batch-mixing) and cLISI (cell-type conservation) scores, aggregating distributions across methods for the Figure 3B comparison plot. For kBET, call set.seed beforehand and set the n.sample parameter of runKBET to fix the number of subsampled cells, then aggregate acceptance rates across the five methods into the Figure 3C boxplot. Supervised Harmony was the best performer here (higher kBET and iLISI, cLISI close to 1), which is the expected qualitative outcome on comparable HSPC datasets.

## Related tools

- **SingCellaR** (R package hosting the runLISI and runKBET wrappers used to compute iLISI/cLISI and kBET acceptance rates on the loaded integration embeddings) — https://github.com/supatt-lab/SingCellaR
- **kBET** (Underlying batch-effects test whose acceptance rates are computed by runKBET with a fixed seed and fixed n.sample subsample size)
- **R** (Runtime environment in which SingCellaR, runLISI, and runKBET are executed)

## Examples

```
set.seed(42); runKBET(Human_HSPC, n.sample = 100)  # run after set.seed; likewise runLISI(Human_HSPC) per integration embedding
```

## Evaluation signals

- Both the iLISI/cLISI comparison plot (Figure 3B) and the kBET acceptance-rate boxplot (Figure 3C) are produced for all five integration methods.
- cLISI scores for the best method (Supervised Harmony) are close to 1, with concurrently higher kBET and iLISI scores than the other four methods.
- Repeated runs with the same fixed seed and the same n.sample subsample size yield nearly identical kBET scores (only small run-to-run variation is tolerated).
- All five methods (Supervised Harmony, Harmony, Seurat RPCA, Scanorama, Limma) have non-empty iLISI/cLISI and kBET distributions — no method silently dropped out.
- iLISI and kBET (batch-mixing metrics) increase while cLISI stays near 1, i.e. mixing does not come at the cost of cell-type conservation.

## Limitations

- kBET scores vary slightly between runs because of different seed-number settings and the number of subsampled cells; this can be minimized by calling set.seed and fixing the n.sample parameter before running kBET.
- Small run-to-run variation in kBET is expected even with a fixed seed and fixed subsample size.
- UMAP/plot orientations can differ from the protocol because of software version and seed-number differences.

## Evidence

- [discussion] This issue can be solved by setting the seed number prior to running kBET using the 'set.seed' function.: [evidence span withheld — anchor status: not_found]
- [discussion] fix the number of cells for the kBET analysis using the n.sample parameter described in the 'runKBET' function: [evidence span withheld — anchor status: not_found]
- [discussion] The user may encounter slightly different kBET scores from this protocol. This is due to the different seed number settings and the number of subsampling cells: "The user may encounter slightly different kBET scores from this protocol. This is due to the different seed number settings and the number of subsampling cells"
- [readme] an integrative analysis tool for analysing single-cell RNA sequencing data: "an integrative analysis tool for analysing single-cell RNA sequencing data"
