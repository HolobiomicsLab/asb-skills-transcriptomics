---
name: network-regularized-regression-fitting
description: Use when you have a gene-level p-value table (gene, pvalue) and a pathway
  database such as GO biological process (terms of size 50-500), and redundant/overlapping
  pathways make standard enrichment tests unstable or incoherent.
when_to_use_negative:
- The analysis is a standard ORA/GSEA on non-redundant pathways where inter-pathway
  dependencies are not a concern — the added network regularization is unnecessary
- Input is already an enrichment result table or a feature matrix rather than gene-level
  p-values — pareg regresses differential expression p-values, not precomputed enrichment
  scores
- The pathway database lacks usable similarity structure (e.g., no shared genes),
  so the term_network matrix provides no regularization signal
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: pareg
  role: 'Core method: fits the regularized generalized linear model with LASSO and
    network regularization on the p-value table, and provides compute_term_similarities,
    pathway_similarities data, and the plot/ggraph network view'
  repo: https://github.com/cbg-ethz/pareg
- name: R
  role: Implementation environment; pareg is an R/Bioconductor package installed via
    BiocManager::install('pareg')
  repo: https://bioconductor.org/packages/release/bioc/html/pareg.html
- name: tidyverse
  role: Data manipulation (e.g., arranging the exported fit by desc(abs(enrichment)))
    within the pareg vignette workflow
- name: ggraph
  role: Renders the pathway network view of the fitted model (top non-isolated terms
    with min_similarity threshold)
- name: enrichplot
  role: Optional downstream visualization of the pareg fit via as_enrichplot_object(fit)
  repo: https://github.com/YuLab-SMU/enrichplot
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad522
    title: Coherent pathway enrichment estimation by modeling inter-pathway dependencies
      using regularized regression
  source_content_hash: sha256:6cccdc4d8a339c7c44be7a595712e330fd748eb02edf7270ca1c9c2b07bcf4a8
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# network-regularized-regression-fitting

## Summary

Fit pareg's regularized generalized linear model, which regresses gene-level differential expression p-values onto pathway terms while a network regularization term based on a pathway similarity matrix (e.g., Jaccard) couples similar pathways, yielding coherent enrichment scores across redundant gene sets.

## When to use

Use when you have a gene-level p-value table (gene, pvalue) and a pathway database such as GO biological process (terms of size 50-500), and redundant/overlapping pathways make standard enrichment tests unstable or incoherent.

## When NOT to use

- The analysis is a standard ORA/GSEA on non-redundant pathways where inter-pathway dependencies are not a concern — the added network regularization is unnecessary
- Input is already an enrichment result table or a feature matrix rather than gene-level p-values — pareg regresses differential expression p-values, not precomputed enrichment scores
- The pathway database lacks usable similarity structure (e.g., no shared genes), so the term_network matrix provides no regularization signal

## Inputs

- Gene-level p-value table with columns (gene, pvalue), e.g., differential expression p-values from TCGA-BRCA
- Pathway term database as a (term, gene) table, e.g., GO Biological Process filtered to term sizes 50-500
- Pathway similarity matrix (Jaccard or pre-computed pathway_similarities from the pareg package)

## Outputs

- Fitted pareg model object (regularized GLM fit)
- Dataframe of pathway enrichment scores ranked by desc(abs(enrichment)), e.g., top-25 terms matching Fig 2a
- Pathway network view of top-50 non-isolated terms rendered with plot(fit, min_similarity = 0.1) via ggraph (Fig 2b)

## How to apply

Load the gene-level p-value table and the pathway term database in R, then compute the pairwise pathway similarity matrix with pareg's compute_term_similarities helper using similarity_function = jaccard (or load pre-computed measures via data(pathway_similarities, package = 'pareg')). Fit the model with pareg on the p-value table, setting network_param = 1 and term_network to the Jaccard similarity matrix so that LASSO regularization is combined with network regularization over pathway dependencies — this is what classifies pareg as a modular enrichment analysis tool. Export the fit to a dataframe and rank terms by desc(abs(enrichment)); the exploratory TCGA-BRCA analysis retains the top-25 terms for comparison and the top-50 non-isolated terms for the network view. The rationale is that modeling inter-pathway dependencies directly in the enrichment computation makes pareg more robust to noise than competing methods.

## Related tools

- **pareg** (Core method: fits the regularized generalized linear model with LASSO and network regularization on the p-value table, and provides compute_term_similarities, pathway_similarities data, and the plot/ggraph network view) — https://github.com/cbg-ethz/pareg
- **R** (Implementation environment; pareg is an R/Bioconductor package installed via BiocManager::install('pareg')) — https://bioconductor.org/packages/release/bioc/html/pareg.html
- **tidyverse** (Data manipulation (e.g., arranging the exported fit by desc(abs(enrichment))) within the pareg vignette workflow)
- **ggraph** (Renders the pathway network view of the fitted model (top non-isolated terms with min_similarity threshold))
- **enrichplot** (Optional downstream visualization of the pareg fit via as_enrichplot_object(fit)) — https://github.com/YuLab-SMU/enrichplot

## Examples

```
mat_sim <- compute_term_similarities(df_terms, similarity_function = jaccard); fit <- pareg(df_genes, df_terms, network_param = 1, term_network = mat_sim)
```

## Evaluation signals

- The ranked top-25 enrichment table must contain axoneme assembly, retrograde vesicle-mediated transport, and synaptic vesicle recycling among the leading hits for the TCGA-BRCA analysis
- The network figure (plot(fit, min_similarity = 0.1)) must show clustered pathway groups consistent with the reported TCGA-BRCA findings, not scattered isolated terms
- Exported fit dataframe rows must be orderable by desc(abs(enrichment)) and cover all terms in the input term database
- The term_network argument must be a square similarity matrix over the same terms as the term database (e.g., from compute_term_similarities with jaccard)
- On synthetic benchmarks where active pathway clusters are selected as true positives, the fit should rank those activated pathways above terms whose genes are drawn from a Uniform null

## Limitations

- Results depend on the chosen similarity measure — on GO Biological Process, different pathway similarity measures can differ substantially, affecting the network regularization
- Large and redundant pathway databases (KEGG, Reactome, GO) complicate enrichment computation; pareg requires term size filtering (e.g., 50-500) and a similarity threshold (min_similarity = 0.1) to keep the network view interpretable
- pareg provides pre-computed similarity measures only for selected pathway databases; other databases require computing similarities yourself

## Evidence

- [abstract] pareg is a novel gene set enrichment method based on a regularized generalized linear model that incorporates dependencies between gene sets: "develop a novel gene set enrichment method, called pareg, which is based on a regularized generalized linear model and directly incorporates dependencies between gene sets"
- [methods] Network regularization rationale: "The network regularization term is based on a pathway similarity matrix (e.g., defined by Jaccard similarity) and thus classifies this method as a modular enrichment analysis tool"
- [methods] Similarity computation helper: "compute_term_similarities(
  df_terms,
  similarity_function = jaccard
)"
- [methods] Pre-computed similarities loading: "data(pathway_similarities, package = "pareg")"
- [readme] Project description: "Pathway enrichment computations using a regularized regression approach to incorporate inter-pathway relations in the statistical model."
