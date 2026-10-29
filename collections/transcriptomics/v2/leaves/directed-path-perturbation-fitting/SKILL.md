---
name: directed-path-perturbation-fitting
description: Use when a significant causal ACE pair (e.g. after Bonferroni correction
  at alpha = 0.05) defines a source and sink node in a directed SEMtree subnetwork,
  and you must recover the connecting directed path and assess which of its nodes/edges
  are group-perturbed.
when_to_use_negative:
- When no significant source-to-sink ACE pair exists after Bonferroni correction —
  SEMpath directed extraction requires a validated source/sink pair from SEMace()
- When the subnetwork is undirected or you want community/cluster-level perturbation
  rather than a single directed route — use TAHC/EBC clustering or SEMbap workflows
  instead
- When edges lack a data-driven causal orientation (e.g. raw active subnetworks before
  Chu–Liu–Edmonds tree conversion), since path = 'directed' needs directed edges
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SEMpath
  role: Extracts the directed source-to-sink path subgraph from the parent SEMtree/igraph
    network and fits the path model
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMrun
  role: Fits the path subgraph SEM with the binary group vector to obtain node group
    effects and Brown's combined P-values for activation/inhibition
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMac
  role: 'Preceding step: estimates ACEs (type = ''optimal'', effect = ''source2sink'')
    to identify the significant source and sink nodes defining the path'
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: transformData
  role: Nonparanormal transform of the RNA-seq expression matrix to relax the normality
    assumption before SEM fitting
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: org.Hs.eg.db
  role: Bioconductor package used with mapIds to convert Entrez IDs (55054, 1234)
    to gene symbols (ATG16L1, CCR5)
- name: SEMdata
  role: 'Supplies the GSE172114 COVID-19 expression matrix and {0: control, 1: case}
    group vector'
  repo: https://github.com/fernandoPalluzzi/SEMdata
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:6d0b025440499d5c01610d58930bd62e645991f01a614c21ed045f6c8029144d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - SEMac
  - SEMdata
  - SEMgraph
  - SEMpath
  - SEMrun
  - SEMtree()
  - org.Hs.eg.db
  - transformData
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# directed-path-perturbation-fitting

## Summary

Extract a directed source-to-sink path from a SEM-based subnetwork with SEMpath(path = "directed") and fit group perturbation on the path subgraph with SEMrun, identifying activation/inhibition routes between condition groups (e.g. the ATG16L1→CCR5 path in the COVID-19 GSE172114 Steiner tree).

## When to use

Use when a significant causal ACE pair (e.g. after Bonferroni correction at alpha = 0.05) defines a source and sink node in a directed SEMtree subnetwork, and you must recover the connecting directed path and assess which of its nodes/edges are group-perturbed.

## When NOT to use

- When no significant source-to-sink ACE pair exists after Bonferroni correction — SEMpath directed extraction requires a validated source/sink pair from SEMace()
- When the subnetwork is undirected or you want community/cluster-level perturbation rather than a single directed route — use TAHC/EBC clustering or SEMbap workflows instead
- When edges lack a data-driven causal orientation (e.g. raw active subnetworks before Chu–Liu–Edmonds tree conversion), since path = 'directed' needs directed edges

## Inputs

- Directed igraph subnetwork (e.g. SEMtree() Steiner-tree ST module from the COVID-19 GSE172114 dataset)
- Expression matrix transformed with transformData() (nonparanormal transform)
- Binary group vector {0: control, 1: case} from SEMdata
- Significant source and sink gene identifiers (Entrez IDs, e.g. 55054 = ATG16L1, 1234 = CCR5) from the SEMace() Bonferroni-corrected ACE table

## Outputs

- Directed path subgraph (reported: 14 nodes, 13 edges with high pairwise correlation) connecting source to sink
- SEMrun group-effect table of per-node perturbation coefficients and P-values (exactly 2 non-significant genes, p >= 0.05, expected)
- Brown's combined P-values classifying the route as activation or inhibition
- Gene symbol annotations via org.Hs.eg.db::mapIds

