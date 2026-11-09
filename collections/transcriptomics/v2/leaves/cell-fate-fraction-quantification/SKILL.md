---
name: cell-fate-fraction-quantification
description: Use when you have simulated single-cell trajectories from a PRESCIENT
  perturbation run (e.g., TF overexpression/knockdown sweeps) and need to measure
  how perturbation magnitude shifts fate fractions at the terminal time point, with
  statistical testing of reciprocal fate changes.
when_to_use_negative:
- When association of a gene with a specific fate should be interpreted as causal
  evidence — the article warns that association does not necessarily imply causality
- When the analysis goal is trajectory shape or dynamics rather than terminal fate
  composition (e.g., interpolation of held-out timepoints)
- When perturbing non-TF targets such as signaling pathway effectors without confirming
  the model supports them — the paper's perturbation validation was limited to TFs
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: PRESCIENT
  role: Generative model used to simulate perturbed cell trajectories whose terminal
    fates are then quantified
  repo: https://github.com/gifford-lab/prescient
- name: PyTorch
  role: Loads the trained PRESCIENT checkpoint and data_pt object and runs the perturbation
    simulations
- name: scikit-learn
  role: Provides the PCA object (data_pt['pca']) fit to normalized expression, through
    which simulated cells are represented
- name: UMAP
  role: Provides the UMAP object (data_pt['um']) used for visualizing fate assignments
    of simulated cells
- name: scanpy
  role: Pre-processing of scRNA-seq data (e.g., Veres et al. beta-cell time course)
    prior to simulation and fate quantification
- name: Seurat
  role: Recommended alternative to scanpy for pre-processing the normalized expression
    inputs
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:c02788d44c19ad53279b92d0573ed43df2b615585f69334ca6902e3e384d925f
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cell-fate-fraction-quantification

## Summary

Quantify the fraction of simulated cells assigned to each terminal cell fate (e.g., endocrine vs. exocrine, neutrophil vs. monocyte) at the final time point of PRESCIENT perturbation trajectories, enabling magnitude-dependent assessment of in silico TF perturbation effects.

## When to use

Use when you have simulated single-cell trajectories from a PRESCIENT perturbation run (e.g., TF overexpression/knockdown sweeps) and need to measure how perturbation magnitude shifts fate fractions at the terminal time point, with statistical testing of reciprocal fate changes.

## When NOT to use

- When association of a gene with a specific fate should be interpreted as causal evidence — the article warns that association does not necessarily imply causality
- When the analysis goal is trajectory shape or dynamics rather than terminal fate composition (e.g., interpolation of held-out timepoints)
- When perturbing non-TF targets such as signaling pathway effectors without confirming the model supports them — the paper's perturbation validation was limited to TFs

## Inputs

- PRESCIENT simulated trajectories at the terminal time point (from trained model checkpoints, loaded with PyTorch)
- data_pt serialized torch dictionary containing celltype labels, gene features, timepoint assignments, PCA and UMAP objects
- Perturbation sweep definitions (gene names, magnitudes e.g. -2.5 to 10, starting population e.g. day-0 or day-2 cells)
- Random non-TF control gene lists for null comparisons

## Outputs

- Fate-fraction table per perturbation and magnitude (e.g., perturbation_fate_fractions.csv)
- Endocrine and exocrine cell fractions at the final simulated timepoint per perturbation magnitude
- Neutrophil and monocyte fractions per TF perturbation magnitude
- Statistical test results (p-values, e.g., p < 0.05 significance of reciprocal fate scaling)

## How to apply

After running PRESCIENT perturbation simulations from a defined starting population (e.g., day-0 Veres beta-cell differentiation cells or day-2 Weinreb hematopoiesis progenitors), classify each simulated terminal cell into annotated fates using the celltype labels stored in the data_pt torch object. Sweep perturbation magnitudes (e.g., z from -2.5 knockdown to 10 overexpression in the hematopoiesis workflow) for each target TF and compute the fraction of cells assigned to each fate at the final simulated time point. For endocrine/exocrine shifts (NEUROG3, NKX6.1 vs. PTF1A, HES1), test the reciprocal scaling of fate fractions for significance at p < 0.05. Include random non-TF control genes as negative controls, which should yield no fate shift. Write fate-fraction measurements and statistical test results to a CSV such as perturbation_fate_fractions.csv, reproducing Fig. 3a-h and Fig. 4c/S4c.

## Related tools

- **PRESCIENT** (Generative model used to simulate perturbed cell trajectories whose terminal fates are then quantified) — https://github.com/gifford-lab/prescient
- **PyTorch** (Loads the trained PRESCIENT checkpoint and data_pt object and runs the perturbation simulations)
- **scikit-learn** (Provides the PCA object (data_pt['pca']) fit to normalized expression, through which simulated cells are represented)
- **UMAP** (Provides the UMAP object (data_pt['um']) used for visualizing fate assignments of simulated cells)
- **scanpy** (Pre-processing of scRNA-seq data (e.g., Veres et al. beta-cell time course) prior to simulation and fate quantification)
- **Seurat** (Recommended alternative to scanpy for pre-processing the normalized expression inputs)

## Examples

```
prescient process_data --input veres_normalized.csv --output data_pt.pt
```

## Evaluation signals

- TF knockdowns (negative z, e.g., -2.5) decrease the target lineage fraction while overexpression (e.g., z = 10) increases it, as verified for Lmo4/Cebpe/Mxd1/Dach1 (neutrophil) and Irf8/Irf5/Klf4/Nr4a1 (monocyte)
- Non-TF random control genes produce no significant fate shift
- NEUROG3/NKX6.1 day-0 overexpression increases endocrine fractions with a reciprocal decrease in exocrine fractions, significant at p < 0.05, in a magnitude-dependent manner
- Output CSV (perturbation_fate_fractions.csv) contains fate fractions for every perturbation-magnitude pair and reproduces Fig. 3a-h, Fig. 4c, and S4c
- ARX and PAX4 perturbation simulations reproduce the expected alpha-cell vs. beta-cell fate specification results

## Limitations

- The model requires the final time point of the dataset to be at steady-state, constraining which terminal fates can be reliably quantified
- Information is lost about individual genes when transforming data into PCA space, and lowly-expressed genes important to cell fate decisions may be dropped altogether
- Association of certain genes with specific cell fates does not necessarily imply causality; challenges remain to confidently suggest gene sets for experimental perturbation
- Model performance depends on growth rates and would benefit from better estimates of proliferation

## Evidence

- [other] perturbation magnitude sweep: [evidence span withheld — anchor status: not_found]
- [other] fate fraction quantification: [evidence span withheld — anchor status: not_found]
- [other] endocrine/exocrine reciprocity: [evidence span withheld — anchor status: not_found]
- [other] non-TF negative controls: [evidence span withheld — anchor status: not_found]
- [discussion] PCA information loss: "information is lost about individual genes when transforming data into PCA space"
- [abstract] complex perturbations: "PRESCIENT is able to accommodate complex perturbations of multiple genes, at different time points and from different starting cell populations"
- [discussion] causality caveat: "the association of certain genes with specific cell fates does not necessarily imply causality"
