---
name: regulator-list-construction-in-mat-files
description: Use when preparing SINGE inputs for pseudotemporal single-cell expression
  data and only a subset of genes (e.g., transcription factors) should be considered
  as candidate regulators rather than all genes in gene_list.mat.
when_to_use_negative:
- The analysis goal is to infer edges from every gene as a potential regulator — omitting
  regix lets SINGE search the full gene list
- The expression data are not ordered by pseudotime (no ptime array), since SINGE
  requires pseudotemporal ordering for Granger regression
- Regulator identities cannot be justified (e.g., no known transcription factor list),
  as an arbitrary subset will bias the ranked interaction list
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SINGE
  role: Consumes the regix-containing .mat file; version 0.3.0 introduced the candidate-regulator
    subset functionality used during GLG regression and aggregation
  repo: https://github.com/gitter-lab/SINGE
- name: MATLAB
  role: Environment in which the .mat input files with X, ptime, regix, and branches
    are created and in which SINGE is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:2e2d7bd0e44bff84faf406198956e6d3783f5ea1cc355fba17f53980f391d4e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB
  - SINGE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# regulator-list-construction-in-mat-files

## Summary

Constructs a candidate regulator subset for SINGE by earmarking a subset of the gene list as regulators and passing their row indices as the optional regix array inside the input .mat file, restricting network inference to regulator-target edges.

## When to use

Use when preparing SINGE inputs for pseudotemporal single-cell expression data and only a subset of genes (e.g., transcription factors) should be considered as candidate regulators rather than all genes in gene_list.mat.

## When NOT to use

- The analysis goal is to infer edges from every gene as a potential regulator — omitting regix lets SINGE search the full gene list
- The expression data are not ordered by pseudotime (no ptime array), since SINGE requires pseudotemporal ordering for Granger regression
- Regulator identities cannot be justified (e.g., no known transcription factor list), as an arbitrary subset will bias the ranked interaction list

## Inputs

- MAT-file with sparse expression matrix X, pseudotime array ptime (e.g., data1/X_SCODE_data.mat)
- gene_list.mat file listing gene names corresponding to rows of X
- regix index vector of candidate regulator row indices (embedded in the data .mat file)

## Outputs

- Input .mat file annotated with regix for downstream SINGE GLG and Aggregate runs
- Regulator-restricted SINGE_Ranked_Edge_List.txt and SINGE_Gene_Influence.txt (indirectly, after running SINGE)

## How to apply

Starting from the ordered single-cell expression matrix X and gene_list.mat, decide which genes are candidate transcriptional regulators and record their row indices in the expression matrix as an index vector named regix. Store regix in the input .mat file alongside X, ptime, and optionally branches (e.g., data1/X_SCODE_data.mat or data_bifurcated/X_data_bifurcated.mat formats); SINGE version 0.3.0 introduced this regulator-list functionality. When running SINGE GLG over the hyperparameter combinations in default_hyperparameters.txt, the regression restricts candidate regulators to the regix subset, so the ranked edge list contains only regulator-target edges from those genes. The regix indices must correspond to rows of X, not arbitrary gene IDs, so verify alignment with gene_list.mat before running. If regix is omitted, SINGE treats the full gene list as candidate regulators, which changes the search space and the modified Borda aggregation output.

## Related tools

- **SINGE** (Consumes the regix-containing .mat file; version 0.3.0 introduced the candidate-regulator subset functionality used during GLG regression and aggregation) — https://github.com/gitter-lab/SINGE
- **MATLAB** (Environment in which the .mat input files with X, ptime, regix, and branches are created and in which SINGE is executed)

## Examples

```
bash SINGE.sh PATH_TO_RUNTIME standalone data1/X_SCODE_data.mat data1/gene_list.mat Output default_hyperparameters.txt
```

## Evaluation signals

- regix indices are valid row indices into X and map to the intended gene names in gene_list.mat
- The input .mat file contains all required arrays (sparse X, ptime, regix; branches if the trajectory forks) and loads in MATLAB without error
- The resulting SINGE_Ranked_Edge_List.txt contains regulator genes drawn only from the regix subset
- Downstream SINGE runs complete without glmnet_matlab memory segmentation violations and avoid the lambda=0 complete-graph degenerate output

## Limitations

- Regulator subset choice directly constrains the inferred network; genes excluded from regix can never appear as regulators in the ranked edge list
- The master branch of SINGE may be unstable — a versioned release (>= 0.3.0 for regix support) should be used for reproducible analysis
- For large count-based datasets the .mat input should be log-transformed (X1 = log(1+X)) and --family gaussian used, or glmnet_matlab may encounter memory segmentation violations

## Evidence

- [readme] optional indices of regulators (array of index values `regix`): [evidence span withheld — anchor status: external_doc]
- [methods] SINGE version 0.3.0 introduced functionality where a subset of the gene list is earmarked as candidate regulators.: "SINGE version 0.3.0 introduced functionality where a subset of the gene list is earmarked as candidate regulators."
- [readme] data - Path to matfile with ordered single-cell expression data (sparse matrix `X`), pseudotime values (array `ptime`): [evidence span withheld — anchor status: external_doc]
