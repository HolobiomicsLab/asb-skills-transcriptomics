---
name: integration-baseline-pipeline-comparison
description: Use when benchmarking whether a clustering/integration method resolves
  perturbation-unique cell states in control/perturbed single-cell transcriptome data
  with known ground-truth labels, across perturbation-severity, marker-gene, and cell-proportion
  conditions.
when_to_use_negative:
- The dataset is a single condition with no control/perturbed treatment labels — ICAT's
  semi-supervised clustering requires control-defined cell states to anchor the sslouvain
  step, so no comparison arm can be run
- No ground-truth population labels exist, so agreement metrics (ARI, F1, LISI against
  truth) cannot be computed and the benchmark cannot be scored
- The goal is batch-effect removal for downstream visualization rather than testing
  whether integration workflows miss perturbation-unique cell states
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ICAT / icat-sc
  role: 'Method under comparison: self-supervised NCFS feature weighting plus control-guided
    semi-supervised Louvain clustering producing sslouvain assignments in the upweighted
    X_icat space'
  repo: https://github.com/BradhamLab/icat
- name: scanpy
  role: Preprocessing (sc.pp.log1p), neighbor computation, Louvain clustering, and
    the AnnData framework through which all arms of the comparison run
- name: AnnData
  role: Data structure holding concatenated control/perturbed matrices, treatment
    labels, and returned X_icat / sslouvain outputs
- name: Seurat
  role: 'Integration baseline arm: Seurat integration followed by Louvain clustering,
    scored against ground truth'
- name: Scanorama
  role: 'Integration baseline arm: Scanorama integration followed by Louvain clustering,
    scored against ground truth'
- name: Harmony
  role: 'Integration baseline arm: Harmony integration followed by Louvain clustering,
    scored against ground truth'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:1656e1f9aaa4da83882d85751255216fe493d10fdbda8bc7100586770bccd6d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - AnnData
  - Harmony
  - ICAT / icat-sc
  - Scanorama
  - Seurat
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# integration-baseline-pipeline-comparison

## Summary

Head-to-head benchmarking of ICAT's control-guided semi-supervised Louvain clustering against Seurat, Scanorama, Harmony, and no-integration Louvain baselines on simulated control/perturbed scRNA-seq datasets, scoring cluster recovery (ARI, LISI, F1) against ground-truth population labels.

## When to use

Use when benchmarking whether a clustering/integration method resolves perturbation-unique cell states in control/perturbed single-cell transcriptome data with known ground-truth labels, across perturbation-severity, marker-gene, and cell-proportion conditions.

## When NOT to use

- The dataset is a single condition with no control/perturbed treatment labels — ICAT's semi-supervised clustering requires control-defined cell states to anchor the sslouvain step, so no comparison arm can be run
- No ground-truth population labels exist, so agreement metrics (ARI, F1, LISI against truth) cannot be computed and the benchmark cannot be scored
- The goal is batch-effect removal for downstream visualization rather than testing whether integration workflows miss perturbation-unique cell states

## Inputs

- Simulated control and perturbed single-cell AnnData objects (perturbation severity 1-25%, marker-gene counts 10-105, varied cell type proportions) from the BradhamLab/ICAT_manuscript archived_data.tar.gz
- Ground-truth population labels for the simulated cells
- Treatment labels (obs['treatment'] with 'control'/'perturbed' values) used by ICAT's semi-supervised clustering
- Control-cell labels for hyperparameter optimization (controls.obs.louvain as NCFS target)

## Outputs

- Per-method, per-condition clustering score table (ARI, LISI, F1) for ICAT, Seurat+Louvain, Scanorama+Louvain, Harmony+Louvain, and no-integration Louvain
- Semi-supervised cluster assignments stored in out.obs['sslouvain'] of the returned AnnData
- Regenerated Fig 3F/G metric comparison panel saved as fig3F_G_metric_panel.png

## How to apply

