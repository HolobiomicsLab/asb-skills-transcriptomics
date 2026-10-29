---
name: steiner-tree-subnetwork-analysis
description: Use when you have seed/disease genes plus group-labeled RNA-seq data
  and need a data-driven directed subnetwork whose significant causal paths (Bonferroni-corrected
  ACE testing) explain disease perturbation.
when_to_use_negative:
- Input is a raw count matrix without group labels or seed genes — Steiner terminals
  cannot be defined and SEMace source2sink effects are not estimable
- The goal is whole-interactome structure learning rather than extracting a perturbed
  backbone around seed genes (SEMtree offers TAHC, EBC, WTC and DAG-based alternatives)
- Only undirected perturbation modules are needed without causal direction — the five
  active subnetwork detection methods output undirected edges and the Chu–Liu–Edmonds
  tree conversion step is what adds directionality
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: SEMtree
  role: Builds the Steiner-tree (ST) perturbed backbone subnetwork with Kou's algorithm
    from p-value-weighted edges
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMace
  role: Estimates Average Causal Effects on the ST subnetwork with type="optimal",
    effect="source2sink" to rank directed paths by |z|
- name: SEMpath
  role: Extracts the directed source-to-sink path subgraph and fits it with group
    information
- name: SEMrun
  role: Fits node group perturbation on the path subgraph; reports activation/inhibition
    via Brown's combined P-values
- name: transformData
  role: Nonparanormal transformation of the expression matrix to relax normality assumptions
    before SEM fitting
- name: org.Hs.eg.db
  role: Entrez-to-SYMBOL gene ID conversion of path nodes via mapIds
- name: SEMdata
  role: Provides the GSE172114 COVID-19 expression matrix, group vector, and reference
    networks/pathways
  repo: https://github.com/fernandoPalluzzi/SEMdata
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:12bea9fbe7bcbde30e93b63b714ed5d43b7851bd776faeb192f2d4747d3096c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - SEMace
  - SEMdata
  - SEMgraph
  - SEMpath
  - SEMrun
  - SEMtree
  - SEMtree()
  - org.Hs.eg.db
  - transformData
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# steiner-tree-subnetwork-analysis

## Summary

