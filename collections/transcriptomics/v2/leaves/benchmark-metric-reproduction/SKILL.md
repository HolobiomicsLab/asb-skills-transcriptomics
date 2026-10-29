---
name: benchmark-metric-reproduction
description: Use when a task specifies a published numeric benchmark (e.g. accuracy
  0.94, specificity 0.98, sensitivity 0.90 on a 30% held-out test set) that must be
  verified by re-running the classifier end-to-end.
when_to_use_negative:
- The classifier inputs are not labeled CD8 T cell profiles (e.g. mixed TIL populations
  where Treg/CD4T/NKT cells are present and filterCD8T filtering dominates) rather
  than a defined held-out CD8 TIL test set
- The goal is classifying new data rather than verifying a published metric — use
  the classifier directly or its maintained successor (ProjecTILs) instead of benchmark
  reproduction
- No reference/ground-truth labels exist for the held-out cells, so accuracy, specificity
  and sensitivity cannot be computed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: TILPRED
  role: Logistic regression classifier whose predictTilState() function produces predictedState
    labels evaluated against held-out labels
  repo: https://github.com/carmonalab/TILPRED
- name: SingleCellExperiment
  role: Container format for the B16CD8TIL_SCE test/training expression data and state
    labels
- name: AUCell
  role: Bioconductor dependency used by TILPRED for gene-signature enrichment scoring
    during classification
- name: doParallel
  role: Parallel backend required by TILPRED (nCores argument of predictTilState)
- name: doRNG
  role: Reproducible parallel RNG dependency for TILPRED runs
- name: carmonalab/testTILPRED
  role: Example R Notebook demonstrating test-set usage of TILPRED
  repo: https://github.com/carmonalab/testTILPRED
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:16f397cc3d5521e2b0ef94fc5aebb0febec1c49794fb4e4e89b6b41866296d9c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/carmonalab/TILPRED
  repo_url: https://github.com/carmonalab/TILPRED
  tools:
  - AUCell
  - R
  - SingleCellExperiment
  - TILPRED
  - carmonalab/testTILPRED
  - doParallel
  - doRNG
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# benchmark-metric-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce a published classification benchmark by re-running a trained predictor (here TILPRED, a multinomial logistic regression CD8 TIL state classifier) on a held-out test split and computing accuracy with 95% CI, mean specificity and mean sensitivity against reported values.

## When to use

Use when a task specifies a published numeric benchmark (e.g. accuracy 0.94, specificity 0.98, sensitivity 0.90 on a 30% held-out test set) that must be verified by re-running the classifier end-to-end.

## When NOT to use

- The classifier inputs are not labeled CD8 T cell profiles (e.g. mixed TIL populations where Treg/CD4T/NKT cells are present and filterCD8T filtering dominates) rather than a defined held-out CD8 TIL test set
- The goal is classifying new data rather than verifying a published metric — use the classifier directly or its maintained successor (ProjecTILs) instead of benchmark reproduction
- No reference/ground-truth labels exist for the held-out cells, so accuracy, specificity and sensitivity cannot be computed

## Inputs

- B16CD8TIL_SCE SingleCellExperiment example dataset (annotated CD8 TIL state labels)
- Train/test partition assigning 30% of cells to the held-out test split
- predictTilState() output: predictedState labels and stateProbabilityMatrix

## Outputs

- tilpred_test_metrics.csv with accuracy, 95% CI, mean specificity, mean sensitivity
- Confusion matrix over the four CD8 TIL states (exhausted, memory-like, naive, EM-like)

## How to apply

Load the reference example dataset (B16CD8TIL_SCE SingleCellExperiment object shipped with TILPRED) and partition cells so that 30% are held out as a test split with their annotated CD8 TIL state labels retained. Run predictTilState() in mouse mode (human=FALSE) with default scoreThreshold, cellCycleThreshold and filterCD8T settings on the test-split expression matrix to obtain predictedState labels and the stateProbabilityMatrix over the four states: exhausted, memory-like, naive and EM-like. Build the confusion matrix of predictedState vs held-out reference labels, then compute overall accuracy with a 95% confidence interval plus mean specificity and mean sensitivity per state. Write the metrics to a CSV (tilpred_test_metrics.csv) and compare against the reported benchmark: accuracy 0.94 (95% CI 0.92-0.95), mean specificity 0.98, mean sensitivity 0.90.

## Related tools

- **TILPRED** (Logistic regression classifier whose predictTilState() function produces predictedState labels evaluated against held-out labels) — https://github.com/carmonalab/TILPRED
- **SingleCellExperiment** (Container format for the B16CD8TIL_SCE test/training expression data and state labels)
- **AUCell** (Bioconductor dependency used by TILPRED for gene-signature enrichment scoring during classification)
- **doParallel** (Parallel backend required by TILPRED (nCores argument of predictTilState))
- **doRNG** (Reproducible parallel RNG dependency for TILPRED runs)
- **carmonalab/testTILPRED** (Example R Notebook demonstrating test-set usage of TILPRED) — https://github.com/carmonalab/testTILPRED

## Examples

```
library(TILPRED); data(B16CD8TIL_SCE); pred <- predictTilState(B16CD8TIL_SCE, human=FALSE)
```

## Evaluation signals

- Reproduced accuracy falls within the published 95% CI of 0.92-0.95 (point estimate 0.94)
- Mean specificity across states is approximately 0.98 and mean sensitivity approximately 0.90
- Confusion matrix rows/columns cover exactly the four reference states: exhausted, memory-like, naive, EM-like
- stateProbabilityMatrix returned by predictTilState contains per-cell probability scores over the four states
- tilpred_test_metrics.csv is written with metrics matching the benchmark table

## Limitations

- TILPRED is no longer maintained; the authors recommend ProjecTILs instead, so long-term reproducibility depends on archived package versions
- The reference map is limited to the B16 melanoma model in a single mouse strain; metrics may not generalize to other strains or cancer types
- Performance depends on default settings (scoreThreshold, cellCycleThreshold, filterCD8T) and non-CD8 T cells are diverted to Treg/CD4T/NKT classes via signature enrichment, so impure input populations degrade the CD8 benchmark

## Evidence

- [methods] test-split accuracy benchmark: "predictTilState(data, nCores, human, scoreThreshold, cellCycleThreshold, filterCD8T)"
- [methods] example dataset load: "data(B16CD8TIL_SCE) # example SingleCellExperiment object"
- [readme] classifier definition: [evidence span withheld — anchor status: not_found]
- [abstract] reference states: "Unsupervised clustering and gene-signature analysis revealed four distinct CD8 TIL states - exhausted, memory-like, naïve and effector memory-like (EM-like)"
- [readme] deprecation: "**TILPRED is no longer maintained.** We recommend to use instead [ProjecTILs](https://github.com/carmonalab/ProjecTILs)"
- [methods] example notebook: "please see this [R Notebook](https://github.com/carmonalab/testTILPRED)"
