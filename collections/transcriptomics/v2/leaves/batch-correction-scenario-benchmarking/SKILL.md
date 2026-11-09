---
name: batch-correction-scenario-benchmarking
description: Use when a cell-attribute-aware DA-testing workflow (e.g. ELVAR/EVA clustering)
  must be stress-tested for robustness to whether batch correction is applied beforehand,
  and to quantify how correction shifts significance of DA-test results.
when_to_use_negative:
- When the question is whether batch correction itself changes DA significance for
  a non-attribute-aware method (e.g. plain Louvain or Milo) — the article's claim
  that correction is dispensable is specific to cell-attribute aware clustering
- When cell-type labels or the clustering attribute contain suspected false positives
  and the goal is annotation-noise robustness rather than batch-correction robustness
  (that is a separate false-positive-flipping scenario)
- When the study has very strong clustering structure where ELVAR's advantage over
  Louvain and its robustness profile are less pronounced
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: ELVAR
  role: R-package implementing EVA cell-attribute aware clustering and DA testing
    (Eva_partitions, SelOptAlpha, ProcessEVA, DoDA) re-run per batch-correction variant
  repo: https://github.com/aet21/ELVAR
- name: Seurat
  role: Builds Seurat objects, runs FindVariableFeatures (vst)/ScaleData/RunPCA/FindNeighbors
    (dims 1:8, k.param=20) and provides integration-corrected embeddings as one preprocessing
    variant
- name: MASS
  role: Backing negative binomial regression used by DoDA with Wald and LRT statistics
- name: igraph
  role: Builds the undirected cell-cell graph (graph.adjacency) with condition vertex
    attributes on which Eva_partitions runs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:a634396b7cd6806121a7e8e5558e8dfea4a330650b3a338e7eeea31ccb4cd7e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/ELVAR
  tools:
  - ELVAR
  - EVA
  - MASS
  - Matrix
  - R
  - Seurat
  - dplyr
  - igraph
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# batch-correction-scenario-benchmarking

## Summary

A robustness benchmarking procedure that re-runs the ELVAR differential abundance pipeline on the same cell-cell kNN graph under multiple preprocessing variants (no batch correction, Harmony-corrected embeddings, Seurat integration-corrected embeddings) and compares DA-detection sensitivity and z-statistic distributions, since attribute-aware clustering may obviate batch correction.

## When to use

Use when a cell-attribute-aware DA-testing workflow (e.g. ELVAR/EVA clustering) must be stress-tested for robustness to whether batch correction is applied beforehand, and to quantify how correction shifts significance of DA-test results.

## When NOT to use

- When the question is whether batch correction itself changes DA significance for a non-attribute-aware method (e.g. plain Louvain or Milo) — the article's claim that correction is dispensable is specific to cell-attribute aware clustering
- When cell-type labels or the clustering attribute contain suspected false positives and the goal is annotation-noise robustness rather than batch-correction robustness (that is a separate false-positive-flipping scenario)
- When the study has very strong clustering structure where ELVAR's advantage over Louvain and its robustness profile are less pronounced

## Inputs

- scRNA-Seq count matrices for each study (e.g. Seurat objects) with per-cell condition/phenotype metadata
- Harmony-corrected embeddings for each dataset
- Seurat integration-corrected embeddings for each dataset
- Sample-replicate labels and cell-type annotations for the DoDA regression

## Outputs

- Per-scenario DA-test results: Wald and LRT statistics from DoDA negative binomial regression
- Sensitivity aggregates (fraction of significant DA detections) per batch-correction scenario
- z-statistic distributions across replicates per scenario, reproducible as Fig. 6 / Supplementary Fig. S8 patterns

## How to apply

For each dataset build a Seurat object with FindVariableFeatures (vst), ScaleData and RunPCA, then construct the cell-cell kNN graph via FindNeighbors (dims 1:8, k.param=20) and prepare three preprocessing variants: no batch correction, Harmony-corrected embeddings, and Seurat integration-corrected embeddings. For each variant run Eva_partitions on the state-manifold (resolution=1, threshold=0.0000001, Vattr.name set to the condition attribute) over alpha 0.1-0.9 with SelOptAlpha (purity and modularity within the 75th centile of max), using ~100 repeats per setting for robustness. Select condition-enriched communities with ProcessEVA (binomial test with Bonferroni threshold) and run DoDA negative binomial regression with the cell-state variable and sample-replicate variable, recording Wald and LRT statistics. Aggregate sensitivity (fraction of significant DA detections) and z-statistic distributions across replicates for every batch-correction scenario; batch-corrected runs should rarely exceed the sensitivity of the no-correction run, supporting ELVAR's claim that attribute-aware clustering can denoise data and avoid batch correction.

## Related tools

- **ELVAR** (R-package implementing EVA cell-attribute aware clustering and DA testing (Eva_partitions, SelOptAlpha, ProcessEVA, DoDA) re-run per batch-correction variant) — https://github.com/aet21/ELVAR
- **Seurat** (Builds Seurat objects, runs FindVariableFeatures (vst)/ScaleData/RunPCA/FindNeighbors (dims 1:8, k.param=20) and provides integration-corrected embeddings as one preprocessing variant)
- **MASS** (Backing negative binomial regression used by DoDA with Wald and LRT statistics)
- **igraph** (Builds the undirected cell-cell graph (graph.adjacency) with condition vertex attributes on which Eva_partitions runs)

## Examples

```
devtools::install_github("aet21/ELVAR"); eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha=0.8, Vattr.name="age")
```

## Evaluation signals

- Batch-corrected runs (Harmony or Seurat integration) should rarely exceed the sensitivity of the no-correction run
- Degree of statistical significance (z-statistics from DoDA) is generally higher without prior batch correction, though this is study-specific
- The no-correction run must reproduce the tutorial's highly significant naive-to-mature Cd4+ T-cell DA result (Wald and LRT statistics from DoDA)
- With ~100 repeats per setting, sensitivity and z-statistic distributions across replicates should be stable enough to reproduce Fig. 6 patterns
- SelOptAlpha-selected alpha must satisfy purity and modularity within the 75th centile of max with cluster count departing from Louvain, consistently across correction variants

## Limitations

- The advantage of skipping batch correction is study-specific; the degree of significance without correction varies across the four datasets tested
- ELVAR is more sensitive to false positives in the clustering attribute (sample/phenotype label) than in cell-type annotations, so label noise can confound the comparison
- ELVAR has increased computational complexity and runtimes compared to Milo and the Louvain-analog, deemed not practically prohibitive but multiplied further by per-scenario repeats (e.g. 900 runs for nine alpha values)
- No benchmark comparison of ELVAR against miloDE was performed; the authors flag this as an open question

## Evidence

- [discussion] Batch correction prior to ELVAR rarely improved sensitivity: [evidence span withheld — anchor status: not_found]
- [abstract] ELVAR avoids batch correction: "leveraging cell attribute information when inferring cell communities can denoise single-cell data, avoid the need for batch correction and help retrieve more robust cell states"
- [methods] Eva_partitions call: "eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha= 0.8, Vattr.name= "age");"
- [methods] SelOptAlpha call: "selAlpha.o <- SelOptAlpha(nC.m,nc0,pur.m,mod.m,alpha.v=seq(0.1,0.9,0.1));"
- [readme] ELVAR installation: "devtools::install_github("aet21/ELVAR")"
- [methods] Graph construction: "seu.o <- FindNeighbors(seu.o, dims = 1:8, k.param=20,verbose=FALSE);"
