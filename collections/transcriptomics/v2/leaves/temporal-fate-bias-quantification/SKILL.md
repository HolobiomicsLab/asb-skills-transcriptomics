---
name: temporal-fate-bias-quantification
description: Use when simulating TF overexpression with PRESCIENT across multiple
  time points (day 0–6) or starting populations (e.g., SOX2+, NKX6.1+, NEUROG3 subpopulations)
  and you need fate-bias shifts toward endocrine lineages per condition.
when_to_use_negative:
- The dataset's final time point is not at steady-state — PRESCIENT's model requires
  the final time point of the dataset to be at steady-state.
- The question concerns individual gene-level trajectories rather than fate-proportion
  shifts; information is lost about individual genes when data are transformed into
  PCA space.
- 'You need causal validation of gene–fate relationships in a wet lab: association
  of certain genes with specific cell fates does not necessarily imply causality.'
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: PRESCIENT
  role: Generative model used to run in silico perturbation simulations from different
    timepoints and starting cell populations in gene/PCA space
  repo: https://github.com/gifford-lab/prescient
- name: PyTorch
  role: Loads the data_pt torch object and trained PRESCIENT model checkpoints for
    simulation
- name: scikit-learn
  role: Provides the sklearn.decomposition.PCA object in data_pt['pca'] used to project
    simulated trajectories back from PCA space
- name: UMAP
  role: Provides the umap.UMAP object in data_pt['um'] used to visualize/project simulated
    perturbed trajectories
- name: prescient-analysis
  role: Contains pre-processing, analysis, and visualization notebooks (including
    veres-fate.sh training scripts for the Veres et al. 2019 β-cell dataset)
  repo: https://github.com/gifford-lab/prescient-analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:e4398778abe5b018c973d4da59e56ba718818f7980a3f69f408288707a4d7e1c
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
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# temporal-fate-bias-quantification

## Summary

Quantify how in silico transcription factor perturbations in PRESCIENT shift cell fate outcomes (e.g., β-cell, α-cell, EC-cell proportions) as a function of perturbation time point and starting subpopulation during in vitro β-cell differentiation.

## When to use

Use when simulating TF overexpression with PRESCIENT across multiple time points (day 0–6) or starting populations (e.g., SOX2+, NKX6.1+, NEUROG3 subpopulations) and you need fate-bias shifts toward endocrine lineages per condition.

## When NOT to use

- The dataset's final time point is not at steady-state — PRESCIENT's model requires the final time point of the dataset to be at steady-state.
- The question concerns individual gene-level trajectories rather than fate-proportion shifts; information is lost about individual genes when data are transformed into PCA space.
- You need causal validation of gene–fate relationships in a wet lab: association of certain genes with specific cell fates does not necessarily imply causality.

## Inputs

- data_pt torch object (serialized dictionary with data, celltype, genes, tps, x, xp, pca, y) produced by prescient process_data
- trained PRESCIENT model checkpoint (2-layer, 400-unit models as in veres-fate.sh)
- sklearn.decomposition.PCA object fit to normalized expression (data_pt['pca'])
- umap.UMAP object fit to PCs (data_pt['um'])
- list of TF perturbation targets (early-acting: NKX2.2, NKX6.1, PAX4, ARX; late-acting: PDX1, MAFA, MAFB, ISL1)

## Outputs

- perturbation_temporal_results.csv (day-resolved endocrine-fate outcome shifts for perturbations introduced at days 0–6)
- perturbation_subpopulation_results.csv (fate-bias shifts toward β-, α-, and EC-cell outcomes per starting subpopulation and perturbation set)

## How to apply

