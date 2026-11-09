---
name: single-cell-integration-benchmarking
description: Use when you have per-cell UMAP embeddings from multiple integration
  methods plus batch and cell-type metadata and need to objectively compare batch
  mixing versus cell-type conservation across covariates.
when_to_use_negative:
- When only one integration embedding exists and no method comparison is needed —
  LISI/kBET benchmarking across methods requires multiple embeddings to compare
- When the goal is de novo integration of datasets rather than evaluating existing
  integration embeddings; run the integration methods first (e.g., Harmony, Scanorama)
  before applying this benchmarking skill
- When cell-type annotations are absent — cLISI requires cell-type labels to score
  cell-type conservation
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SingCellaR
  role: R package that loads the integration embeddings and metadata and provides
    the runLISI and runKBET wrappers used to score each method
  repo: https://github.com/supatt-lab/SingCellaR
- name: R
  role: Language in which the SingCellaR wrappers and set.seed-based reproducibility
    control are executed
- name: kBET
  role: Batch-effect test computing acceptance rates via subsampling, invoked through
    runKBET with a fixed seed and fixed n.sample
- name: scanorama
  role: One of the five integration methods whose embedding is benchmarked (installed
    as a Python module via pip for runScanorama)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:f1d65775f215a83e8a28d30825b31c55d8df32d392d5559b8cb928ce0000e0de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - R
  - SingCellaR
  - kBET
  - scanorama
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# single-cell-integration-benchmarking

## Summary

Benchmark how well five single-cell data integration methods (Supervised Harmony, Harmony, Seurat RPCA, Scanorama, Limma) mix batches and donors while conserving AUCell-derived cell types on HSPC data, using iLISI/cLISI scores and kBET acceptance rates computed inside SingCellaR.

## When to use

Use when you have per-cell UMAP embeddings from multiple integration methods plus batch and cell-type metadata and need to objectively compare batch mixing versus cell-type conservation across covariates.

## When NOT to use

- When only one integration embedding exists and no method comparison is needed — LISI/kBET benchmarking across methods requires multiple embeddings to compare
- When the goal is de novo integration of datasets rather than evaluating existing integration embeddings; run the integration methods first (e.g., Harmony, Scanorama) before applying this benchmarking skill
- When cell-type annotations are absent — cLISI requires cell-type labels to score cell-type conservation

## Inputs

- Deposited UMAP embeddings from five integration methods (Supervised Harmony, Harmony, Seurat RPCA, Scanorama, Limma)
- Per-cell batch and donor metadata
- Per-cell AUCell-derived cell type annotations
- SingCellaR object loaded with the embeddings and metadata

## Outputs

- Per-cell iLISI and cLISI score distributions for each of the five integration embeddings
- Figure 3B-style iLISI/cLISI comparison plot aggregating all five methods
- kBET acceptance rates per method
- Figure 3C-style kBET acceptance-rate boxplot across the five methods
- Ranked integration-method assessment (e.g., Supervised Harmony highest kBET/iLISI and cLISI close to 1)

## How to apply

Load the deposited UMAP embeddings for each of the five integration methods (Supervised Harmony, Harmony, Seurat RPCA, Scanorama, Limma) together with per-cell batch and donor (and AUCell-derived cell type) annotations into a SingCellaR object. For each embedding, run the runLISI wrapper to compute per-cell iLISI (batch-mixing) and cLISI (cell-type conservation) scores. For each embedding, call set.seed beforehand to fix the random seed and use the n.sample parameter of runKBET to fix the number of subsampled cells, which stabilizes kBET acceptance rates (small run-to-run variation is otherwise expected). Aggregate the iLISI and cLISI distributions across the five methods to draw the Figure 3B comparison plot, and aggregate kBET acceptance rates into the Figure 3C boxplot. Judge the best method by higher kBET and iLISI scores together with a cLISI close to 1 — as Supervised Harmony achieved on this HSPC dataset.

## Related tools

- **SingCellaR** (R package that loads the integration embeddings and metadata and provides the runLISI and runKBET wrappers used to score each method) — https://github.com/supatt-lab/SingCellaR
- **R** (Language in which the SingCellaR wrappers and set.seed-based reproducibility control are executed)
- **kBET** (Batch-effect test computing acceptance rates via subsampling, invoked through runKBET with a fixed seed and fixed n.sample)
- **scanorama** (One of the five integration methods whose embedding is benchmarked (installed as a Python module via pip for runScanorama))

## Examples

```
set.seed(1); runKBET(Human_HSPC, n.sample = 50); runLISI(Human_HSPC)
```

## Evaluation signals

- Both plots (iLISI/cLISI comparison and kBET acceptance-rate boxplot) are produced for all five integration methods
- kBET acceptance rates are stable across repeated runs when the same seed (set.seed) and same n.sample subsample size are used; only small run-to-run variation is acceptable
- Best-performing method shows higher kBET and iLISI scores with cLISI close to 1 (as Supervised Harmony did on this HSPC dataset)
- Every embedding is scored with both metrics (iLISI for batch mixing, cLISI for cell-type conservation) rather than a single measure

## Limitations

- kBET scores vary slightly between runs due to different seed number settings and the number of subsampling cells; fix the seed with set.seed and the number of cells with the n.sample parameter
- The scanorama Python module must be installed (pip install scanorama) or runScanorama fails; the fa2 module and its R environmental path must also be set for runFA2_ForceDirectedGraph
- UMAP/FDG plot orientations may differ from the protocol due to software version and seed number differences

## Evidence

- [results] Supervised Harmony shows higher kBET and iLISI scores, and cLISI score is close to 1, indicating better data integration for this HSPC dataset: "Supervised Harmony shows higher kBET and iLISI scores, and cLISI score is close to 1, indicating better data integration for this HSPC dataset"
- [discussion] fix the number of cells for the kBET analysis using the n.sample parameter: [evidence span withheld — anchor status: not_found]
- [discussion] This issue can be solved by setting the seed number prior to running kBET using the 'set.seed' function.: [evidence span withheld — anchor status: not_found]
- [discussion] The user may encounter slightly different kBET scores from this protocol.: "The user may encounter slightly different kBET scores from this protocol. This is due to the different seed number settings and the number of subsampling cells"
- [readme] SingCellaR, an analysis package in R: "an integrative analysis tool for analysing single-cell RNA sequencing data"
