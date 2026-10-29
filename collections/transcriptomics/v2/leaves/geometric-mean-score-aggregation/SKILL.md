---
name: geometric-mean-score-aggregation
description: Use when benchmarking multiple trajectory inference methods (e.g., Totem
  vs Slingshot vs TinGa) scored with the four dynverse metrics against dynverse gold
  standards, and a single comparable overall score per method/topology is needed.
when_to_use_negative:
- Any input metric can be zero or negative, making the geometric mean undefined or
  uninterpretable
- Only a single metric is computed, so there is nothing to aggregate into an overall
  score
- Metrics are not all on a comparable positive scale and would need normalization
  before combining
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Implementation environment for computing geometric means, Friedman and Wilcoxon
    signed-rank tests with Bonferroni adjustment, and writing benchmark_metrics.csv
- name: Totem
  role: Benchmarked method whose per-topology scores are aggregated and compared against
    Slingshot and TinGa
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Benchmarked dynverse implementation compared via the aggregated overall scores,
    superior for linear trajectories
- name: TinGa
  role: Benchmarked method ranked second-best for nonlinear trajectories in the aggregated
    score comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:6035fdec6311e220b1ceda56cbe68ab60aeca7d9ef6d9964d33134659b852b77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - BiocManager
  - R
  - SingleCellExperiment
  - Slingshot
  - TinGa
  - Totem
  - dyndimred
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# geometric-mean-score-aggregation

## Summary

Aggregates the four dynverse trajectory-inference metrics (correlation of geodesic distances, feature-importance correlation, F1 branches, and HIM) into a single geometric-mean overall score per method and topology, then stratifies method rankings by topology (linear vs nonlinear) with paired nonparametric tests.

## When to use

Use when benchmarking multiple trajectory inference methods (e.g., Totem vs Slingshot vs TinGa) scored with the four dynverse metrics against dynverse gold standards, and a single comparable overall score per method/topology is needed.

## When NOT to use

- Any input metric can be zero or negative, making the geometric mean undefined or uninterpretable
- Only a single metric is computed, so there is nothing to aggregate into an overall score
- Metrics are not all on a comparable positive scale and would need normalization before combining

## Inputs

- Per-trajectory dynverse metric values for each method: correlation of geodesic distances, featureimp correlation, F1 branches, HIM
- Trajectory topology labels (linear, bifurcation, multifurcation, tree) for stratification
- Method labels (Totem, Slingshot, TinGa or other compared methods)

## Outputs

- Geometric-mean overall score per method per topology
- benchmark_metrics.csv with the per-method/per-topology score table
- Friedman test and Bonferroni-adjusted Wilcoxon signed-rank pairwise comparisons of overall scores

## How to apply

Score each inferred trajectory against its dynverse gold standard using the four dynverse metrics: correlation of geodesic distances, feature-importance (featureimp) correlation, F1 branches, and HIM. For each method and each topology group (linear, bifurcation, multifurcation, tree), combine the four metric values into a geometric mean overall score, which rewards methods that perform uniformly well across all metrics rather than excelling on one. Write the per-method/per-topology table to benchmark_metrics.csv. To judge superiority, run a Friedman test across methods and follow up with Wilcoxon signed-rank tests using Bonferroni adjustment (in this benchmark, overall Bonferroni-adjusted P ≤ 0.001 and pairwise P ≤ 0.01). Always report the score separately for linear and nonlinear topologies, since method superiority is topology-dependent (Totem best for nonlinear, Slingshot for linear in the reference benchmark).

## Related tools

- **R** (Implementation environment for computing geometric means, Friedman and Wilcoxon signed-rank tests with Bonferroni adjustment, and writing benchmark_metrics.csv)
- **Totem** (Benchmarked method whose per-topology scores are aggregated and compared against Slingshot and TinGa) — https://github.com/elolab/Totem
- **Slingshot** (Benchmarked dynverse implementation compared via the aggregated overall scores, superior for linear trajectories)
- **TinGa** (Benchmarked method ranked second-best for nonlinear trajectories in the aggregated score comparison)

## Evaluation signals

- Overall score per method/topology equals the 4th root of the product of the four dynverse metric values (geometric mean of geodesic-distance correlation, featureimp correlation, F1 branches, HIM)
- benchmark_metrics.csv contains one row per method per topology with a positive-valued overall score
- Topology-stratified rankings reproduce the expected pattern: Totem statistically superior for nonlinear trajectories, Slingshot for linear
- Friedman test over methods yields overall Bonferroni-adjusted P ≤ 0.001 and pairwise Wilcoxon Bonferroni-adjusted P ≤ 0.01 for the reported winner
- All four constituent metrics remain non-negative before aggregation

## Limitations

- Geometric-mean aggregation implicitly weights all four dynverse metrics equally and assumes positive metric values
- Scores are conditional on standardized inputs (5-dimensional LMDS embeddings from dyndimred) and on the specific 216 tree-shaped dynverse trajectories; they do not transfer to other benchmarks unchanged
- Rankings are topology-dependent, so reporting only a single overall score masks that Slingshot outperforms Totem on linear trajectories

## Evidence

- [other] Aggregate the four metrics per method and topology into a geometric mean overall score: [evidence span withheld — anchor status: not_found]
- [results] Totem achieved statistically superior overall scores: [evidence span withheld — anchor status: not_found]
- [other] Scoring against the dynverse gold standard: [evidence span withheld — anchor status: not_found]
- [readme] README framing of the benchmarked tool: "A user-friendly tool for inferring tree-shaped trajectories from single cell data."
