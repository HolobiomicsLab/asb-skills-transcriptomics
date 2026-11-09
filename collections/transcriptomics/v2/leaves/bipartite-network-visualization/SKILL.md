---
name: bipartite-network-visualization
description: Use when you need to visually explore bipartite relationships between
  two entity types, such as KEGG pathways and the top-ranked eeSNV genes annotated
  to them, following enrichment analysis.
when_to_use_negative:
- When relationships are not bipartite (e.g. gene-gene co-expression), a plain force-directed
  graph without bipartite structure is more appropriate
- When quantitative enrichment statistics alone suffice and no interactive exploration
  of gene-pathway membership is needed
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0092
tools:
- name: Gephi
  role: Force-directed layout and interactive exploration of the bipartite pathway-gene
    graph
- name: Cytoscape
  role: Alternative graph analysis software to render and interactively explore the
    bipartite network
- name: DAVID
  role: Source of KEGG pathway enrichment p-values defining which genes are annotated
    to which pathways
- name: SSrGE
  role: Produces the ranked eeSNV gene lists (rank_genes()) whose top members populate
    the gene nodes of the graph
  repo: https://github.com/lanagarmire/SSrGE.git
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:084148ef8db383e3bb6c6e9a973dd1e6db62f934108f1abf13244a7fe457e41b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/SSrGE.git
  tools:
  - Cytoscape
  - DAVID
  - Gephi
  - Python
  - SSrGE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# bipartite-network-visualization

## Summary

Constructs a bipartite pathway-gene graph (edge: gene annotated to pathway, e.g. from KEGG enrichment via DAVID) and renders it with force-directed graph drawing (spring-like attractive forces, electrical repulsions) in Gephi or Cytoscape to explore the bipartite relationships interactively.

## When to use

Use when you need to visually explore bipartite relationships between two entity types, such as KEGG pathways and the top-ranked eeSNV genes annotated to them, following enrichment analysis.

## When NOT to use

- When relationships are not bipartite (e.g. gene-gene co-expression), a plain force-directed graph without bipartite structure is more appropriate
- When quantitative enrichment statistics alone suffice and no interactive exploration of gene-pathway membership is needed

## Inputs

- Pathway-gene annotation pairs from KEGG enrichment results (DAVID p-values per pathway)
- Aggregated enrichment scores (sum -log10(p-value)) per pathway
- Ranked gene lists (e.g. top 100 SSrGE-ranked eeSNV genes per dataset)

## Outputs

- Bipartite pathway-gene graph (node and edge tables)
- Force-directed network rendering compatible with Gephi or Cytoscape

## How to apply

After aggregating enrichment evidence per pathway (e.g. summing -log10(p-value) across member genes for KEGG pathways from DAVID), build a bipartite graph whose nodes are pathways and genes, with an edge for each gene annotated to a pathway. Export the node/edge table and load it into Gephi or Cytoscape. Render using force-directed graph drawing, which models spring-like attractive forces between nodes connected by edges and electrical repulsions between nodes. Judge success by comparing the rendered network against the reference figure (e.g. Fig. 5), including the antigen processing and presentation pathway with sum -log10(p-value) of 15.80.

## Related tools

- **Gephi** (Force-directed layout and interactive exploration of the bipartite pathway-gene graph)
- **Cytoscape** (Alternative graph analysis software to render and interactively explore the bipartite network)
- **DAVID** (Source of KEGG pathway enrichment p-values defining which genes are annotated to which pathways)
- **SSrGE** (Produces the ranked eeSNV gene lists (rank_genes()) whose top members populate the gene nodes of the graph) — https://github.com/lanagarmire/SSrGE.git

## Examples

```
ssrge = SSrGE(snv_id_list=snv_id_list, gene_id_list=gene_id_list); ssrge.fit(X, Y); print ssrge.rank_genes()  # then submit top 100 genes to DAVID, export pathway-gene edges, load into Gephi/Cytoscape and apply force-directed layout
```

## Evaluation signals

- Rendered network reproduces Fig. 5 of the article (bipartite pathway-gene structure)
- Antigen processing and presentation pathway node carries the aggregated sum -log10(p-value) of 15.80
- Every edge corresponds to a gene annotated to that pathway in the KEGG/DAVID results (no spurious edges)
- Force-directed layout converges with spring-like attractive forces on edges and electrical repulsions between all nodes
- Graph opens and is interactive in both Gephi and Cytoscape without node/edge loss

## Limitations

- Sum of -log10(p-value) aggregation across datasets is a heuristic, not a formal meta-analysis p-value
- LASSO may arbitrarily select one of highly correlated SNV features, so gene membership in the network may not be unique
- Interactive graph software output depends on layout seeds; exact node positions may vary between runs

## Evidence

- [discussion] Graph analysis software such as Gephi36 or Cytoscape54 can be utilized to explore the bipartite relationships in an interactive manner.: "Graph analysis software such as Gephi36 or Cytoscape54 can be utilized to explore the bipartite relationships in an interactive manner."
- [discussion] force-directed graph drawing spring/electrical: "We have used force-directed graph drawing algorithms involve spring-like attractive forces and electrical repulsions between nodes that are connected by edges."
- [methods] SSrGE ranking function: "print ssrge.rank_genes()"
- [other] Validation target: antigen processing and presentation pathway: [evidence span withheld — anchor status: not_found]
