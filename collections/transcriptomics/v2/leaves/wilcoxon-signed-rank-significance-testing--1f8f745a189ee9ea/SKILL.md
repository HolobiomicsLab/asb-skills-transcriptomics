---
name: wilcoxon-signed-rank-significance-testing
description: Use when comparing paired per-replicate runtime (or other paired measurement)
  values across methods — e.g. three replicate subsamples per tool — and you need
  a distribution-free significance call without assuming normality.
when_to_use_negative:
- Measurements are independent/unpaired across conditions rather than paired on identical
  inputs — the signed-rank test requires paired observations
- The variable of interest is categorical or the sample provides no replicate pairing
  (e.g. a single runtime per method), making rank-based paired testing undefined
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Environment for timing the benchmark runs (via wall-clock timers around singlenet
    and competitor calls) and for running the Wilcoxon signed-rank test on the compiled
    runtime table
- name: dsCellNet
  role: Method under test whose ~12 min runtime per 10,000-cell replicate is the reference
    arm of the paired comparison
  repo: https://github.com/songzh523/dsCellNet
- name: SingleCellSignalR
  role: Comparison arm expected to take 2-3 h on 10,000 cells, providing the largest
    paired runtime difference
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.047
    title: 'dsCellNet: A new computational tool to infer cell–cell communication networks
      in the developing and aging brain'
  source_content_hash: sha256:7331ab291c7d48d69713e4f1e35fc0de3e326a222e49a2f962e1168c8770c00b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/songzh523/dsCellNet
  repo_url: https://github.com/songzh523/dsCellNet
  tools:
  - CellPhoneDB
  - NATMI
  - R
  - SingleCellSignalR
  - dsCellNet
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# wilcoxon-signed-rank-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A non-parametric paired significance test used in the dsCellNet benchmarking study to verify that dsCellNet's ~12 min runtime for 10,000 E15.5 cells is significantly faster than NATMI, CellPhoneDB, and SingleCellSignalR at p < 0.01.

## When to use

Use when comparing paired per-replicate runtime (or other paired measurement) values across methods — e.g. three replicate subsamples per tool — and you need a distribution-free significance call without assuming normality.

## When NOT to use

- Measurements are independent/unpaired across conditions rather than paired on identical inputs — the signed-rank test requires paired observations
- The variable of interest is categorical or the sample provides no replicate pairing (e.g. a single runtime per method), making rank-based paired testing undefined

## Inputs

- Runtime table with columns (method, replicate, minutes) from timing dsCellNet, NATMI, CellPhoneDB and SingleCellSignalR on three replicate 10,000-cell subsamples of E15.5 mouse cortex scRNA-seq (GSE153164)

## Outputs

- Wilcoxon signed-rank p-values comparing dsCellNet runtime against each competing method
- Significance verdict at p < 0.01 confirming dsCellNet's runtime advantage

## How to apply

Compile per-method, per-replicate wall-clock runtimes into a table with columns (method, replicate, minutes), as done for dsCellNet vs NATMI, CellPhoneDB and SingleCellSignalR on three independently subsampled 10,000-cell E15.5 replicates (fixed seed, from GSE153164's 11,670 cells). Because replicates are paired across methods on identical inputs, apply the Wilcoxon signed-rank test to the paired differences (e.g. dsCellNet vs each competitor) rather than a t-test, avoiding any normality assumption on runtime distributions. Confirm significance at p < 0.01 and check that dsCellNet reproduces the reported ~12 min per replicate while SingleCellSignalR takes up to 2-3 h. In R, the test can be run with wilcox.test(x, y, paired=TRUE) on the per-replicate minute vectors.

## Related tools

- **R** (Environment for timing the benchmark runs (via wall-clock timers around singlenet and competitor calls) and for running the Wilcoxon signed-rank test on the compiled runtime table)
- **dsCellNet** (Method under test whose ~12 min runtime per 10,000-cell replicate is the reference arm of the paired comparison) — https://github.com/songzh523/dsCellNet
- **SingleCellSignalR** (Comparison arm expected to take 2-3 h on 10,000 cells, providing the largest paired runtime difference)

## Examples

```
wilcox.test(dsCellNet_minutes, ssc_minutes, paired=TRUE, alternative="less")
```

## Evaluation signals

- The runtime table shows dsCellNet at approximately 12 minutes per replicate for 10,000 cells
- Wilcoxon signed-rank test comparing dsCellNet against NATMI, CellPhoneDB and SingleCellSignalR yields p < 0.01
- All three replicate subsamples were generated with a fixed seed so paired differences are reproducible
- Per-replicate runtimes for each method are recorded to minutes in the (method, replicate, minutes) table

## Limitations

- Reported runtimes are specific to 10,000-cell subsamples of the E15.5 dataset with default parameters; scaling to other cell counts or datasets may change runtimes
- Wall-clock timing is hardware-dependent; results are only comparable within the same execution environment
- dsCellNet's version number is not stated, so exact reproducibility of the benchmarked timing may vary across package versions

## Evidence

- [results] SingleCellSignalR takes up to 2-3 h to finish for samples with 10,000 single cells; among the top two performing methods NATMI takes the least time, but dsCellNet only requires 12 min for 10,000 single cells.: [evidence span withheld — anchor status: not_found]
