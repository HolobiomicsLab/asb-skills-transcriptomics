---
name: wall-clock-runtime-profiling
description: Use when comparing the computational cost of pareg versus standard enrichment
  methods, or when deciding whether pareg's ~30-minute parallelized runtime is acceptable
  for a given enrichment analysis.
when_to_use_negative:
- The goal is enrichment performance (precision/recall or robustness to noise) rather
  than computational cost — use the performance-evaluation benchmark instead.
- The input is a real study (e.g. TCGA breast cancer differential expression) where
  runtime on synthetic data does not represent the production workload scale.
- Only a single method is being run and no cross-method runtime comparison is required.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: pareg
  role: Enrichment method whose wall-clock runtime (up to ~30 min parallelized, network_param
    = 1, term_network = mat_similarities) is the primary measurement target
  repo: https://github.com/cbg-ethz/pareg
- name: R
  role: Language used to run pareg, MGSA, FET, topGO, blitzGSEA and record timing;
    pareg and Snakemake workflows are distributed as R/Bioconductor assets
  repo: https://github.com/cbg-ethz/pareg
- name: tidyverse
  role: R toolkit used to aggregate the recorded runtimes into the summary table runtime_benchmark_results.csv
- name: Snakemake
  role: Workflow engine used to orchestrate reproducible benchmark runs in the pareg
    repository
  repo: https://github.com/cbg-ethz/pareg
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad522
    title: Coherent pathway enrichment estimation by modeling inter-pathway dependencies
      using regularized regression
  source_content_hash: sha256:5cd653bd16100124399c4e666fbd68a1cb020c1c5991ad5a04fadc21e25bac17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/pareg
  tools:
  - R
  - Snakemake
  - pareg
  - tidyverse
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# wall-clock-runtime-profiling

## Summary

Benchmarks the wall-clock runtime of pareg against competing enrichment methods (MGSA, FET, topGO, blitzGSEA) on a fixed synthetic study to quantify the computational cost of network-regularized regression enrichment.

## When to use

Use when comparing the computational cost of pareg versus standard enrichment methods, or when deciding whether pareg's ~30-minute parallelized runtime is acceptable for a given enrichment analysis.

## When NOT to use

- The goal is enrichment performance (precision/recall or robustness to noise) rather than computational cost — use the performance-evaluation benchmark instead.
- The input is a real study (e.g. TCGA breast cancer differential expression) where runtime on synthetic data does not represent the production workload scale.
- Only a single method is being run and no cross-method runtime comparison is required.

## Inputs

- Synthetic pathway-term dataframe (df_terms: 2 pathway groups x 10 pathways, 15 genes per group plus 50 background genes)
- Synthetic study dataframe (df_study: Beta(0.1,1) p-values for active-pathway genes, Uniform p-values for the rest), generated with set.seed(42)
- Jaccard-based pathway similarity matrix (mat_similarities) computed via compute_term_similarities

## Outputs

- Wall-clock runtime measurements for pareg, MGSA, FET, topGO, and blitzGSEA on the synthetic study
- Aggregated summary table saved as runtime_benchmark_results.csv
- Comparison against Supplementary Fig S2 runtime ranges

## How to apply

Generate a fixed synthetic benchmark using set.seed(42): a df_terms pathway database with 2 pathway groups x 10 pathways (15 genes per group plus 50 background genes) and a df_study with Beta(0.1,1) p-values for genes in the 5 active pathways and Uniform p-values elsewhere. Run pareg on df_study with the Jaccard-based term similarity matrix (network_param = 1, term_network = mat_similarities), recording wall-clock time, then run MGSA, the Fisher exact test, topGO, and blitzGSEA on the same df_study, timing each. Aggregate all runtimes into a summary table (runtime_benchmark_results.csv) and compare against the paper's Supplementary Fig S2 envelope. Rationale: the article reports that all competing methods complete in under 2 minutes while pareg takes up to ~30 minutes even when parallelized, so the benchmark validates that the observed cost overhead is inherent to the network-regularized GLM rather than an artifact of the setup.

## Related tools

- **pareg** (Enrichment method whose wall-clock runtime (up to ~30 min parallelized, network_param = 1, term_network = mat_similarities) is the primary measurement target) — https://github.com/cbg-ethz/pareg
- **R** (Language used to run pareg, MGSA, FET, topGO, blitzGSEA and record timing; pareg and Snakemake workflows are distributed as R/Bioconductor assets) — https://github.com/cbg-ethz/pareg
- **tidyverse** (R toolkit used to aggregate the recorded runtimes into the summary table runtime_benchmark_results.csv)
- **Snakemake** (Workflow engine used to orchestrate reproducible benchmark runs in the pareg repository) — https://github.com/cbg-ethz/pareg

## Examples

```
set.seed(42); df_terms <- create_synthetic_pathway_database(); df_study <- create_synthetic_study(df_terms, active_terms = similarity_sample(compute_term_similarities(df_terms, similarity_function = jaccard), 5)); system.time(pareg(df_study, df_terms, network_param = 1, term_network = mat_similarities))
```

## Evaluation signals

- runtime_benchmark_results.csv exists and contains wall-clock times for all five methods (pareg, MGSA, FET, topGO, blitzGSEA).
- All competing methods (MGSA, FET, topGO, blitzGSEA) complete in under 2 minutes on the synthetic study.
- pareg's recorded runtime falls within the reported up-to-~30-minute parallelized envelope from Supplementary Fig S2.
- The synthetic data generation is reproducible: same set.seed(42), 2 pathway groups x 10 pathways, 15 genes per group plus 50 background genes, 5 active pathways.
- pareg is run with network_param = 1 and term_network = mat_similarities, matching the configuration whose runtime is benchmarked in the paper.

## Limitations

- pareg is the slowest method: all competing enrichment methods run in less than 2 minutes, while pareg takes up to ~30 minutes even in parallelized mode.
- Runtime is measured on a small synthetic database (20 pathways, 2 groups); scaling to real pathway databases (KEGG, Reactome, GO) with many redundancies can be much larger.
- Benchmarked runtime depends on parallelization mode; the ~30-minute figure applies to the parallelized setting, so single-threaded runs may be slower.

## Evidence

- [supplementary] pareg is the slowest method: all competing enrichment methods run in less than 2 minutes, while pareg takes up to approximately 30 minutes when run in parallelized mode.: [evidence span withheld — anchor status: not_found]
- [methods] compute_term_similarities with jaccard: "compute_term_similarities(
  df_terms,
  similarity_function = jaccard
)"
- [abstract] pareg uses regularized generalized linear model with dependencies between gene sets: "develop a novel gene set enrichment method, called pareg, which is based on a regularized generalized linear model and directly incorporates dependencies between gene sets"
- [abstract] The GitHub repository contains Snakemake workflows to reproduce all results: "The GitHub repository also contains the Snakemake workflows needed to reproduce all results presented here."
- [methods] Beta and Uniform p-value sampling for synthetic study: "The p-values of genes considered to be differentially expressed are sampled from a Beta distribution centered at $0$. The p-values for all other genes are drawn from a Uniform distribution."
