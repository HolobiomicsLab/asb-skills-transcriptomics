---
name: monte-carlo-replication-analysis
description: Use when comparing cell annotation methods under stochastic simulation
  conditions and a single run would not capture sampling variability — e.g., benchmarking
  CASi vs scmap, CHETAH, scPred on simulated multi-timepoint scRNA-seq data.
when_to_use_negative:
- When the goal is a single annotated dataset rather than a controlled comparison
  of methods — the neural network's step-wise annotation of real t1 data is handled
  by CrossTimeAnnotation alone, not by a replication loop.
- 'When the initial timepoint''s data differ dramatically from later timepoints: the
  CASi neural network ''would fail to borrow useful information and could not be very
  well trained'', so benchmark rankings would be dominated by a known failure mode
  rather than sampling variability.'
- When the novel cell type is transcriptomically very similar to known cell types
  — the paper notes 'the novel cell detection framework may not work well', so scenario-3
  results would reflect this limitation more than method ranking.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CASi
  role: Cross-timepoint annotation method under test; CreateCASiList builds t0/t1/label
    inputs and CrossTimeAnnotation trains the Keras/TensorFlow neural network classifier
    per replicate
  repo: https://github.com/yizhuo-wang/CASi
- name: keras
  role: TensorFlow backend dependency required to train CASi's neural network in each
    replicate
- name: scmap
  role: Competing supervised annotation method run on identical t0/t1 splits in every
    replicate
- name: CHETAH
  role: Competing supervised annotation method run on identical t0/t1 splits in every
    replicate
- name: scPred
  role: Competing supervised annotation method run on identical t0/t1 splits in every
    replicate
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:29fd73860dd2d48dc66dba50be80a023f04dc4dfb71dc58745dfa2f62fa46825
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yizhuo-wang/CASi
  tools:
  - CASi
  - CHETAH
  - keras
  - scPred
  - scmap
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# monte-carlo-replication-analysis

## Summary

A benchmarking skill for estimating the variability and ranking of method performance (per-cell annotation accuracy and ARI) by repeating a simulation-and-annotation pipeline across many Monte Carlo repetitions, as done in the CASi paper with 200 replicates per scenario.

## When to use

Use when comparing cell annotation methods under stochastic simulation conditions and a single run would not capture sampling variability — e.g., benchmarking CASi vs scmap, CHETAH, scPred on simulated multi-timepoint scRNA-seq data.

## When NOT to use

- When the goal is a single annotated dataset rather than a controlled comparison of methods — the neural network's step-wise annotation of real t1 data is handled by CrossTimeAnnotation alone, not by a replication loop.
- When the initial timepoint's data differ dramatically from later timepoints: the CASi neural network 'would fail to borrow useful information and could not be very well trained', so benchmark rankings would be dominated by a known failure mode rather than sampling variability.
- When the novel cell type is transcriptomically very similar to known cell types — the paper notes 'the novel cell detection framework may not work well', so scenario-3 results would reflect this limitation more than method ranking.

## Inputs

- Public PBMC scRNA-seq dataset (>60,000 sorted cells from eight immune cell types) as the simulation source pool
- Simulated longitudinal t0/t1 count matrices per replicate per scenario (shifted compositions / disappearing type / novel type)
- t0 ground-truth cell labels used to train each method
- Competing method implementations: scmap, CHETAH, scPred, and CASi

## Outputs

- Per-cell annotation accuracy for each method, scenario, and Monte Carlo replicate
- ARI comparisons across the three scenarios (as in supplementary Figure 11)
- Aggregated accuracy/ARI summary table, e.g., annotation_benchmark_results.csv

## How to apply

Define distinct simulation scenarios (here: same cell types with shifted compositions, a disappearing cell type, and a novel emerging cell type) and, for each, repeatedly simulate paired t0/t1 scRNA-seq datasets from a source pool (the PBMC dataset of >60,000 sorted cells from eight immune cell types, with five cell types drawn as the simulation pool as in pbmc_demo). In each replicate, run all competing methods on identical t0/t1 splits: build a CASi list with CreateCASiList (t0 counts, t1 counts, t0 labels), train the Keras/TensorFlow neural network via CrossTimeAnnotation, and run scmap, CHETAH, and scPred on the same data. Record per-cell annotation accuracy for every method in every replicate, plus adjusted Rand index (ARI) for clustering comparison as reported in supplementary Figure 11. Aggregate accuracies and ARIs across the 200 Monte Carlo repetitions per scenario and write the final table (e.g., annotation_benchmark_results.csv) so rankings reflect distributional evidence rather than one lucky split.

