---
name: precision-recall-auc-benchmarking
description: Use when you need to quantitatively compare competing gene set enrichment
  methods on synthetic studies where the true positive pathways are known by construction,
  across a noise/similarity grid.
when_to_use_negative:
- Real experimental studies with no ground truth for which pathways are truly active
  — PR-AUC against simulated truth is only meaningful on synthetic data where positives
  are sampled by construction
- Evaluating a single enrichment method in isolation without competitor baselines
  — this skill is specifically a comparative benchmarking design
- Datasets where pathway redundancy/inter-pathway dependencies are irrelevant — the
  (η, ρ) similarity sweep is the point of the benchmark
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: pareg
  role: Benchmarked enrichment method and source of compute_term_similarities and
    similarity_sample helpers used to build the synthetic ground truth
  repo: https://github.com/cbg-ethz/pareg
- name: Snakemake
  role: Orchestration of the benchmark runs over the (η, ρ) grid via the workflows
    in the pareg repository
  repo: https://github.com/cbg-ethz/pareg
- name: R
  role: Implementation language for the benchmark, available as an R/Bioconductor
    package
  repo: https://bioconductor.org/packages/release/bioc/html/pareg.html
- name: tidyverse
  role: Data wrangling and aggregation of per-study PR-AUC results into the benchmark
    table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad522
    title: Coherent pathway enrichment estimation by modeling inter-pathway dependencies
      using regularized regression
  source_content_hash: sha256:abd5758dcd504ba8aa45dd813b4dd26d448f267ad8f79c866f32b84ec8835455
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/pareg
  tools:
  - R
  - Snakemake
  - pareg
  - tidyverse
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Precision-Recall AUC Benchmarking

## Summary

Benchmark enrichment methods by computing per-method precision-recall AUC against a simulated ground truth of active pathways, sweeping noise levels η and similarity factors ρ to expose robustness differences as done for pareg vs MGSA, FET, topGO, blitzGSEA, and a null model.

## When to use

Use when you need to quantitatively compare competing gene set enrichment methods on synthetic studies where the true positive pathways are known by construction, across a noise/similarity grid.

## When NOT to use

- Real experimental studies with no ground truth for which pathways are truly active — PR-AUC against simulated truth is only meaningful on synthetic data where positives are sampled by construction
- Evaluating a single enrichment method in isolation without competitor baselines — this skill is specifically a comparative benchmarking design
- Datasets where pathway redundancy/inter-pathway dependencies are irrelevant — the (η, ρ) similarity sweep is the point of the benchmark

## Inputs

- Synthetic pathway database (gene sets with pronounced clustering of pathways)
- Pairwise pathway similarity matrix (e.g., Jaccard) computed with compute_term_similarities
- Simulated gene p-value lists per synthetic study (DE genes ~ Beta(0.1, 1); background ~ Uniform)
- Ground-truth active pathway labels from similarity_sample
- Benchmark outputs from pareg, MGSA, FET, topGO, blitzGSEA, and null model runs

## Outputs

- Per-method precision-recall AUC values for each (η, ρ) grid setting
- Aggregated mean PR-AUC table written to benchmark_pr_auc.csv
- Benchmark comparison figures (e.g., Fig. 1a and 1c style PR-AUC comparisons)

## How to apply

Generate a synthetic pathway database with pronounced pathway clustering and compute pairwise Jaccard term similarities with pareg's compute_term_similarities helper. For each similarity factor ρ ∈ {0, 0.5, 1}, draw active pathways as true positives via pareg's similarity_sample on the similarity matrix, define DE genes as the union of active pathway genes, and simulate p-values from Beta(0.1, 1) for DE genes and Uniform for background genes while sweeping η ∈ {0, 0.25, 0.5, 0.75}. Run pareg (network_param = 1, term_network = similarity matrix), MGSA, FET, topGO, blitzGSEA, and a null model on every synthetic study, orchestrating runs with the Snakemake workflows from the pareg repository (inst/scripts/). Compute PR-AUC per method per (η, ρ) setting using the sampled active pathways as ground truth, aggregate means across the grid, and write the results to benchmark_pr_auc.csv. Pareg's network regularization on the pathway similarity matrix is expected to yield the highest mean PR-AUC in all (η, ρ) settings (Fig. 1a, 1c), demonstrating robustness to noise.

## Related tools

- **pareg** (Benchmarked enrichment method and source of compute_term_similarities and similarity_sample helpers used to build the synthetic ground truth) — https://github.com/cbg-ethz/pareg
- **Snakemake** (Orchestration of the benchmark runs over the (η, ρ) grid via the workflows in the pareg repository) — https://github.com/cbg-ethz/pareg
- **R** (Implementation language for the benchmark, available as an R/Bioconductor package) — https://bioconductor.org/packages/release/bioc/html/pareg.html
- **tidyverse** (Data wrangling and aggregation of per-study PR-AUC results into the benchmark table)

## Examples

```
active_terms <- similarity_sample(mat_similarities, 5)
```

## Evaluation signals

- benchmark_pr_auc.csv exists with one PR-AUC row per method per (η, ρ) grid setting
- The grid covers similarity factors ρ ∈ {0, 0.5, 1} and noise levels η ∈ {0, 0.25, 0.5, 0.75} with all combinations present
- Active pathways sampled with similarity_sample match exactly the set used as ground-truth positives for PR computation
- Null model PR-AUC is clearly below all real methods (sanity lower bound on the metric)
- Pareg (network_param = 1, term_network = similarity matrix) achieves the highest mean PR-AUC in each (η, ρ) setting, matching Fig. 1a/1c expectations

## Limitations

- Benchmark relies on synthetic p-value distributions (Beta(0.1, 1) for DE, Uniform for background); conclusions are conditioned on this simulation model
- Only a subset of noise levels per Fig. 1 panel (η = 0, 0.25, 0.5) is visualized, so high-noise behavior may need additional runs
- Pareg's advantage depends on the similarity matrix supplied as term_network; with ρ = 0 (no similarity signal) the network regularization offers less benefit

## Evidence

- [abstract] We show that pareg is more robust to noise than competing methods.: "We show that pareg is more robust to noise than competing methods."
- [methods] similarity_sample: "active_terms <- similarity_sample(mat_similarities, 5)"
- [methods] Beta/Uniform p-value simulation: "The p-values of genes considered to be differentially expressed are sampled from a Beta distribution centered at $0$. The p-values for all other genes are drawn from a Uniform distribution."
- [methods] true positive definition: "we select a subset of pathways to be activated. In a performance evaluation, these would be considered to be true positives."
- [readme] Snakemake workflows in repo: "Pathway enrichment computations using a regularized regression approach to incorporate inter-pathway relations in the statistical model."
