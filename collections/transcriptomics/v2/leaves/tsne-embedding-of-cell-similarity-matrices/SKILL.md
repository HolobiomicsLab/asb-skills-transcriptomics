---
name: tsne-embedding-of-cell-similarity-matrices
description: Use when you have cell-to-cell similarity matrices from scRNA-seq methods
  (e.g. SSRE vs Corr, SIMLR, MPSSC, SSR, native t-SNE) and need 2D embeddings to visually
  compare how well each separates known cell types/stages.
when_to_use_negative:
- The goal is clustering accuracy metrics (NMI, Purity, ARI) rather than visual comparison
  of similarity structure — use spectral clustering on the learned similarities instead
- The dataset is large-scale, where SSRE similarity learning is relatively time-consuming
  and parallel computing would be needed
- Input is raw gene expression without first computing a cell-to-cell similarity matrix
  via one of the six methods
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SSRE
  role: 'Learns the cell-to-cell similarity matrix (default: all three pairwise similarities)
    that is embedded via modified t-SNE for visualization'
  repo: https://github.com/CSUBioGroup/SSRE
- name: SC3
  role: Source of the Goolam and Yan scRNA-seq expression matrices used as input
  repo: https://github.com/hemberg-lab/SC3
- name: MPSSC
  role: Competing similarity-learning method (MATLAB) whose learned similarity matrix
    is embedded for comparison
  repo: https://github.com/ishspsy/project
- name: MATLAB
  role: Language in which the SSRE implementation and competing similarity-embedding
    analyses are executed
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: Alternative language for the SSRE implementation used to compute similarities
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:b4afc6f2b32140d31197e6cf3a1e47928bb1113beafcd5dd9ff81a055ffa9c00
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - MPSSC
  - Python
  - SC3
  - SSRE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# tsne-embedding-of-cell-similarity-matrices

## Summary

Embeds cell-to-cell similarity matrices learned by competing scRNA-seq methods (native t-SNE, Corr, SIMLR, MPSSC, SSR, SSRE) into 2D via a modified t-SNE, so that cell-type/stage separation on the Goolam and Yan datasets can be compared visually. This demonstrates SSRE's extension from clustering to visualization of single-cell data.

## When to use

Use when you have cell-to-cell similarity matrices from scRNA-seq methods (e.g. SSRE vs Corr, SIMLR, MPSSC, SSR, native t-SNE) and need 2D embeddings to visually compare how well each separates known cell types/stages.

## When NOT to use

- The goal is clustering accuracy metrics (NMI, Purity, ARI) rather than visual comparison of similarity structure — use spectral clustering on the learned similarities instead
- The dataset is large-scale, where SSRE similarity learning is relatively time-consuming and parallel computing would be needed
- Input is raw gene expression without first computing a cell-to-cell similarity matrix via one of the six methods

## Inputs

- Goolam scRNA-seq expression matrix (five developmental stages) from hemberg-lab/SC3
- Yan scRNA-seq expression matrix (seven developmental stages) from hemberg-lab/SC3
- Cell-to-cell similarity matrices learned by native t-SNE, Corr, SIMLR, MPSSC, SSR, and SSRE

## Outputs

- 2D t-SNE embedding coordinates (one per cell) for each of the six similarity matrices, per dataset
- Stage-colored panel figure, assembled as figure3_tsne_panels.png

## How to apply

Load the Goolam (five developmental stages) and Yan (seven developmental stages) scRNA-seq expression matrices from the hemberg-lab/SC3 repository. Obtain the cell-to-cell similarity matrices from each of the six methods, using the SSRE implementation from CSUBioGroup/SSRE (MATLAB or Python) with the default combination of all three pairwise similarities (users can choose one or two similarities, but all three are used by default). For each similarity matrix, run the paper's modified t-SNE embedding to produce 2D coordinates for every cell, and arrange stage-colored panels per dataset to match Figure 3's layout. The rationale is that SSRE's sparse subspace representation plus similarity enhancement yields similarities that better reveal the block structure of scRNA-seq data, so its embeddings should separate stages more clearly.

## Related tools

- **SSRE** (Learns the cell-to-cell similarity matrix (default: all three pairwise similarities) that is embedded via modified t-SNE for visualization) — https://github.com/CSUBioGroup/SSRE
- **SC3** (Source of the Goolam and Yan scRNA-seq expression matrices used as input) — https://github.com/hemberg-lab/SC3
- **MPSSC** (Competing similarity-learning method (MATLAB) whose learned similarity matrix is embedded for comparison) — https://github.com/ishspsy/project
- **MATLAB** (Language in which the SSRE implementation and competing similarity-embedding analyses are executed) — https://github.com/CSUBioGroup/SSRE
- **Python** (Alternative language for the SSRE implementation used to compute similarities) — https://github.com/CSUBioGroup/SSRE

## Examples

```
[P] = clus_fin_update(rho, lam, lam2, eta, c, in_X, true_labs);
```

## Evaluation signals

- SSRE panels of the Goolam and Yan embeddings show oocyte and 2-cell stage groups visually separated from other stage groups
- SSRE panels show clearer stage separation than the competing methods (Corr, SIMLR, MPSSC, SSR, native t-SNE) in the Figure 3-style layout
- On the Yan dataset, cells are clustered more accurately according to oocyte, 2-cell, and other cell types than in competing methods' embeddings
- All six methods (native t-SNE, Corr, SIMLR, MPSSC, SSR, SSRE) produce a complete 2D embedding for every cell in both datasets

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is suggested as a possible acceleration strategy but is not implemented
- Visualization is an extension of SSRE rather than its core function — the primary output of the framework is cell-type clustering via similarity learning combined with spectral clustering
- Users can substitute one or two of the three pairwise similarities, which would deviate from the default (all three) configuration used in the reported Figure 3 comparison

## Evidence

- [abstract] Visualization extension: "In addition, SSRE can be extended to visualization of scRNA-seq data and identification of differentially expressed genes."
- [abstract] Implementation availability: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [discussion] Similarity combination default: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
- [discussion] Sparse structure embedding rationale: "SSRE greatly improved the clustering performance by appropriately combining multiple similarity measurements and adopting the embedding of sparse structure."
- [readme] t-SNE implementation reference: [evidence span withheld — anchor status: not_found]
