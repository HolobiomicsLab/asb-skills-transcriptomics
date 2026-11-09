---
name: cell-type-subpopulation-stratification
description: Use when simulating perturbed trajectories from a differentiation time
  series and you need fate-outcome shifts computed per starting cell state rather
  than per whole timepoint.
when_to_use_negative:
- The question is about temporal (day 0–6) timing of perturbations rather than starting-population
  effects — use the day-resolved workflow instead
- Input scRNA-seq data has not yet been pre-processed with Seurat or scanpy and converted
  to the data_pt torch object via prescient process_data
- The dataset's final time point is not at steady-state, violating the model's constraint
  and undermining fate-bias interpretation
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: PRESCIENT
  role: Generative model used to run in silico TF perturbation simulations initialized
    from each stratified subpopulation
  repo: https://github.com/gifford-lab/prescient
- name: PyTorch
  role: Loads the data_pt torch object and the trained model checkpoint for simulation
- name: scikit-learn
  role: Provides the PCA object (data_pt['pca']) used to map simulated states between
    expression and PC space
- name: UMAP
  role: Provides the umap.UMAP object (data_pt['um']) used to project perturbed trajectories
    for visualization and fate quantification
- name: prescient-analysis
  role: Notebooks for pre-processing, analysis, and visualization of PRESCIENT applied
    to in vitro β-cell differentiation (Veres et al. 2019)
  repo: https://github.com/gifford-lab/prescient-analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-23518-w
    title: Generative modeling of single-cell time series with PRESCIENT enables prediction
      of cell trajectories with interventions
  source_content_hash: sha256:18afa08ba2c9dbd5285515943841a8a0014aceaecf05b6bfdf03543fd0d9e246
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
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cell-type-subpopulation-stratification

## Summary

Stratifies cells from an in vitro β-cell differentiation scRNA-seq time series into labeled starting subpopulations (e.g., SOX2+, NKX6.1+, and neurog3-early/mid/late) so that in silico TF perturbations can be simulated from each population separately, revealing whether perturbation effects depend on developmental stage.

## When to use

Use when simulating perturbed trajectories from a differentiation time series and you need fate-outcome shifts computed per starting cell state rather than per whole timepoint.

## When NOT to use

- The question is about temporal (day 0–6) timing of perturbations rather than starting-population effects — use the day-resolved workflow instead
- Input scRNA-seq data has not yet been pre-processed with Seurat or scanpy and converted to the data_pt torch object via prescient process_data
- The dataset's final time point is not at steady-state, violating the model's constraint and undermining fate-bias interpretation

## Inputs

- PRESCIENT data_pt torch object (serialized dictionary with data, celltype, genes, tps, x, xp, pca, y) from prescient process_data
- Trained PRESCIENT model checkpoint (e.g., a 2-layer 400-unit model trained with estimated proliferation rates on the Veres et al. 2019 β-cell differentiation dataset)
- Cell type annotations and marker gene expression (SOX2, NKX6.1, NEUROG3) used to define starting subpopulations
- TF perturbation sets: early-acting (NKX2.2, NKX6.1, PAX4, ARX) and late-acting (PDX1, MAFA, MAFB, ISL1)

## Outputs

- perturbation_subpopulation_results.csv with fate-bias shifts toward beta-, alpha-, and EC-cell outcomes for each perturbation condition and starting population
- Per-subpopulation simulated trajectories projected into PCA/UMAP space

## How to apply

