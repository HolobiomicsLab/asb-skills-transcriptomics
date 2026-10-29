---
name: pareg-regularized-regression-enrichment
description: Use when a study has per-gene differential expression p-values and a
  pathway database whose redundant, overlapping gene sets require coherent (inter-pathway-aware)
  enrichment scores rather than independent per-pathway tests.
when_to_use_negative:
- 'When wall-clock time is the binding constraint: pareg takes up to ~30 minutes parallelized
  versus <2 minutes for MGSA, FET, topGO and blitzGSEA on the same benchmark'
- When the pathway database is small and non-redundant, so inter-pathway dependency
  modeling offers little benefit over standard over-representation analysis
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: pareg
  role: 'Core enrichment method: regularized generalized linear model with network
    regularization from a pathway similarity matrix; also supplies compute_term_similarities,
    jaccard, similarity_sample, and as_enrichplot_object helpers'
  repo: https://github.com/cbg-ethz/pareg
- name: R
  role: Runtime environment; pareg is distributed as an R/Bioconductor package
  repo: https://bioconductor.org/packages/release/bioc/html/pareg.html
- name: tidyverse
  role: Data wrangling and plotting of enrichment results in the vignette workflow
- name: enrichplot
  role: Visualization of pareg results after conversion with as_enrichplot_object
  repo: https://github.com/YuLab-SMU/enrichplot
- name: Snakemake
  role: Workflows shipped in the GitHub repository to reproduce all results presented
    in the paper
  repo: https://github.com/cbg-ethz/pareg
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad522
    title: Coherent pathway enrichment estimation by modeling inter-pathway dependencies
      using regularized regression
  source_content_hash: sha256:e651bb1c2b274a3f40a0982c11c25071d1dcfe29fb895ef3d2e5c70a11d124e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/pareg
  tools:
  - R
  - Snakemake
  - enrichplot
  - pareg
  - tidyverse
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pareg-regularized-regression-enrichment

## Summary

Compute gene set enrichment with pareg, an R/Bioconductor method that regresses differential expression p-values and incorporates dependencies between pathways (e.g. shared genes) through a similarity-matrix-based network regularization term.

## When to use

Use when a study has per-gene differential expression p-values and a pathway database whose redundant, overlapping gene sets require coherent (inter-pathway-aware) enrichment scores rather than independent per-pathway tests.

## When NOT to use

- When wall-clock time is the binding constraint: pareg takes up to ~30 minutes parallelized versus <2 minutes for MGSA, FET, topGO and blitzGSEA on the same benchmark
- When the pathway database is small and non-redundant, so inter-pathway dependency modeling offers little benefit over standard over-representation analysis

## Inputs

- Dataframe of differential expression p-values per gene (df_study; Beta-distributed for active genes, Uniform for background in the synthetic benchmark)
- Pathway-term dataframe mapping genes to terms (df_terms)
- Pairwise pathway similarity matrix (e.g. Jaccard-based, from compute_term_similarities or the pre-computed pathway_similarities data object)

## Outputs

- Per-pathway enrichment scores (pareg fit object)
- enrichplot-compatible object via as_enrichplot_object for network visualization
- Runtime summary table (e.g. runtime_benchmark_results.csv) when benchmarking against MGSA, FET, topGO, blitzGSEA

## How to apply

Install pareg via BiocManager (or the GitHub dev version), then build a pathway-term dataframe (genes per term) and a term-similarity matrix with pareg's compute_term_similarities(..., similarity_function = jaccard). Fit the model on the study's p-value table using a network regularization term (e.g. term_network = mat_similarities with network_param = 1), which makes pareg a modular enrichment analysis tool because the regularization is based on pathway similarity. Convert results with as_enrichplot_object for enrichplot/ggraph visualization in a pathway network view, and optionally run the synthetic-data protocol (2 pathway groups x 10 pathways, 15 genes per group plus 50 background genes; Beta(0.1,1) p-values for 5 active pathways sampled via similarity_sample, Uniform p-values for the rest, set.seed(42)) to benchmark behavior. Be aware of runtime: pareg can take up to ~30 minutes even in parallelized mode, while competing methods (MGSA, FET, topGO, blitzGSEA) finish in under 2 minutes.

## Related tools

- **pareg** (Core enrichment method: regularized generalized linear model with network regularization from a pathway similarity matrix; also supplies compute_term_similarities, jaccard, similarity_sample, and as_enrichplot_object helpers) — https://github.com/cbg-ethz/pareg
- **R** (Runtime environment; pareg is distributed as an R/Bioconductor package) — https://bioconductor.org/packages/release/bioc/html/pareg.html
- **tidyverse** (Data wrangling and plotting of enrichment results in the vignette workflow)
- **enrichplot** (Visualization of pareg results after conversion with as_enrichplot_object) — https://github.com/YuLab-SMU/enrichplot
- **Snakemake** (Workflows shipped in the GitHub repository to reproduce all results presented in the paper) — https://github.com/cbg-ethz/pareg

## Examples

```
BiocManager::install("pareg"); mat_similarities <- compute_term_similarities(df_terms, similarity_function = jaccard); fit <- pareg(df_study, df_terms, network_param = 1, term_network = mat_similarities)
```

## Evaluation signals

- The enrichment fit object is created and convertible: as_enrichplot_object(fit) returns a valid enrichplot object without error
- On the synthetic benchmark (set.seed(42); Beta(0.1,1) p-values for 5 active pathways, Uniform elsewhere), enriched scores concentrate on the sampled active terms selected via similarity_sample(mat_similarities, 5)
- In runtime benchmarks, competing methods (MGSA, FET, topGO, blitzGSEA) complete in under 2 minutes while pareg's runtime falls within the reported up-to-~30-minute parallelized envelope
- The Jaccard similarity matrix is symmetric with diagonal 1 and matches the pathway-term dataframe's term set
- Results can be rendered in a pathway network view using ggraph/enrichplot

## Limitations

- pareg is the slowest method in the runtime comparison: it takes up to approximately 30 minutes in parallelized mode, versus <2 minutes for all competing enrichment methods
- Similarity measures can differ substantially between choices (observed on GO Biological Process), so the enrichment output depends on the chosen term similarity matrix
- As a regularized regression with network regularization, results depend on the network regularization parameter (network_param, e.g. 1) and the supplied term_network

## Evidence

- [abstract] novel gene set enrichment method based on a regularized generalized linear model: "develop a novel gene set enrichment method, called pareg, which is based on a regularized generalized linear model"
- [methods] network regularization makes it a modular enrichment analysis tool: "The network regularization term is based on a pathway similarity matrix (e.g., defined by Jaccard similarity) and thus classifies this method as a modular enrichment analysis tool"
- [methods] computing term similarities with the jaccard helper: "compute_term_similarities(
  df_terms,
  similarity_function = jaccard
)"
- [supplementary] runtime benchmark finding against competing methods: [evidence span withheld — anchor status: not_found]
- [readme] availability as R/Bioconductor package: "BiocManager::install("pareg")"
