---
name: phase-portrait-fitting-quality-evaluation
description: Use when comparing RNA velocity methods on the same scRNA-seq dataset
  and needing a quantitative, gene-level measure of phase portrait fit quality (e.g.,
  reproducing Fig. 2d-g or benchmarking TFvelo against scVelo/Dynamo/UniTVelo/cellDancer).
when_to_use_negative:
- Datasets lacking unspliced/spliced counts where the un/spliced-based baselines (scVelo,
  Dynamo, UniTVelo, cellDancer) cannot run at all, breaking the common-gene comparison
- When the goal is velocity vector accuracy downstream (CBDir/ICCoh trajectory metrics)
  rather than gene-level phase portrait fitting quality
- Very large-scale datasets where TFvelo's runtime is prohibitive compared to scVelo
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TFvelo
  role: Method under test; models gene dynamics from TF abundance to produce phase
    portrait fits of target genes
  repo: github.com/xiaoyeye/TFvelo
- name: ScVelo
  role: Un/spliced-based baseline whose EM dynamical model phase portraits are compared
    against TFvelo's
  repo: theislab/scvelo
- name: velocyto
  role: Steady-state baseline evaluated on the same pancreas dataset for phase-portrait
    comparison
- name: UniTVelo
  role: Time-function baseline evaluated on the same pancreas dataset for phase-portrait
    comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:9f0651f7eeeb0ddcc2944e279af3acef7a1719f93f3945df3cad61948f58de88
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ScVelo
  - TFvelo
  - UniTVelo
  - VeloAE
  - velocyto
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# phase-portrait-fitting-quality-evaluation

## Summary

Evaluates RNA velocity methods by measuring how well each method fits gene dynamics on phase portraits, quantified via intra-class and inter-class distance metrics across methods and genes. Used in the TFvelo paper to show TF-based velocity estimation fits the pancreas phase portraits significantly better than un/spliced-based baselines.

## When to use

Use when comparing RNA velocity methods on the same scRNA-seq dataset and needing a quantitative, gene-level measure of phase portrait fit quality (e.g., reproducing Fig. 2d-g or benchmarking TFvelo against scVelo/Dynamo/UniTVelo/cellDancer).

## When NOT to use

- Datasets lacking unspliced/spliced counts where the un/spliced-based baselines (scVelo, Dynamo, UniTVelo, cellDancer) cannot run at all, breaking the common-gene comparison
- When the goal is velocity vector accuracy downstream (CBDir/ICCoh trajectory metrics) rather than gene-level phase portrait fitting quality
- Very large-scale datasets where TFvelo's runtime is prohibitive compared to scVelo

## Inputs

- E15.5 mouse pancreas scRNA-seq dataset (h5ad with spliced/unspliced counts)
- TFvelo output stored in TFvelo_pancreas/rc.h5ad
- Velocity results from baselines (scVelo, velocyto, VeloAE, UniTVelo) run on the same pancreas dataset
- Metric scripts provided in the repository's 'baselines' folder

## Outputs

- Intra-class and inter-class distance statistics with paired t-test p-values (expected: p = 4.23e-51 and p = 5.65e-117)
- Fitting error, CBDir, ICCoh, and velocity consistency metrics on commonly modeled genes
- Figure panels reproducing Fig. 2d-g and Fig. S5 comparisons
- TFvelo_pancreas_demo/TFvelo.h5ad with pseudotime and phase-portrait fits of best fitted genes

## How to apply

Run the method under test (e.g., TFvelo via TFvelo_run_demo.py --dataset_name pancreas) and each baseline (scVelo, velocyto/VeloAE, UniTVelo, cellDancer) on the same dataset, each in its own environment using code from the 'baselines' folder. Restrict the comparison to genes commonly modeled by all methods so the phase-portrait fits are directly comparable. Compute intra-class distance (compactness of fitted phase portraits within method class) and inter-class distance (separation between classes), then apply paired t-tests across genes. Validation requires recovering the reported significance levels: intra-class distance p = 4.23e-51 and inter-class distance p = 5.65e-117, favoring TFvelo over un/spliced-based baselines. Assemble the metric comparisons into panels reproducing Fig. 2d-g and Fig. S5.

## Related tools

- **TFvelo** (Method under test; models gene dynamics from TF abundance to produce phase portrait fits of target genes) — github.com/xiaoyeye/TFvelo
- **ScVelo** (Un/spliced-based baseline whose EM dynamical model phase portraits are compared against TFvelo's) — theislab/scvelo
- **velocyto** (Steady-state baseline evaluated on the same pancreas dataset for phase-portrait comparison)
- **UniTVelo** (Time-function baseline evaluated on the same pancreas dataset for phase-portrait comparison)

## Examples

```
python TFvelo_run_demo.py --dataset_name pancreas && python TFvelo_analysis_demo.py --dataset_name pancreas
```

## Evaluation signals

- Paired t-test on intra-class distance recovers p = 4.23e-51 (TFvelo lower than baselines)
- Paired t-test on inter-class distance recovers p = 5.65e-117 (TFvelo higher than baselines)
- Metrics computed only on genes commonly modeled by all compared methods
- Produced panels match Fig. 2d-g and Fig. S5 comparisons
- TFvelo results stored at TFvelo_pancreas/rc.h5ad and TFvelo_pancreas_demo/TFvelo.h5ad as expected

## Limitations

- TFvelo is slower than scVelo and other baselines, a weakness for large-scale datasets; parallelization over more CPUs reduces runtime
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14), so per-gene fit quality varies
- Baselines each require a separate environment with their required packages, increasing reproduction overhead

## Evidence

- [results] lower intra-class distance and higher inter-class distance (paired t-test p = 4.23e-51 and p = 5.65e-117): [evidence span withheld — anchor status: not_found]
- [methods] Compute the evaluation metrics on the genes commonly modeled by all methods: [evidence span withheld — anchor status: not_found]
- [readme] the visualization of results can be obtained by: "the visualization of results can be obtained by 
``npython TFvelo_analysis_demo.py --dataset_name pancreas"
- [readme] The code for baselines and metrics are provided in the folder "baselines": "The code for baselines and metrics are provided in the folder "baselines". You may need to create an environment and install the required packages for each baseline method."
- [discussion] the time efficiency of TFvelo is lower than baseline approaches in the same framework, e.g., scVelo: "the time efficiency of TFvelo is lower than baseline approaches in the same framework, e.g., scVelo. This could be a weakness when being applied to the large-scale datasets"
