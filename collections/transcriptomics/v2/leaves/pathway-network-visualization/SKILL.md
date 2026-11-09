---
name: pathway-network-visualization
description: Use when a pareg fit (or an enrichment result convertible via as_enrichplot_object)
  must be inspected for inter-pathway structure — clustered groups of related terms
  — rather than only a top-N term ranking.
when_to_use_negative:
- When only a flat top-N ranked term table is needed and no inter-pathway structure
  is of interest — a ranked dataframe by desc(abs(enrichment)) suffices without the
  network view.
- When the pathway database or the enriched term set is so small or sparse that no
  term pairs share genes (all similarities near zero), the network collapses to isolated
  nodes and adds no insight.
- When working with an enrichment result not produced by pareg and not convertible
  via as_enrichplot_object — the plot(fit, ...) path expects a pareg fit object.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: pareg
  role: Provides the fitted enrichment model whose plot() method renders the pathway
    network view, plus as_enrichplot_object and compute_term_similarities helpers
  repo: https://github.com/cbg-ethz/pareg
- name: ggraph
  role: Underlying graph visualization library used by pareg to draw the pathway network
    view
- name: enrichplot
  role: Visualization ecosystem that the fit is converted into via as_enrichplot_object
    for interoperable enrichment plotting
  repo: https://github.com/YuLab-SMU/enrichplot
- name: tidyverse
  role: Used for data wrangling and ranking terms by desc(abs(enrichment)) before
    selecting the top-50 non-isolated terms
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad522
    title: Coherent pathway enrichment estimation by modeling inter-pathway dependencies
      using regularized regression
  source_content_hash: sha256:e78cf8449b85f863ddf6c01b80a686f1e7c8f921b7d3fa1b08bffac753579e62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/pareg
  tools:
  - R
  - enrichplot
  - ggraph
  - pareg
  - tidyverse
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# pathway-network-visualization

## Summary

Render pareg enrichment fits as a pathway network view, where nodes are enriched GO terms and edges are drawn between terms whose similarity (e.g. Jaccard) exceeds a threshold, to expose clustered pathway groups rather than an unstructured ranked list.

## When to use

Use when a pareg fit (or an enrichment result convertible via as_enrichplot_object) must be inspected for inter-pathway structure — clustered groups of related terms — rather than only a top-N term ranking.

## When NOT to use

- When only a flat top-N ranked term table is needed and no inter-pathway structure is of interest — a ranked dataframe by desc(abs(enrichment)) suffices without the network view.
- When the pathway database or the enriched term set is so small or sparse that no term pairs share genes (all similarities near zero), the network collapses to isolated nodes and adds no insight.
- When working with an enrichment result not produced by pareg and not convertible via as_enrichplot_object — the plot(fit, ...) path expects a pareg fit object.

## Inputs

- A fitted pareg model object (pareg fit on a gene p-value table with columns gene, pvalue)
- A pathway similarity matrix, e.g. from compute_term_similarities(df_terms, similarity_function = jaccard) or data(pathway_similarities)
- A ranked enrichment dataframe from as.data.frame(fit) with an enrichment score column

## Outputs

- A ggraph/ggplot network visualization of enriched pathway terms (pathway network view, analogous to Fig 2b)
- An enrichplot-compatible object from as_enrichplot_object(fit)
- A filtered set of top-50 non-isolated terms above min_similarity = 0.1

## How to apply

Fit the pareg regularized GLM on the gene-level p-value table (with network_param and a term_network similarity matrix computed via compute_term_similarities using the jaccard similarity_function), then convert the fit with as_enrichplot_object and render it with ggraph-backed plotting. Following the TCGA-BRCA exploratory analysis (Fig 2b), select the top-50 non-isolated terms from the fit dataframe ranked by desc(abs(enrichment)) and call plot(fit, min_similarity = 0.1) so edges only connect terms with Jaccard similarity above 0.1, yielding clustered pathway groups. The rationale is that pareg's network regularization makes coherent terms score together, so the network view reveals pathway clusters that a flat ranking obscures. Validate by checking that the leading enriched terms (axoneme assembly, retrograde vesicle-mediated transport, synaptic vesicle recycling in the TCGA-BRCA case) appear as clustered nodes consistent with the published figure.

## Related tools

- **pareg** (Provides the fitted enrichment model whose plot() method renders the pathway network view, plus as_enrichplot_object and compute_term_similarities helpers) — https://github.com/cbg-ethz/pareg
- **ggraph** (Underlying graph visualization library used by pareg to draw the pathway network view)
- **enrichplot** (Visualization ecosystem that the fit is converted into via as_enrichplot_object for interoperable enrichment plotting) — https://github.com/YuLab-SMU/enrichplot
- **tidyverse** (Used for data wrangling and ranking terms by desc(abs(enrichment)) before selecting the top-50 non-isolated terms)

## Examples

```
mat_similarities <- compute_term_similarities(df_terms, similarity_function = jaccard); obj <- as_enrichplot_object(fit)
```

## Evaluation signals

- The rendered network is built from the top-50 non-isolated terms, with edges only between terms whose similarity exceeds min_similarity = 0.1.
- The ranked top-25 table underlying the visualization contains axoneme assembly, retrograde vesicle-mediated transport, and synaptic vesicle recycling among the leading hits (TCG-BRCA Fig 2a check).
- The network figure shows clustered pathway groups consistent with the reported TCGA-BRCA findings (Fig 2b check).
- The fit converts cleanly to an enrichplot object via as_enrichplot_object(fit) without error, confirming schema compatibility.
- Similarity values feeding the edges come from the Jaccard matrix computed by compute_term_similarities or loaded via data(pathway_similarities).

## Limitations

- Pathway similarity measures can differ substantially from each other (as observed on GO Biological Process), so the choice of Jaccard versus another measure changes the network edges.
- Growing size and redundancy of pathway databases (KEGG, Reactome, GO) complicate enrichment and visualization; filtering terms to a size range (e.g. 50–500 genes) is needed to keep the network interpretable.
- The network view is exploratory and matches a published figure only when the same similarity matrix, network_param = 1, and term-size filters are used.

## Evidence

- [readme] Pathway enrichment computations using a regularized regression approach to incorporate inter-pathway relations in the statistical model.: "Pathway enrichment computations using a regularized regression approach to incorporate inter-pathway relations in the statistical model."
- [methods] The network regularization term is based on a pathway similarity matrix (e.g., defined by Jaccard similarity): "The network regularization term is based on a pathway similarity matrix (e.g., defined by Jaccard similarity) and thus classifies this method as a modular enrichment analysis tool"
- [methods] ...and also visualized in a pathway network view.: [evidence span withheld — anchor status: not_found]
- [methods] obj <- as_enrichplot_object(fit): "as_enrichplot_object(fit)"
- [methods] we compute pairwise pathway similarities with `pareg`'s helper function: "we compute pairwise pathway similarities with `pareg`'s helper function"
