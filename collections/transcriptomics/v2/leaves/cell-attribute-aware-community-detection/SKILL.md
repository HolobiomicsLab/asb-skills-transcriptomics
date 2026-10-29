---
name: cell-attribute-aware-community-detection
description: Use when performing DA-testing of cell-types from scRNA-Seq/snRNA-Seq
  data where condition labels (age, disease stage, severity, phenotype) exist per
  cell and standard Louvain, Milo, or DA-seq clustering yields weakly significant
  composition shifts.
when_to_use_negative:
- When no per-cell biological condition attribute is available to guide clustering
  — EVA degenerates to standard Louvain behavior and its advantage disappears
- When the dataset displays strong clustering structure — the paper reports ELVAR's
  improvement over the Louvain-analog was less pronounced in such studies
- When phenotype/sample labels are unreliable — the paper reports ELVAR is more sensitive
  to false positives in the clustering attribute, which can degrade community quality
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: ELVAR
  role: R-package implementing the full cell-attribute-aware clustering plus DA-testing
    pipeline (Eva_partitions, SelOptAlpha, ProcessEVA, DoDA)
  repo: https://github.com/aet21/ELVAR
- name: EVA
  role: Node-attribute aware community detection algorithm, a generalization of Louvain,
    used to cluster cells by state-space and attribute similarity
  repo: https://github.com/aet21/ELVAR
- name: Seurat
  role: Builds the Seurat object (FindVariableFeatures vst, ScaleData, RunPCA) and
    the cell-cell kNN graph via FindNeighbors (dims=1:8, k.param=20)
- name: igraph
  role: Holds the undirected state-manifold graph and carries the condition attribute
    as a vertex attribute
- name: MASS
  role: Supports the negative-binomial regressions used in the DoDA step
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_004
  - task_005
  - task_007
  - task_008
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:131b7b446c3fce6d4c035ab837ab74d69962ea253e59e0ad518ee332f550d44b
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
  - ggplot2
  - igraph
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# cell-attribute-aware-community-detection

## Summary

ELVAR-style cell-attribute aware community detection adapts the EVA algorithm — a generalization of Louvain — to cluster cells on the single-cell state-manifold by both expression similarity and attribute similarity (e.g. age, disease stage), improving sensitivity of downstream differential abundance (DA) testing of cell-types.

## When to use

Use when performing DA-testing of cell-types from scRNA-Seq/snRNA-Seq data where condition labels (age, disease stage, severity, phenotype) exist per cell and standard Louvain, Milo, or DA-seq clustering yields weakly significant composition shifts.

## When NOT to use

- When no per-cell biological condition attribute is available to guide clustering — EVA degenerates to standard Louvain behavior and its advantage disappears
- When the dataset displays strong clustering structure — the paper reports ELVAR's improvement over the Louvain-analog was less pronounced in such studies
- When phenotype/sample labels are unreliable — the paper reports ELVAR is more sensitive to false positives in the clustering attribute, which can degrade community quality

## Inputs

- scRNA-Seq or snRNA-Seq count matrix (e.g. lungCD4Tinfo from the ELVAR package, Tabula Muris Senis lung CD4+ T-cells, Chua nasopharyngeal swab, Becker colon snRNA-Seq)
- Cell metadata with a condition attribute (age, disease stage, Covid-19 severity, smell-loss phenotype) and a sample/replicate variable
- Cell-cell nearest-neighbor graph as an undirected igraph object with the condition attribute attached as a vertex attribute

## Outputs

- EVA community partition per alpha run (purity and modularity scores)
- Attribute-enriched, Bonferroni-significant merged communities per condition group (ProcessEVA output)
- Wald and LRT statistics from negative-binomial-regression DA tests (DoDA output, e.g. elvar_da_statistics.csv)
- Optimal alpha selection with purity/modularity/cluster-count criteria (SelOptAlpha)

## How to apply

Build a cell-cell nearest-neighbor state-manifold graph from a Seurat object (FindNeighbors, dims=1:8, k.param=20), convert it to an undirected igraph graph with the diagonal zeroed, and attach the condition attribute (e.g. age) as a vertex attribute via vertex_attr. Run Eva_partitions with resolution=1, threshold=1e-7, and the attribute name as Vattr.name, sweeping alpha from 0.1 to 0.9 (~100 runs per alpha) because alpha controls the purity-vs-modularity trade-off; select the optimal alpha with SelOptAlpha requiring purity and modularity within the 75th centile of their maxima and a cluster count deviating from the alpha=0 Louvain behavior. Identify attribute-enriched communities with ProcessEVA using a binomial enrichment test with a Bonferroni significance threshold, merging multiple enriched clusters per condition group, then run negative-binomial-regression DA testing with DoDA (varDA = cell-state/condition variable, varREP = sample/replicate variable) to obtain Wald and LRT statistics. Compare against the Louvain-analog benchmark (alpha=0, sequential or non-sequential) to confirm the attribute-aware clustering recovers stronger DA significance.

