---
name: pseudotime-normalization-single-cell
description: Use when single-cell expression data has an ordering along an inferred
  trajectory and must be formatted (ptime array, 0-100 rescale, sparse X, optional
  branches/regix) before running SINGE GLG.
when_to_use_negative:
- Data have no temporal ordering or trajectory — the article notes caveats about poor
  performance with uninformative pseudotimes
- Data are bulk time-series rather than ordered single-cell expression measurements
- You already have a ranked regulator-gene edge list and only need aggregation (use
  SINGE Aggregate instead)
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: SINGE
  role: Consumes the rescaled pseudotime input for kernel-based GLG Granger regression
    and ensemble aggregation
  repo: https://github.com/gitter-lab/SINGE
- name: glmnet_matlab
  role: Backend used by SINGE GLG for the kernel-based Lasso Granger regression over
    the normalized pseudotemporally ordered data
  repo: http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- name: dynwrap (dynverse add_pseudotime)
  role: Assigns pseudotimes to cells after trajectory inference when they must be
    generated rather than rescaled
  repo: https://rdrr.io/github/dynverse/dynwrap/man/add_pseudotime.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:fd1c59173e881d549b5646c4e985131b8d8b51292692a4929448c6e27b3d4500
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB R2018a
  - SINGE
  - dynwrap (dynverse add_pseudotime)
  - glmnet_matlab
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# pseudotime-normalization-single-cell

## Summary

Prepares pseudotemporally ordered single-cell gene expression data for SINGE's kernel-based Granger regression by rescaling pseudotimes to the 0-100 range the Gaussian kernel smoothing expects and packaging them with the sparse expression matrix in a .mat file.

## When to use

Use when single-cell expression data has an ordering along an inferred trajectory and must be formatted (ptime array, 0-100 rescale, sparse X, optional branches/regix) before running SINGE GLG.

## When NOT to use

- Data have no temporal ordering or trajectory — the article notes caveats about poor performance with uninformative pseudotimes
- Data are bulk time-series rather than ordered single-cell expression measurements
- You already have a ranked regulator-gene edge list and only need aggregation (use SINGE Aggregate instead)

## Inputs

- matfile with ordered single-cell expression data (sparse matrix X) and pseudotime values (array ptime)
- gene_list.mat with gene names corresponding to rows of X
- optional regix array of regulator index values
- optional branches matrix for branching trajectories (e.g. data_bifurcated/X_data_bifurcated.mat)
- hyperparameter file (e.g. default_hyperparameters.txt) including kernel-width, dT, num-lags, lambda, family

## Outputs

- Normalized SINGE input Data structure with ptime rescaled to 0-100
- SINGE_Ranked_Edge_List.txt of ranked regulator-target edges
- SINGE_Gene_Influence.txt of genes ranked by SINGE influence

## How to apply

Load the expression matrix X (sparse), gene_list, and pseudotimes; if pseudotimes are absent, infer a trajectory with dynverse/dynwrap and attach them with the add_pseudotime function. Rescale the pseudotimes to the 0-100 range required by the SINGE kernel smoothing before storing them as the ptime array in the input .mat file. For count-based data, log-transform (X1 = log(1+X)) and run GLG with the --family gaussian hyperparameter; optionally apply the prob-zero-removal zero-handling strategy. For branching trajectories, supply a branches matrix (e.g. data_bifurcated/X_data_bifurcated.mat) or split the trajectory into sub-trajectories per cell fate and analyze each independently, because kernel-based Granger regression smooths irregular pseudotimes and missing values along the ordered axis. Mark candidate regulators via the optional regix index array (SINGE v0.3.0+).

## Related tools

- **SINGE** (Consumes the rescaled pseudotime input for kernel-based GLG Granger regression and ensemble aggregation) — https://github.com/gitter-lab/SINGE
- **glmnet_matlab** (Backend used by SINGE GLG for the kernel-based Lasso Granger regression over the normalized pseudotemporally ordered data) — http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- **dynwrap (dynverse add_pseudotime)** (Assigns pseudotimes to cells after trajectory inference when they must be generated rather than rescaled) — https://rdrr.io/github/dynverse/dynwrap/man/add_pseudotime.html

## Examples

```
docker run -v $(pwd):/SINGE -w /SINGE agitter/singe:0.5.1 standalone data1/X_SCODE_data.mat data1/gene_list.mat Output default_hyperparameters.txt
```

## Evaluation signals

- All ptime values lie within the 0-100 range expected by the SINGE kernel smoothing
- Length of the ptime array matches the number of cells (columns) of the sparse expression matrix X
- If a branches matrix is present, branch assignments cover the same cells as ptime and X
- Same input .mat file, gene_list, hyperparameters, and --date reproduce identical SINGE output via the generated random seed
- Generated output matches tests/reference output when compared with tests/compare_example_output.sh

## Limitations

- Uninformative pseudotimes or individual regulators can yield poor network inference performance
- Because subsampling and zero-removal use pseudo-random sample removals, results are only reproducible if the same --date input is supplied
- The glmnet_matlab package encounters memory segmentation violations with some settings
- The repository master branch may be unstable; use a versioned release for stable data analysis

## Evidence

- [abstract] Kernel-based Granger causality regression smooths irregular pseudotimes: "SINGE uses kernel-based Granger causality regression to smooth irregular pseudotimes and missing expression values."
- [readme] SINGE input matfile format with ptime and branches: [evidence span withheld — anchor status: external_doc]
- [readme] Example linear and bifurcated trajectory datasets: [evidence span withheld — anchor status: external_doc]
- [methods] Generating pseudotimes by trajectory inference: "pseudotimes can be generated by inferring a trajectory from the cells."
- [methods] Log-transform recommendation for count data: "we recommend log-transforming the count-based data (`X1 = log(1+X)`) and using the `--family gaussian` hyperparameter instead."
