---
name: knn-neighbor-graph-construction
description: Use when a velocity or trajectory pipeline needs a cell-neighborhood
  graph over preprocessed scRNA-seq AnnData (e.g. after scvelo pp.filter_and_normalize
  with top-2000 highly variable genes) to support moment smoothing or local averaging.
when_to_use_negative:
- The count matrix has not yet been filtered and normalized (scvelo pp.filter_and_normalize
  is a prerequisite; building the graph on raw counts gives unreliable neighborhoods)
- The analysis needs splicing-based RNA velocity dynamics rather than a cell graph
  for TF-expression smoothing — TFvelo's graph step exists to support TF-based dynamics
  without relying on unspliced/spliced kinetics
- The input is not a single-cell count AnnData (e.g. bulk or already-aggregated expression
  tables have no meaningful per-cell neighborhoods)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ScVelo
  role: Provides pp.neighbors (n_neighbors=30) and pp.moments used to build the k-NN
    cell graph and smooth total mRNA counts in the TFvelo preprocessing step
  repo: https://github.com/theislab/scvelo
- name: anndata
  role: Holds the count matrix and the resulting neighbor graph/annotations in the
    .h5ad object passed through the pipeline
- name: scanpy
  role: Part of the pinned environment (scanpy==1.8.2) underlying the preprocessing
    and AnnData handling
- name: TFvelo
  role: Pipeline that exposes the graph size as the --n_neighbors hyperparameter and
    consumes the graph via smoothed TF expression for gene-dynamics fitting
  repo: https://github.com/xiaoyeye/TFvelo
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:9179e41cb36dd25f4b31e6fc383b5c09e453448083f3aa24dcf45683efe34c0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scvelo
  tools:
  - ScVelo
  - TFvelo
  - anndata
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# knn-neighbor-graph-construction

## Summary

Constructs a k-nearest-neighbor cell graph on filtered/normalized scRNA-seq counts with scvelo pp.neighbors (n_neighbors=30 in TFvelo), providing the connectivity structure over which scvelo pp.moments smooths total (unspliced+spliced) mRNA counts before TF-model fitting.

## When to use

Use when a velocity or trajectory pipeline needs a cell-neighborhood graph over preprocessed scRNA-seq AnnData (e.g. after scvelo pp.filter_and_normalize with top-2000 highly variable genes) to support moment smoothing or local averaging.

## When NOT to use

- The count matrix has not yet been filtered and normalized (scvelo pp.filter_and_normalize is a prerequisite; building the graph on raw counts gives unreliable neighborhoods)
- The analysis needs splicing-based RNA velocity dynamics rather than a cell graph for TF-expression smoothing — TFvelo's graph step exists to support TF-based dynamics without relying on unspliced/spliced kinetics
- The input is not a single-cell count AnnData (e.g. bulk or already-aggregated expression tables have no meaningful per-cell neighborhoods)

## Inputs

- Filtered and normalized scRNA-seq AnnData (.h5ad) with top-2000 highly variable genes retained (e.g. loaded via anndata read_h5ad or the auto-downloaded pancreas dataset)
- Value for the number of neighbors (TFvelo default n_neighbors=30)

## Outputs

- k-NN neighbor graph stored in the AnnData object (connectivities/distances)
- Smoothed first/second-order moments of total (unspliced+spliced) mRNA counts computed over the neighbor graph
- Preprocessed AnnData written to the 'TFvelo_'+dataset_name+'_demo' folder for downstream TF-model fitting

## How to apply

First filter and normalize the count matrix with scvelo pp.filter_and_normalize, retaining the top-2000 highly variable genes, since graph edges on unnormalized data are dominated by library-size effects. Then run scvelo pp.neighbors with n_neighbors=30 (TFvelo's setting, exposed as the --n_neighbors hyperparameter) to build the k-NN connectivity graph in the AnnData object. Use the resulting graph as the input to scvelo pp.moments computed on the total (unspliced+spliced) mRNA counts, so that TF expression used by TFvelo's linear weighting (WX_method=lsq_linear, correlation-initialized) is locally smoothed. Persist the graph in the preprocessed AnnData that is saved to the 'TFvelo_'+dataset_name+'_demo' folder, because downstream TF-model fitting and TFvelo_analysis_demo.py visualization (pseudotime, streamplot on UMAP) consume the same neighborhood structure.

## Related tools

- **ScVelo** (Provides pp.neighbors (n_neighbors=30) and pp.moments used to build the k-NN cell graph and smooth total mRNA counts in the TFvelo preprocessing step) — https://github.com/theislab/scvelo
- **anndata** (Holds the count matrix and the resulting neighbor graph/annotations in the .h5ad object passed through the pipeline)
- **scanpy** (Part of the pinned environment (scanpy==1.8.2) underlying the preprocessing and AnnData handling)
- **TFvelo** (Pipeline that exposes the graph size as the --n_neighbors hyperparameter and consumes the graph via smoothed TF expression for gene-dynamics fitting) — https://github.com/xiaoyeye/TFvelo

## Examples

```
import scvelo as scv; scv.pp.neighbors(adata, n_neighbors=30); scv.pp.moments(adata)
```

## Evaluation signals

- The preprocessed .h5ad file exists in the 'TFvelo_'+dataset_name+'_demo' folder, loads with anndata, and contains the neighbor graph
- The graph was built on the normalized, HVG-restricted matrix (top-2000 highly variable genes), not on raw counts
- n_neighbors matches the configured value (30 in the TFvelo demo) and pp.moments were computed over that graph on total (unspliced+spliced) counts
- Downstream TF-model fitting (TFvelo_run_demo.py) and visualization (TFvelo_analysis_demo.py: pseudotime, streamplot on UMAP) run without errors on the saved AnnData
- No numeric acceptance threshold is reported in the paper, so structural checks (file loads, graph present, per-gene TF annotations present) are the qualitative acceptance criterion

## Limitations

- TFvelo is slower than scVelo baselines, which is a weakness for large-scale datasets; parallelizing with more CPUs (--n_jobs) reduces runtime
- No numeric quality threshold for the graph/moments is reported in the paper, so validation is structural rather than quantitative
- Choice of n_neighbors directly affects the smoothing that feeds TF weight estimation; the paper only documents the demo value (30)

## Evidence

- [methods] Neighbor graph with 30 neighbors before moment smoothing: [evidence span withheld — anchor status: not_found]
- [readme] n_neighbors hyperparameter documented in README: "--n_neighbors: The number of neighbors."
- [readme] Default demo run that performs download, preprocessing and TF model fitting: "This will automatically download, preprocess and run TF model on pancrease dataset. The result will be stored in 'TFvelo_pancreas/rc.h5ad'."
- [methods] HVG restriction prior to graph construction: [evidence span withheld — anchor status: not_found]
