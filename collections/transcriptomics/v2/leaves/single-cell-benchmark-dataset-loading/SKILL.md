---
name: single-cell-benchmark-dataset-loading
description: Use when a study needs the Goolam or Yan scRNA-seq matrices (or the MPSSC
  benchmark .mat sets) with cell/true_labs labels to benchmark similarity learning
  and clustering methods.
when_to_use_negative:
- The input is already a precomputed cell-to-cell similarity matrix rather than a
  raw expression matrix — skip loading and proceed to embedding or clustering
- The target dataset is a large-scale scRNA-seq set beyond the nine benchmark .mat
  files, since SSRE is relatively time-consuming on large-scale datasets and MPSSC
  large runs required a 6-CPU, 800 GB computing cluster
- A clustering method other than SSRE/SIMLR/MPSSC/SSR/t-SNE defines its own data schema
  that does not follow the in_X / true_labs .mat convention
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SC3
  role: Source repository (hemberg-lab/SC3) from which the Goolam and Yan benchmark
    single-cell expression datasets are obtained
  repo: https://github.com/hemberg-lab/SC3
- name: SSRE
  role: MATLAB/Python implementation that consumes the loaded expression matrices
    and learns cell-to-cell similarities using all three pairwise similarities by
    default
  repo: https://github.com/CSUBioGroup/SSRE
- name: MPSSC
  role: MATLAB pipeline whose Data directory hosts the nine benchmark .mat datasets
    (in_X, true_labs) and example scripts for loading and evaluating them
  repo: https://github.com/ishspsy/project
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:bfef8578ca0d721aa71e4a5bfafbbb3fdb37c512f27d5b0a80ac968dd2c9d407
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hemberg-lab/SC3
  tools:
  - MATLAB
  - MPSSC
  - Python
  - SC3
  - SSRE
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-benchmark-dataset-loading

## Summary

Loads the benchmark scRNA-seq expression matrices (e.g., Goolam with five and Yan with seven developmental stages) from the hemberg-lab/SC3 data repository, and the nine .mat datasets (Deng, Ting, Treutlein, Ginhoux, Buettner, Pollen, Zeisel, Tasic, Macosko) from the MPSSC repository, so similarity-learning methods like SSRE, SIMLR, MPSSC, SSR and native t-SNE can be compared on identical inputs.

## When to use

Use when a study needs the Goolam or Yan scRNA-seq matrices (or the MPSSC benchmark .mat sets) with cell/true_labs labels to benchmark similarity learning and clustering methods.

## When NOT to use

- The input is already a precomputed cell-to-cell similarity matrix rather than a raw expression matrix — skip loading and proceed to embedding or clustering
- The target dataset is a large-scale scRNA-seq set beyond the nine benchmark .mat files, since SSRE is relatively time-consuming on large-scale datasets and MPSSC large runs required a 6-CPU, 800 GB computing cluster
- A clustering method other than SSRE/SIMLR/MPSSC/SSR/t-SNE defines its own data schema that does not follow the in_X / true_labs .mat convention

## Inputs

- Goolam scRNA-seq expression matrix (five developmental stages) from hemberg-lab/SC3
- Yan scRNA-seq expression matrix (seven developmental stages) from hemberg-lab/SC3
- MPSSC benchmark .mat files (Data_Deng.mat, Data_Ting.mat, Data_Treutlin.mat, Data_Ginhoux.mat, Data_Buettner.mat, Data_Pollen.mat, Data_Zeisel.mat, Data_Tasic.mat, Data_Macosko.mat) each with in_X (n cells x p genes) and true_labs

## Outputs

- In-memory n-by-p gene expression matrices ready for similarity learning or clustering
- Ground-truth label vectors (true_labs) aligned with cell rows for NMI, Purity and ARI evaluation
- Dataset scripts/loads compatible with the SSRE (MATLAB/Python) and MPSSC (MATLAB) pipelines

## How to apply

Fetch the Goolam and Yan expression matrices, covering five and seven developmental stages respectively, from the hemberg-lab/SC3 repository, or the nine .mat files (each containing in_X, an n-by-p gene expression matrix with n cells and p genes, plus true_labs ground-truth labels) from the ishspsy/project Data directory. For the SSRE comparison, load each matrix into MATLAB or Python, then compute or load the cell-to-cell similarity matrices for the six methods (native t-SNE, Corr, SIMLR, MPSSC, SSR, SSRE), using the default SSRE combination of all three pairwise similarities. For MPSSC-style runs, follow the README penalty parameters (rho=0.2, lam=0.0001, lam2=lam, eta=1, c=0.1) so benchmark results are comparable to the published figures. Validate each loaded dataset by checking the presence of in_X and true_labs and the expected number of stage groups before passing the matrices to similarity learning or modified t-SNE embedding.

## Related tools

- **SC3** (Source repository (hemberg-lab/SC3) from which the Goolam and Yan benchmark single-cell expression datasets are obtained) — https://github.com/hemberg-lab/SC3
- **SSRE** (MATLAB/Python implementation that consumes the loaded expression matrices and learns cell-to-cell similarities using all three pairwise similarities by default) — https://github.com/CSUBioGroup/SSRE
- **MPSSC** (MATLAB pipeline whose Data directory hosts the nine benchmark .mat datasets (in_X, true_labs) and example scripts for loading and evaluating them) — https://github.com/ishspsy/project

## Examples

```
load('Data_Deng.mat'); [P] = clus_fin_update(0.2, 0.0001, 0.0001, 1, 0.1, in_X, true_labs); [NMI,~,clus_labs,~]=calc2_nmis(max(true_labs), double(P),true_labs);
```

## Evaluation signals

- Each loaded .mat contains both in_X (n-by-p expression matrix) and true_labs, with n equal to the number of cells and p the number of genes
- Goolam data resolves to five developmental-stage groups and Yan data to seven, with oocyte and 2-cell groups identifiable among Yan stages
- Downstream similarity learning with SSRE (default combination of all three pairwise similarities) yields 2D modified t-SNE panels in which oocyte and 2-cell cells are visually separated from other stages, matching Figure 3's reported result
- Benchmark evaluation metrics (NMI, Purity, ARI) compute without error against true_labs, as in the MPSSC calc2_nmis/purity/RandIndex workflow
- Data source URLs resolve to the original publications (e.g., Deng → science.sciencemag.org/content/343/6167/193, Pollen → nature.com/articles/nbt.2967) as documented in the MPSSC README

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is suggested as a future acceleration strategy
- MPSSC's large-scale scRNA-seq runs required a computing cluster with 6 CPUs and 800 GB of memory, so loading and running on a laptop (README baseline: MacBook Pro 2.7 GHz, 8 GB, MATLAB 2016b) is only feasible for small-scale sets
- Benchmark data are limited to the specific published sets (Goolam/Yan from SC3; nine .mat sets in MPSSC) and need manual curation for other studies

## Evidence

- [other] Goolam and Yan scRNA-seq expression matrices (five and seven developmental stages, respectively) from the hemberg-lab/SC3 data repository: [evidence span withheld — anchor status: not_found]
- [abstract] The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE.: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [readme] Here n and p are number of cells and genes, respectively.: "Note: one can use any data set that consists of in_X and true_labs, where in_X is an n by p gene 
expression matrix and true_labs is the ground truth labels."
- [readme] The 9 data sets are provided in the directory: [evidence span withheld — anchor status: not_found]
- [abstract] Tested on ten real scRNA-seq datasets and five simulated datasets: "Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods."
