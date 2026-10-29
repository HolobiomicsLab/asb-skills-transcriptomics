---
name: transcription-factor-perturbation-analysis
description: Use when you have a trained PRESCIENT model on time-series scRNA-seq
  data and need to test whether perturbing candidate TFs (e.g. neutrophil or monocyte
  regulators) shifts simulated cells toward or away from expected terminal fates.
when_to_use_negative:
- The dataset's final time point is not at steady-state, violating PRESCIENT's assumption
  for terminal fate assignment
- Genes of interest are lowly expressed and may have been dropped during scRNA-seq
  preprocessing or lost in the PCA-space transformation
- The goal is causal validation rather than in silico hypothesis generation — association
  of a gene with a fate does not imply causality
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: PRESCIENT
  role: Generative model used to simulate perturbed cell trajectories to the final
    time point
  repo: https://github.com/gifford-lab/prescient
- name: PyTorch
  role: Loads the model checkpoint and data_pt torch object used for simulations
- name: scikit-learn
  role: Provides the PCA object (data_pt['pca']) fit to normalized expression used
    to produce PCs
- name: UMAP
  role: Provides the UMAP object (data_pt['um']) fit to PCs used to produce UMAP dims
    for visualization
- name: prescient-analysis
  role: Analysis and visualization notebooks for the Weinreb et al. hematopoiesis
    perturbation experiments
  repo: https://github.com/gifford-lab/prescient-analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:3d4a0d939875d942f11974f7831c09b2aabdac804762f0a804cb57b19b5d7167
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - UMAP
  - prescient-analysis
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# transcription-factor-perturbation-analysis

## Summary

In silico perturbation analysis using a trained PRESCIENT generative model to sweep expression perturbations (z from -2.5 knockdown to +10 overexpression) on lineage-regulating transcription factors and quantify resulting shifts in cell fate fractions, validating that perturbations recapitulate expected differentiation directionality.

## When to use

Use when you have a trained PRESCIENT model on time-series scRNA-seq data and need to test whether perturbing candidate TFs (e.g. neutrophil or monocyte regulators) shifts simulated cells toward or away from expected terminal fates.

## When NOT to use

- The dataset's final time point is not at steady-state, violating PRESCIENT's assumption for terminal fate assignment
- Genes of interest are lowly expressed and may have been dropped during scRNA-seq preprocessing or lost in the PCA-space transformation
- The goal is causal validation rather than in silico hypothesis generation — association of a gene with a fate does not imply causality

## Inputs

- Trained PRESCIENT model checkpoint (2-layer, 400-unit model, e.g. from weinreb-fate.sh)
- data_pt torch object (serialized dictionary with normalized expression, sklearn PCA, umap.UMAP, timepoint splits)
- List of target TFs (e.g. Lmo4, Cebpe, Mxd1, Dach1, Irf8, Irf5, Klf4, Nr4a1) and random non-TF control genes
- Day-2 progenitor cell population from the Weinreb et al. mouse hematopoiesis dataset

## Outputs

- Fate fraction curves: fraction of simulated cells assigned to neutrophil and monocyte fates at the terminal time point per gene and per perturbation magnitude z
- Perturbation sweep results reproducing Fig. 3a-h directionality plots
- Non-TF control null results showing no fate shift

## How to apply

Load the trained PRESCIENT model checkpoint and the data_pt torch object (containing normalized expression, sklearn PCA, umap.UMAP objects, and timepoint splits) with PyTorch. Select day-2 progenitor cells from the Weinreb et al. mouse hematopoiesis dataset as the starting population for all simulations. For each TF — granulopoiesis regulators Lmo4, Cebpe, Mxd1, Dach1 and monocyte regulators Irf8, Irf5, Klf4, Nr4a1 — apply expression perturbations z spanning -2.5 (knockdown) to 10 (overexpression) and simulate trajectories to the final time point with the trained model. Run the same perturbation sweep on random non-TF control genes as negative controls. Quantify the fraction of simulated cells assigned to neutrophil and monocyte fates at the terminal time point for each gene and magnitude, and verify directionality: knockdown decreases and overexpression increases the target lineage fraction while non-TF controls show no fate shift.

## Related tools

- **PRESCIENT** (Generative model used to simulate perturbed cell trajectories to the final time point) — https://github.com/gifford-lab/prescient
- **PyTorch** (Loads the model checkpoint and data_pt torch object used for simulations)
- **scikit-learn** (Provides the PCA object (data_pt['pca']) fit to normalized expression used to produce PCs)
- **UMAP** (Provides the UMAP object (data_pt['um']) fit to PCs used to produce UMAP dims for visualization)
- **prescient-analysis** (Analysis and visualization notebooks for the Weinreb et al. hematopoiesis perturbation experiments) — https://github.com/gifford-lab/prescient-analysis

## Examples

```
import torch; data_pt = torch.load('data_pt'); model = torch.load('model.pt'); simulate fate fractions for z in [-2.5, 10] on Lmo4/Cebpe/Irf8 starting from day-2 progenitors
```

## Evaluation signals

- TF knockdowns (z = -2.5) produce a relative decrease in the target lineage fraction (neutrophil for Lmo4/Cebpe/Mxd1/Dach1; monocyte for Irf8/Irf5/Klf4/Nr4a1)
- TF overexpressions (up to z = 10) produce a relative increase in the target lineage fraction, matching Fig. 3a-h
- Random non-TF control genes yield no systematic fate shift across the perturbation sweep
- Fate fractions are computed on the same day-2 starting population across all genes for comparability
- Simulation runs use the same trained model seed(s) as the weinreb-fate.sh fitting (5 seeds of 2-layer 400-unit models)

## Limitations

- The model requires the final time point of the dataset to be at steady-state
- Information is lost about individual genes when transforming data into PCA space, and lowly-expressed genes important to cell fate decisions may be dropped altogether
- Association of certain genes with specific cell fates does not necessarily imply causality; challenges remain to confidently suggest gene sets for experimental perturbation
- Paper perturbations were limited to transcription factors, though non-TF targets such as signaling pathway effectors can also be tested

## Evidence

- [intro] PRESCIENT enables large unbiased in silico perturbation experiments to aid the design of in vitro genetic perturbational screens: "PRESCIENT enables large unbiased in silico perturbation experiments to aid the design of in vitro genetic perturbational screens"
- [methods] The prescient process_data command generates data_pt: "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary) that contains all the necessary information for downstream training, simulations"
- [methods] data_pt contains PCA and UMAP objects: "data_pt["pca"]: sklearn.decomposition.PCA object fit to normalized expression and used to produce PCs."
- [discussion] Model requires final time point at steady-state: "the model is subject to constraints and assumptions, for example requiring that the final time point of the dataset be at steady-state."
- [readme] weinreb-fate.sh trains 5 seeds of 2-layer 400-unit models: [evidence span withheld — anchor status: external_doc]
