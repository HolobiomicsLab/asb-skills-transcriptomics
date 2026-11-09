---
name: precision-recall-f1-evaluation
description: Use when comparing differential network inference methods (e.g., BoostDiff
  vs GENIE3, ARACNE, RF Difftrees, EBcoexpress) on data with known ground-truth edges
  and you need precision/recall/F1 per network-size setting.
when_to_use_negative:
- The analysis goal is differential expression testing rather than differential network
  (edge-level) inference — precision/recall on predicted edges is not the right metric
- No ground-truth network is available (e.g., real disease cohorts like GSE156063
  or GSE126124) — use pathway enrichment or correlation-distribution diagnostics instead
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BoostDiff
  role: Differential network inference method under benchmark; run with n_estimators=50
    on simulated disease/control pairs
  repo: github.com/scibiome/boostdiff_inference
- name: dcanr
  role: Simulates ground-truth GRNs and disease/control expression data via the one-gene-knockdown
    scheme
- name: minet
  role: Provides the ARACNE comparator for edge prediction on the simulated datasets
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:e9224b8f0c6d28c64986b0ed0e343b5bbd7e6a1153d590b4806e7ea2dcc0615f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BoostDiff
  - Python
  - R
  - conda
  - dcanr
  - minet
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# precision-recall-f1-evaluation

## Summary

Benchmark protocol for comparing differential gene regulatory network (GRN) inference methods by ranking predicted edges and computing precision, recall, and F1 on the top-100 edges against simulated ground-truth GRNs from the dcanr SimulatorGRN one-gene-knockdown scheme.

## When to use

Use when comparing differential network inference methods (e.g., BoostDiff vs GENIE3, ARACNE, RF Difftrees, EBcoexpress) on data with known ground-truth edges and you need precision/recall/F1 per network-size setting.

## When NOT to use

- The analysis goal is differential expression testing rather than differential network (edge-level) inference — precision/recall on predicted edges is not the right metric
- No ground-truth network is available (e.g., real disease cohorts like GSE156063 or GSE126124) — use pathway enrichment or correlation-distribution diagnostics instead

## Inputs

- Simulated ground-truth GRNs (50/150/300/500 nodes) from dcanr SimulatorGRN one-gene-knockdown
- Simulated disease and control expression matrices (100 samples per condition), tab-separated gene-by-sample files with first column 'Gene'
- Predicted edge rankings from each differential network method (BoostDiff and the nine comparators)

## Outputs

- benchmark_performance.csv with precision, recall, and F1 per method per network-size setting
- Top-100 edge sets per method for comparison against ground-truth edges

## How to apply

Simulate ground-truth GRNs and disease/control expression data with the dcanr SimulatorGRN one-gene-knockdown scheme (100 samples per condition) for 50/150/300-node networks (500 simulations each) and a 500-node network (250 simulations). Run each candidate method — BoostDiff with the recommended low ensemble size n_estimators=50 to avoid overfitting, plus the nine comparators (GENIE3, ARACNE via minet, RF Difftrees, Differential GENIE3, z-score, EBcoexpress, Diffcoex, GGM, chNet) — on the same simulated disease/control pairs. Rank predicted edges per method and restrict evaluation to the top-100 edges. Compute precision, recall, and F1 per method per network-size setting and write the results to benchmark_performance.csv. The key decision point is the complexity setting: the validation expectation is that RF Difftrees leads in the 50-gene setting while BoostDiff leads in the 150-, 300-, and 500-gene settings.

## Related tools

- **BoostDiff** (Differential network inference method under benchmark; run with n_estimators=50 on simulated disease/control pairs) — github.com/scibiome/boostdiff_inference
- **dcanr** (Simulates ground-truth GRNs and disease/control expression data via the one-gene-knockdown scheme)
- **minet** (Provides the ARACNE comparator for edge prediction on the simulated datasets)

## Examples

```
from boostdiff.main_boostdiff import BoostDiff; model.run(file_etha, file_ctrl, output_folder, n_estimators=50, n_features=2500, n_subsamples=20, n_processes=4)
```

## Evaluation signals

- benchmark_performance.csv reproduces the reported pattern: RF Difftrees is the leading method in the 50-node setting while BoostDiff leads in the 150-, 300-, and 500-node settings
- Precision, recall, and F1 are computed per method per network-size setting (50/150/300/500 nodes), not aggregated across settings
- Edge evaluation is restricted to the top-100 ranked edges per method for all methods identically
- Simulations use the dcanr one-gene-knockdown scheme with 100 samples per condition: 500 simulations for 50/150/300-node networks and 250 simulations for the 500-node network

## Limitations

- Method ranking is complexity-dependent: no single method wins across all network sizes (RF Difftrees wins at 50 genes, BoostDiff at 150-500 genes)
- BoostDiff should use a relatively low number of base learners (n_estimators=50) on real transcriptomics data to avoid overfitting
- Large datasets (e.g., the Crohn's disease dataset) require a high-performance computing cluster to run

## Evidence

- [abstract] BoostDiff outperforms existing differential network methods on simulated data evaluated in four different complexity settings.: "BoostDiff outperforms existing differential network methods on simulated data evaluated in four different complexity settings."
- [results] RF Difftrees top in 50-gene setting; BoostDiff best in complex settings.: [evidence span withheld — anchor status: not_found]
- [methods] Rank predicted edges per method and evaluate inference performance on the top-100 edges.: [evidence span withheld — anchor status: not_found]
- [readme] README recommends low base-learner count to avoid overfitting.: "we recommend to set a relatively low number of base learners, e.g. set `n_estimators=50`, so that 50 adaptively boosted differential trees will be built to avoid overfitting."
