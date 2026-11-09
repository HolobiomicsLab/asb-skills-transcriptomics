---
name: chi-square-differential-abundance-testing
description: Use when cell communities (from EVA or Louvain clustering) must be tested
  for differential enrichment between two conditions (e.g. perturbed vs normal cells),
  and you need chi-square P-values plus ARI to quantify clustering recovery.
when_to_use_negative:
- 'When per-replicate/per-sample differential testing is required: the pooled condition-by-state
  chi-square ignores replicate structure, whereas the paper''s DoDA step explicitly
  supplies a replicate variable (varREP="mouse")'
- When the goal is DA-testing on real data without attribute information; the paper's
  advantage over the Louvain-analog was more pronounced only in settings with weaker
  clustering structure
- When the input lacks a cell attribute (condition/phenotype label) to attach to the
  graph vertices — EVA's purity-modularity trade-off cannot be defined without it
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ELVAR
  role: R package providing Eva_partitions (cell-attribute aware EVA clustering),
    SelOptAlpha, ProcessEVA and DoDA used to generate partitions and DA calls tested
    by chi-square
  repo: aet21/ELVAR
- name: R
  role: Implementation language; chisq.test and ARI computations are run in R
- name: Seurat
  role: Creates the Seurat object, computes variable features/PCA, and builds the
    kNN graph (FindNeighbors) feeding the state-manifold graph
- name: igraph
  role: Constructs the undirected cell-cell state-manifold graph (graph.adjacency)
    and holds the condition label as a vertex attribute
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:7d61a12b68925e78dc7ab5598a7603a2113dd70d9d44464edaf9bb2a0bf837f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/ELVAR
  tools:
  - ELVAR
  - EVA
  - Matrix
  - R
  - Seurat
  - dplyr
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# chi-square-differential-abundance-testing

## Summary

Tests whether inferred cell communities are differentially enriched between biological conditions by building condition-by-state contingency tables from the ELVAR/EVA partition of the single-cell state-manifold graph and running chi-square tests, complemented by adjusted Rand Index evaluation against simulated ground-truth labels.

## When to use

Use when cell communities (from EVA or Louvain clustering) must be tested for differential enrichment between two conditions (e.g. perturbed vs normal cells), and you need chi-square P-values plus ARI to quantify clustering recovery.

## When NOT to use

- When per-replicate/per-sample differential testing is required: the pooled condition-by-state chi-square ignores replicate structure, whereas the paper's DoDA step explicitly supplies a replicate variable (varREP="mouse")
- When the goal is DA-testing on real data without attribute information; the paper's advantage over the Louvain-analog was more pronounced only in settings with weaker clustering structure
- When the input lacks a cell attribute (condition/phenotype label) to attach to the graph vertices — EVA's purity-modularity trade-off cannot be defined without it

## Inputs

- Seurat object with condition labels in metadata (e.g. seu.o from Tabula Muris Senis-derived monocytes)
- Cell-cell kNN state-manifold igraph object with condition as a vertex attribute (gr.o)
- Ground-truth simulated cell-state/condition labels for ARI computation
- EVA partition object (eva.o) and processed significant-community object (sigcl.o) from ProcessEVA

## Outputs

- Condition-by-state contingency tables per inferred partition
- Chi-square test P-values per enriched community (differential-abundance calls)
- Adjusted Rand Index values comparing EVA vs Louvain partitions to ground truth
- Optimal alpha selection output (SelOptAlpha over alpha.v = seq(0.1, 0.9, 0.1))

## How to apply

Build the cell-cell k-nearest-neighbour state-manifold graph with igraph (undirected, self-loops removed) from the Seurat object (FindNeighbors, dims 1:8, k.param=20) and assign the condition label as a vertex attribute. Run Eva_partitions across the alpha purity-parameter grid (0.1 to 0.9 in steps of 0.1, resolution=1, threshold=1e-7) plus the Louvain baseline (alpha=0 behaviour), then build condition-by-state contingency tables from each partition. For each enriched community, run R's chisq.test on the contingency table to obtain differential-abundance P-values, and compute the adjusted Rand Index between the inferred partition and the ground-truth simulated labels (e.g. 200 TMS-derived monocytes, 100 perturbed vs 100 normal). Use SelOptAlpha to select the optimal alpha from the purity/modularity trade-off. Rationale: the alpha parameter forces EVA to partition cells not only by transcriptomic state-space similarity but also by attribute similarity, which in the paper yields higher ARI and more significant chi-square P-values than Louvain, though mainly at larger alpha values.

## Related tools

- **ELVAR** (R package providing Eva_partitions (cell-attribute aware EVA clustering), SelOptAlpha, ProcessEVA and DoDA used to generate partitions and DA calls tested by chi-square) — aet21/ELVAR
- **R** (Implementation language; chisq.test and ARI computations are run in R)
- **Seurat** (Creates the Seurat object, computes variable features/PCA, and builds the kNN graph (FindNeighbors) feeding the state-manifold graph)
- **igraph** (Constructs the undirected cell-cell state-manifold graph (graph.adjacency) and holds the condition label as a vertex attribute)

## Examples

```
sigcl.o <- ProcessEVA(eva.o,seu.o,attrName="age"); da.o <- DoDA(seu.o,sigcl.o,varDA="naive",varREP="mouse");
```

## Evaluation signals

- EVA partition at the optimal alpha must yield a higher adjusted Rand Index against the ground-truth simulated labels than the Louvain (alpha=0) baseline
- Chi-square P-values for enriched communities from EVA must be more significant than those from Louvain, consistent with Supplementary Fig. S1
- Contingency tables must have dimensions condition x inferred state (e.g. 2 conditions x number of communities) with marginals summing to 200 simulated monocytes (100 perturbed, 100 normal)
- SelOptAlpha must be run over the full alpha.v = seq(0.1, 0.9, 0.1) grid at resolution=1 and threshold=1e-7 before DA calls are made
- Expect the EVA-over-Louvain improvement to appear only at larger alpha values, not across the whole grid

## Limitations

- Improvement of EVA over Louvain appeared only at larger purity parameter (alpha) values in the simulated monocyte benchmark
- Biological inter-subject variability and technical batch effects can confound naïve DA-analyses, though ELVAR largely circumvents the need for prior batch correction
- ELVAR is sensitive to false positives in the clustering attribute (sample/phenotype label), which can degrade community quality
- No benchmark against miloDE was performed; the authors flag this as an open question, and runtimes are higher than Milo or the Louvain-analog

## Evidence

- [readme] ELVAR is an R-package for differential abundance (DA) testing of cell-types in single-cell RNA-Seq data.: "ELVAR is an R-package for differential abundance (DA) testing of cell-types in single-cell RNA-Seq data."
- [methods] DoDA call with condition and replicate variables: "da.o <- DoDA(seu.o,sigcl.o,varDA="naive",varREP="mouse");"
- [methods] Eva_partitions call with alpha purity parameter, resolution and threshold: "eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha= 0.8, Vattr.name= "age");"
- [intro] Confounding motivation for careful DA testing: "biological inter-subject variability, as well as technical batch effects, can potentially confound naïve DA-analyses"
- [discussion] Sensitivity advantage over competing methods: "ELVAR displayed higher sensitivity than Milo and DA-seq to detect biologically important and plausible DA-shifts"
