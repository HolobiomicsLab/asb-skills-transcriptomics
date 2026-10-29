---
name: simulation-noise-parameter-tuning
description: Use when benchmarking enrichment methods (pareg, MGSA, FET, topGO, blitzGSEA)
  on simulated differential-expression studies where robustness to noise must be quantified
  at a fixed recall target.
when_to_use_negative:
- The goal is enriching a real experimental study rather than measuring a method's
  robustness to synthetic noise
- Ground-truth active pathways are unavailable — precision-recall interpolation at
  fixed recall requires known true positives
- The pathway database lacks enough redundancy/clustering for similarity_sample to
  define a meaningful activation set
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: pareg
  role: Provides compute_term_similarities, similarity_sample and jaccard helpers
    to build the similarity matrix and sample active pathways, plus the regularized-regression
    enrichment method being benchmarked
  repo: https://github.com/cbg-ethz/pareg
- name: Snakemake
  role: Runs the benchmark workflows that reproduce the noise-level sweeps and precision-recall
    evaluation
  repo: https://github.com/cbg-ethz/pareg
- name: tidyverse
  role: R data manipulation for constructing precision-recall curves and computing
    medians across replicates
- name: enrichplot
  role: Visualization of enrichment results via as_enrichplot_object
  repo: https://github.com/YuLab-SMU/enrichplot
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad522
    title: Coherent pathway enrichment estimation by modeling inter-pathway dependencies
      using regularized regression
  source_content_hash: sha256:40ba54ef96f286222cc02d18df9b8af7c916cdfbe2b5bbf0ce3ead95dee1388c
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# simulation-noise-parameter-tuning

## Summary

A synthetic-benchmark design for evaluating pathway enrichment methods under controlled noise: activate pathways via similarity sampling, inject noise by setting η (e.g. η=0.25) and ρ (ρ=0.5), then score methods on precision-recall curves to obtain median precision at fixed recall (80%).

## When to use

Use when benchmarking enrichment methods (pareg, MGSA, FET, topGO, blitzGSEA) on simulated differential-expression studies where robustness to noise must be quantified at a fixed recall target.

## When NOT to use

- The goal is enriching a real experimental study rather than measuring a method's robustness to synthetic noise
- Ground-truth active pathways are unavailable — precision-recall interpolation at fixed recall requires known true positives
- The pathway database lacks enough redundancy/clustering for similarity_sample to define a meaningful activation set

## Inputs

- Synthetic pathway database with pronounced pathway clustering
- Simulated differential-expression p-values (DE genes from Beta(0.1, 1), background from Uniform(0, 1))
- Pairwise pathway similarity matrix (e.g. Jaccard) computed with compute_term_similarities
- Noise-level parameters η and ρ (e.g. η=0.25, ρ=0.5)
- Set of activated pathways from similarity_sample used as ground-truth positives

## Outputs

- Pathway-level enrichment scores per method per benchmark replicate
- Precision-recall curves per method
- Median precision at recall = 0.80 per method under each noise level
- Summary table of median precision-at-80%-recall values comparable to Fig 1b/1d

## How to apply

Generate a synthetic pathway database with a pronounced clustering of pathways, then sample DE gene p-values from Beta(0.1, 1) and background gene p-values from Uniform(0, 1). Select active pathways as true positives using pareg's similarity_sample helper on the Jaccard pathway-similarity matrix (e.g. similarity_sample(mat_similarities, 5)). Run each enrichment method at the chosen noise levels (η=0.25, ρ=0.5) to obtain pathway-level enrichment scores, construct per-method precision-recall curves, and interpolate precision at recall = 0.80. Take the median precision across benchmark replicates per method to obtain a noise-robustness summary (reference values: pareg 94%, MGSA 37%, FET 12%, topGO 5%).

## Related tools

- **pareg** (Provides compute_term_similarities, similarity_sample and jaccard helpers to build the similarity matrix and sample active pathways, plus the regularized-regression enrichment method being benchmarked) — https://github.com/cbg-ethz/pareg
- **Snakemake** (Runs the benchmark workflows that reproduce the noise-level sweeps and precision-recall evaluation) — https://github.com/cbg-ethz/pareg
- **tidyverse** (R data manipulation for constructing precision-recall curves and computing medians across replicates)
- **enrichplot** (Visualization of enrichment results via as_enrichplot_object) — https://github.com/YuLab-SMU/enrichplot

## Examples

```
active_terms <- similarity_sample(mat_similarities, 5)
```

## Evaluation signals

- Computed median precision at recall = 0.8 reproduces reported values within replicate variability: pareg ≈ 94%, MGSA ≈ 37%, FET ≈ 12%, topGO ≈ 5% at η=0.25
- Precision-recall curves are constructed per method with pathways activated by similarity_sample treated as true positives
- DE p-values follow Beta(0.1, 1) and background p-values follow Uniform(0, 1) as specified
- Benchmark is run at the defined noise levels (η=0.25, ρ=0.5) across multiple replicates before taking medians

## Limitations

- Evaluation depends on the synthetic pathway database design; similarity_sample picks a subset of pathways to activate, so results reflect clustered-database scenarios
- Median precision-at-recall is specific to the chosen recall target (0.80) and noise levels; other settings may reorder methods
- Replicate variability sets the tolerance for validating against reported values

## Evidence

- [results] At a fixed recall of 80% under noise level η=0.25 in the synthetic benchmark, pareg achieves a median precision of 94%, MGSA 37%, FET 12%, and topGO 5%.: [evidence span withheld — anchor status: not_found]
- [methods] The p-values of genes considered to be differentially expressed are sampled from a Beta distribution centered at $0$. The p-values for all other genes are drawn from a Uniform distribution.: "The p-values of genes considered to be differentially expressed are sampled from a Beta distribution centered at $0$."
- [methods] we select a subset of pathways to be activated. In a performance evaluation, these would be considered to be true positives.: "we select a subset of pathways to be activated. In a performance evaluation, these would be considered to be true positives."
- [abstract] The GitHub repository also contains the Snakemake workflows needed to reproduce all results presented here.: "The GitHub repository also contains the Snakemake workflows needed to reproduce all results presented here."
- [readme] Pathway enrichment computations using a regularized regression approach to incorporate inter-pathway relations in the statistical model.: "Pathway enrichment computations using a regularized regression approach to incorporate inter-pathway relations in the statistical model."
