---
name: negative-binomial-gene-count-simulation
description: 'Use when evaluating whether a co-expression estimator applied to log-normalized
  scRNA-seq UMI counts produces false positives: simulate counts for an independent
  gene pair under a negative-binomial model with zero true co-expression and measure
  the spurious correlation each method reports.'
when_to_use_negative:
- The goal is co-expression inference on real scRNA-seq data rather than benchmarking
  estimator false-positive behavior on simulated independent genes
- The simulated pair is intended to have true co-expression — this skill's design
  requires zero true co-expression so any nonzero estimate is a false positive
- The data are not UMI counts with sequencing-depth variation (e.g., bulk RNA-seq
  or already-normalized expression matrices), since the spurious-correlation phenomenon
  being probed arises from log-normalizing variable-depth UMI counts
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: locCSN
  role: Benchmarked co-expression estimator applied to the simulated log-normalized
    counts; reports a spurious estimate of 0.61 for the independent pair
  repo: https://github.com/xuranw/locCSN
- name: Seurat
  role: Used alongside CSCORE in the article's analysis environment; the log-normalization
    transform log(1 + scaled expression) being probed corresponds to Seurat-style
    normalization
- name: CS-CORE
  role: Statistical method whose motivation this simulation supports — modeling sequencing
    depth variations and measurement errors to avoid inflated false positives; the
    article's repository provides the reference implementation
  repo: https://github.com/ChangSuBiostats/CS-CORE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:51fa8dd7881a387291bf47fe242a859667a31ad2f9a88bb4a0c0d6b912195318
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuranw/locCSN
  tools:
  - CS-CORE
  - CSCORE (R package)
  - Seurat
  - locCSN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# negative-binomial-gene-count-simulation

## Summary

A benchmarking simulation that generates UMI counts for a truly independent (non-co-expressed) gene pair from a negative-binomial model calibrated to single-cell expression levels, then runs co-expression estimators on log-normalized counts to quantify spurious (false-positive) co-expression. It exposes the inflated false positives of log-normalization-based methods that motivate CS-CORE.

## When to use

Use when evaluating whether a co-expression estimator applied to log-normalized scRNA-seq UMI counts produces false positives: simulate counts for an independent gene pair under a negative-binomial model with zero true co-expression and measure the spurious correlation each method reports.

## When NOT to use

- The goal is co-expression inference on real scRNA-seq data rather than benchmarking estimator false-positive behavior on simulated independent genes
- The simulated pair is intended to have true co-expression — this skill's design requires zero true co-expression so any nonzero estimate is a false positive
- The data are not UMI counts with sequencing-depth variation (e.g., bulk RNA-seq or already-normalized expression matrices), since the spurious-correlation phenomenon being probed arises from log-normalizing variable-depth UMI counts

## Inputs

- Simulation specification: negative-binomial parameters (mean and dispersion) for two genes with zero true co-expression, calibrated so mean ranks match 269 and 351 among 28,412 genes in excitatory neurons of the ref. 17 postmortem brain dataset
- Number of simulated cells per gene pair
- List of co-expression estimators to benchmark (locCSN, Pearson, propr, Spearman, SpQN)

## Outputs

- coexpression_estimates_independent_pair.csv containing the five co-expression estimates on log-normalized counts
- Per-method spurious co-expression estimates for the independent gene pair (expected: locCSN=0.61, Pearson=0.14, propr=0.50, Spearman=0.07, SpQN=0.14)

## How to apply

Simulate UMI counts for two genes under a negative-binomial model with zero true co-expression, setting mean levels so the genes sit at mean ranks 269 and 351 among 28,412 genes (as in excitatory neurons of the Alzheimer's disease postmortem brain dataset, ref. 17). Compute sequencing-depth scale factors from the simulated raw counts and derive both scaled counts and log-normalized counts via log(1 + scaled expression). Apply each estimator under test — locCSN, Pearson correlation, propr, Spearman correlation, and SpQN — to the log-normalized counts of the pair. Compare each estimate against the reported reference values (locCSN=0.61, Pearson=0.14 with p<0.01, propr=0.50, Spearman=0.07 with p=0.02, SpQN=0.14); large estimates such as locCSN=0.61 and propr=0.50 demonstrate inflated false positives on truly independent genes. Write the five estimates to coexpression_estimates_independent_pair.csv.

## Related tools

- **locCSN** (Benchmarked co-expression estimator applied to the simulated log-normalized counts; reports a spurious estimate of 0.61 for the independent pair) — https://github.com/xuranw/locCSN
- **Seurat** (Used alongside CSCORE in the article's analysis environment; the log-normalization transform log(1 + scaled expression) being probed corresponds to Seurat-style normalization)
- **CS-CORE** (Statistical method whose motivation this simulation supports — modeling sequencing depth variations and measurement errors to avoid inflated false positives; the article's repository provides the reference implementation) — https://github.com/ChangSuBiostats/CS-CORE

## Evaluation signals

- Each reproduced estimate on log-normalized counts matches the reported value for the independent pair: locCSN=0.61, Pearson=0.14 (p<0.01), propr=0.50, Spearman=0.07 (p=0.02), SpQN=0.14
- Since the simulated pair has zero true co-expression, any statistically significant Pearson/Spearman p-value is itself evidence of the spurious-correlation artifact being measured
- Simulated gene mean levels reproduce the target mean ranks (269 and 351) among 28,412 genes calibrated to excitatory neurons
- Output file coexpression_estimates_independent_pair.csv contains exactly five estimates, one per benchmarked method

## Limitations

- The reference values correspond to one specific simulation setting (mean ranks 269 and 351 of 28,412 genes); other mean/dispersion choices may shift the spurious estimates
- The article's own measurement model uses a Poisson distribution; the negative binomial is used here for simulation, and the authors note adaptation of CS-CORE to the negative binomial is left to future work
- Conclusions are limited to log-normalization-based estimators (locCSN, Pearson, propr, Spearman, SpQN); methods operating on raw or appropriately scaled counts are not characterized by this benchmark

## Evidence

- [abstract] most existing methods suffered from inflated false positives as well as biased co-expression estimates and clustering analysis: "most existing methods suffered from inflated false positives as well as biased co-expression estimates and clustering analysis"
- [intro] gene co-expressions measured via correlations of UMI counts across cells can be seriously confounded by varying sequencing depths: "gene co-expressions measured via correlations of UMI counts across cells can be seriously confounded by varying sequencing depths, resulting in inflated false positive findings"
- [discussion] It can be useful to adapt CS-CORE to model xij∣zij using other distributions: "It can be useful to adapt CS-CORE to model xij∣zij using other distributions that model nonnegative integers such as the negative binomial distribution"
- [readme] locCSN is a package for estimating gene co-expression networks: [evidence span withheld — anchor status: external_doc]
