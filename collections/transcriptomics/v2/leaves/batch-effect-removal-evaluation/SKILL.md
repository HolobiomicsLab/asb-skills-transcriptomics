---
name: batch-effect-removal-evaluation
description: 'Use when assessing whether a scRNA-seq clustering method removes batch
  effects: you have count matrices with multiple batch metadata fields (e.g., donor,
  region, sample IDs) and reference cell-type labels to score with ARI.'
when_to_use_negative:
- The method under test explicitly requires batch information to correct batch effects
  (e.g., scVI, whose ARI dropped to 0.242 here when batch information was not utilized)
- No reference cell-type labels are available, so clustering accuracy cannot be scored
  with ARI
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DESC
  role: Deep embedding algorithm that clusters scRNA-seq data while gradually removing
    batch effects via iterative self-learning; trained with and without batch labels
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: Preprocessing (log1p, filter_genes_dispersion, scale) and ARI-based clustering
    evaluation
  repo: https://github.com/theislab/scanpy
- name: Keras
  role: Backend used to train the DESC deep embedding model
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:c0fd18db1df2fedccf405da95c3172268c6c9a39efd877e23e189e1d6a82b877
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - DESC
  - Keras
  - Scanpy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# batch-effect-removal-evaluation

## Summary

Evaluates whether a clustering pipeline (e.g., DESC) removes batch effects without explicit batch labels by comparing clustering accuracy (ARI against reference cell-type labels) across multiple batch definitions, including the no-batch-information case.

## When to use

Use when assessing whether a scRNA-seq clustering method removes batch effects: you have count matrices with multiple batch metadata fields (e.g., donor, region, sample IDs) and reference cell-type labels to score with ARI.

## When NOT to use

- The method under test explicitly requires batch information to correct batch effects (e.g., scVI, whose ARI dropped to 0.242 here when batch information was not utilized)
- No reference cell-type labels are available, so clustering accuracy cannot be scored with ARI

## Inputs

- GSE115746 macaque retina bipolar-cell count matrix (21,017 foveal + 9,285 peripheral cells)
- Cell metadata containing batch definitions: macaque id, region id, sample id
- Reference bipolar-cell type labels for ARI computation

## Outputs

- ari_results.csv with ARI values for each batch definition (macaque id, region id, sample id, none)
- DESC soft-clustering assignments for each batch condition

## How to apply

Download the count matrix with its batch metadata fields — for the macaque retina bipolar-cell dataset, GSE115746 provides 21,017 foveal and 9,285 peripheral cells annotated with macaque id, region id, and sample id. Preprocess with Scanpy (sc.pp.log1p, sc.pp.filter_genes_dispersion for highly variable gene selection, sc.pp.scale(max_value=6)), then run desc.train (dims=[n_genes,64,32], tol=0.005, n_neighbors=10, batch_size=256, louvain_resolution=[0.8,1.0], use_GPU=False, num_Cores=1) four times, supplying batch labels as macaque id, region id, sample id, and no batch information. Because DESC removes batch effects through iterative self-learning rather than explicit batch modeling, high accuracy without batch labels is the key test. Compute the adjusted Rand index of each clustering against the reference bipolar-cell type labels with scikit-learn/Scanpy and write all values to ari_results.csv.

## Related tools

- **DESC** (Deep embedding algorithm that clusters scRNA-seq data while gradually removing batch effects via iterative self-learning; trained with and without batch labels) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocessing (log1p, filter_genes_dispersion, scale) and ARI-based clustering evaluation) — https://github.com/theislab/scanpy
- **Keras** (Backend used to train the DESC deep embedding model)

## Examples

```
res = desc.train(adata, dims=[adata.shape[1],64,32], tol=0.005, n_neighbors=10, batch_size=256, louvain_resolution=[0.8,1.0], save_dir='save', use_GPU=False, num_Cores=1)
```

## Evaluation signals

- ARI with any batch definition (macaque id, region id, sample id) falls within 0.919–0.970
- ARI without batch information is ~0.920, matching the paper's reported value
- ARI computation must be run against the reference bipolar-cell type labels for all four conditions
- Desc training terminates by the stability criterion: proportion of cells changing cluster assignment between consecutive steps < tol = 0.005
- Cells are well mixed across batches even when no batch information is supplied

## Limitations

- DESC removes batch effects only as long as technical differences across batches are smaller than true biological variations
- Reproducing the paper's exact results requires tensorflow < 2.0 with desc 2.0.3; python must be 3.5.x or 3.6.x
- Without batch information, methods like scVI that explicitly incorporate batch information degrade sharply (ARI 0.242 on this dataset), so this evaluation design is specific to self-learning approaches

## Evidence

- [discussion] High ARI without batch labels: "we use ARI to compare the performance of different clustering algorithms. Larger values of ARI indicate higher accuracy in clustering."
- [abstract] Iterative self-learning rationale: "Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations"
- [readme] Reproducibility requirements: "you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper."
- [methods] HVG selection: "sc.pp.filter_genes_dispersion(adata,n_top_genes=1000)"
