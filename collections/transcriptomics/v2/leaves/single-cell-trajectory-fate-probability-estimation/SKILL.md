---
name: single-cell-trajectory-fate-probability-estimation
description: Use when time-series scRNA-seq data with time-point labels and cell type
  annotations is available and the goal is to predict cell fate bias — the probability
  a cell enters a particular fate given its initial state — including under simulated
  perturbations.
when_to_use_negative:
- The final time point of the dataset cannot be assumed to be at steady-state, violating
  the model's constraint
- The question concerns individual-gene resolution in a static snapshot rather than
  trajectories — information about individual genes is lost when data are transformed
  into PCA space
- The goal is causal validation of fate associations rather than in silico prediction;
  association of genes with fates does not imply causality and perturbation work in
  the paper was limited to TFs
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PRESCIENT
  role: Generative modeling framework that learns the differentiation landscape from
    time-series scRNA-seq and simulates trajectories for fate probability and perturbation
    prediction
  repo: https://github.com/gifford-lab/prescient
- name: prescient-analysis
  role: Notebooks and pre-trained models for pre-processing, analysis, and visualization
    of PRESCIENT on Weinreb et al. 2020 hematopoiesis and Veres et al. 2019 beta-cell
    differentiation datasets
  repo: https://github.com/gifford-lab/prescient-analysis
- name: PyTorch
  role: Generates and stores the PRESCIENT torch object (data_pt) used for downstream
    training and simulations
- name: scanpy
  role: Recommended tool for pre-processing scRNA-seq data before PRESCIENT training
- name: Seurat
  role: Alternative recommended tool for pre-processing scRNA-seq data before PRESCIENT
    training
- name: scikit-learn
  role: Provides the sklearn.decomposition.PCA object fit to normalized expression
    and used to produce PCs stored in data_pt
- name: UMAP
  role: Provides the umap.UMAP object fit to PCs used to produce UMAP dimensions stored
    in data_pt
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:7e93daca88cc31a1893d733805c5425e2ddeb42e0be76c8f6c56e02e92554e8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gifford-lab/prescient
  tools:
  - PRESCIENT
  - PyTorch
  - Seurat
  - UMAP
  - prescient-analysis
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# single-cell-trajectory-fate-probability-estimation

## Summary

Fit a PRESCIENT generative model to longitudinal scRNA-seq data to learn an underlying differentiation landscape, then simulate trajectories from arbitrary initial cell states to estimate fate probabilities for downstream fates (e.g. alpha-, beta-, EC-cell in Veres et al. in vitro beta-cell differentiation).

## When to use

Use when time-series scRNA-seq data with time-point labels and cell type annotations is available and the goal is to predict cell fate bias — the probability a cell enters a particular fate given its initial state — including under simulated perturbations.

## When NOT to use

- The final time point of the dataset cannot be assumed to be at steady-state, violating the model's constraint
- The question concerns individual-gene resolution in a static snapshot rather than trajectories — information about individual genes is lost when data are transformed into PCA space
- The goal is causal validation of fate associations rather than in silico prediction; association of genes with fates does not imply causality and perturbation work in the paper was limited to TFs

## Inputs

- Longitudinal (time-series) scRNA-seq data as normalized gene expression (.csv, .tsv, .txt, or .h5ad)
- Time-point labels for each cell
- Cell type annotations for each cell
- Estimated cell proliferation / growth rates (recommended for fate bias tasks)
- Serialized PRESCIENT data_pt torch object containing expression matrix, gene features, cell type labels, timepoints, PCA and UMAP objects

## Outputs

- data_pt: serialized PyTorch dictionary with expression matrix, gene feature list, cell type labels, timepoint assignments, PCA and UMAP objects
- Per-cell fate bias probabilities (probability of entering each fate given initial state)
- Simulated perturbed vs. unperturbed fate probability distributions
- Ranked perturbation hit table (TF, fate, log2FC, t-statistic, FDR), e.g. tf_perturbation_hits.csv

## How to apply