Extract a perturbed backbone subnetwork as a Steiner tree (Kou's algorithm) from a weighted interactome using SEMtree(), then interrogate it with SEMace, SEMpath and SEMrun to find significant source-to-sink perturbation paths (e.g., ATG16L1 -> CCR5, 14 nodes/13 edges) in the GSE172114 COVID-19 dataset.

## When to use

Use when you have seed/disease genes plus group-labeled RNA-seq data and need a data-driven directed subnetwork whose significant causal paths (Bonferroni-corrected ACE testing) explain disease perturbation.

## When NOT to use

- Input is a raw count matrix without group labels or seed genes — Steiner terminals cannot be defined and SEMace source2sink effects are not estimable
- The goal is whole-interactome structure learning rather than extracting a perturbed backbone around seed genes (SEMtree offers TAHC, EBC, WTC and DAG-based alternatives)
- Only undirected perturbation modules are needed without causal direction — the five active subnetwork detection methods output undirected edges and the Chu–Liu–Edmonds tree conversion step is what adds directionality

## Inputs

- RNA-seq expression matrix and {0: control, 1: case} group vector from SEMdata (e.g., COVID-19 GSE172114: 46 critical vs 23 non-critical cases)
- Seed (disease) gene list used as Steiner terminals
- Reference interactome or pathway igraph object (e.g., SEMgraph::kegg, 5007 nodes / 44755 edges)
- Edge weights from gene perturbation p-values

## Outputs

- Steiner-tree (ST) perturbed backbone subnetwork (Kou's algorithm) as an igraph object
- Bonferroni-corrected ACE table ranked by decreasing |z|, restricted to source-to-sink pairs
- Directed path subgraph (e.g., ATG16L1 -> CCR5, 14 nodes, 13 edges) with per-node group perturbation coefficients and Brown's combined P-values
- Gene symbol annotations via org.Hs.eg.db::mapIds

## How to apply

First transform the expression matrix with SEMgraph::transformData() (nonparanormal transform) to relax the normality assumption — for GSE172114 this is the {0: control, 1: case} COVID-19 data from SEMdata. Weight network edges with perturbation p-values and run SEMtree() with the Steiner-tree (ST) backbone filter using Kou's algorithm, supplying seed genes as Steiner terminals. Rank causal paths by running SEMace() on the ST subnetwork with type = "optimal" (optimal adjustment set blocking all backdoor paths), effect = "source2sink", and declare significance after Bonferroni correction at alpha = 0.05. Extract a candidate directed path with SEMpath(path = "directed") from a Bonferroni-significant source-sink pair (e.g., ATG16L1/55054 to CCR5/1234), then fit group perturbation on the path subgraph with SEMrun (node group effects; Brown's combined P-values for activation/inhibition). Finally map Entrez IDs to symbols with org.Hs.eg.db::mapIds and verify the recovered topology matches the reported 14 nodes / 13 edges structure.

## Related tools

- **SEMtree** (Builds the Steiner-tree (ST) perturbed backbone subnetwork with Kou's algorithm from p-value-weighted edges) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMace** (Estimates Average Causal Effects on the ST subnetwork with type="optimal", effect="source2sink" to rank directed paths by |z|)
- **SEMpath** (Extracts the directed source-to-sink path subgraph and fits it with group information)
- **SEMrun** (Fits node group perturbation on the path subgraph; reports activation/inhibition via Brown's combined P-values)
- **transformData** (Nonparanormal transformation of the expression matrix to relax normality assumptions before SEM fitting)
- **org.Hs.eg.db** (Entrez-to-SYMBOL gene ID conversion of path nodes via mapIds)
- **SEMdata** (Provides the GSE172114 COVID-19 expression matrix, group vector, and reference networks/pathways) — https://github.com/fernandoPalluzzi/SEMdata

## Examples

```
data.npn <- transformData(covid$exprs)$data; st <- SEMtree(kegg, data.npn, covid$group, seed = seeds, filter = "ST"); ace <- SEMace(st, data.npn, type = "optimal", effect = "source2sink", method = "bonferroni", alpha = 0.05)
```

## Evaluation signals

- Recovered path topology matches the reported structure exactly: 14 nodes and 13 edges for ATG16L1 -> CCR5
- Exactly 2 of the 14 path genes have non-significant group perturbation coefficients (p >= 0.05); the remaining 12 are perturbed
- The ACE table contains only source-to-sink pairs significant after Bonferroni correction at alpha = 0.05, ranked by decreasing |z|
- High pairwise correlation among the 13 edges on the recovered path (co-expression consistency check)
- Nonparanormal transform applied before any SEM fitting (data.npn <- transformData(exprs)$data) so normality assumptions are relaxed

## Limitations

- Bonferroni correction is conservative; the same correction style is used elsewhere in the workflow (e.g., SEMbap with method = "bonferroni"), which may miss weaker perturbation signals
- Steiner-tree extraction depends on the seed gene list — different Steiner terminals yield different backbones
- The article's discussion section text is absent from the source document, so boundary conditions beyond the reported COVID-19 example are not documented

## Evidence

- [methods] Perturbed backbone as a Steiner tree (Kou's algorithm): "# Perturbed backbone as a Steiner tree (Kou's algorithm)"
- [methods] ACE estimation with optimal adjustment set: "ace <- SEMace(graph = alsData$graph, data = data.npn, type = "optimal", effect = "source2sink", method = "BH", alpha = 0.05)"
- [methods] Nonparanormal transform rationale: "we apply a nonparanormal transform to relax the normality assumption."
- [abstract] SEMtree captures biologically relevant subnetworks: "Compared to existing methods, SEMtree() is able to capture biologically relevant subnetworks with simple visualization of directed paths, good perturbation extraction, and classifier performance."
- [methods] Path extraction with SEMpath: "path <- SEMpath(alsData$graph, data.npn, alsData$group, from = source, to = sink, path = "directed", verbose = TRUE)"
- [readme] SEMdata provides COVID-19 dataset: [evidence span withheld — anchor status: external_doc]
