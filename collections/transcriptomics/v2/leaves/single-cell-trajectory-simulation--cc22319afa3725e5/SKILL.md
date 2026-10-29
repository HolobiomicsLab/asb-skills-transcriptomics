---
name: single-cell-trajectory-simulation
description: Use when you have longitudinal scRNA-seq data (normalized expression,
  time-point labels, cell type annotations, e.
when_to_use_negative:
- The dataset's final time point is not at steady state, since the PRESCIENT model
  constrains the terminal timepoint to be the steady-state target of the potential
- You need single-gene-resolution interpretation at low abundance, since transforming
  data into PCA space loses information about individual genes and lowly-expressed
  fate-relevant genes may be dropped altogether in scRNA-seq
- You want causal claims about gene-fate relationships — association of genes with
  fates does not imply causality, so simulation results should guide rather than replace
  experimental perturbation
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: PRESCIENT
  role: 'Core generative framework: fits the differentiation potential landscape from
    time-series scRNA-seq and simulates trajectories and perturbations'
  repo: https://github.com/gifford-lab/prescient
- name: prescient
  role: PyPI CLI whose `prescient process_data` command builds the data_pt torch object
    needed for training and simulation
  repo: https://github.com/gifford-lab/prescient
- name: prescient-analysis
  role: Pre-processing scripts, training bash scripts (weinreb-interpolate.sh, weinreb-fate.sh,
    veres-fate.sh), pre-trained models, and visualization notebooks
  repo: https://github.com/gifford-lab/prescient-analysis
- name: PyTorch
  role: Trains the potential-energy model and runs trajectory simulations from the
    data_pt torch object and model checkpoints
- name: scanpy
  role: Pre-processing of scRNA-seq matrices (e.g. the Weinreb lineage-tracing dataset)
    into .h5ad/.csv before process_data
- name: scikit-learn
  role: Provides the sklearn.decomposition.PCA object stored in data_pt["pca"] and
    used to produce PCs for simulation space
- name: UMAP
  role: Provides the umap.UMAP object stored in data_pt["um"] used to project simulated
    perturbed trajectories for fate-outcome visualization
- name: wot
  role: Baseline method producing linearly interpolated held-out timepoint populations
    for Wasserstein-distance comparison
  repo: https://github.com/broadinstitute/wot
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:59528253f2fa127629fafeb242dde9b9196cd56644d5d96bf9c3e5a13a49c316
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - UMAP
  - prescient
  - prescient-analysis
  - scanpy
  - scikit-learn
  - wot
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-trajectory-simulation

## Summary

Simulate cell differentiation trajectories in silico with PRESCIENT, a generative model that learns a potential-energy landscape from time-series scRNA-seq data, enabling held-out timepoint generation, clonal fate-bias prediction, and in silico gene perturbation experiments in physical time.

## When to use

Use when you have longitudinal scRNA-seq data (normalized expression, time-point labels, cell type annotations, e.g. Weinreb hematopoiesis days 2/4/6 or Veres beta-cell differentiation) and need to simulate trajectories, recover held-out timepoints, predict clonal fate bias, or test gene perturbation effects on fate outcomes.

## When NOT to use

- The dataset's final time point is not at steady state, since the PRESCIENT model constrains the terminal timepoint to be the steady-state target of the potential
- You need single-gene-resolution interpretation at low abundance, since transforming data into PCA space loses information about individual genes and lowly-expressed fate-relevant genes may be dropped altogether in scRNA-seq
- You want causal claims about gene-fate relationships — association of genes with fates does not imply causality, so simulation results should guide rather than replace experimental perturbation

## Inputs

- Longitudinal scRNA-seq data with normalized gene expression, integer time-point labels (e.g. days 2/4/6), and cell type annotations (.csv/.tsv/.txt/.h5ad)
- Trained PRESCIENT model checkpoint (2-layer 400-unit PyTorch models)
- data_pt torch object (data, celltype, genes, tps, x/xp/xu splits, sklearn PCA, umap.UMAP object, growth weights w)
- Empirical or expression-derived proliferation/growth rates for growth-weighted training
- Perturbation specifications: target TFs, perturbation magnitudes (z in [-2.5, 10]), starting timepoints/subpopulations

## Outputs

- Simulated terminal-timepoint populations in PCA/gene space, projectable to UMAP
- Wasserstein testing/training distances for held-out timepoint recovery (e.g. wasserstein_distances.csv)
- Clonal fate-bias metrics per method/proliferation regime: Pearson r, AUROC, covered fraction (fate_bias_metrics.csv)
- Per-perturbation fate-fraction shifts (e.g. perturbation_temporal_results.csv, perturbation_subpopulation_results.csv)

## How to apply

Pre-process the scRNA-seq matrix with scanpy or Seurat (.csv/.tsv/.txt/.h5ad accepted), then run `prescient process_data` to build the data_pt torch object containing normalized expression, celltype, genes, tps, timepoint-split tensors (x/xp/xu), sklearn PCA and UMAP objects, and pre-computed growth weights w. Train the PRESCIENT potential model in PyTorch (2-layer 400-unit models, e.g. 5 random seeds as in the Weinreb task scripts), using the final time point as the steady-state terminal target and optionally incorporating empirically measured or expression-derived proliferation rates — proliferation-aware training greatly improves clonal fate-bias prediction (mean r = 0.347 ± 0.029, AUROC = 0.692 ± 0.012, beating WOT and FateID). Simulate trajectories by rolling cells forward from chosen initial populations (e.g. day-2 progenitors) under the learned potential to generate held-out timepoint marginals, then score with Wasserstein distance against held-out cells (as in the day-4 holdout comparison vs. WOT interpolation). For perturbations, apply in silico expression shifts (z from -2.5 knockdown to +10 overexpression) to known TFs (e.g. Lmo4/Cebpe/Mxd1/Dach1 for neutrophils, Irf8/Irf5/Klf4/Nr4a1 for monocytes, NKX2.2/NKX6.1/PAX4/ARX vs. PDX1/MAFA/MAFB/ISL1 in beta-cell differentiation) on selected starting populations and timepoints, then quantify fate-fraction shifts at the terminal time point and project simulated cells back through the stored PCA and UMAP objects for visualization.

## Related tools

- **PRESCIENT** (Core generative framework: fits the differentiation potential landscape from time-series scRNA-seq and simulates trajectories and perturbations) — https://github.com/gifford-lab/prescient
- **prescient** (PyPI CLI whose `prescient process_data` command builds the data_pt torch object needed for training and simulation) — https://github.com/gifford-lab/prescient
- **prescient-analysis** (Pre-processing scripts, training bash scripts (weinreb-interpolate.sh, weinreb-fate.sh, veres-fate.sh), pre-trained models, and visualization notebooks) — https://github.com/gifford-lab/prescient-analysis
- **PyTorch** (Trains the potential-energy model and runs trajectory simulations from the data_pt torch object and model checkpoints)
- **scanpy** (Pre-processing of scRNA-seq matrices (e.g. the Weinreb lineage-tracing dataset) into .h5ad/.csv before process_data)
- **scikit-learn** (Provides the sklearn.decomposition.PCA object stored in data_pt["pca"] and used to produce PCs for simulation space)
- **UMAP** (Provides the umap.UMAP object stored in data_pt["um"] used to project simulated perturbed trajectories for fate-outcome visualization)
- **wot** (Baseline method producing linearly interpolated held-out timepoint populations for Wasserstein-distance comparison) — https://github.com/broadinstitute/wot

## Examples

```
prescient process_data --in data.h5ad --out data_pt.pt && bash weinreb-fate.sh  # then simulate from day-2 progenitors with the trained checkpoint
```

## Evaluation signals

- Proliferation-aware PRESCIENT fate-bias runs reach mean Pearson r ≈ 0.347 ± 0.029 and AUROC ≈ 0.692 ± 0.012 on the 335-cell Weinreb test set, improving over the no-proliferation variant and WOT/FateID baselines
- Simulated held-out day-4 populations from a days-2/6-trained model are closer (Wasserstein distance) to actual day-4 cells than WOT's linearly interpolated day-4 population and than the day-2/day-6 marginal distances
- TF perturbation directionality holds: knockdown (z = -2.5) of granulopoiesis TFs (Lmo4, Cebpe, Mxd1, Dach1) decreases the neutrophil fraction and overexpression (z = 10) increases it, with analogous behavior for monocyte TFs (Irf8, Irf5, Klf4, Nr4a1), while random non-TF controls yield no fate shift
- Temporal specificity check: early-acting beta-cell TFs (NKX2.2, NKX6.1, PAX4, ARX) significantly increase final beta-cell proportion (p < 0.05) from pancreatic progenitors but not from later endocrine-pathway populations, and the NKX6.1/NEUROG3 (z = 10) endocrine effect diminishes when introduced at later days 0–6
- data_pt torch object exists with expected keys (data, celltype, genes, tps, x/xp/xu, pca, um) and wasserstein_distances.csv / fate_bias_metrics.csv contain rows for all model variants and baselines

## Limitations

- The model requires the final time point of the dataset to be at steady-state
- Performance depends on growth-rate estimates; the model would likely benefit from better estimates of proliferation
- Information is lost about individual genes when transforming data into PCA space
- Lowly-expressed genes important to cell fate decisions may be dropped altogether in the scRNA-seq data
- Association of certain genes with specific cell fates does not necessarily imply causality; challenges remain to confidently suggest gene sets for experimental perturbation
- Paper perturbation demonstrations were limited to transcription factors, although non-TF targets such as signaling pathway effectors can also be tested

## Evidence

- [abstract] PRESCIENT (Potential eneRgy undErlying Single Cell gradIENTs), a generative modeling framework that learns an underlying differentiation landscape from time-series scRNA-seq data: "We introduce PRESCIENT (Potential eneRgy undErlying Single Cell gradIENTs), a generative modeling framework that learns an underlying differentiation landscape from time-series scRNA-seq data."
- [methods] process_data generates data_pt torch object: "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary)"
- [methods] PRESCIENT inputs: "For training, all that is needed is normalized gene expression, time-point labels, and cell type annotations."
- [discussion] Steady-state constraint on final time point: "the model is subject to constraints and assumptions, for example requiring that the final time point of the dataset be at steady-state"
- [readme] Interpolation training script spec: [evidence span withheld — anchor status: external_doc]
- [results] Perturbation result in hematopoiesis: [evidence span withheld — anchor status: not_found]
- [discussion] Non-TF targets possible but untested: "While this work was limited to TFs to show the utility of the method, non-TF targets, such as signaling pathway effectors, can also be tested using PRESCIENT."