## Related tools

- **CASi** (Cross-timepoint annotation method under test; CreateCASiList builds t0/t1/label inputs and CrossTimeAnnotation trains the Keras/TensorFlow neural network classifier per replicate) — https://github.com/yizhuo-wang/CASi
- **keras** (TensorFlow backend dependency required to train CASi's neural network in each replicate)
- **scmap** (Competing supervised annotation method run on identical t0/t1 splits in every replicate)
- **CHETAH** (Competing supervised annotation method run on identical t0/t1 splits in every replicate)
- **scPred** (Competing supervised annotation method run on identical t0/t1 splits in every replicate)

## Examples

```
R: require(keras); require(CASi); casilist <- CreateCASiList(t0_counts, t1_counts, t0_labels); ann <- CrossTimeAnnotation(casilist); # repeat 200x per scenario, then aggregate accuracy/ARI into annotation_benchmark_results.csv
```

## Evaluation signals

- Each of the 200 repetitions per scenario uses identical t0/t1 splits across all methods (CASi, scmap, CHETAH, scPred) — verify no method received different data in any replicate.
- Scenario 1 (shifted compositions) and Scenario 2 (disappearing cell type) should show CASi with the highest accuracy and ARI across replicates, per the paper's finding.
- Scenario 3 (novel cell type) should show a substantial rise in both accuracy and ARI when the novel-cell detection pipeline (DetectNovelCells) is added, with CASi at state-of-the-art performance.
- Aggregated ARI values should reproduce the ordering reported in supplementary Figure 11; compare your summary table against it.
- The final output is a single aggregated CSV (annotation_benchmark_results.csv) with per-scenario, per-method accuracy/ARI summaries across all 200 replicates — check row counts and scenario labels.

## Limitations

- The ARI comparison results for the three scenarios are only in supplementary Figure 11, not the main text, so the reference distribution must be read from the supplementary file.
- The neural network fails if the initial timepoint's data are dramatically different from later timepoints, which can dominate replicate-level results.
- Novel cell detection may fail when the novel cell type is transcriptomically similar to known cell types, weakening scenario-3 conclusions.
- CASi's annotation step pools all samples together for training, which may be inappropriate when cell populations differ across conditions.

## Evidence

- [results] For scenarios 1 and 2 CASi has the highest accuracy and ARI; when a novel cell type appears (scenario 3), the novel-cell detection pipeline substantially improves annotation: [evidence span withheld — anchor status: not_found]
- [supplementary] The results of three scenarios are included in the supplementary file Figure 11.: "The results of three scenarios are included in the supplementary file Figure 11."
- [methods] We obtain a publicly available dataset of peripheral blood mononuclear cells (PBMC)[1], containing more than 60,000 sorted cells from eight immune cell types.: "We obtain a publicly available dataset of peripheral blood mononuclear cells (PBMC)[1], containing more than 60,000 sorted cells from eight immune cell types."
- [intro] Our framework implements a neural network classifier to assign cell labels in a supervised way, and it can achieve high accuracy in the cross-timepoint setting.: "Our framework implements a neural network classifier to assign cell labels in a supervised way, and it can achieve high accuracy in the cross-timepoint setting."
- [discussion] If the initial timepoint’s data is dramatically different from later timepoint’s data, the neural network would fail to borrow useful information and could not be very well trained: "If the initial timepoint’s data is dramatically different from later timepoint’s data, the neural network would fail to borrow useful information and could not be very well trained"
- [readme] Before using CASi, please make sure to set up TensorFlow from https://tensorflow.rstudio.com/install/, and that the dependent package "keras" is available to use.: [evidence span withheld — anchor status: external_doc]
