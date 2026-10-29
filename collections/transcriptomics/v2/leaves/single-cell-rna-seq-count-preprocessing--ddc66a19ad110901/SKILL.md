---
name: single-cell-rna-seq-count-preprocessing
description: Use when raw scRNA-seq count matrices (e.g., Mouse Cell Atlas bone marrow)
  must be converted into SINGE-compatible .mat files with expression matrix X, gene_list,
  pseudotimes, and an optional TF-only regix before GLG network inference.
when_to_use_negative:
- The expression data are already pseudotemporally ordered, log-transformed, and packaged
  in a SINGE-format .mat file with gene_list and ptime — rerunning preprocessing would
  duplicate work.
- The trajectory method already provides pseudotimes and no regulator restriction
  is needed, so regix and dynwrap add_pseudotime steps are unnecessary.
- The task is target-gene selection or aggregation of GLG outputs (SINGE Aggregate),
  not preparation of the expression input.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SINGE
  role: Consumes the preprocessed .mat input (X, ptime, regix) in GLG mode; its hyperparameter
    conventions (--family, --lambda) dictate the preprocessing choices
  repo: https://github.com/gitter-lab/SINGE
- name: glmnet_matlab
  role: Regression backend invoked during GLG; --family gaussian on log-transformed
    counts avoids its memory segmentation violations
  repo: http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- name: dynverse
  role: Used to infer trajectories from the cells and generate pseudotimes for the
    ordered expression data
  repo: http://dynverse.org/
- name: dynwrap
  role: Provides add_pseudotime to assign pseudotemporal values when the chosen trajectory
    method does not supply them
  repo: https://rdrr.io/github/dynverse/dynwrap/man/add_pseudotime.html
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:65926070ccb784578462314b0341b1d6b7835a7e4d64393aaca32ea373d4fa70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB R2018a
  - SINGE
  - dynverse
  - dynwrap
  - glmnet_matlab
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# single-cell-rna-seq-count-preprocessing

## Summary

Prepares pseudotemporal single-cell RNA-seq count data into the .mat inputs SINGE requires: log-transformed expression matrix (X), gene_list, ptime values, and an optional regix vector restricting candidate regulators. It also selects the glmnet family setting appropriate for count-based data.

## When to use

Use when raw scRNA-seq count matrices (e.g., Mouse Cell Atlas bone marrow) must be converted into SINGE-compatible .mat files with expression matrix X, gene_list, pseudotimes, and an optional TF-only regix before GLG network inference.

## When NOT to use

- The expression data are already pseudotemporally ordered, log-transformed, and packaged in a SINGE-format .mat file with gene_list and ptime — rerunning preprocessing would duplicate work.
- The trajectory method already provides pseudotimes and no regulator restriction is needed, so regix and dynwrap add_pseudotime steps are unnecessary.
- The task is target-gene selection or aggregation of GLG outputs (SINGE Aggregate), not preparation of the expression input.

## Inputs

- scRNA-seq count matrix (counts to be log-transformed as X1 = log(1+X))
- gene list file (e.g., data1/gene_list.mat) with names matching rows of X
- pseudotimes inferred via dynverse (or cells requiring trajectory inference)
- optional regulator index set (e.g., 149 AnimalTFDB 3.0 transcription factors) encoded as regix
- optional branches matrix for branching trajectories

## Outputs

- .mat input file with sparse matrix X (log-transformed), ptime array, gene_list, and optional regix/branches (e.g., data_bifurcated/X_data_bifurcated.mat style)
- hyperparameter file (e.g., default_hyperparameters.txt) with --family gaussian and nonzero lambda values
- pseudotemporal ordering of cells suitable for GLG Granger regression

## How to apply

Load the count matrix and log-transform it as X1 = log(1+X), then run glmnet with the --family gaussian hyperparameter rather than poisson, because count-based data processed this way performs better and avoids glmnet_matlab memory segmentation violations on large datasets. Infer a trajectory from the cells with the dynverse package to generate pseudotimes; if the chosen trajectory method does not emit them, assign them after the fact using the dynwrap add_pseudotime function. Assemble the input .mat file containing the sparse expression matrix X, the ptime array, the gene_list matching rows of X, and — if restricting regulators — a regix vector indexing candidate regulators (e.g., the 149 AnimalTFDB 3.0 transcription factors for the bone marrow case study). For branching trajectories, construct the branches matrix using dynverse or analyze each sub-trajectory (branch) independently. Avoid lambda=0 in downstream hyperparameter files since a non-sparse, complete graph over thousands of genes is impractical and increases segmentation-violation risk.

## Related tools

- **SINGE** (Consumes the preprocessed .mat input (X, ptime, regix) in GLG mode; its hyperparameter conventions (--family, --lambda) dictate the preprocessing choices) — https://github.com/gitter-lab/SINGE
- **glmnet_matlab** (Regression backend invoked during GLG; --family gaussian on log-transformed counts avoids its memory segmentation violations) — http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- **dynverse** (Used to infer trajectories from the cells and generate pseudotimes for the ordered expression data) — http://dynverse.org/
- **dynwrap** (Provides add_pseudotime to assign pseudotemporal values when the chosen trajectory method does not supply them) — https://rdrr.io/github/dynverse/dynwrap/man/add_pseudotime.html

## Examples

```
X1 = log(1+X); save('X_data.mat', 'X1', 'ptime', 'gene_list', 'regix'); docker run -v $(pwd):/SINGE -w /SINGE agitter/singe:0.5.1 standalone X_data.mat gene_list.mat Output default_hyperparameters.txt
```

## Evaluation signals

- The .mat file contains a sparse matrix X, a ptime array with one value per cell, a gene_list matching the rows of X, and (if used) a regix vector whose entries are valid row indices (e.g., 149 TF indices).
- Log-transformed counts follow exactly X1 = log(1+X) and the hyperparameter file specifies --family gaussian, not poisson.
- No GLG hyperparameter combination uses lambda=0; all lambda entries yield sparse solutions.
- glmnet_matlab runs complete without memory segmentation violations on the large dataset.
- For branching data, branches sub-trajectories are either encoded in the .mat branches matrix or each branch is stored and analyzed independently.

## Limitations

- The glmnet_matlab package encounters memory segmentation violations with certain settings on large datasets, motivating --family gaussian and avoiding lambda=0.
- A complete (non-sparse) graph from lambda=0 over thousands of genes is impractical and increases segmentation-violation risk.
- For branching trajectories, SINGE v0.5.0 combines GLG results across all branches by default; branch-specific networks require storing per-branch GLG results and running SINGE Aggregate separately.
- Pseudotimes may be uninformative for some datasets, a caveat the authors note about individual regulator performance.

## Evidence

- [methods] Log-transform and gaussian family choice: "we recommend log-transforming the count-based data (`X1 = log(1+X)`) and using the `--family gaussian` hyperparameter instead."
- [methods] Regulator restriction feature: "SINGE version 0.3.0 introduced functionality where a subset of the gene list is earmarked as candidate regulators."
- [readme] Input .mat schema: [evidence span withheld — anchor status: external_doc]
- [methods] Pseudotime generation guidance: "pseudotimes can be generated by inferring a trajectory from the cells."
- [methods] Segmentation violation caveat: "the glmnet_matlab package encounters memory segmentation violations with this setting"