## Related tools

- **ELVAR** (R-package implementing the full cell-attribute-aware clustering plus DA-testing pipeline (Eva_partitions, SelOptAlpha, ProcessEVA, DoDA)) — https://github.com/aet21/ELVAR
- **EVA** (Node-attribute aware community detection algorithm, a generalization of Louvain, used to cluster cells by state-space and attribute similarity) — https://github.com/aet21/ELVAR
- **Seurat** (Builds the Seurat object (FindVariableFeatures vst, ScaleData, RunPCA) and the cell-cell kNN graph via FindNeighbors (dims=1:8, k.param=20))
- **igraph** (Holds the undirected state-manifold graph and carries the condition attribute as a vertex attribute)
- **MASS** (Supports the negative-binomial regressions used in the DoDA step)

## Examples

```
eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha=0.8, Vattr.name="age"); sigcl.o <- ProcessEVA(eva.o, seu.o, attrName="age"); da.o <- DoDA(seu.o, sigcl.o, varDA="naive", varREP="mouse")
```

## Evaluation signals

- Purity and modularity of the chosen alpha run fall within the 75th centile of their maxima across the 0.1–0.9 alpha sweep (checkable against the purmodInfo boxplots)
- Cluster count at the optimal alpha deviates from the standard Louvain (alpha=0) partition, confirming attribute-aware clustering is active
- Wald/LRT P-values from DoDA are more significant than the Louvain-analog (LVdet/LVnonseq) benchmark, e.g. one-sided Wilcoxon rank-sum tests on P-value distributions across 100 runs
- Biological direction matches the paper: naive Cd4+ T-cell fraction significantly decreases and mature/activated fraction increases with age; binomial enrichment survives Bonferroni correction and ≥2 replicates contribute per enriched group
- Simulated ground-truth recovery: adjusted Rand Index against simulated perturbation labels exceeds the Louvain baseline, at least at larger alpha values

## Limitations

- EVA's improvement over Louvain was more pronounced in studies with weaker clustering structure and less noticeable with strong clustering
- ELVAR is sensitive to false positives in the clustering attribute (sample/phenotype label), which can affect the quality of inferred communities; cell-type annotation FPs are less damaging
- Increased computational complexity and runtimes compared to Milo and the Louvain-analog (up to 100 runs per each of nine alpha values, i.e. 900 runs), though deemed not practically prohibitive
- No benchmark against miloDE was performed; the authors flag this as an open question

## Evidence

- [abstract] Here we present a differential abundance testing paradigm called ELVAR that uses cell attribute aware clustering when inferring differentially enriched communities within the single-cell manifold.: "Here we present a differential abundance testing paradigm called ELVAR that uses cell attribute aware clustering when inferring differentially enriched communities within the single-cell manifold."
- [intro] we here adapt a node-attribute aware community detection algorithm called EVA17, which is a generalization of the very popular Louvain clustering method18: "we here adapt a node-attribute aware community detection algorithm called EVA17, which is a generalization of the very popular Louvain clustering method18"
- [methods] eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha= 0.8, Vattr.name= "age");: "eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha= 0.8, Vattr.name= "age");"
- [methods] selAlpha.o <- SelOptAlpha(nC.m,nc0,pur.m,mod.m,alpha.v=seq(0.1,0.9,0.1));: "selAlpha.o <- SelOptAlpha(nC.m,nc0,pur.m,mod.m,alpha.v=seq(0.1,0.9,0.1));"
- [discussion] to identify an optimal a parameter value may require a total of 100 runs for each of nine a parameter values, so a total of 900 runs: "to identify an optimal a parameter value may require a total of 100 runs for each of nine a parameter values, so a total of 900 runs"
- [readme] ELVAR is an R-package for differential abundance (DA) testing of cell-types in single-cell RNA-Seq data. It implements an Extended Louvain clustering Algorithm (EVA): "ELVAR is an R-package for differential abundance (DA) testing of cell-types in single-cell RNA-Seq data. It implements an Extended Louvain clustering Algorithm (EVA)"
