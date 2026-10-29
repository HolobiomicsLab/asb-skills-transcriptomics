---
name: pathway-enrichment-estimation
description: Use when a gene-level p-value table (gene, pvalue) and a pathway database
  (e.g., GO BP terms of size 50-500) must be converted into coherent pathway-level
  enrichment scores that account for redundant/overlapping gene sets.
when_to_use_negative:
- The input is already a pathway-level enrichment result and only visualization is
  needed (use enrichplot directly rather than re-fitting pareg).
- The pathway database has negligible overlap/redundancy between terms, so network
  regularization on inter-pathway dependencies adds no modeling benefit.
- No gene-to-pathway mapping or similarity matrix is available, leaving no term_network
  for the regularized regression.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3517
tools:
- name: pareg
  role: 'Core method: regularized generalized linear model estimating pathway enrichment
    scores with network regularization on a pathway similarity matrix'
  repo: https://github.com/cbg-ethz/pareg
- name: R
  role: Implementation environment; pareg is distributed as an R/Bioconductor package
  repo: https://bioconductor.org/packages/release/bioc/html/pareg.html
- name: tidyverse
  role: Data wrangling and ranking of enrichment results (arrange by desc(abs(enrichment)))
- name: ggraph
  role: Rendering the pathway network view of clustered enriched terms from the pareg
    fit
- name: enrichplot
  role: Visualization of the pareg fit after conversion via as_enrichplot_object
  repo: https://github.com/YuLab-SMU/enrichplot
- name: Snakemake
  role: Workflow engine for reproducible benchmark and analysis pipelines in inst/scripts/
  repo: https://github.com/cbg-ethz/pareg
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad522
    title: Coherent pathway enrichment estimation by modeling inter-pathway dependencies
      using regularized regression
  source_content_hash: sha256:fde1e04243d6c9aaa595026416d0501a8873bdf5836345c2930966cd80cd9d7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/pareg
  tools:
  - R
  - Snakemake
  - enrichplot
  - ggraph
  - pareg
  - tidyverse
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# pathway-enrichment-estimation

## Summary

Estimate pathway enrichment scores by regressing gene-level differential-expression p-values with pareg, a regularized generalized linear model that directly incorporates inter-pathway dependencies (e.g., Jaccard similarity) via network regularization.

## When to use

Use when a gene-level p-value table (gene, pvalue) and a pathway database (e.g., GO BP terms of size 50-500) must be converted into coherent pathway-level enrichment scores that account for redundant/overlapping gene sets.

## When NOT to use

- The input is already a pathway-level enrichment result and only visualization is needed (use enrichplot directly rather than re-fitting pareg).
- The pathway database has negligible overlap/redundancy between terms, so network regularization on inter-pathway dependencies adds no modeling benefit.
- No gene-to-pathway mapping or similarity matrix is available, leaving no term_network for the regularized regression.

## Inputs

- Gene-level differential-expression p-value table with columns gene, pvalue (e.g., TCGA-BRCA)
- Pathway/term database (e.g., GO Biological Process filtered to terms of size 50-500) as a term-to-gene mapping
- Pathway similarity matrix (Jaccard or other measure), pre-computed via compute_term_similarities or loaded from data(pathway_similarities)

## Outputs

- pareg fit object with per-pathway enrichment scores, exportable to a dataframe ranked by desc(abs(enrichment))
- Pathway network view (ggraph) of top non-isolated terms at min_similarity = 0.1
- enrichplot-compatible object via as_enrichplot_object(fit)
- Benchmark summary table of median precision at fixed recall (e.g., pareg ~94%, FET ~12%, topGO ~5% at recall 0.80, noise eta=0.25)

## How to apply

