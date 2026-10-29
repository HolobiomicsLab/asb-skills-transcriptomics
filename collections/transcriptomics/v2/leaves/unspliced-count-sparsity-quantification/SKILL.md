---
name: unspliced-count-sparsity-quantification
description: Use when loading a new scRNA-seq h5ad into TFvelo_run_demo.py and you
  need to measure zero-fraction in unspliced counts to justify applying a regulation-based
  (TF) velocity model instead of a splicing-based one.
when_to_use_negative:
- The dataset already has dense, well-populated unspliced counts where a splicing-based
  method like ScVelo's dynamical EM model will fit well
- The input lacks any unspliced/spliced layers and the goal is pure TF-weight analysis
  rather than quantifying splicing-count coverage
- You only need trajectory visualization (pseudotime/streamplot) from an already-fitted
  TFvelo.h5ad via TFvelo_analysis_demo.py
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TFvelo
  role: framework whose TFvelo_run_demo.py preprocess() step loads the raw dataset
    and quantifies unspliced count sparsity before the TF model fit
  repo: github.com/xiaoyeye/TFvelo
- name: anndata
  role: reads the raw h5ad file containing spliced/unspliced layers in the preprocess()
    loader branch
- name: ScVelo
  role: splicing-based baseline whose dependence on unspliced counts motivates the
    sparsity check; part of TFvelo's framework and baselines folder
  repo: theislab/scvelo
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:a3d18a4915b867ec0683cb956c4832f757751d42ac655e42dd10d067c138c63c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ScVelo
  - TFvelo
  - anndata
  - scanpy
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# unspliced-count-sparsity-quantification

## Summary

Quantifies how sparse the unspliced count layer of a raw single-cell h5ad dataset is, as a pre-check during TFvelo's preprocess() step to determine whether splicing-based velocity (e.g., ScVelo dynamical modeling) is viable or a TF-regulation-based fit is needed.

## When to use

Use when loading a new scRNA-seq h5ad into TFvelo_run_demo.py and you need to measure zero-fraction in unspliced counts to justify applying a regulation-based (TF) velocity model instead of a splicing-based one.

## When NOT to use

- The dataset already has dense, well-populated unspliced counts where a splicing-based method like ScVelo's dynamical EM model will fit well
- The input lacks any unspliced/spliced layers and the goal is pure TF-weight analysis rather than quantifying splicing-count coverage
- You only need trajectory visualization (pseudotime/streamplot) from an already-fitted TFvelo.h5ad via TFvelo_analysis_demo.py

## Inputs

- raw single-cell h5ad file with unspliced/spliced count layers (read via anndata.read_h5ad)
- dataset name passed via --dataset_name to TFvelo_run_demo.py

## Outputs

- sparsity metric (fraction of zeros in the unspliced count matrix) reported for the raw dataset
- recovered counts stored as rc.h5ad in the 'TFvelo_' + dataset_name output folder

## How to apply

Add a dataset-specific branch to the preprocess() function of TFvelo_run_demo.py (e.g., 'if args.dataset_name == gastrulation_erythroid: adata = ad.read_h5ad(...)') and inspect the unspliced layer of the loaded AnnData object. Compute the fraction of zero entries in the unspliced count matrix of the raw data before fitting; TFvelo treats high unspliced sparsity as motivation for modeling gene dynamics from transcription-factor expression levels rather than splicing information. Run the fit with default hyperparameters (init_weight_method=correlation, WX_method=lsq_linear, TF_databases=ENCODE+ChEA) so sparsity quantification is reported in the same run that produces rc.h5ad in the dataset-specific output folder (e.g., 'TFvelo_pancreas/rc.h5ad'). If unspliced counts are absent or extremely sparse, splicing-based baselines in the 'baselines' folder (ScVelo EM) are expected to fail, reinforcing the choice of TFvelo.

## Related tools

- **TFvelo** (framework whose TFvelo_run_demo.py preprocess() step loads the raw dataset and quantifies unspliced count sparsity before the TF model fit) — github.com/xiaoyeye/TFvelo
- **anndata** (reads the raw h5ad file containing spliced/unspliced layers in the preprocess() loader branch)
- **ScVelo** (splicing-based baseline whose dependence on unspliced counts motivates the sparsity check; part of TFvelo's framework and baselines folder) — theislab/scvelo

## Examples

```
python TFvelo_run_demo.py --dataset_name gastrulation_erythroid
```

## Evaluation signals

- The raw AnnData object is loaded via ad.read_h5ad inside the dataset-specific preprocess() branch before any model fitting occurs
- The sparsity statistic is computed on the unspliced layer of the RAW data (not the recovered counts) and is reported alongside the fit for the chosen --dataset_name
- The run completes with default hyperparameters (init_weight_method=correlation, WX_method=lsq_linear, TF_databases=ENCODE+ChEA) and writes rc.h5ad to the 'TFvelo_' + dataset_name folder
- Sparsity findings are consistent with the motivation that regulation-based fitting is needed when unspliced counts are too sparse for ScVelo-style dynamics

## Limitations

- TFvelo is slower than baseline approaches in the same framework, e.g., scVelo, which could be a weakness when applied to large-scale datasets
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14), so sparsity quantification does not guarantee every gene will be recoverable
- Datasets entirely lacking unspliced/spliced counts cannot use splicing-based velocity models, making the sparsity check a one-way decision toward regulation-based fitting

## Evidence

- [abstract] abstract: "we propose TFvelo, which expands RNA velocity concept to various single-cell datasets without relying on splicing information"
- [intro] intro: "RNA velocity models always treat each gene independently and do not take the underlying regulation into consideration"
- [readme] readme: "This will automatically download, preprocess and run TF model on pancrease dataset. The result will be stored in 'TFvelo_pancreas/rc.h5ad'."