Load the archived simulated control/perturbed AnnData objects from BradhamLab/ICAT_manuscript's archived_data.tar.gz (perturbation severity 1-25%, 10-105 marker genes, varied cell-proportion conditions) together with the ground-truth population labels. Run the ICAT arm: concatenate control and perturbed cells, apply sc.pp.log1p, grid-search Louvain parameters (n_neighbors 3-50, resolution 0.3-1.2) on control cells via optimize_louvain maximizing the Calinski-Harabasz Index, grid-search NCFS sigma (0.5-3) and reg (0.25-3) via optimize_ncfs using MCC of a weighted 5-NN under k-fold cross-validation, then call models.icat with treatment labels and collect the sslouvain assignments. Run the baseline arms on the same combined matrices: Seurat+Louvain, Scanorama+Louvain, Harmony+Louvain, and no-integration Louvain clustering of the merged control/perturbed data. Score each method's clustering against the ground-truth labels with ARI, LISI, and F1 for every severity, marker-gene, and cell-proportion condition, then aggregate per-method per-condition metrics to regenerate the Fig 3F/G comparison panel (fig3F_G_metric_panel.png). The validation criterion is that only ICAT resolves the perturbation-unique cellular response and its metric values stay highly stable as perturbation intensity and cell-state resolution vary.

## Related tools

- **ICAT / icat-sc** (Method under comparison: self-supervised NCFS feature weighting plus control-guided semi-supervised Louvain clustering producing sslouvain assignments in the upweighted X_icat space) — https://github.com/BradhamLab/icat
- **scanpy** (Preprocessing (sc.pp.log1p), neighbor computation, Louvain clustering, and the AnnData framework through which all arms of the comparison run)
- **AnnData** (Data structure holding concatenated control/perturbed matrices, treatment labels, and returned X_icat / sslouvain outputs)
- **Seurat** (Integration baseline arm: Seurat integration followed by Louvain clustering, scored against ground truth)
- **Scanorama** (Integration baseline arm: Scanorama integration followed by Louvain clustering, scored against ground truth)
- **Harmony** (Integration baseline arm: Harmony integration followed by Louvain clustering, scored against ground truth)

## Examples

```
model = models.icat(ctrl_value="control", ncfs_kws={'reg': 1, 'sigma': 3}, neighbor_kws={'n_neighbors': 15}, cluster_kws={'resolution': 0.75}); out = model.cluster(adata, adata.obs['treatment'])
```

## Evaluation signals

- Only ICAT resolves the perturbation-unique cellular response across perturbation-severity and cell-proportion conditions, while Scanorama, Seurat, Harmony, and no-integration baselines fail to separate it
- ICAT's metric values remain highly stable as perturbation intensity (1-25% severity) and marker-gene count (10-105) vary; baselines degrade with severity
- sslouvain assignments keep originally identified control labels constant while perturbed cells join clusters freely regardless of treatment status
- optimize_louvain returns the grid point maximizing the Calinski-Harabasz Index over n_neighbors 3-50 and resolution 0.3-1.2; optimize_ncfs returns sigma/reg maximizing MCC in a weighted 5-NN k-fold cross-validation
- The aggregated per-method per-condition table reproduces the reported Fig 3F/G pattern and the panel is saved as fig3F_G_metric_panel.png

## Limitations

- ICAT does not automatically compute UMAP, tSNE, or other reduced-dimension visualizations; the upweighted count matrix in adata.obsm['X_icat'] must be passed to these algorithms manually
- Hyperparameters are dataset-dependent — the README recommends grid-searching Louvain and NCFS parameters on your own data prior to running the full workflow
- The manuscript analysis was run on a high-performance cluster with a CentOS conda environment and 'is not guaranteed to run on your system out-of-the-box'; icat-sc installs via pip on Linux machines

## Evidence

- [abstract] We empirically validate ICAT in a developmental model and find that only ICAT identifies a perturbation-unique cellular response.: "We empirically validate ICAT in a developmental model and find that only ICAT identifies a perturbation-unique cellular response."
- [abstract] ICAT is robust to low signal strength, high perturbation severity, and disparate cell type proportions without requiring a priori knowledge of cell states or marker genes: [evidence span withheld — anchor status: not_found]
- [intro] perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells: "perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells"
- [readme] We have also provided grid search functions to find the "best" `n_neighbor` and `resolution` parameters for Louvain and Semi-supervised Louvain clustering steps: "We have also provided grid search functions to find the "best" `n_neighbor` and `resolution` parameters for Louvain and Semi-supervised Louvain clustering steps"
