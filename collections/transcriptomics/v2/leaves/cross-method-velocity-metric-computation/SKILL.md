---
name: cross-method-velocity-metric-computation
description: Use when benchmarking two or more RNA velocity methods on the same dataset
  and you need comparable per-gene metrics (e.g., phase-portrait fitting quality,
  directional coherence) restricted to commonly modeled genes.
when_to_use_negative:
- Only a single velocity method is applied and no cross-method comparison is intended
  — the metrics are defined relative to multiple methods on commonly modeled genes
- The dataset contains splicing information that scVelo-style dynamics modeling can
  fully exploit and no benchmarking against regulation-based fitting is needed
- Genes modeled by only a subset of methods must be included — restricting to commonly
  modeled genes is what makes the comparison valid, so partial-coverage genes should
  be excluded rather than forced in
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: TFvelo
  role: Method under evaluation; its phase-portrait fitting is compared against baselines
    via the computed metrics
  repo: github.com/xiaoyeye/TFvelo
- name: ScVelo
  role: Baseline un/spliced-based RNA velocity method run on the same pancreas dataset
    for metric comparison
  repo: theislab/scvelo
- name: UniTVelo
  role: Baseline time-function-based velocity method run on the same dataset; its
    code is in the 'baselines' folder
- name: velocyto
  role: Baseline steady-state velocity method executed on the same dataset for benchmarking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:385f0e3b34e5b4d2baa198b27ba5367f8dfafd43ae798aa90af37c6aced69e94
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ScVelo
  - TFvelo
  - UniTVelo
  - VeloAE
  - velocyto
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# cross-method-velocity-metric-computation

## Summary

Computes quantitative RNA-velocity evaluation metrics — intra-class distance, inter-class distance, fitting error, CBDir, ICCoh, and velocity consistency — on the genes commonly modeled by all compared methods, to benchmark TFvelo against un/spliced-based baselines (scVelo, Dynamo, UniTVelo, cellDancer) as in Fig. 2d-g and Fig. S5.

## When to use

Use when benchmarking two or more RNA velocity methods on the same dataset and you need comparable per-gene metrics (e.g., phase-portrait fitting quality, directional coherence) restricted to commonly modeled genes.

## When NOT to use

- Only a single velocity method is applied and no cross-method comparison is intended — the metrics are defined relative to multiple methods on commonly modeled genes
- The dataset contains splicing information that scVelo-style dynamics modeling can fully exploit and no benchmarking against regulation-based fitting is needed
- Genes modeled by only a subset of methods must be included — restricting to commonly modeled genes is what makes the comparison valid, so partial-coverage genes should be excluded rather than forced in

## Inputs

- Annotated single-cell dataset used identically across methods (pancreas E15.5 mouse, AnnData .h5ad)
- Velocity results/h5ad outputs from each method (e.g., TFvelo_pancreas/rc.h5ad, TFvelo_pancreas_demo/TFvelo.h5ad, scVelo/UniTVelo/cellDancer outputs)
- Metric scripts from the 'baselines' folder of the TFvelo repository
- Set of genes commonly modeled by all compared methods

## Outputs

- Per-method metric values: intra-class distance, inter-class distance, fitting error, CBDir, ICCoh, velocity consistency
- Paired t-test significance values for intra-class and inter-class distance comparisons
- Comparison figure panels reproducing Fig. 2d-g and Fig. S5

## How to apply

Run each velocity method (TFvelo, scVelo, UniTVelo, cellDancer, etc.) on the same preprocessed dataset — here the E15.5 mouse pancreas dataset — using each method's own environment and the code provided in the 'baselines' folder of the TFvelo repository. Restrict metric computation to the genes commonly modeled by all methods so the comparison is on equal footing. Compute the six metrics — intra-class distance, inter-class distance, fitting error, CBDir, ICCoh, and velocity consistency — with the metric scripts in the baselines folder. Assess significance of intra-class vs. inter-class distance differences with paired t-tests (TFvelo vs. baselines on pancreas: p = 4.23e-51 for intra-class distance and p = 5.65e-117 for inter-class distance), then assemble metric comparisons into figure panels reproducing Fig. 2d-g and Fig. S5.

## Related tools

- **TFvelo** (Method under evaluation; its phase-portrait fitting is compared against baselines via the computed metrics) — github.com/xiaoyeye/TFvelo
- **ScVelo** (Baseline un/spliced-based RNA velocity method run on the same pancreas dataset for metric comparison) — theislab/scvelo
- **UniTVelo** (Baseline time-function-based velocity method run on the same dataset; its code is in the 'baselines' folder)
- **velocyto** (Baseline steady-state velocity method executed on the same dataset for benchmarking)

## Examples

```
python TFvelo_run_demo.py --dataset_name pancreas; python TFvelo_analysis_demo.py --dataset_name pancreas  # then run baseline scripts from the 'baselines' folder and compute metrics on commonly modeled genes
```

## Evaluation signals

- Intra-class distance paired t-test against baselines recovers p = 4.23e-51 on the pancreas dataset
- Inter-class distance paired t-test against baselines recovers p = 5.65e-117 on the pancreas dataset
- Metrics are computed only on genes commonly modeled by all methods, not on each method's full gene set
- Assembled panels visually match Fig. 2d-g and Fig. S5 comparisons from the manuscript
- Fitting error, CBDir, ICCoh, and velocity consistency are all reported for every compared method

## Limitations

- Each baseline requires a separate conda environment with its own package requirements, so environment drift across methods can affect comparability
- Restricting to commonly modeled genes shrinks the evaluated gene set and may understate differences in gene coverage between methods
- TFvelo is slower than baseline approaches such as scVelo in the same framework, which is a practical constraint when scaling the benchmark to large datasets

## Evidence

- [methods] common-genes restriction: [evidence span withheld — anchor status: not_found]
- [results] significance levels: [evidence span withheld — anchor status: not_found]
- [readme] baselines folder: "The code for baselines and metrics are provided in the folder "baselines". You may need to create an environment and install the required packages for each baseline method."
- [methods] workflow validation: [evidence span withheld — anchor status: not_found]
- [discussion] slower than baselines: "the time efficiency of TFvelo is lower than baseline approaches in the same framework, e.g., scVelo. This could be a weakness when being applied to the large-scale datasets"
