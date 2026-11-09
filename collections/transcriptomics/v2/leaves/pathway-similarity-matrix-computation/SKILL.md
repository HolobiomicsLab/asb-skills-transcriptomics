---
name: pathway-similarity-matrix-computation
description: Use when a regularized-regression enrichment method (pareg) needs a pathway
  similarity matrix for its network term, or when benchmarking whether an alternative
  similarity measure matches a Jaccard-based baseline.
when_to_use_negative:
- When no inter-pathway dependencies are relevant and a plain over-representation
  analysis (FET/topGO-style) suffices without network regularization
- When the goal is enrichment testing on real differential-expression data without
  a benchmarking component and the default Jaccard matrix is already adequate
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0089
tools:
- name: pareg
  role: Provides compute_term_similarities(), jaccard, similarity_sample, and pre-computed
    pathway_similarities matrices; its network regularization term consumes the similarity
    matrix
  repo: https://github.com/cbg-ethz/pareg
- name: R
  role: Implementation environment (pareg is an R/Bioconductor package)
  repo: https://bioconductor.org/packages/release/bioc/html/pareg.html
- name: tidyverse
  role: Data handling for building similarity comparison tables and plots
- name: ggpairs
  role: Pairwise scatter plots to compare how similarity measures differ across GO
    Biological Process
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad522
    title: Coherent pathway enrichment estimation by modeling inter-pathway dependencies
      using regularized regression
  source_content_hash: sha256:91e0f5b0fc8c549b8af917a1eb16fb81127dcbba14818c8c7dd24d8818054dfa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/pareg
  tools:
  - R
  - ggpairs
  - pareg
  - tidyverse
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# pathway-similarity-matrix-computation

## Summary

Compute and use pairwise pathway (term) similarity matrices — e.g. via pareg's compute_term_similarities() helper with the Jaccard index or an alternative measure like the overlap coefficient — to quantify inter-pathway dependencies for network-regularized enrichment analysis and to swap similarity measures in benchmark comparisons.

## When to use

Use when a regularized-regression enrichment method (pareg) needs a pathway similarity matrix for its network term, or when benchmarking whether an alternative similarity measure matches a Jaccard-based baseline.

## When NOT to use

- When no inter-pathway dependencies are relevant and a plain over-representation analysis (FET/topGO-style) suffices without network regularization
- When the goal is enrichment testing on real differential-expression data without a benchmarking component and the default Jaccard matrix is already adequate

## Inputs

- Pathway database as a term-gene mapping (df_terms), e.g. from GO Biological Process or a synthetic database with two 15-gene gene groups and 50 background genes
- Similarity function choice (e.g. jaccard or overlap coefficient) passed to compute_term_similarities()
- Optional: pre-computed similarity matrices from data(pathway_similarities, package = 'pareg')

## Outputs

- Pairwise pathway similarity matrix (mat_similarities) usable as pareg's term_network argument
- Similarity heatmaps (Heatmap with colorRamp2(c(0,1), c('white','black'))) and pairwise comparison plots (ggpairs)
- Benchmark table (similarity_benchmark.csv) with columns similarity measure, PR-AUC, precision-at-80%-recall

## How to apply

From a pathway database (df_terms mapping terms to gene sets), call pareg's compute_term_similarities() helper with similarity_function = jaccard to build the baseline matrix, or a non-Jaccard measure (e.g. the overlap coefficient) to test robustness. Pre-computed similarity matrices are also available via data(pathway_similarities, package = 'pareg') for selected pathway databases. Use mat_similarities both to select active terms in synthetic studies with similarity_sample(mat_similarities, 5) and as the term_network argument when running pareg (network_param = 1), which drives the network regularization that makes pareg a modular enrichment analysis tool. Because similarity measures can differ substantially even on the same ontology — visible on GO Biological Process with heatmap and ggpairs comparisons — always benchmark the alternative against the Jaccard baseline using PR-AUC and precision-at-80%-recall (Jaccard baseline: ~94% precision at 80% recall at noise level η=0.25).

## Related tools

- **pareg** (Provides compute_term_similarities(), jaccard, similarity_sample, and pre-computed pathway_similarities matrices; its network regularization term consumes the similarity matrix) — https://github.com/cbg-ethz/pareg
- **R** (Implementation environment (pareg is an R/Bioconductor package)) — https://bioconductor.org/packages/release/bioc/html/pareg.html
- **tidyverse** (Data handling for building similarity comparison tables and plots)
- **ggpairs** (Pairwise scatter plots to compare how similarity measures differ across GO Biological Process)

## Examples

```
mat_similarities <- compute_term_similarities(df_terms, similarity_function = jaccard); active_terms <- similarity_sample(mat_similarities, 5)
```

## Evaluation signals

- Both similarity variants (alternative and Jaccard) must produce ranked enrichment scores evaluable against known true-positive active terms
- Comparison table reports PR-AUC and precision-at-80%-recall for the alternative measure side-by-side with the Jaccard baseline (median precision 94% at 80% recall for η=0.25)
- Similarity matrix is symmetric with entries in [0, 1] (heatmap scale colorRamp2(c(0, 1))) and dims equal the number of terms
- similarity_sample(mat_similarities, 5) selects 5 active terms whose p-values are drawn from Beta(0.1, 1), all others from Uniform(0, 1)
- pareg fit with network_param = 1 succeeds with term_network set to either similarity matrix

## Limitations

- Pathway similarity measures can differ substantially from each other even on the same ontology (GO Biological Process), so results are measure-dependent
- Synthetic-benchmark conclusions are tied to the vignette design (two 15-gene groups, 10 pathways per group, 50 background genes) and specific noise level η=0.25; other designs may behave differently
- The Jaccard-based baseline to match or exceed is pareg's reported 94% median precision at 80% recall; an alternative measure failing to reach it does not invalidate the measure in general, only in this benchmark

## Evidence

- [methods] compute_term_similarities helper: "we compute pairwise pathway similarities with `pareg`'s helper function"
- [methods] compute_term_similarities signature: "compute_term_similarities(
  df_terms,
  similarity_function = jaccard
)"
- [methods] similarity_sample usage: "active_terms <- similarity_sample(mat_similarities, 5)"
- [intro] measures differ: "On the Gene Ontology's Biological Process subcategory, we can observe how much pathway similarity measures can differ from each other."
- [methods] modular classification: "The network regularization term is based on a pathway similarity matrix (e.g., defined by Jaccard similarity) and thus classifies this method as a modular enrichment analysis tool"
- [intro] pre-computed matrices: "pareg provides various pre-computed similarity measures (`r names(pathway_similarities[[1]])`) for selected pathway databases (`r names(pathway_similarities)`) in matrix form."
- [readme] installation: "Install the latest stable version from Bioconductor:
```r
BiocManager::install("pareg")
```"
