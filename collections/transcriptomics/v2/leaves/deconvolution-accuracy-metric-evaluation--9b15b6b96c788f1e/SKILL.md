---
name: deconvolution-accuracy-metric-evaluation
description: Use when a deconvolution method (e.g., uniPort label_transfer) has predicted
  per-spot cell-type proportions for spatial data with known ground-truth proportions,
  and quantitative accuracy metrics are needed to compare against baselines.
when_to_use_negative:
- The spatial data has no ground-truth cell-type proportions, so metrics cannot be
  validated (e.g., real 10x Visium slices with only clusters, not synthetic benchmarks)
- You are still at the preprocessing/HVG-selection stage and no deconvolution output
  proportion vectors exist yet
- Evaluating integration quality (batch correction, latent alignment) rather than
  deconvolution proportion accuracy — use domain-classifier or silhouette metrics
  instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3516
tools:
- name: uniPort
  role: Provides the OT-plan-based label transfer that produces the deconvoluted proportions,
    plus metric utilities for evaluation
  repo: https://github.com/caokai1073/uniport
- name: scanpy
  role: Preprocessing (normalize_total, log1p with offset 1, MaxAbsScaler scaling,
    k=2000 HVG selection) of scRNA and spot data before deconvolution and metric computation
- name: scikit-learn
  role: Metric and classifier utilities (e.g., MaxAbsScaler, kNN classifier) used
    in the evaluation workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:4034c881c18da573fb457c880ea3e896fd8de4db15fb058344b16d95a98d6e93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/caokai1073/uniport
  tools:
  - scanpy
  - scikit-learn
  - uniPort
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3516
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# deconvolution-accuracy-metric-evaluation

## Summary

Benchmark the accuracy of spatial transcriptomic cell-type deconvolution (e.g., uniPort's OT-plan-based label transfer) by computing PCC, SSIM, RMSE, and JSD between predicted and ground-truth spot cell-type proportions, as demonstrated on the synthetic STARmap dataset against Tangram and SpaOTsc.

## When to use

Use when a deconvolution method (e.g., uniPort label_transfer) has predicted per-spot cell-type proportions for spatial data with known ground-truth proportions, and quantitative accuracy metrics are needed to compare against baselines.

## When NOT to use

- The spatial data has no ground-truth cell-type proportions, so metrics cannot be validated (e.g., real 10x Visium slices with only clusters, not synthetic benchmarks)
- You are still at the preprocessing/HVG-selection stage and no deconvolution output proportion vectors exist yet
- Evaluating integration quality (batch correction, latent alignment) rather than deconvolution proportion accuracy — use domain-classifier or silhouette metrics instead

## Inputs

- Predicted per-spot cell-type proportion matrix (n_spots x n_celltypes) from deconvolution, e.g., via uniport label_transfer on the saved OT plan
- Ground-truth per-spot cell-type proportion matrix for the benchmark spatial data (e.g., synthetic STARmap, 189 spots x 882 genes)
- Published baseline metrics for comparison (e.g., Tangram PCC 0.619, SpaOTsc PCC 0.409)

## Outputs

- Table of accuracy metrics (PCC, SSIM, RMSE, JSD) for the evaluated method on the benchmark
- Comparison table with baseline methods (uniPort, Tangram, SpaOTsc rows)

## How to apply

After running the deconvolution (e.g., uniport.Run with save_OT=True on the synthetic STARmap scRNA reference of 14249 cells x 34041 genes and 189 spots x 882 genes, then uniport label_transfer mapping OT plan rows to reference cell-type annotations), extract the predicted per-spot cell-type proportion vectors. Compute PCC, RMSE, and JSD between predicted and ground-truth proportion vectors using the uniPort metric utilities (metrics.label_transfer); SSIM can additionally be computed for spatial map comparison. Tabulate the metrics alongside published baselines — for STARmap, uniPort PCC 0.449 / RMSE 0.157 / JSD 0.569 versus Tangram PCC 0.619 and SpaOTsc PCC 0.409 — and judge correctness by reproducing the reported values within tolerance. The rationale is that multiple complementary metrics capture proportion accuracy (PCC/RMSE), structural similarity (SSIM), and distributional divergence (JSD) rather than relying on a single score.

## Related tools

- **uniPort** (Provides the OT-plan-based label transfer that produces the deconvoluted proportions, plus metric utilities for evaluation) — https://github.com/caokai1073/uniport
- **scanpy** (Preprocessing (normalize_total, log1p with offset 1, MaxAbsScaler scaling, k=2000 HVG selection) of scRNA and spot data before deconvolution and metric computation)
- **scikit-learn** (Metric and classifier utilities (e.g., MaxAbsScaler, kNN classifier) used in the evaluation workflow)

## Examples

```
import uniport as up; adata, OT = up.Run(adatas=[scrna, spots], adata_cm=adata_cm, save_OT=True); pred = up.label_transfer(ot=OT, meta=scrna.obs['celltype'])
```

## Evaluation signals

- Reproduced uniPort metrics match the reported synthetic STARmap values within tolerance: PCC ~0.449, RMSE ~0.157, JSD ~0.569
- The benchmark comparison table contains rows for all three methods: uniPort, Tangram, and SpaOTsc
- Predicted proportions for each spot are non-negative and sum to 1 (valid probability vectors) before metric computation
- Relative ordering holds: uniPort PCC (0.449) is below Tangram (0.619) but above SpaOTsc (0.409)

## Limitations

- SSIM was reported in the benchmark framing but PCC/RMSE/JSD values are the concrete reproduced figures; ensure metric definitions match those used in the original benchmarking paper
- Results are specific to the synthetic STARmap benchmark (189 spots); performance on real tissue data may differ
- Storing the full OT plan (save_OT=True) requires substantial memory — verify memory capacity before computing label transfers for evaluation

## Evidence

- [abstract] uniPort provides a flexible label transfer framework to deconvolute heterogeneous spatial transcriptomic data using an optimal transport plan, instead of embedding latent space.: "uniPort provides a flexible label transfer framework to deconvolute heterogeneous spatial transcriptomic data using an optimal transport plan, instead of embedding latent space."
- [discussion] Synthetic STARmap dataset dimensions: [evidence span withheld — anchor status: not_found]
- [discussion] Memory caveat for OT plan: "Note that storing a dense matrix T needs more memory. Therefore, computing capacity with respect to memory should be ascertained before using this option"
- [readme] save_OT parameter description: "**save_OT**: if True, output a global OT plan. Need more memory. Default: False"
