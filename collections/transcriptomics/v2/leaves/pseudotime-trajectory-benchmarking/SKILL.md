---
name: pseudotime-trajectory-benchmarking
description: 'Use when a splicing-free velocity or trajectory method must be validated
  against a multi-omics baseline: compare pseudotime ordering, streamplot direction,
  and gene phase-portrait dynamics on a shared dataset.'
when_to_use_negative:
- The dataset has reliable spliced/unspliced counts and splicing-based models (scVelo/velocyto)
  are appropriate — TFvelo's benchmarking here targets datasets lacking splicing information
- Large-scale datasets where TFvelo's lower time efficiency versus scVelo is prohibitive
- Genes known to be poorly modeled — TFvelo cannot fit the dynamics of some genes
  well (Fig. S14)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: TFvelo
  role: Method under test — infers pseudotime, trajectory, and gene dynamics from
    RNA-only counts using TF regulation
  repo: github.com/xiaoyeye/TFvelo
- name: MultiVelo
  role: Multi-omics baseline for comparison of pseudotime and differentiation directions;
    code in the package's 'baselines' folder
- name: scVelo
  role: Framework on which TFvelo is built; also a reference baseline for time-efficiency
    comparisons
  repo: github.com/theislab/scvelo
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:b942fb62871cadec79d126ab927448bae20562003a44590fbf79d3ad9af42e07
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MultiVelo
  - TFvelo
  - scVelo
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# pseudotime-trajectory-benchmarking

## Summary

Benchmark a RNA-velocity/trajectory method (e.g., TFvelo, RNA-only) against a multi-omics baseline (MultiVelo) by comparing inferred pseudotime ordering, stream direction on UMAP, and per-gene phase-portrait fits on example genes such as AHI1, NTRK2, and GRIN2B.

## When to use

Use when a splicing-free velocity or trajectory method must be validated against a multi-omics baseline: compare pseudotime ordering, streamplot direction, and gene phase-portrait dynamics on a shared dataset.

## When NOT to use

- The dataset has reliable spliced/unspliced counts and splicing-based models (scVelo/velocyto) are appropriate — TFvelo's benchmarking here targets datasets lacking splicing information
- Large-scale datasets where TFvelo's lower time efficiency versus scVelo is prohibitive
- Genes known to be poorly modeled — TFvelo cannot fit the dynamics of some genes well (Fig. S14)

## Inputs

- 10x multi-omics embryonic mouse brain scRNA-seq data as an AnnData .h5ad object
- TF-target databases (ENCODE and ChEA, ENCODE.zip decompressed before use)
- MultiVelo baseline code from the package's 'baselines' folder

## Outputs

- rc.h5ad — TFvelo velocity result file
- TFvelo.h5ad — annotated result with pseudotime after TFvelo_analysis_demo.py
- UMAP pseudotime and streamplot figures in the 'figures' folder
- Phase-portrait fits for target genes AHI1, NTRK2, GRIN2B under both methods

## How to apply

Preprocess the 10x multi-omics embryonic mouse brain dataset into an AnnData (.h5ad) object by adding a load branch to the preprocess() function in TFvelo_run_demo.py, per the README's customization procedure. Run TFvelo on RNA-only counts with default parameters (TF_databases ENCODE and ChEA, WX_method lsq_linear, init_weight_method correlation) to yield rc.h5ad, then run the MultiVelo baseline from the package's 'baselines' folder on the same data. Visualize TFvelo's pseudotime and streamplot on UMAP with TFvelo_analysis_demo.py, storing the annotated result as TFvelo.h5ad. Plot phase portraits of AHI1, NTRK2, and GRIN2B under both methods; accept the run when TFvelo's pseudotime ordering and differentiation direction match the MultiVelo/reference direction (Fig. 5a-c) and gene fits show the desired clockwise TF-to-target curve — e.g., TFvelo captures the correct AHI1 dynamic where MultiVelo constructs a mistaken cyclic dynamic.

## Related tools

- **TFvelo** (Method under test — infers pseudotime, trajectory, and gene dynamics from RNA-only counts using TF regulation) — github.com/xiaoyeye/TFvelo
- **MultiVelo** (Multi-omics baseline for comparison of pseudotime and differentiation directions; code in the package's 'baselines' folder)
- **scVelo** (Framework on which TFvelo is built; also a reference baseline for time-efficiency comparisons) — github.com/theislab/scvelo

## Examples

```
python TFvelo_run_demo.py --dataset_name your_dataset_name && python TFvelo_analysis_demo.py --dataset_name your_dataset_name
```

## Evaluation signals

- TFvelo's inferred pseudotime ordering on the mouse brain UMAP agrees with MultiVelo's pseudotime and the differentiation directions in Fig. 5a-c
- Streamplot arrows on UMAP point in the same differentiation direction as the reference
- Phase portraits of AHI1, NTRK2, and GRIN2B show the desired clockwise TF-to-target curve; TFvelo recovers the correct (non-cyclic) AHI1 dynamic that MultiVelo mistakenly fits as cyclic
- Run artifacts exist: TFvelo_mousebrain/rc.h5ad and TFvelo_mousebrain_demo/TFvelo.h5ad with figures saved to 'figures'
- Default hyperparameters were used (WX_method=lsq_linear, init_weight_method=correlation, TF_databases=ENCODE and ChEA), matching 'default parameters can reproduce the results in manuscript'

## Limitations

- TFvelo is slower than scVelo baselines in the same framework, which could be a weakness on large-scale datasets; running with more CPUs in parallel reduces time
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14)
- The 10x multi-omics dataset is referenced in the paper without accession or identifier, so obtaining the exact input requires external sourcing
- The current linear model for TF weight estimation may be insufficient; no implementation for alternatives is provided

## Evidence

- [results] TFvelo matches MultiVelo's pseudotime and differentiation directions without ATAC-seq, and on example genes it captures dynamics MultiVelo misses: MultiVelo mistakenly constructs a cyclic dynamic for AHI1 while TFvelo captures the correct dynamic.: [evidence span withheld — anchor status: not_found]
- [readme] you can define a personalized name for the dataset, and simply add the following codes into the preprocess() function in TFvelo_run_demo.py: "simply add the following codes into the preprocess() function in TFvelo_run_demo.py"
- [readme] This will show the pseudotime and streamplot on UMAP, and also the phase portrait fitting of best fitted genes.: "This will show the pseudotime and streamplot on UMAP, and also the phase portrait fitting of best fitted genes."
- [discussion] TFvelo can fit the gene dynamics well, showing a desired clockwise curve on the phase portrait between the TFs representation and the target: "TFvelo can fit the gene dynamics well, showing a desired clockwise curve on the phase portrait between the TFs representation and the target"
- [readme] The code for baselines and metrics are provided in the folder "baselines".: "The code for baselines and metrics are provided in the folder "baselines"."
