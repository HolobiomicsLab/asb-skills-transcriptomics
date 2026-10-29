---
name: rare-population-proportion-scaling
description: Use when benchmarking rare-cell detection tools across controlled rare-population
  sizes and DE strengths, or when deciding whether a detector (e.g. GapClust) retains
  accuracy as the planted minor cluster grows from 2 to 100 cells.
when_to_use_negative:
- The goal is general clustering of major cell types — Seurat/Scater-style workflows
  are the article's designated tools for major populations, not this rare-cell proportion
  sweep
- You need to call a rare population in a single real dataset rather than benchmark
  detectors against simulated ground truth — run GapClust directly instead of the
  simulation sweep
- The rare population is not 'rare' relative to dataset size (planted sizes here stay
  ≤ 100 cells per simulated profile)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GapClust
  role: Primary rare-cell detector run on every simulated dataset; calls the minor
    cluster via a second-order-derivative-type distance score
  repo: github.com/fabotao/GapClust
- name: RaceID
  role: Competitor rare-cell detector run with default settings on the same simulated
    count matrices for F1 comparison
- name: GiniClust
  role: Competitor rare-cell detector run with default settings on the same simulated
    count matrices for F1 comparison
- name: CellSIUS
  role: Competitor rare-cell detector run with default settings on the same simulated
    count matrices for F1 comparison
- name: FiRE
  role: Competitor rare-cell detector run with default settings on the same simulated
    count matrices for F1 comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:3572aafaac937fd972a40094ea6ca395ab6f6f0baefa7f261d21da541e261ebb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellSIUS
  - FiRE
  - GapClust
  - GiniClust
  - RaceID
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# rare-population-proportion-scaling

## Summary

Generates Splatter-simulated scRNA-seq count matrices in which the size of a planted rare population (2–100 cells) and its differential-expression level (de.prob 0.1–0.8) are systematically swept, so that rare-cell detectors can be benchmarked by F1 score across rare-population proportions.

## When to use

Use when benchmarking rare-cell detection tools across controlled rare-population sizes and DE strengths, or when deciding whether a detector (e.g. GapClust) retains accuracy as the planted minor cluster grows from 2 to 100 cells.

## When NOT to use

- The goal is general clustering of major cell types — Seurat/Scater-style workflows are the article's designated tools for major populations, not this rare-cell proportion sweep
- You need to call a rare population in a single real dataset rather than benchmark detectors against simulated ground truth — run GapClust directly instead of the simulation sweep
- The rare population is not 'rare' relative to dataset size (planted sizes here stay ≤ 100 cells per simulated profile)

## Inputs

- Splatter simulation parameters: rare population size sweep (2–100 cells) and de.prob sweep (0.1–0.8), 99 profiles per DE-probability scenario
- Ground-truth rare-cell labels from the Splatter simulations
- Simulated raw count matrices (genes x cells) to feed each detector

## Outputs

- Per-dataset, per-method F1 scores for GapClust, RaceID, GiniClust, CellSIUS and FiRE
- Aggregated F1 table/panels matching Supplementary Fig. 2a-e
- Ranking of methods per simulation setting (expected: GapClust best in all settings)

## How to apply

Generate 99 simulated expression profiles per differential-expression scenario with Splatter, sweeping the rare population size from 2 to 100 cells and de.prob from 0.1 to 0.8 while retaining Splatter's ground-truth rare-cell labels. Run GapClust on every simulated dataset, exploiting its second-order-derivative-type distance score that captures the inter-cluster gap to call the minor cluster. Run the competitors RaceID, GiniClust, CellSIUS and FiRE on the same simulated count matrices with their default settings. Compare each method's called rare cells against the Splatter ground-truth labels, compute the F1 score per dataset and per method, and aggregate scores into panels matching Supplementary Fig. 2a-e; a correct application confirms GapClust ranks best (highest F1) in every simulated setting.

## Related tools

- **GapClust** (Primary rare-cell detector run on every simulated dataset; calls the minor cluster via a second-order-derivative-type distance score) — github.com/fabotao/GapClust
- **RaceID** (Competitor rare-cell detector run with default settings on the same simulated count matrices for F1 comparison)
- **GiniClust** (Competitor rare-cell detector run with default settings on the same simulated count matrices for F1 comparison)
- **CellSIUS** (Competitor rare-cell detector run with default settings on the same simulated count matrices for F1 comparison)
- **FiRE** (Competitor rare-cell detector run with default settings on the same simulated count matrices for F1 comparison)

## Examples

```
devtools::install_github("fabotao/GapClust"); GapClust::GapClust(expr_simulated)  # per Splatter-simulated matrix with planted rare cells
```

## Evaluation signals

- Every simulated dataset carries ground-truth rare-cell labels from Splatter, so per-method F1 scores are computable by set comparison of called vs planted rare cells
- F1 scores aggregate into panels matching Supplementary Fig. 2a-e across all five DE-probability scenarios
- Invariant to verify: GapClust ranks best by F1 among GapClust/RaceID/GiniClust/CellSIUS/FiRE in every simulated setting
- Scenario grid check: 99 profiles per de.prob scenario, rare sizes spanning 2–100 cells and de.prob spanning 0.1–0.8

## Limitations

- Results are simulation-grounded (Splatter) for the DE sweep; competitor tool versions are not reported in the article section
- GapClust run parameters (e.g. candidate k threshold skewness<k> > 2) are not fully reported in the article section, so reproduction may need the GitHub repo
- The design targets a single minor cluster separated by an inter-cluster gap; FiRE is noted in the article as failing to discriminate outliers from minor cell types, which affects comparison interpretation

## Evidence

- [abstract] Here, we develop GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets: "Here, we develop GapClust, a light-weight algorithm to detect rare cell types from ultra-large scRNA-seq datasets"
- [intro] A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information.: "A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information."
- [discussion] In terms of accuracy, GapClust consistently performs better than existing approaches in extensive simulation experiments: "In terms of accuracy, GapClust consistently performs better than existing approaches in extensive simulation experiments"
- [readme] In terms of rare cell detection, GapClust offers best performance compared with other methods, quantified by F<sub>1</sub> score.: "In terms of rare cell detection, GapClust offers best performance compared with other methods, quantified by F<sub>1</sub> score."
