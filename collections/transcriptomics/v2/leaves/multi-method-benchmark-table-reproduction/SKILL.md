---
name: multi-method-benchmark-table-reproduction
description: 'Use when replicating a published single-cell clustering benchmark: multiple
  clustering methods must be run on the same pre-annotated scRNA-seq matrices with
  known cluster counts and NMI/ARI assembled into a comparison table.'
when_to_use_negative:
- Datasets lack pre-annotated cell-type labels, so NMI/ARI against ground truth cannot
  be computed
- Large-scale datasets where runtime is critical — the article notes SSRE is 'relatively
  time-consuming in large-scale datasets' and defers acceleration to future parallel
  computing
- The goal is a single method's clustering rather than a multi-method comparison table
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: SSRE
  role: 'Focal method: adaptive gene selection plus similarity enhancement over three
    pairwise similarities, then spectral clustering on the learned similarity matrix'
  repo: https://github.com/CSUBioGroup/SSRE
- name: MPSSC
  role: 'Comparison method: spectral clustering based on learning similarity matrix
    with sparse structure via ADMM; provides MATLAB routines and .mat datasets with
    true_labs for NMI/ARI computation'
  repo: https://github.com/ishspsy/project
- name: SC3
  role: 'Comparison method: unsupervised clustering of cells from single-cell RNA-Seq
    experiments, installed from BioConductor or GitHub'
  repo: https://github.com/hemberg-lab/SC3
- name: SIMLR
  role: Comparison method for multi-kernel similarity learning clustering
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: MATLAB
  role: Execution environment for SSRE and MPSSC benchmark runs (MPSSC examples used
    MATLAB 2016b)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:15f4a4ac0e48cc5cffb40408660621075059c3f2e73aa4a9d769912282503492
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - MPSSC
  - Python
  - SC3
  - SIMLR
  - SSRE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# multi-method-benchmark-table-reproduction

## Summary

Reproduce the NMI/ARI clustering benchmark tables of the SSRE paper by running SSRE and ten comparison methods (SSR, SC, SIMLR, MPSSC, Corr, SNN-Cliq, NMF, SC3, dropClust, Seurat) on ten pre-annotated scRNA-seq datasets and aggregating results into a 10-dataset × 11-method table.

## When to use

Use when replicating a published single-cell clustering benchmark: multiple clustering methods must be run on the same pre-annotated scRNA-seq matrices with known cluster counts and NMI/ARI assembled into a comparison table.

## When NOT to use

- Datasets lack pre-annotated cell-type labels, so NMI/ARI against ground truth cannot be computed
- Large-scale datasets where runtime is critical — the article notes SSRE is 'relatively time-consuming in large-scale datasets' and defers acceleration to future parallel computing
- The goal is a single method's clustering rather than a multi-method comparison table

## Inputs

- Ten pre-annotated scRNA-seq expression matrices (gene-by-cell) with known cell-type labels: Treutlein, Deng, Ting, Macosko, Yan, Goolam, Song (GSE85908), Engel (GSE74597), Haber (GSE92332), Vento (E-MTAB-6678)
- Known number of cell types per dataset (true cluster count for each method)
- SSRE MATLAB/Python implementation from https://github.com/CSUBioGroup/SSRE
- MATLAB implementations of comparison methods, e.g. MPSSC code from https://github.com/ishspsy/project and SC3 from https://github.com/hemberg-lab/SC3

## Outputs

- Cluster assignments per method per dataset
- NMI and ARI values per method per dataset
- Aggregated 10-dataset × 11-method benchmark table mirroring Tables 2 and 3

## How to apply

Download each of the ten datasets (Treutlein, Deng, Ting, Macosko, Yan, Goolam, Song/GSE85908, Engel/GSE74597, Haber/GSE92332, Vento/E-MTAB-6678) from its GEO/ArrayExpress accession and build a gene-by-cell expression matrix with ground-truth cell-type labels. Run SSRE (MATLAB or Python from CSUBioGroup/SSRE) using its two-step procedure of adaptive gene selection followed by similarity enhancement over three pairwise similarities, then spectral clustering on the learned similarity matrix with the known number of cell types. Run each of the ten comparison methods on identical preprocessed matrices; e.g. for MPSSC (MATLAB, from ishspsy/project) set penalty parameters rho=0.2, lam=lam2=0.0001, eta=1, c=0.1 and call the main function to get the target matrix P, or install SC3 from BioConductor/GitHub. Compute NMI and ARI of each method's cluster assignments against the pre-annotated labels per dataset and aggregate into the 10×11 table mirroring Tables 2 and 3. Validate by matching reproduced values to the published table entries within a small numerical tolerance; SSRE should achieve best or tied-first NMI/ARI on seven datasets and second on three (NMI) and two (ARI).

## Related tools

- **SSRE** (Focal method: adaptive gene selection plus similarity enhancement over three pairwise similarities, then spectral clustering on the learned similarity matrix) — https://github.com/CSUBioGroup/SSRE
- **MPSSC** (Comparison method: spectral clustering based on learning similarity matrix with sparse structure via ADMM; provides MATLAB routines and .mat datasets with true_labs for NMI/ARI computation) — https://github.com/ishspsy/project
- **SC3** (Comparison method: unsupervised clustering of cells from single-cell RNA-Seq experiments, installed from BioConductor or GitHub) — https://github.com/hemberg-lab/SC3
- **SIMLR** (Comparison method for multi-kernel similarity learning clustering) — https://github.com/BatzoglouLabSU/SIMLR
- **MATLAB** (Execution environment for SSRE and MPSSC benchmark runs (MPSSC examples used MATLAB 2016b))

## Examples

```
load('Data_Deng.mat'); rho=0.2; lam=0.0001; lam2=lam; eta=1; c=0.1; [P]=clus_fin_update(rho,lam,lam2,eta,c,in_X,true_labs); [NMI,~,clus_labs,~]=calc2_nmis(max(true_labs),double(P),true_labs); ARI=RandIndex(clus_labs,true_labs)
```

## Evaluation signals

- Reproduced NMI/ARI for SSRE and each comparison method match the corresponding entries of Tables 2 and 3 within a small numerical tolerance
- SSRE achieves best or tied-first NMI and ARI on seven of the ten datasets and ranks second on three (NMI) and two (ARI) datasets
- Each method is run on the same preprocessed gene-by-cell matrices with the correct known number of cell types
- For MPSSC runs, the pipeline returns the triple [NMI, Purity, ARI] from calc2_nmis and RandIndex against true_labs
- Learned SSRE similarity matrix combined with spectral clustering reveals the expected block structure of the scRNA-seq data

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is only proposed as a possible acceleration strategy, not implemented
- Memory-intensive comparison runs may require a computing cluster: the MPSSC large-scale examples were run on 6 CPUs with 800 GB of memory
- Users may choose one or two of the three pairwise similarities for gene selection and similarity enhancement, but all three are used by default — deviating changes results

## Evidence

- [results] SSRE outperformed all comparison methods in most cases, achieving the best or tied-first NMI and ARI on seven datasets: [evidence span withheld — anchor status: not_found]
- [intro] Two-step procedure quote: "SSRE designs a two-step procedure, i.e., 1) adaptive gene selection and 2) similarity enhancement."
- [abstract] Repo availability quote: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [discussion] Time limitation quote: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework"
- [readme] MPSSC example invocation quote: [evidence span withheld — anchor status: not_found]
- [discussion] Default similarities quote: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