Load the serialized data_pt torch dictionary (containing data, celltype, genes, tps, x, xp, pca, y) and the trained PRESCIENT model checkpoint with PyTorch. For each timepoint from day 0 to day 6, initialize simulations from the x tensors split by timepoint, apply in silico overexpression of TFs (e.g., NKX6.1 and NEUROG3, with z = 10 noted for NEUROG3), run PRESCIENT's simulation function in gene/PCA space, and project trajectories through the stored pca and UMAP objects to quantify endocrine-fate outcome shifts per day. Separately, for each starting subpopulation (SOX2+, NKX6.1+, neurog3-early, neurog3-mid, neurog3-late), apply the early-acting TF set (NKX2.2, NKX6.1, PAX4, ARX) and the late-acting set (PDX1, MAFA, MAFB, ISL1) as distinct simulation initializations and compute fate-bias shifts toward β-, α-, and EC-cell outcomes. Write day-resolved results to perturbation_temporal_results.csv and subpopulation-resolved results to perturbation_subpopulation_results.csv; the rationale is that early endocrine-specification TF perturbations increase final β-cell proportion (p < 0.05) when applied in pancreatic progenitors, an effect that diminishes along the endocrine induction pathway.

## Related tools

- **PRESCIENT** (Generative model used to run in silico perturbation simulations from different timepoints and starting cell populations in gene/PCA space) — https://github.com/gifford-lab/prescient
- **PyTorch** (Loads the data_pt torch object and trained PRESCIENT model checkpoints for simulation)
- **scikit-learn** (Provides the sklearn.decomposition.PCA object in data_pt['pca'] used to project simulated trajectories back from PCA space)
- **UMAP** (Provides the umap.UMAP object in data_pt['um'] used to visualize/project simulated perturbed trajectories)
- **prescient-analysis** (Contains pre-processing, analysis, and visualization notebooks (including veres-fate.sh training scripts for the Veres et al. 2019 β-cell dataset)) — https://github.com/gifford-lab/prescient-analysis

## Examples

```
import torch; data_pt = torch.load('data_pt'); model = torch.load('veres_model.pt'); sim = prescient.simulate(model, data_pt['x'][day0], tps=[0,6], gene='NEUROG3', z=10)
```

## Evaluation signals

- Simulated endocrine-fate effect of NKX6.1 and NEUROG3 overexpression diminishes monotonically with perturbation day across days 0–6.
- Early-acting TF perturbations (NKX2.2, NKX6.1, PAX4, ARX) show significant β-cell proportion increase (p < 0.05) in pancreatic progenitors but minimal effect in cells further along endocrine induction.
- PDX1 perturbation increases β-cell fractions in both progenitors and neurog3-late starting cells, while early-acting TFs do not — reproducing distinct early vs. late TF patterns.
- Results qualitatively match Figures 4d and 5b, c of the paper when visualized as fate-bias shifts by timepoint and subpopulation.
- Both output CSVs (perturbation_temporal_results.csv, perturbation_subpopulation_results.csv) contain β-, α-, and EC-cell fate-shift columns for every simulated condition.

## Limitations

- The model requires the final time point of the dataset to be at steady-state.
- Information is lost about individual genes when transforming data into PCA space, and lowly-expressed genes important to cell fate decisions may be dropped altogether in the scRNA-seq data.
- Association of certain genes with specific cell fates does not necessarily imply causality, and challenges remain to confidently suggest gene sets for experimental perturbation.
- Model performance depends on growth rates and would benefit from better estimates of proliferation.

## Evidence

- [abstract] PRESCIENT is able to accommodate complex perturbations of multiple genes, at different time points and from different starting cell populations: "PRESCIENT is able to accommodate complex perturbations of multiple genes, at different time points and from different starting cell populations"
- [methods] process_data produces data_pt: "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary)"
- [methods] PCA object in data_pt: "data_pt["pca"]: sklearn.decomposition.PCA object fit to normalized expression and used to produce PCs."
- [discussion] steady-state assumption: "requiring that the final time point of the dataset be at steady-state"
- [readme] veres-fate.sh training script: [evidence span withheld — anchor status: external_doc]