## How to apply

First rank candidate causal pairs by running SEMace() on the subnetwork with type = 'optimal' and effect = 'source2sink', keeping only Bonferroni-corrected significant pairs ordered by decreasing |z|; in the COVID-19 ST module this yields the ATG16L1 (Gene ID 55054) → CCR5 (Gene ID 1234) pair. Apply a nonparanormal transform (transformData()) to the expression matrix beforehand to relax the normality assumption. Call SEMpath(graph, data.npn, group, from = source, to = sink, path = 'directed') to extract the directed path subgraph — for ATG16L1→CCR5 expect a topology of 14 nodes and 13 edges. Then fit the path subgraph with SEMrun(node group effects, fit = 1) to obtain per-node group coefficients and Brown's combined P-values for activation/inhibition. Map Entrez IDs to symbols with org.Hs.eg.db::mapIds and count perturbed (p < 0.05) versus unperturbed genes — the reported path has exactly two unperturbed genes out of 14 nodes.

## Related tools

- **SEMpath** (Extracts the directed source-to-sink path subgraph from the parent SEMtree/igraph network and fits the path model) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMrun** (Fits the path subgraph SEM with the binary group vector to obtain node group effects and Brown's combined P-values for activation/inhibition) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMac** (Preceding step: estimates ACEs (type = 'optimal', effect = 'source2sink') to identify the significant source and sink nodes defining the path) — https://github.com/fernandoPalluzzi/SEMgraph
- **transformData** (Nonparanormal transform of the RNA-seq expression matrix to relax the normality assumption before SEM fitting) — https://github.com/fernandoPalluzzi/SEMgraph
- **org.Hs.eg.db** (Bioconductor package used with mapIds to convert Entrez IDs (55054, 1234) to gene symbols (ATG16L1, CCR5))
- **SEMdata** (Supplies the GSE172114 COVID-19 expression matrix and {0: control, 1: case} group vector) — https://github.com/fernandoPalluzzi/SEMdata

## Examples

```
path <- SEMpath(alsData$graph, data.npn, alsData$group, from = source, to = sink, path = "directed", verbose = TRUE)
```

## Evaluation signals

- Topology check: extracted directed path must have exactly 14 nodes and 13 edges
- Perturbation check: exactly 2 of the path's genes have non-significant group coefficients (p >= 0.05); the remaining 12 are perturbed
- ACE-table check: only source-to-sink pairs significant under Bonferroni correction at alpha = 0.05 appear, ranked by decreasing |z|
- Edge quality: pairwise correlations along the 13 edges are high
- ID mapping check: mapIds returns symbols ATG16L1 (55054) and CCR5 (1234) for the path endpoints

## Limitations

- The nonparanormal transform must be applied first; SEM fitting on raw non-normal RNA-seq counts violates normality assumptions
- Directed path extraction depends on the causal orientation of the parent network — results change if the ST backbone or DAG orientation differs
- Reported node counts (14 nodes, 13 edges, 2 unperturbed) are specific to the GSE172114 Steiner-tree module and Bonferroni ACE filtering; other networks/subnetworks will differ

## Evidence

- [methods] Path extraction and fitting (SEMpath): "path <- SEMpath(alsData$graph, data.npn, alsData$group, from = source, to = sink, path = "directed", verbose = TRUE)"
- [methods] Node perturbation with group vector: "sem1 <- SEMrun(graph = alsData$graph, data = data.npn, group = alsData$group)"
- [methods] Nonparanormal transform rationale: "we apply a nonparanormal transform to relax the normality assumption."
- [methods] ACE estimation parameters: "ace <- SEMace(graph = alsData$graph, data = data.npn, type = "optimal", effect = "source2sink", method = "BH", alpha = 0.05)"
- [readme] Package capability for perturbed path finding: "Perturbed paths finding, community searching and sample scoring, together with graph plotting utilities, tracing model architecture modifications and perturbation (i.e., activation or repression)"
