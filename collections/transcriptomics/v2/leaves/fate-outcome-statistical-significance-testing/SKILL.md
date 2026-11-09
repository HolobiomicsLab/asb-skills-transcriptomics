---
name: fate-outcome-statistical-significance-testing
description: Use when PRESCIENT perturbation ensembles initialized from a defined
  starting population (e.g., day-0 cells) must be evaluated for statistically significant,
  magnitude-dependent shifts in endocrine vs exocrine or alpha vs beta fate fractions.
when_to_use_negative:
- The trained model's dataset does not reach steady-state at the final timepoint,
  since PRESCIENT's fate-outcome predictions are subject to that constraint
- Evaluating associations of genes with fates as causal without perturbation simulation
  — the paper explicitly notes association of genes with fates does not imply causality
- Cell fate prediction on data where lowly-expressed fate-decision genes were dropped
  in preprocessing, since information is lost transforming into PCA space
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: PRESCIENT
  role: Generates perturbed trajectories and final-timepoint fate outcomes from the
    trained potential-energy model
  repo: https://github.com/gifford-lab/prescient
- name: PyTorch
  role: Training and simulation backend for the PRESCIENT potential-energy model and
    perturbation ensembles
- name: scanpy
  role: Preprocessing of the Veres et al. scRNA-seq dataset providing normalized expression
    and cell type annotations used for fate assignment
- name: scikit-learn
  role: PCA fit to normalized expression in the data_pt dictionary used for downstream
    training and simulation
- name: UMAP
  role: UMAP object fit to PCs in data_pt for embedding/visualization of simulated
    fates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:adad251cf0f4a0b09670d07ae11e5d0240760ccc497a19924d6601b0a5b47c38
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# fate-outcome-statistical-significance-testing

## Summary

Quantifies cell-fate fractions at the final simulated timepoint of PRESCIENT perturbation ensembles and statistically tests whether perturbing fate-associated transcription factors (e.g., NEUROG3/NKX6.1 vs PTF1A/HES1 on the Veres beta-cell model) causes magnitude-dependent, reciprocal shifts in fate outcome at p < 0.05.

## When to use

Use when PRESCIENT perturbation ensembles initialized from a defined starting population (e.g., day-0 cells) must be evaluated for statistically significant, magnitude-dependent shifts in endocrine vs exocrine or alpha vs beta fate fractions.

## When NOT to use

- The trained model's dataset does not reach steady-state at the final timepoint, since PRESCIENT's fate-outcome predictions are subject to that constraint
- Evaluating associations of genes with fates as causal without perturbation simulation — the paper explicitly notes association of genes with fates does not imply causality
- Cell fate prediction on data where lowly-expressed fate-decision genes were dropped in preprocessing, since information is lost transforming into PCA space

## Inputs

- Trained PRESCIENT potential-energy model on the Veres et al. 7-timepoint beta-cell differentiation dataset
- Serialized data_pt torch dictionary with celltype labels, gene features, timepoint assignments, and growth weights
- Perturbation specifications: TF names (NEUROG3, NKX6.1, PTF1A, HES1, ARX, PAX4), perturbation magnitudes, and day-0 starting population
- Final-timepoint simulated cell populations with predicted cell type assignments

## Outputs

- perturbation_fate_fractions.csv containing endocrine/exocrine (and alpha/beta) fate fractions per perturbation and magnitude
- Statistical test results for magnitude-dependent, reciprocal fate shifts (p < 0.05)
- Reproduction of Fig. 4c and Fig. S4c fate-fraction summaries

## How to apply

Run ensembles of in silico overexpression perturbations (NEUROG3/NKX6.1 for endocrine induction, PTF1A/HES1 for exocrine, and ARX/PAX4 for alpha vs beta specification) from day-0 cells across increasing perturbation magnitudes using the trained PRESCIENT potential-energy model on the 7 timepoints of the Veres et al. dataset. For each perturbation and magnitude, simulate to the final timepoint, assign cells to annotated cell types, and compute endocrine and exocrine fate fractions. Test the expected reciprocal relationship — increased endocrine fraction paired with decreased exocrine fraction — for significance at p < 0.05, treating magnitude as the scaling variable. Write the fate-fraction measurements and statistical test results to perturbation_fate_fractions.csv, reproducing Fig. 4c and S4c of the paper.

## Related tools

- **PRESCIENT** (Generates perturbed trajectories and final-timepoint fate outcomes from the trained potential-energy model) — https://github.com/gifford-lab/prescient
- **PyTorch** (Training and simulation backend for the PRESCIENT potential-energy model and perturbation ensembles)
- **scanpy** (Preprocessing of the Veres et al. scRNA-seq dataset providing normalized expression and cell type annotations used for fate assignment)
- **scikit-learn** (PCA fit to normalized expression in the data_pt dictionary used for downstream training and simulation)
- **UMAP** (UMAP object fit to PCs in data_pt for embedding/visualization of simulated fates)

## Examples

```
prescient simulate --data data_pt --model veres_fate.pt --perturb NEUROG3,NKX6.1 --start day0 --magnitudes grid; python tests/fate_fractions.py --out perturbation_fate_fractions.csv
```

## Evaluation signals

- Endocrine TF perturbations (NEUROG3, NKX6.1) at day 0 increase endocrine fractions in a magnitude-dependent manner with p < 0.05
- Endocrine fraction increases are reciprocally mirrored by exocrine fraction decreases across perturbation magnitudes
- Perturbation results written to perturbation_fate_fractions.csv match Fig. 4c and S4c of the paper
- ARX vs PAX4 simulations reproduce the expected alpha-cell vs beta-cell fate specification shift
- Fate assignments at the final simulated timepoint are consistent with the cell type annotations stored in data_pt

## Limitations

- The model requires the final time point of the dataset to be at steady-state, which constrains validity of fate-fraction endpoints
- Model performance depends on growth rates and would benefit from better estimates of proliferation
- Information is lost about individual genes when transforming data into PCA space; lowly-expressed genes important to cell fate decisions may be dropped altogether
- Association of certain genes with specific cell fates does not necessarily imply causality; perturbations tested were limited to transcription factors

## Evidence

- [results] endocrine magnitude shift: [evidence span withheld — anchor status: not_found]
- [methods] perturbation setup: [evidence span withheld — anchor status: not_found]
- [methods] output artifact: [evidence span withheld — anchor status: not_found]
- [discussion] interpretation caveat: "the association of certain genes with specific cell fates does not necessarily imply causality."
- [discussion] scope limitation: "While this work was limited to TFs to show the utility of the method, non-TF targets, such as signaling pathway effectors, can also be tested using PRESCIENT."
- [discussion] steady-state requirement: "the model is subject to constraints and assumptions, for example requiring that the final time point of the dataset be at steady-state."
