---
name: alpha-parameter-optimization-purity-modularity
description: Use when running cell-attribute aware community detection (EVA/ELVAR)
  on a cell-cell kNN graph and an alpha value weighting attribute similarity must
  be chosen from multiple candidate partitions.
when_to_use_negative:
- Cell types are already strongly clustered so attribute-aware weighting adds little;
  the article notes ELVAR's improvement over the Louvain-analog is more pronounced
  in studies with weaker clustering structure
- Single fixed-alpha exploratory clustering where a full 900-run alpha sweep is computationally
  prohibitive (ELVAR has increased runtimes compared to Milo or the Louvain-analog)
- The graph lacks a meaningful cell attribute (e.g. no condition/phenotype label),
  since alpha weighting on a degenerate attribute cannot improve community detection
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: ELVAR
  role: R-package providing Eva_partitions (per-alpha stochastic partitioning) and
    SelOptAlpha (alpha selection from purity/modularity/cluster-count profiles)
  repo: https://github.com/aet21/ELVAR
- name: EVA
  role: Extended Louvain algorithm whose alpha parameter balances state-space similarity
    vs cell-attribute similarity during community detection
- name: Seurat
  role: Builds the Seurat object, runs PCA (RunPCA) and constructs the kNN state-manifold
    graph (FindNeighbors, dims 1:8, k.param=20) that SelOptAlpha-based optimization
    operates on
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:ef0f0c775b3358dd22bcb90cb3a24f01c466752bcfafaceb71ef24106bab2cc9
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# alpha-parameter-optimization-purity-modularity

## Summary

Selects the optimal EVA alpha parameter (weight on cell-attribute similarity in the Extended Louvain objective) by running Eva_partitions over alpha = 0.1–0.9 with ~100 repeats each, then applying SelOptAlpha to pick the alpha whose partitions combine high purity and modularity with a cluster count that departs from the Louvain baseline.

## When to use

Use when running cell-attribute aware community detection (EVA/ELVAR) on a cell-cell kNN graph and an alpha value weighting attribute similarity must be chosen from multiple candidate partitions.

## When NOT to use

- Cell types are already strongly clustered so attribute-aware weighting adds little; the article notes ELVAR's improvement over the Louvain-analog is more pronounced in studies with weaker clustering structure
- Single fixed-alpha exploratory clustering where a full 900-run alpha sweep is computationally prohibitive (ELVAR has increased runtimes compared to Milo or the Louvain-analog)
- The graph lacks a meaningful cell attribute (e.g. no condition/phenotype label), since alpha weighting on a degenerate attribute cannot improve community detection

## Inputs

- Cell-cell kNN graph (igraph) built from Seurat FindNeighbors (dims 1:8, k.param=20) with a vertex attribute encoding the biological condition (e.g. age)
- Matrix of community counts per run and alpha (nC.m) and the Louvain reference cluster count (nc0)
- Matrices of purity (pur.m) and modularity (mod.m) per run and alpha, from ~100 Eva_partitions repeats at each of nine alpha values

## Outputs

- Selected optimal alpha value (single scalar from 0.1–0.9)
- SelOptAlpha result object with per-alpha purity, modularity, and cluster-count profiles
- Final EVA partition (eva.o) at the optimal alpha for downstream ProcessEVA/DoDA

## How to apply

First run Eva_partitions on the state-manifold graph (resolution=1, threshold=0.0000001, Vattr.name set to the condition attribute such as 'age') for each alpha in seq(0.1, 0.9, 0.1), repeating ~100 times per alpha because Louvain-style community detection is stochastic; this yields ~900 runs in total. For each run, record the number of communities (nC), purity (pur), and modularity (mod). Feed these matrices to SelOptAlpha together with the Louvain reference cluster count (nc0), selecting as optimal the alpha whose purity and modularity both fall within the 75th percentile of their maxima while the cluster count departs from the Louvain-analog, indicating attribute-driven communities. The rationale is that attribute-aware weighting should denoise the manifold and produce more robust cell states for downstream differential abundance testing, but only at an alpha large enough to use the attribute yet not so large that state-space similarity is ignored.

## Related tools

- **ELVAR** (R-package providing Eva_partitions (per-alpha stochastic partitioning) and SelOptAlpha (alpha selection from purity/modularity/cluster-count profiles)) — https://github.com/aet21/ELVAR
- **EVA** (Extended Louvain algorithm whose alpha parameter balances state-space similarity vs cell-attribute similarity during community detection)
- **Seurat** (Builds the Seurat object, runs PCA (RunPCA) and constructs the kNN state-manifold graph (FindNeighbors, dims 1:8, k.param=20) that SelOptAlpha-based optimization operates on)

## Examples

```
selAlpha.o <- SelOptAlpha(nC.m,nc0,pur.m,mod.m,alpha.v=seq(0.1,0.9,0.1));
```

## Evaluation signals

- The chosen alpha lies within seq(0.1, 0.9, 0.1) and its purity and modularity both fall in the 75th centile of the maximum across alphas
- The cluster count at the optimal alpha departs from the Louvain-analog reference count (nc0), evidencing attribute-driven communities
- Per-alpha variability across the ~100 repeats is quantified, showing selection is robust to Louvain stochasticity
- Downstream ProcessEVA/DoDA at the optimal alpha reproduces the expected DA result (e.g. the highly significant naive-to-mature Cd4+ T-cell shift in lung tissue via Wald and LRT statistics)

## Limitations

- Identifying the optimal alpha requires ~900 runs (100 per nine alpha values); on a 100-node server this is ~9 instances, and ELVAR has increased runtimes compared to Milo or the Louvain-analog
- The improvement of ELVAR over its Louvain-analog is less noticeable in studies with strong clustering structure
- Alpha selection depends on the quality of the clustering attribute: false positives in the sample/phenotype label can degrade the inferred communities

## Evidence

- [methods] SelOptAlpha call grounding: "selAlpha.o <- SelOptAlpha(nC.m,nc0,pur.m,mod.m,alpha.v=seq(0.1,0.9,0.1));"
- [methods] Eva_partitions parameters grounding: "eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha= 0.8, Vattr.name= "age");"
- [discussion] Computational cost of alpha sweep: "to identify an optimal a parameter value may require a total of 100 runs for each of nine a parameter values, so a total of 900 runs, which on a 100-node server would require 9 instances"
