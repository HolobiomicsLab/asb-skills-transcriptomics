---
name: detection-f1-score-evaluation
description: Use when comparing rare-cell detectors (GapClust, RaceID, GiniClust,
  CellSIUS, FiRE) on simulated count matrices that carry ground-truth rare-cell labels
  and you need per-method F1 scores.
when_to_use_negative:
- The dataset is real experimental data without ground-truth rare-cell labels — no
  true-positive basis exists to compute F1
- You need runtime/memory benchmarks rather than detection accuracy (the paper measures
  time and memory separately)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GapClust
  role: Rare-cell detector being benchmarked; calls the minor cluster via a second-order-derivative-type
    distance score
  repo: github.com/fabotao/GapClust
- name: RaceID
  role: Competitor rare-cell detector run on the same simulated count matrices
- name: GiniClust
  role: Competitor rare-cell detector run on the same simulated count matrices
- name: CellSIUS
  role: Competitor rare-cell detector run on the same simulated count matrices
- name: FiRE
  role: Competitor rare-cell detector run on the same simulated count matrices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:8f8d41c66c4c9cb68418ea64fc176fb3e14b61e596bfc873b364c4137ea87c14
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellSIUS
  - FiRE
  - GapClust
  - GiniClust
  - RaceID
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# detection-f1-score-evaluation

## Summary

Benchmark rare-cell detection methods on Splatter-simulated scRNA-seq datasets by scoring each method's called rare cells against planted ground-truth labels with the F1 score, as done in the GapClust evaluation.

## When to use

Use when comparing rare-cell detectors (GapClust, RaceID, GiniClust, CellSIUS, FiRE) on simulated count matrices that carry ground-truth rare-cell labels and you need per-method F1 scores.

## When NOT to use

- The dataset is real experimental data without ground-truth rare-cell labels — no true-positive basis exists to compute F1
- You need runtime/memory benchmarks rather than detection accuracy (the paper measures time and memory separately)

## Inputs

- Splatter-simulated scRNA-seq count matrices with ground-truth rare-cell labels (rare size 2–100 cells, de.prob 0.1–0.8)
- Rare-cell calls from each compared method (GapClust, RaceID, GiniClust, CellSIUS, FiRE) on the same count matrices

## Outputs

- Per-dataset, per-method F1 score for rare-cell detection
- Aggregated F1 tables reproducing the panels of Supplementary Fig. 2a-e

## How to apply

Generate simulated expression profiles with Splatter, sweeping rare population size from 2 to 100 cells and de.prob from 0.1 to 0.8 (five DE-probability scenarios, 99 profiles each), retaining the planted ground-truth rare-cell labels. Run each detector on the same simulated count matrices with default settings — GapClust exploits its second-order-derivative-type distance score to call the minor cluster, while RaceID, GiniClust, CellSIUS and FiRE run at defaults. For every dataset and method, match the called rare-cell set against the ground-truth labels, derive precision and recall, and compute F1 = 2*precision*recall/(precision+recall). Aggregate F1 values into a per-scenario table that mirrors Supplementary Fig. 2a-e and check whether GapClust ranks best in all simulated settings.

## Related tools

- **GapClust** (Rare-cell detector being benchmarked; calls the minor cluster via a second-order-derivative-type distance score) — github.com/fabotao/GapClust
- **RaceID** (Competitor rare-cell detector run on the same simulated count matrices)
- **GiniClust** (Competitor rare-cell detector run on the same simulated count matrices)
- **CellSIUS** (Competitor rare-cell detector run on the same simulated count matrices)
- **FiRE** (Competitor rare-cell detector run on the same simulated count matrices)

## Evaluation signals

- Every F1 value falls within [0, 1] and is computed against the Splatter-planted ground-truth labels, not inferred clusters
- The aggregated table spans five simulation scenarios (de.prob 0.1–0.8) and matches the panels of Supplementary Fig. 2a-e
- GapClust achieves the highest F1 among all compared methods in every simulated setting, per the reported finding
- Called rare-cell sets from competitors are produced on identical count matrices and with default settings, ensuring a fair F1 comparison

## Limitations

- No version numbers are given for the compared tools (GiniClust, RaceID, CellSIUS, FiRE, EDGE), so exact reproducibility of the F1 comparison is limited
- Results for most real datasets are deferred to unlinked supplementary figures (Supplementary Figs. 14-22, Supplementary Table 5), so only the Splatter benchmark F1 panels are directly verifiable
- No parameter settings for GapClust runs (e.g. K, skewness threshold) are reported in the section, constraining replication of its called rare-cell sets

## Evidence

- [other] F1 per dataset and per method against ground truth: [evidence span withheld — anchor status: not_found]
- [other] Best performance among all compared methods in every simulation setting: [evidence span withheld — anchor status: not_found]
- [readme] F1 quantification of rare cell detection performance: "In terms of rare cell detection, GapClust offers best performance compared with other methods, quantified by F<sub>1</sub> score."
- [other] Simulation design sweeping rare population size and de.prob: [evidence span withheld — anchor status: not_found]