Pre-process the longitudinal scRNA-seq data (e.g. with Seurat or scanpy; inputs accepted as .csv, .tsv, .txt, or .h5ad) so it contains normalized gene expression, time-point labels, and cell type annotations. Run `prescient process_data` to generate the serialized torch dictionary `data_pt`, which holds the normalized expression matrix, gene feature list, cell type labels, timepoint assignments, and the sklearn PCA and umap.UMAP objects fit to the normalized expression and PCs. Train the PRESCIENT model (e.g. 2-layer 400-unit models with estimated proliferation rates, 5 seeds) so the learned potential landscape captures differentiation dynamics; the final time point of the dataset should be at steady-state. To estimate fate probabilities, simulate high-dimensional trajectories from a chosen starting cell population with arbitrary initializations in physical time, and compute the fraction of simulated cells entering each terminal fate. For perturbational analyses, simulate z=5 overexpression of a target gene from the relevant starting population, repeat over ~10 random initializations, and compare fate probabilities perturbed versus unperturbed with paired t-tests across seeds, applying Benjamini-Hochberg FDR correction (hits at FDR < 0.01 with log2FC > 0.5).

## Related tools

- **PRESCIENT** (Generative modeling framework that learns the differentiation landscape from time-series scRNA-seq and simulates trajectories for fate probability and perturbation prediction) — https://github.com/gifford-lab/prescient
- **prescient-analysis** (Notebooks and pre-trained models for pre-processing, analysis, and visualization of PRESCIENT on Weinreb et al. 2020 hematopoiesis and Veres et al. 2019 beta-cell differentiation datasets) — https://github.com/gifford-lab/prescient-analysis
- **PyTorch** (Generates and stores the PRESCIENT torch object (data_pt) used for downstream training and simulations)
- **scanpy** (Recommended tool for pre-processing scRNA-seq data before PRESCIENT training)
- **Seurat** (Alternative recommended tool for pre-processing scRNA-seq data before PRESCIENT training)
- **scikit-learn** (Provides the sklearn.decomposition.PCA object fit to normalized expression and used to produce PCs stored in data_pt)
- **UMAP** (Provides the umap.UMAP object fit to PCs used to produce UMAP dimensions stored in data_pt)

## Examples

```
prescient process_data
```

## Evaluation signals

- data_pt contains the expected serialized entries: normalized expression, gene feature list, cell type labels, timepoint assignments, data_pt['pca'] (sklearn PCA) and data_pt['um'] (umap.UMAP)
- Fate predictions on held-out lineage tracing data (Weinreb et al.) reproduce known progenitor fate biases, and improve when cell proliferation is accounted for
- In silico perturbation of well-studied regulatory genes of hematopoiesis and beta-cell differentiation produces the expected changes in fate outcome
- Unbiased screen of 200+ TFs at FDR < 0.01 with log2FC > 0.5 recovers significant fate shifts (e.g. 10 alpha-cell and 18 beta-cell fate hits in the Veres et al. hit list)
- Repeated simulations over 10 random initializations yield consistent fate probability estimates across seeds under paired t-tests

## Limitations

- The model requires the final time point of the dataset to be at steady-state
- Model performance depends on growth rates and would benefit from better estimates of proliferation
- Information is lost about individual genes when transforming data into PCA space, and lowly-expressed genes important to cell fate decisions may be dropped altogether
- Association of certain genes with specific cell fates does not necessarily imply causality; challenges remain in confidently suggesting gene sets for experimental perturbation
- Reported perturbation work was limited to transcription factors, though non-TF targets such as signaling pathway effectors can also be tested

## Evidence

- [abstract] PRESCIENT (Potential eneRgy undErlying Single Cell gradIENTs), a generative modeling framework that learns an underlying differentiation landscape from time-series scRNA-seq data.: "PRESCIENT (Potential eneRgy undErlying Single Cell gradIENTs), a generative modeling framework that learns an underlying differentiation landscape from time-series scRNA-seq data."
- [methods] process_data generates the serialized data_pt torch object: "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary) that contains all the necessary information for downstream training, simulations, and"
- [methods] Input requirements for training: "PRESCIENT takes as input longitudinal scRNA-seq data. For training, all that is needed is normalized gene expression, time-point labels, and cell type annotations."
- [discussion] Large-scale TF perturbation screen validating fate probability estimates: "large-scale screen for in vitro β-cell differentiation in which we perturbed 200+ TFs and identified target genes that could cause significant shifts in β-cell, α-cell, and EC-cell fates."
- [intro] Purpose of perturbation simulations: "PRESCIENT enables large unbiased in silico perturbation experiments to aid the design of in vitro genetic perturbational screens"
- [readme] README description of the tool: "Software for PRESCIENT (Potential eneRgy undErlying Single Cell gradIENTs), a generative model for modeling single-cell time-series."
