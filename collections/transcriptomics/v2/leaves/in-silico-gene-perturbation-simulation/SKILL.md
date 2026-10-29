---
name: in-silico-gene-perturbation-simulation
description: Use when you have a trained PRESCIENT model and a serialized data_pt
  torch object and need to predict how overexpression or knockdown of specific genes
  (TFs or non-TFs) alters terminal cell-fate proportions.
when_to_use_negative:
- The dataset's final time point is not at steady-state, violating a PRESCIENT model
  assumption
- The gene of interest was dropped from the expression matrix (e.g. lowly-expressed
  genes important to fate decisions may be absent, and information is lost about individual
  genes in PCA space)
- You need experimentally validated causal effects — association of certain genes
  with specific cell fates does not necessarily imply causality
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: PRESCIENT
  role: Generative model used to simulate perturbed differentiation trajectories and
    quantify fate-bias shifts
  repo: https://github.com/gifford-lab/prescient
- name: PyTorch
  role: Loads the data_pt torch object and trained model checkpoint, and runs perturbation
    simulations
- name: scikit-learn
  role: Provides the PCA object stored in data_pt through which perturbed trajectories
    are projected
- name: UMAP
  role: UMAP object fit to PCs, used to visualize/project simulated perturbed trajectories
- name: scanpy
  role: Recommended pre-processing of the longitudinal scRNA-seq data (e.g. Veres
    et al. dataset) before prescient process_data
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:8123ca497d5ebe67445b6ef8478ec200851695ace45486ece628ff9b1db67f7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - Seurat
  - UMAP
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# in-silico-gene-perturbation-simulation

## Summary

Simulate single-cell differentiation trajectories under in silico gene perturbations using a trained PRESCIENT potential-energy model, quantifying shifts in fate fractions for target lineages relative to unperturbed simulations.

## When to use

Use when you have a trained PRESCIENT model and a serialized data_pt torch object and need to predict how overexpression or knockdown of specific genes (TFs or non-TFs) alters terminal cell-fate proportions.

## When NOT to use

- The dataset's final time point is not at steady-state, violating a PRESCIENT model assumption
- The gene of interest was dropped from the expression matrix (e.g. lowly-expressed genes important to fate decisions may be absent, and information is lost about individual genes in PCA space)
- You need experimentally validated causal effects — association of certain genes with specific cell fates does not necessarily imply causality

## Inputs

- Trained PRESCIENT model checkpoint (PyTorch)
- Serialized data_pt torch dictionary (normalized expression, gene features, celltype labels, timepoint assignments, sklearn PCA object, UMAP object, growth weights)
- List of target genes (e.g. TFs intersected with the Veres et al. highly variable gene set) and perturbation magnitudes z
- Starting population definition (day/timepoint split or cell-type annotated subpopulation)

## Outputs

- Fate fractions at the terminal timepoint per gene and perturbation magnitude
- perturbation_fate_fractions.csv (fate-fraction measurements and statistical tests)
- tf_perturbation_hits.csv (ranked table: TF, fate, log2FC, t-statistic, FDR)
- perturbation_temporal_results.csv and perturbation_subpopulation_results.csv (day- and subpopulation-resolved fate shifts)

## How to apply

Load the trained PRESCIENT checkpoint and data_pt (normalized expression, sklearn PCA, UMAP, celltype labels, timepoints) with PyTorch, then select the starting cell population (e.g. day-2 progenitors in Weinreb hematopoiesis or day-0 cells in Veres beta-cell differentiation). Apply expression perturbations to the initial cell states over a sweep of magnitudes (z from -2.5 for knockdown to 10 for overexpression), simulate trajectories to the final time point, and project results through the stored pca and umap objects to quantify fate fractions (neutrophil/monocyte or endocrine/exocrine/alpha/beta/EC). For an unbiased screen, simulate each TF at z = 5 over 10 random initializations, run paired t-tests across initializations, and apply Benjamini-Hochberg FDR correction retaining hits at FDR < 0.01 and log2(FC) > 0.5. Include random non-TF control genes as negative controls, and verify expected directionality: knockdown decreases and overexpression increases the target lineage fraction. Temporal/stage specificity can be probed by initializing perturbations from each timepoint or from annotated subpopulations (e.g. neurog3-early/mid/late).

## Related tools

- **PRESCIENT** (Generative model used to simulate perturbed differentiation trajectories and quantify fate-bias shifts) — https://github.com/gifford-lab/prescient
- **PyTorch** (Loads the data_pt torch object and trained model checkpoint, and runs perturbation simulations)
- **scikit-learn** (Provides the PCA object stored in data_pt through which perturbed trajectories are projected)
- **UMAP** (UMAP object fit to PCs, used to visualize/project simulated perturbed trajectories)
- **scanpy** (Recommended pre-processing of the longitudinal scRNA-seq data (e.g. Veres et al. dataset) before prescient process_data)

## Examples

```
data_pt = torch.load('data.pt'); sim = prescient.simulate(model, data_pt['x'][data_pt['tps']==0], genes={'NEUROG3': 10}); endo_frac = compute_fate_fractions(sim, fate='endocrine')
```

## Evaluation signals

- Directionality check: TF knockdown (z = -2.5) decreases and overexpression (z = 10) increases the target lineage fraction
- Negative-control check: perturbation sweeps on random non-TF control genes yield no fate shift
- Statistical significance: endocrine/exocrine reciprocal shifts significant at p < 0.05; unbiased screen hits pass FDR < 0.01 with log2FC > 0.5
- Temporal specificity: endocrine-fate effects of NKX6.1/NEUROG3 (z = 10) diminish when perturbations are introduced at later days 0-6, and early-acting vs late-acting TF sets show distinct effects by starting subpopulation
- Ensemble stability: paired t-tests across 10 random initializations agree on fate-probability differences

## Limitations

- Model requires the final time point of the dataset to be at steady-state
- Model performance depends on growth rates and would benefit from better estimates of proliferation
- Information is lost about individual genes when transforming data into PCA space, and lowly-expressed genes important to cell fate decisions may be dropped altogether
- Association of certain genes with specific cell fates does not necessarily imply causality; there remain challenges to confidently suggest gene sets for experimental perturbation

## Evidence

- [abstract] PRESCIENT is able to accommodate complex perturbations of multiple genes, at different time points and from different starting cell populations: "PRESCIENT is able to accommodate complex perturbations of multiple genes, at different time points and from different starting cell populations"
- [methods] The prescient process_data command generates data_pt: "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary) that contains all the necessary information for downstream training, simulations"
- [methods] data_pt contains PCA and UMAP objects: "data_pt["pca"]: sklearn.decomposition.PCA object fit to normalized expression and used to produce PCs."
- [discussion] Perturbations of well-studied regulatory genes produce expected fate changes: "We show that inducing perturbations in well-studied regulatory genes of hematopoiesis and β-cell differentiation result in expected changes in fate outcome."
- [discussion] Large-scale screen of 200+ TFs identified significant fate-shift genes: "large-scale screen for in vitro β-cell differentiation in which we perturbed 200+ TFs and identified target genes that could cause significant shifts in β-cell, α-cell, and EC-cell fates."
- [discussion] Model requires final time point at steady-state: "the model is subject to constraints and assumptions, for example requiring that the final time point of the dataset be at steady-state."