Load the gene-level p-value table and the pathway term database, then compute pairwise pathway similarities with pareg's compute_term_similarities helper using the Jaccard similarity function (or load pre-computed matrices via data(pathway_similarities, package = "pareg")). Fit the pareg regularized generalized linear model with network_param = 1 and term_network set to the Jaccard similarity matrix, so the network regularization term (classifying the method as a modular enrichment analysis tool) penalizes incoherent enrichment across similar pathways. Export the fit to a dataframe and rank terms by desc(abs(enrichment)) for reporting or for benchmarking against methods like MGSA, FET, topGO, and blitzGSEA. For visualization, select the top non-isolated terms and render the pathway network view with plot(fit, min_similarity = 0.1) using ggraph, or convert with as_enrichplot_object for the enrichplot/clusterProfiler ecosystem. For synthetic benchmarks, draw DE p-values from Beta(0.1, 1), background p-values from Uniform(0, 1), select active pathways with similarity_sample, and interpolate precision at fixed recall (e.g., 0.80) from precision-recall curves.

## Related tools

- **pareg** (Core method: regularized generalized linear model estimating pathway enrichment scores with network regularization on a pathway similarity matrix) — https://github.com/cbg-ethz/pareg
- **R** (Implementation environment; pareg is distributed as an R/Bioconductor package) — https://bioconductor.org/packages/release/bioc/html/pareg.html
- **tidyverse** (Data wrangling and ranking of enrichment results (arrange by desc(abs(enrichment))))
- **ggraph** (Rendering the pathway network view of clustered enriched terms from the pareg fit)
- **enrichplot** (Visualization of the pareg fit after conversion via as_enrichplot_object) — https://github.com/YuLab-SMU/enrichplot
- **Snakemake** (Workflow engine for reproducible benchmark and analysis pipelines in inst/scripts/) — https://github.com/cbg-ethz/pareg

## Examples

```
mat_similarities <- compute_term_similarities(df_terms, similarity_function = jaccard); fit <- pareg(df_pvalues, network_param = 1, term_network = mat_similarities)
```

## Evaluation signals

- Ranked top-25 terms from the TCGA-BRCA fit must contain axoneme assembly, retrograde vesicle-mediated transport, and synaptic vesicle recycling among leading hits (matching Fig 2a).
- The pathway network view (top-50 non-isolated terms, min_similarity = 0.1) must show clustered pathway groups consistent with reported TCGA-BRCA findings (Fig 2b).
- In the synthetic benchmark at noise eta=0.25, rho=0.5, interpolated median precision at recall 0.80 must reproduce ~94% for pareg vs ~37% MGSA, ~12% FET, ~5% topGO within replicate variability.
- The term similarity matrix must be symmetric, non-negative (Jaccard in [0,1]), and cover all terms in the fitted pathway database.
- Enrichment dataframe must contain an enrichment score column for every term in the input database with no missing values after fit export.

## Limitations

- Pathway databases (KEGG, Reactome, GO) are large and redundant, which complicates statistical enrichment computation; the network regularization is the remedy but requires a meaningful similarity measure, and different similarity measures can differ substantially (observable on GO Biological Process).
- Benchmark performance depends on the synthetic construction: DE p-values from Beta(0.1, 1), background from Uniform(0, 1), and true positives defined by similarity_sample; results are subject to replicate variability.
- pareg's pre-computed pathway_similarities are only available for selected pathway databases.

## Evidence

- [abstract] pareg is a novel gene set enrichment method based on a regularized generalized linear model that incorporates dependencies between gene sets (e.g. shared genes) directly in the enrichment computation.: "develop a novel gene set enrichment method, called pareg, which is based on a regularized generalized linear model and directly incorporates dependencies between gene sets"
- [methods] The network regularization is based on a pathway similarity matrix, making pareg a modular enrichment analysis tool.: "The network regularization term is based on a pathway similarity matrix (e.g., defined by Jaccard similarity) and thus classifies this method as a modular enrichment analysis tool"
- [methods] compute_term_similarities with jaccard builds the pathway similarity matrix used as term_network.: "compute_term_similarities(
  df_terms,
  similarity_function = jaccard
)"
- [intro] pareg ships pre-computed similarity matrices for selected pathway databases.: "pareg provides various pre-computed similarity measures (`r names(pathway_similarities[[1]])`) for selected pathway databases (`r names(pathway_similarities)`) in matrix form."
- [readme] pareg is available as an R Bioconductor package.: "Pathway enrichment computations using a regularized regression approach to incorporate inter-pathway relations in the statistical model."