Split the cell states in the PRESCIENT data_pt torch object (serialized dictionary containing data, celltype, genes, tps, x, xp, pca, y) into starting subpopulations using the stored celltype annotations and marker expression (SOX2+, NKX6.1+, neurog3-early, neurog3-mid, neurog3-late). For each subpopulation, initialize PRESCIENT simulations with the early-acting TF set (NKX2.2, NKX6.1, PAX4, ARX) and the late-acting TF set (PDX1, MAFA, MAFB, ISL1) as separate simulation initializations. Run the simulations in gene/PCA space, project through the stored sklearn PCA and umap.UMAP objects, and compute fate-bias shifts toward beta-, alpha-, and EC-cell outcomes for each perturbation-by-population condition. The rationale is that the same TF can have stage-dependent effects — e.g., early endocrine-specification TFs raise final β-cell proportion (p < 0.05) only when introduced in pancreatic progenitors, while PDX1 increases β-cell fractions in both progenitors and late-NEUROG3 cells — which is invisible if populations are pooled.

## Related tools

- **PRESCIENT** (Generative model used to run in silico TF perturbation simulations initialized from each stratified subpopulation) — https://github.com/gifford-lab/prescient
- **PyTorch** (Loads the data_pt torch object and the trained model checkpoint for simulation)
- **scikit-learn** (Provides the PCA object (data_pt['pca']) used to map simulated states between expression and PC space)
- **UMAP** (Provides the umap.UMAP object (data_pt['um']) used to project perturbed trajectories for visualization and fate quantification)
- **prescient-analysis** (Notebooks for pre-processing, analysis, and visualization of PRESCIENT applied to in vitro β-cell differentiation (Veres et al. 2019)) — https://github.com/gifford-lab/prescient-analysis

## Examples

```
data_pt = torch.load('data_pt.pt'); sim = prescient.simulate(model, data_pt['x'][celltype=='SOX2+'], celltype='progenitor', genes=['NKX2.2','NKX6.1','PAX4','ARX'])
```

## Evaluation signals

- Early-acting TF perturbations (NKX2.2, NKX6.1, PAX4, ARX) significantly increase final β-cell proportion (p < 0.05) in pancreatic progenitor populations but show minimal effect in more mature endocrine-pathway populations
- PDX1 perturbation increases β-cell fractions in both progenitors and late-NEUROG3 subpopulations, distinguishing it from the early-acting set
- Qualitative patterns reproduce Figs. 4d and 5b, c of the PRESCIENT paper
- Results table perturbation_subpopulation_results.csv contains one fate-bias-shift row per (perturbation condition, starting subpopulation) combination, covering beta-, alpha-, and EC-cell outcomes
- Simulated trajectories project cleanly through the stored PCA and UMAP objects without errors, confirming consistent dimensionality with data_pt['pca'] and data_pt['um']

## Limitations

- Information is lost about individual genes when transforming data into PCA space, which may obscure subpopulation-defining markers
- Lowly-expressed genes important to cell fate decisions may be dropped altogether in the scRNA-seq data
- Association of certain genes with specific cell fates does not necessarily imply causality, so stratified perturbation shifts suggest but do not prove stage-specific regulatory roles
- There remain challenges to confidently suggest gene sets for experimental perturbation

## Evidence

- [abstract] PRESCIENT is able to accommodate complex perturbations of multiple genes, at different time points and from different starting cell populations: "PRESCIENT is able to accommodate complex perturbations of multiple genes, at different time points and from different starting cell populations"
- [methods] data_pt contents including pca and umap objects: "data_pt["pca"]: sklearn.decomposition.PCA object fit to normalized expression and used to produce PCs."
- [methods] data_pt umap object: "data_pt["um"]: umap.UMAP object fit to PCs used to produce UMAP dims."
- [methods] prescient process_data generates the torch pt file: "The `prescient process_data` command will generate a torch pt file `data_pt` (serialized dictionary) that contains all the necessary information for downstream training, simulations, and"
- [discussion] large-scale TF screen in β-cell differentiation: "large-scale screen for in vitro β-cell differentiation in which we perturbed 200+ TFs and identified target genes that could cause significant shifts in β-cell, α-cell, and EC-cell fates."
- [readme] prescient-analysis covers Veres beta-cell differentiation workflows: [evidence span withheld — anchor status: external_doc]
