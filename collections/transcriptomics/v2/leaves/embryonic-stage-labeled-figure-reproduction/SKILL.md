---
name: embryonic-stage-labeled-figure-reproduction
description: Use when the goal is to visually validate learned cell-to-cell similarity
  matrices by re-embedding them with a modified t-SNE and comparing stage-colored
  panels across competing methods on embryo-development datasets (e.g., Goolam, Yan).
when_to_use_negative:
- The goal is quantitative clustering benchmarking (NMI/Purity/ARI tables) rather
  than the qualitative visual comparison of similarity-based embeddings
- The input is not a cell-to-cell similarity matrix or scRNA-seq expression matrix
  from an embryo-development dataset with stage labels (e.g., a feature table without
  cell annotations)
- Working with large-scale scRNA-seq data, since SSRE is relatively time-consuming
  on large-scale datasets
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: SSRE
  role: Produces the SSRE similarity matrices (default combination of all three pairwise
    similarities) and the modified t-SNE embedding of scRNA-seq data used for the
    SSRE panels
  repo: https://github.com/CSUBioGroup/SSRE
- name: MPSSC
  role: Provides one of the competing similarity matrices (spectral clustering based
    on learning similarity matrix) via clus_fin_update for the comparison panels
  repo: https://github.com/ishspsy/project
- name: SC3
  role: Source of the Goolam and Yan scRNA-seq expression matrices and stage labels
    used as inputs
  repo: https://github.com/hemberg-lab/SC3
- name: SIMLR
  role: One of the competing similarity-learning methods whose learned similarity
    is embedded and plotted for comparison
  repo: https://github.com/ishspsy/project
- name: MATLAB
  role: Implementation language for SSRE and MPSSC similarity learning and figure
    generation
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: Alternative implementation language for SSRE available in the CSUBioGroup/SSRE
    repository
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:3e38984c050f23a1749b51aa75f3593f4408eba60c8ca919e09b742b7168b56f
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# embryonic-stage-labeled-figure-reproduction

## Summary

Reproduces the stage-colored modified t-SNE comparison figure (Figure 3 of the SSRE paper) by embedding cell-to-cell similarity matrices from six methods (native t-SNE, Corr, SIMLR, MPSSC, SSR, SSRE) for the Goolam and Yan scRNA-seq datasets and checking that SSRE's embeddings visually separate oocyte and 2-cell groups.

## When to use

Use when the goal is to visually validate learned cell-to-cell similarity matrices by re-embedding them with a modified t-SNE and comparing stage-colored panels across competing methods on embryo-development datasets (e.g., Goolam, Yan).

## When NOT to use

- The goal is quantitative clustering benchmarking (NMI/Purity/ARI tables) rather than the qualitative visual comparison of similarity-based embeddings
- The input is not a cell-to-cell similarity matrix or scRNA-seq expression matrix from an embryo-development dataset with stage labels (e.g., a feature table without cell annotations)
- Working with large-scale scRNA-seq data, since SSRE is relatively time-consuming on large-scale datasets

## Inputs

- Goolam scRNA-seq expression matrix (five developmental stages) from hemberg-lab/SC3 data
- Yan scRNA-seq expression matrix (seven developmental stages) from hemberg-lab/SC3 data
- Cell-to-cell similarity matrices from native t-SNE, Corr, SIMLR, MPSSC, SSR, and SSRE (SSRE via CSUBioGroup/SSRE, default combination of all three pairwise similarities)
- Ground-truth developmental stage labels for stage coloring

## Outputs

- 2D t-SNE embedding coordinates for every cell under each of the six similarity matrices
- Stage-colored panel grid matching the layout of Figure 3 for Goolam and Yan
- Single assembled figure file: figure3_tsne_panels.png

## How to apply

Load the Goolam (five developmental stages) and Yan (seven developmental stages) expression matrices from the hemberg-lab/SC3 data repository, and obtain or compute the cell-to-cell similarity matrices from native t-SNE, Corr, SIMLR, MPSSC, SSR, and SSRE (use the CSUBioGroup/SSRE implementation with the default combination of all three pairwise similarities). For each similarity matrix, run the paper's modified t-SNE embedding of scRNA-seq data to obtain 2D coordinates per cell. Plot stage-colored panels arranged per dataset in the row/column layout of Figure 3, and inspect the SSRE panels: oocyte and 2-cell cells should be visually separated from the other stage groups in both datasets. Assemble all panels into a single figure file (figure3_tsne_panels.png). MPSSC similarities can be recomputed in MATLAB via clus_fin_update (rho=0.2, lam=0.0001, lam2=lam, eta=1, c=0.1), and clustering/labels can be checked with NMI, Purity, and ARI via calc2_nmis, purity, and RandIndex.

## Related tools

- **SSRE** (Produces the SSRE similarity matrices (default combination of all three pairwise similarities) and the modified t-SNE embedding of scRNA-seq data used for the SSRE panels) — https://github.com/CSUBioGroup/SSRE
- **MPSSC** (Provides one of the competing similarity matrices (spectral clustering based on learning similarity matrix) via clus_fin_update for the comparison panels) — https://github.com/ishspsy/project
- **SC3** (Source of the Goolam and Yan scRNA-seq expression matrices and stage labels used as inputs) — https://github.com/hemberg-lab/SC3
- **SIMLR** (One of the competing similarity-learning methods whose learned similarity is embedded and plotted for comparison) — https://github.com/ishspsy/project
- **MATLAB** (Implementation language for SSRE and MPSSC similarity learning and figure generation) — https://github.com/CSUBioGroup/SSRE
- **Python** (Alternative implementation language for SSRE available in the CSUBioGroup/SSRE repository) — https://github.com/CSUBioGroup/SSRE

## Examples

```
addpath(genpath(pwd)); load('Data_Yan.mat'); [P] = clus_fin_update(0.2, 0.0001, 0.0001, 1, 0.1, in_X, true_labs);
```

## Evaluation signals

- SSRE panels show oocyte and 2-cell stage groups visually separated from other stage groups in both the Goolam and Yan embeddings
- Panels are arranged per dataset (Goolam rows/columns and Yan rows/columns) matching the layout of Figure 3
- All six methods (native t-SNE, Corr, SIMLR, MPSSC, SSR, SSRE) are represented as separate stage-colored panels
- The final output is a single assembled figure file named figure3_tsne_panels.png
- For MPSSC-recomputed similarities, NMI, Purity, and ARI computed via calc2_nmis, purity, and RandIndex return plausible values against true_labs

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is suggested as a possible acceleration strategy but is not implemented
- MPSSC's large-scale runs required a computing cluster (6 CPUs, 800 GB of memory), so reproducing all competing similarities on big data may be resource-intensive
- The validation criterion is qualitative (visual separation of oocyte/2-cell groups), so borderline or ambiguous separations are not scored numerically

## Evidence

- [abstract] In addition, SSRE can be extended to visualization of scRNA-seq data and identification of differentially expressed genes.: "In addition, SSRE can be extended to visualization of scRNA-seq data and identification of differentially expressed genes"
- [abstract] SSRE implementation availability: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [discussion] Users can choose one or two of the three pairwise similarities for gene selection and similarity enhancement; all three are used by default: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
- [readme] MPSSC example invocation with penalty parameters and data format: [evidence span withheld — anchor status: not_found]
- [readme] MPSSC compute environment note: [evidence span withheld — anchor status: external_doc]
