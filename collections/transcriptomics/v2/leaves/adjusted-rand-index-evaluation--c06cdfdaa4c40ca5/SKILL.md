---
name: adjusted-rand-index-evaluation
description: Use when benchmarking cell-type annotation or clustering across timepoints
  and you need an ARI comparison (as in supplementary Figure 11) alongside per-cell
  accuracy for CASi vs scmap, CHETAH, scPred.
when_to_use_negative:
- The task is single-timepoint cross-sectional cell annotation with no t0/t1 split
  — CASi's cross-timepoint benchmark design does not apply
- The reference labels are unknown (pure unsupervised clustering evaluation without
  ground truth), so a Rand-index comparison against truth cannot be computed
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: CASi
  role: Produces t1 cell annotations via CrossTimeAnnotation whose labels are scored
    with ARI and accuracy
  repo: https://github.com/yizhuo-wang/CASi
- name: scmap
  role: Competing supervised annotation method benchmarked with accuracy and ARI on
    identical splits
- name: CHETAH
  role: Competing supervised annotation method benchmarked with accuracy and ARI on
    identical splits
- name: scPred
  role: Competing supervised annotation method benchmarked with accuracy and ARI on
    identical splits
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-024-58566-x
    title: 'CASi: A framework for cross-timepoint analysis of single-cell RNA sequencing
      data'
  source_content_hash: sha256:9be1b8f4aa08721f24cbbd42768c67f48416c178a0b0064d8946fda309cfd032
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yizhuo-wang/CASi
  tools:
  - CASi
  - CHETAH
  - scPred
  - scmap
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# adjusted-rand-index-evaluation

## Summary

Compute adjusted Rand index (ARI) to compare clusterings/cell annotations against reference labels in the CASi cross-timepoint benchmark, complementing per-cell annotation accuracy across simulated PBMC t0/t1 scenarios.

## When to use

Use when benchmarking cell-type annotation or clustering across timepoints and you need an ARI comparison (as in supplementary Figure 11) alongside per-cell accuracy for CASi vs scmap, CHETAH, scPred.

## When NOT to use

- The task is single-timepoint cross-sectional cell annotation with no t0/t1 split — CASi's cross-timepoint benchmark design does not apply
- The reference labels are unknown (pure unsupervised clustering evaluation without ground truth), so a Rand-index comparison against truth cannot be computed

## Inputs

- Predicted t1 cell label vector from each annotation method (CASi, scmap, CHETAH, scPred)
- Ground-truth simulated t1 cell type labels from the PBMC simulation (200 Monte Carlo replicates x 3 scenarios)

## Outputs

- Per-replicate ARI between predicted and reference cell partitions
- Aggregated ARI and accuracy per method per scenario (annotation_benchmark_results.csv)

## How to apply

For each of the three simulated PBMC scenarios (shifted compositions, disappearing cell type, novel emerging cell type), annotate t1 cells with CASi's CrossTimeAnnotation and the competing supervised methods (scmap, CHETAH, scPred) on identical t0/t1 splits. After each of the 200 Monte Carlo repetitions, compare the predicted t1 cell labels against the known simulated ground-truth labels, computing both per-cell annotation accuracy and the adjusted Rand index between predicted and reference partitions. Aggregate accuracies and ARIs across replicates and scenarios, and report the scenario-wise ARI comparison as done in supplementary Figure 11. Correct application means ARI is computed on the same per-replicate label vectors used for accuracy, so the two metrics can be jointly aggregated into annotation_benchmark_results.csv.

## Related tools

- **CASi** (Produces t1 cell annotations via CrossTimeAnnotation whose labels are scored with ARI and accuracy) — https://github.com/yizhuo-wang/CASi
- **scmap** (Competing supervised annotation method benchmarked with accuracy and ARI on identical splits)
- **CHETAH** (Competing supervised annotation method benchmarked with accuracy and ARI on identical splits)
- **scPred** (Competing supervised annotation method benchmarked with accuracy and ARI on identical splits)

## Examples

```
R> casilist <- CreateCASiList(t0_counts, t1_counts, t0_labels); pred <- CrossTimeAnnotation(casilist); ari <- mclust::adjustedRandIndex(pred, t1_true_labels)
```

## Evaluation signals

- ARI for the three scenarios is computed exactly as in supplementary Figure 11 of the CASi paper
- ARI values are computed on the same per-replicate predicted/ground-truth label vectors used for annotation accuracy, over 200 Monte Carlo repetitions
- Aggregated table contains accuracy and ARI per method (CASi, scmap, CHETAH, scPred) per scenario and is written to annotation_benchmark_results.csv
- In scenarios 1 and 2, CASi should show the highest accuracy and ARI among the compared methods; in scenario 3, ARI improves substantially when the novel-cell detection pipeline is enabled

## Limitations

- ARI comparison results for the three scenarios are only available in supplementary Figure 11, not in the main text
- Simulation scenario settings are referenced but not fully described in the main text, limiting reproducibility of the exact ARI benchmark
- CASi's annotation pools all samples together for neural network training, which may distort clustering comparison when cell populations differ across conditions

## Evidence

- [discussion] The results of three scenarios are included in the supplementary file Figure 11.: "The results of three scenarios are included in the supplementary file Figure 11."
- [methods] CrossTimeAnnotation function parameters: "Below are the input parameters of the CrossTimeAnnotation function"
- [intro] CASi trains a neural network to assign supervised cell labels across timepoints: "Our framework implements a neural network classifier to assign cell labels in a supervised way, and it can achieve high accuracy in the cross-timepoint setting."
- [readme] CASi is a comprehensive R package for multi-timepoint scRNA-seq analysis: "CASi is a comprehensive R package for analyzing multi-timepoint scRNA-seq data"
