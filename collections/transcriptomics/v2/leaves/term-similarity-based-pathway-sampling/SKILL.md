---
name: term-similarity-based-pathway-sampling
description: Use when generating synthetic benchmark data for enrichment methods where
  ground-truth active pathways must reflect realistic inter-pathway redundancy (e.g.
  shared genes), so performance can be evaluated as a function of pathway-similarity
  structure.
when_to_use_negative:
- When the goal is enrichment analysis on real experimental p-values rather than constructing
  synthetic benchmarks with known positives
- When pathways in the database are assumed independent and similarity structure should
  not influence the ground truth (rho effectively 0 with uniform sampling suffices)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: pareg
  role: Provides compute_term_similarities (Jaccard similarity matrix construction)
    and similarity_sample (similarity-aware sampling of active pathways) used to build
    the synthetic benchmark
  repo: https://github.com/cbg-ethz/pareg
- name: R
  role: Implementation language in which the pareg helpers and p-value sampling (Beta/Uniform)
    are executed
  repo: https://bioconductor.org/packages/release/bioc/html/pareg.html
- name: Snakemake
  role: Orchestrates the benchmark runs over the (eta, rho) grid from the workflows
    in the pareg repository's inst/scripts directory
  repo: https://github.com/cbg-ethz/pareg
- name: tidyverse
  role: Used for data manipulation and aggregation of benchmark results (e.g. writing
    benchmark_pr_auc.csv)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad522
    title: Coherent pathway enrichment estimation by modeling inter-pathway dependencies
      using regularized regression
  source_content_hash: sha256:b7e0abd0092b908b2aa7dc38fef0116649243d1ebd845558af138a5bc70784bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/pareg
  tools:
  - R
  - Snakemake
  - pareg
  - tidyverse
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# term-similarity-based-pathway-sampling

## Summary

A simulation technique from the pareg workflow that builds a synthetic pathway database with pronounced pathway clustering, computes a pairwise Jaccard term-similarity matrix, and then uses similarity-aware sampling of active pathways so that ground-truth positive gene sets are correlated rather than drawn independently.

## When to use

Use when generating synthetic benchmark data for enrichment methods where ground-truth active pathways must reflect realistic inter-pathway redundancy (e.g. shared genes), so performance can be evaluated as a function of pathway-similarity structure.

## When NOT to use

- When the goal is enrichment analysis on real experimental p-values rather than constructing synthetic benchmarks with known positives
- When pathways in the database are assumed independent and similarity structure should not influence the ground truth (rho effectively 0 with uniform sampling suffices)

## Inputs

- Synthetic pathway database (df_terms: term-to-gene mapping) with clustered pathways
- Pairwise term-similarity matrix computed via compute_term_similarities with the jaccard function
- Similarity factor rho controlling how correlated the sampled active pathways are
- Noise level eta and p-value distributions (Beta(0.1, 1) for DE genes, Uniform for background)

## Outputs

- Set of active (true-positive) pathway identifiers sampled from the similarity matrix
- Simulated differential expression p-value table with DE genes defined as the union of active pathway genes
- Ground-truth positive labels used to compute per-method precision-recall AUC in the benchmark grid

## How to apply

First construct a synthetic pathway database with a pronounced clustering of pathways, then compute the full pairwise term-similarity matrix with pareg's helper `compute_term_similarities(df_terms, similarity_function = jaccard)`. Choose a similarity factor rho (e.g. 0, 0.5, 1) and draw the active (true-positive) pathways from the similarity matrix via `similarity_sample`, which selects correlated sets of terms rather than uniform random picks. Define the differentially expressed genes as the union of the genes of the sampled active pathways, assign them p-values from a Beta(0.1, 1) distribution centered at 0, and background genes p-values from a Uniform distribution, optionally injecting noise level eta. The sampled active terms serve as ground-truth positives for computing precision-recall AUC of each benchmarked method across the (eta, rho) grid.

## Related tools

- **pareg** (Provides compute_term_similarities (Jaccard similarity matrix construction) and similarity_sample (similarity-aware sampling of active pathways) used to build the synthetic benchmark) — https://github.com/cbg-ethz/pareg
- **R** (Implementation language in which the pareg helpers and p-value sampling (Beta/Uniform) are executed) — https://bioconductor.org/packages/release/bioc/html/pareg.html
- **Snakemake** (Orchestrates the benchmark runs over the (eta, rho) grid from the workflows in the pareg repository's inst/scripts directory) — https://github.com/cbg-ethz/pareg
- **tidyverse** (Used for data manipulation and aggregation of benchmark results (e.g. writing benchmark_pr_auc.csv))

## Examples

```
mat_similarities <- compute_term_similarities(df_terms, similarity_function = jaccard); active_terms <- similarity_sample(mat_similarities, 5)
```

## Evaluation signals

- The similarity matrix is symmetric with values in [0, 1] and diagonal of 1 when built with the jaccard similarity_function
- The number of sampled active terms matches the requested count (e.g. similarity_sample(mat_similarities, 5) returns exactly 5 pathways)
- Every differentially expressed gene in the simulated study is a member of at least one sampled active pathway (union definition)
- DE-gene p-values follow the Beta(0.1, 1) shape (mass near 0) while background p-values are Uniform
- Varying the similarity factor rho changes the overlap/coherence of sampled active pathway sets, verifiable by recomputing Jaccard similarity among the selected terms

## Limitations

- Simulated p-values from Beta(0.1, 1) and Uniform distributions are an idealization; real differential expression signal may deviate, affecting transferability of benchmark conclusions
- Results depend on the synthetic database being generated 'with a pronounced clustering of pathways'; a poorly clustered synthetic database will not exercise the similarity-dependent behavior pareg is designed for
- Ground truth is only as good as the sampled active set; the method benchmarks performance relative to this synthetic definition of true positives, not biological truth

## Evidence

- [methods] synthetic db construction: "we generate a synthetic pathway database with a pronounced clustering of pathways"
- [methods] similarity computation: "compute_term_similarities(
  df_terms,
  similarity_function = jaccard
)"
- [methods] active pathway sampling: "active_terms <- similarity_sample(mat_similarities, 5)"
- [methods] true positive definition: "we select a subset of pathways to be activated. In a performance evaluation, these would be considered to be true positives."
- [methods] p-value simulation: "The p-values of genes considered to be differentially expressed are sampled from a Beta distribution centered at $0$. The p-values for all other genes are drawn from a Uniform distribution."
- [intro] precomputed similarities: [evidence span withheld — anchor status: not_found]
