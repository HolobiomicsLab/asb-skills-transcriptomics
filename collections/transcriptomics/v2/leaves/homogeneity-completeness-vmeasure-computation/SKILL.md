---
name: homogeneity-completeness-vmeasure-computation
description: Use when a partitioned clustering solution must be scored against a known
  ground-truth labeling (e.g. Splatter-simulated c1–c5 populations), and per-method
  Homogeneity/Completeness/V-Measure comparisons across competing methods are needed.
when_to_use_negative:
- No ground-truth labeling exists — Homogeneity/Completeness/V-Measure are external
  metrics requiring a reference partition; use internal stability scores (reassign,
  TFIDF5/TFIDF10, SCCAF) instead
- Comparing soft/probabilistic cluster assignments rather than one definitive label
  per cell (the binary assignment matrix X of shape I * N assumption does not hold)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ClusterEnsembles
  role: Provides the ensemble clustering baselines (HGPA, MCLA, HBGF) run with default
    options to compete against scTriangulate on the metrics
  repo: https://github.com/tsano430/ClusterEnsembles
- name: scTriangulate
  role: Generates the candidate reconciled clustering (lazy_run on four conflicting
    annotations) whose labels are scored against the simulated truth
  repo: https://github.com/frankligy/scTriangulate
- name: Splatter
  role: Generates the simulated dataset (splatSimulateGroup, version 1.16.1) supplying
    the ground-state-truth populations c1–c5 used as reference labels
- name: scanpy
  role: Provides the sklearn-metric-compatible Python environment and AnnData handling
    for assembling the benchmark metrics table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:01c413736b60b4e7ecae0051feed6dc24704bdec4cab7eded0a048d869bd2d54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tsano430/ClusterEnsembles
  tools:
  - ClusterEnsembles
  - Splatter
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# homogeneity-completeness-vmeasure-computation

## Summary

Computes external clustering validity metrics — Homogeneity (H(C|K)), Completeness (H(K|C)), and their harmonic mean (V-Measure) — to benchmark a reconciliation method (scTriangulate) against ensemble clustering baselines (HGPA, MCLA, HBGF) on a simulated ground-truth clustering (Splatter clusters c1–c5).

## When to use

Use when a partitioned clustering solution must be scored against a known ground-truth labeling (e.g. Splatter-simulated c1–c5 populations), and per-method Homogeneity/Completeness/V-Measure comparisons across competing methods are needed.

## When NOT to use

- No ground-truth labeling exists — Homogeneity/Completeness/V-Measure are external metrics requiring a reference partition; use internal stability scores (reassign, TFIDF5/TFIDF10, SCCAF) instead
- Comparing soft/probabilistic cluster assignments rather than one definitive label per cell (the binary assignment matrix X of shape I * N assumption does not hold)

## Inputs

- Ground-truth cluster labels (Splatter simulated populations c1–c5 from splatSimulateGroup, batchCells=3000)
- Predicted cluster labels from scTriangulate lazy_run (v0.9.0, default settings, win_fraction_cutoff=0.25)
- Predicted cluster labels from ClusterEnsembles HGPA, MCLA, and HBGF (default options)
- Four partially overlapping input annotations: Broad (c1+c2+c3 merged), Median (c2+c3 merged), Fine (c5 subdivided), Excessive (c2 and c3 subdivided)

## Outputs

- simulation_benchmark_metrics.csv with Homogeneity, Completeness, and V-Measure rows for scTriangulate, HGPA, MCLA, and HBGF, plus the number of clusters each solution recovers

## How to apply

Generate the reference partition from Splatter's splatSimulateGroup (batchCells=3000, 5 clusters, group.prob (0.23,0.15,0.15,0.23,0.24), de.prob (0.15,0.15,0.15,0.2,0.4)) and treat it as the true labels C. Run each candidate method — scTriangulate lazy_run (v0.9.0, default settings including win_fraction_cutoff=0.25) and ClusterEnsembles HGPA/MCLA/HBGF (default options) — on the same four conflicting annotations (Broad merging c1+c2+c3, Median merging c2+c3, Fine subdividing c5, Excessive subdividing c2 and c3) to obtain predicted labels K. Compute metrics per sklearn conventions: Homogeneity = 1 − H(C|K)/H(C), Completeness = 1 − H(K|C)/H(K), and V-Measure as the harmonic mean of the two, per method against the simulated truth. Record each method's metrics alongside the number of clusters recovered in simulation_benchmark_metrics.csv; scTriangulate is judged superior if its V-Measure matches or exceeds HGPA, MCLA, and HBGF while recovering the c1–c5 structure without artifacts.

## Related tools

- **ClusterEnsembles** (Provides the ensemble clustering baselines (HGPA, MCLA, HBGF) run with default options to compete against scTriangulate on the metrics) — https://github.com/tsano430/ClusterEnsembles
- **scTriangulate** (Generates the candidate reconciled clustering (lazy_run on four conflicting annotations) whose labels are scored against the simulated truth) — https://github.com/frankligy/scTriangulate
- **Splatter** (Generates the simulated dataset (splatSimulateGroup, version 1.16.1) supplying the ground-state-truth populations c1–c5 used as reference labels)
- **scanpy** (Provides the sklearn-metric-compatible Python environment and AnnData handling for assembling the benchmark metrics table)

## Examples

```
from sklearn.metrics import homogeneity_completeness_v_measure; h, c, v = homogeneity_completeness_v_measure(true_labels, pred_labels)
```

## Evaluation signals

- simulation_benchmark_metrics.csv contains Homogeneity/Completeness/V-Measure rows for all four methods (scTriangulate, HGPA, MCLA, HBGF)
- scTriangulate's V-Measure matches or exceeds HGPA, MCLA, and HBGF, consistent with the paper's reported result
- scTriangulate recovers exactly the five ground-truth populations c1–c5 without artifacts, whereas ensemble baselines fail to mix-and-match
- Metrics follow sklearn conventions (H(C), H(C|K), H(K|C), harmonic mean for V-Measure), so all values lie in [0, 1] and V-Measure lies between Homogeneity and Completeness

## Limitations

- The simulated benchmark's conclusions are bounded by the specific constructed annotations (Broad/Median/Fine/Excessive); the paper notes programmatic default analytical settings may not be optimal for all study designs
- Ensemble consensus (HGPA/MCLA/HBGF) represents the 'consensus' rather than the most stable solution, so V-Measure comparisons only test this benchmark design, not all integration scenarios
- Metrics are computed against a synthetic ground truth from Splatter, which may not reflect behavior on real multimodal datasets

## Evidence

- [methods] ClusterEnsembles baselines: "using the Python package ClusterEnsembles with default options (https://github.com/tsano430/ClusterEnsembles)."
- [abstract] scTriangulate superiority claim: "When evaluated on diverse multimodal technologies, scTriangulate outperforms alternative approaches to identify high-confidence cell-populations"
- [abstract] Consensus vs stability framing: "Rather than ensemble approaches which select the "consensus", scTriangulate picks the most stable solution through coalitional iteration."
- [readme] Reconciliation of conflicting annotations: "scTriangulate is a Python package to mix-and-match conflicting clustering results in single cell analysis, and generate reconciled clustering solutions."
