---
name: confidence-interval-aggregation-over-replications
description: Use when an evaluation protocol repeats a clustering/analysis over many
  independent replications (e.g. 100 simulated datasets) and per-replication scores
  must be aggregated into means with 95% confidence intervals for method comparison.
when_to_use_negative:
- When only a single dataset replication is available (e.g. the real-data Hockley/Usoskin
  or Tasic analyses) — no confidence interval over replications can be formed
- When the goal is method development of TransferCluster itself (NMF, KTA-based theta
  selection) rather than statistical evaluation of its results
edam_operation: http://edamontology.org/operation_3664
tools:
- name: scRNA (cmd_generate_data.py)
  role: Generates the 100 simulated count-level scRNA-Seq datasets per setting whose
    per-replication ARI scores are aggregated
  repo: https://github.com/nicococo/scRNA/blob/master/scRNA/cmd_generate_data.py
- name: SC3
  role: Downstream clustering method whose per-replication ARI on target cells supplies
    the raw scores for mean/CI aggregation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:e6c5219dce4db5c5f24bf0e9aeab41c0cca9d136d5f9ed73d955da44079daf41
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SC3
  - scRNA (cmd_generate_data.py)
  edam_operation: http://edamontology.org/operation_3664
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# confidence-interval-aggregation-over-replications

## Summary

Aggregate Adjusted Rand Index (ARI) scores over many independent dataset replications (100 simulated scRNA-Seq sets per overlap setting) and report mean ARI with 95% confidence intervals per method, overlap setting, and target size, enabling statistically grounded comparison of TransferCluster vs TargetCluster vs ConcatenateCluster.

## When to use

Use when an evaluation protocol repeats a clustering/analysis over many independent replications (e.g. 100 simulated datasets) and per-replication scores must be aggregated into means with 95% confidence intervals for method comparison.

## When NOT to use

- When only a single dataset replication is available (e.g. the real-data Hockley/Usoskin or Tasic analyses) — no confidence interval over replications can be formed
- When the goal is method development of TransferCluster itself (NMF, KTA-based theta selection) rather than statistical evaluation of its results

## Inputs

- Per-replication ARI scores against known ground-truth target labels, for TransferCluster, TargetCluster (SC3 on target), and ConcatenateCluster (SC3 on source+target)
- 100 simulated scRNA-Seq count datasets per overlap setting (complete, incomplete, no overlap; 10,000 genes, 1800 cells) generated with scRNA/cmd_generate_data.py
- Target datasets downsampled to 10, 50, 100, 200, 400, 600, or 800 cells

## Outputs

- Aggregated ARI results table (simulation_ari_results.csv) with mean ARI and 95% confidence intervals per setting x target size x method
- Comparison against Figure 2B-D of the paper as the expected qualitative outcome

## How to apply

For each experimental cell (overlap setting x target size x method), collect the per-replication ARI computed against ground-truth labels on the target cells only (TransferCluster, TargetCluster = SC3 on target alone, ConcatenateCluster = SC3 on concatenated source+target). Aggregate over the 100 repetitions and compute 95% confidence intervals around the mean ARI. Write the aggregated table to a single results file (simulation_ari_results.csv). Judge correctness by the paper's validation criterion: TransferCluster should show higher mean ARI than both baselines for complete-overlap and incomplete-overlap settings across target sizes (10-800 cells), while in the no-overlap setting its confidence interval should overlap TargetCluster's, indicating no significant degradation.

## Related tools

- **scRNA (cmd_generate_data.py)** (Generates the 100 simulated count-level scRNA-Seq datasets per setting whose per-replication ARI scores are aggregated) — https://github.com/nicococo/scRNA/blob/master/scRNA/cmd_generate_data.py
- **SC3** (Downstream clustering method whose per-replication ARI on target cells supplies the raw scores for mean/CI aggregation)

## Examples

```
python scripts/experiments/main_wrapper_generated_data.py && python scripts/plots/main_plots_generated_data.py
```

## Evaluation signals

- Aggregated table contains one mean ARI + 95% CI entry per overlap setting x target size (10-800 cells) x method over exactly 100 repetitions
- TransferCluster mean ARI > TargetCluster and > ConcatenateCluster for complete-overlap and incomplete-overlap settings across target sizes
- In the no-overlap setting, TransferCluster's 95% CI overlaps TargetCluster's CI (no significant degradation)
- ARI is computed only on target cells against the known ground-truth labels, never on source cells
- Pattern matches Figure 2B-D of the paper when plotted via scripts/plots/main_plots_generated_data.py

## Limitations

- Coverage of uncertainty is limited to simulation variability; no software version numbers are stated for SC3, so exact reproducibility may vary
- If source and target share only a small set of transcripts, information loss is inevitable since only the shared gene set can be used, which will be reflected in the no-overlap setting's CI behavior

## Evidence

- [methods] 100 sets of simulated data were generated for each of the three settings simulating the expression levels of 10 000 genes in 1800 cells: "100 sets of simulated data were generated for each of the three settings simulating the expression levels of 10 000 genes in 1800 cells"
- [discussion] Transfer from large source outperformed baselines: "transferring knowledge from a large well-annotated source dataset to a smaller target dataset was not only more successful than applying SC3"
- [readme] Reproduction scripts for generated-data experiments: "To fully reproduce the experiments of our study you can find the corresponding scripts at the following links"
