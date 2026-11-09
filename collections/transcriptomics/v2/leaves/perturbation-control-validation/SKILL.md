---
name: perturbation-control-validation
description: Use when a generative trajectory model supports in silico gene perturbations
  and you must verify that perturbing known fate-regulating TFs moves lineage fractions
  in the expected direction while non-TF controls do not.
when_to_use_negative:
- The final time point of the dataset is not at steady-state, violating a PRESCIENT
  model constraint
- Perturbed genes of interest are lowly expressed or dropped from the data, since
  information about individual genes is lost when transforming into PCA space
- The goal is to claim causality from gene-fate association alone — the article explicitly
  notes association does not necessarily imply causality
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: PRESCIENT
  role: Generative model used to simulate perturbed trajectories from day-2 progenitors
    to the final time point
  repo: https://github.com/gifford-lab/prescient
- name: PyTorch
  role: Loads the trained checkpoint and the data_pt torch object used for training,
    simulations
- name: scikit-learn
  role: Provides the PCA object fit to normalized expression and used to produce PCs
    in data_pt
- name: UMAP
  role: Provides the UMAP object fit to PCs used to visualize perturbed trajectories
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:80409b7e00573064bf046bff67713b0f97f69ecba67f1632b3f73e7866af18c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - UMAP
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# perturbation-control-validation

## Summary

Validates an in silico perturbation workflow by checking that simulated knockdown/overexpression of known lineage-regulating transcription factors shifts fate fractions in the expected direction, while random non-TF control genes produce no fate shift. Applied here with PRESCIENT simulations on the Weinreb et al. mouse hematopoiesis dataset to confirm the model recovers directional biology rather than spurious shifts.

## When to use

Use when a generative trajectory model supports in silico gene perturbations and you must verify that perturbing known fate-regulating TFs moves lineage fractions in the expected direction while non-TF controls do not.

## When NOT to use

- The final time point of the dataset is not at steady-state, violating a PRESCIENT model constraint
- Perturbed genes of interest are lowly expressed or dropped from the data, since information about individual genes is lost when transforming into PCA space
- The goal is to claim causality from gene-fate association alone — the article explicitly notes association does not necessarily imply causality

## Inputs

- Trained PRESCIENT model checkpoint (2-layer 400-unit model per prescient-analysis training scripts)
- data_pt torch file (serialized dictionary) from `prescient process_data` containing normalized expression, sklearn.decomposition.PCA object, umap.UMAP object, and timepoint splits
- Day-2 progenitor cells from the Weinreb et al. mouse hematopoiesis lineage tracing dataset
- Gene lists: neutrophil TFs (Lmo4, Cebpe, Mxd1, Dach1), monocyte TFs (Irf8, Irf5, Klf4, Nr4a1), and random non-TF control genes

## Outputs

- Terminal-time-point fate fractions (neutrophil and monocyte) per perturbed gene per perturbation magnitude z in [-2.5, 10]
- Directionality table reproducing Fig. 3a-h: TF knockdown decreases and overexpression increases the target lineage fraction; non-TF controls flat
- In silico perturbation simulation trajectories from day-2 progenitors

## How to apply

Load the trained PRESCIENT model checkpoint and the data_pt torch object (normalized expression, sklearn PCA, umap.UMAP objects, and timepoint splits) with PyTorch, then select day-2 progenitor cells as the starting population for all perturbation simulations. For each neutrophil TF (Lmo4, Cebpe, Mxd1, Dach1) and monocyte TF (Irf8, Irf5, Klf4, Nr4a1), sweep expression perturbations z from -2.5 (knockdown) to 10 (overexpression) and simulate trajectories to the final time point with the trained model. Repeat the identical sweep on random non-TF control genes as negative controls. Quantify the fraction of simulated cells assigned to the neutrophil and monocyte fates at the terminal time point for each gene and magnitude, then check directionality: TF knockdown must decrease and overexpression must increase the target lineage fraction, while non-TF controls yield no fate shift, reproducing Fig. 3a-h. Rationale: PRESCIENT accommodates complex perturbations of multiple genes at different time points and from different starting populations, so a directional-plus-null-control pattern demonstrates the model encodes causal-like regulatory structure rather than reacting to any expression change.

## Related tools

- **PRESCIENT** (Generative model used to simulate perturbed trajectories from day-2 progenitors to the final time point) — https://github.com/gifford-lab/prescient
- **PyTorch** (Loads the trained checkpoint and the data_pt torch object used for training, simulations)
- **scikit-learn** (Provides the PCA object fit to normalized expression and used to produce PCs in data_pt)
- **UMAP** (Provides the UMAP object fit to PCs used to visualize perturbed trajectories)

## Examples

```
import torch; data_pt = torch.load('data_pt'); sim_cells = prescient.simulate(model, start_cells=data_pt['day2_progenitors'], z={'Cebpe': 10}, t_final=final_timepoint)
```

## Evaluation signals

- Knockdown (z = -2.5) of each granulopoiesis TF (Lmo4, Cebpe, Mxd1, Dach1) decreases the simulated neutrophil fraction relative to unperturbed
- Overexpression (z up to 10) of each neutrophil TF increases the neutrophil fraction, with mirrored behavior for monocyte TFs (Irf8, Irf5, Klf4, Nr4a1)
- Random non-TF control genes produce no fate shift at any perturbation magnitude
- Results are stable across the 5 trained seeds used in prescient-analysis training scripts
- Terminal fate fractions reproduce the pattern of Fig. 3a-h

## Limitations

- The model requires the final time point of the dataset to be at steady-state
- Information is lost about individual genes when transforming data into PCA space, and lowly-expressed genes important to cell fate decisions may be dropped altogether
- The article's perturbation work was limited to TFs to show the method's utility; non-TF targets such as signaling pathway effectors can also be tested
- Association of certain genes with specific cell fates does not necessarily imply causality, and there remain challenges to confidently suggest gene sets for experimental perturbation

## Evidence

- [methods] In silico perturbation sweep design: "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary) that contains all the necessary information for downstream training, simulations, and"
- [abstract] Perturbation capacity of the model: [evidence span withheld — anchor status: not_found]
- [discussion] Expected directionality of perturbation effects: "We show that inducing perturbations in well-studied regulatory genes of hematopoiesis and β-cell differentiation result in expected changes in fate outcome."
- [discussion] Steady-state requirement: "the model is subject to constraints and assumptions, for example requiring that the final time point of the dataset be at steady-state."
- [readme] Training configuration for Weinreb dataset: [evidence span withheld — anchor status: external_doc]
