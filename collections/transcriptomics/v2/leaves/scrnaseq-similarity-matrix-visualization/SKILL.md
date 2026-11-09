---
name: scrnaseq-similarity-matrix-visualization
description: Use when you need to visually compare cell-type or developmental-stage
  separation produced by different learned cell-to-cell similarity matrices on scRNA-seq
  data, e.g. benchmarking SSRE against Corr, SIMLR, MPSSC, SSR on the Goolam or Yan
  datasets.
when_to_use_negative:
- The goal is quantitative clustering accuracy (NMI/ARI/Purity) rather than visual
  separation comparison — use the SSRE clustering evaluation instead of the visualization
  extension
- Data are not scRNA-seq similarity matrices but already-embedded 2D coordinates or
  a feature table requiring only standard t-SNE
- Datasets are large-scale, where SSRE is known to be relatively time-consuming and
  parallel computing would be required
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SSRE
  role: 'Provides the learned cell-to-cell similarity matrix (default: all three pairwise
    similarities) that is embedded with the modified t-SNE and compared against competing
    methods'
  repo: https://github.com/CSUBioGroup/SSRE
- name: MPSSC
  role: Competing spectral clustering method whose learned similarity matrix is one
    of the embeddings visualized for comparison
  repo: https://github.com/ishspsy/project
- name: SC3
  role: Source of the Goolam and Yan scRNA-seq datasets used as visualization inputs
  repo: https://github.com/hemberg-lab/SC3
- name: MATLAB
  role: One of the two implementation languages for running SSRE and generating similarity-based
    embeddings
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: One of the two implementation languages for running SSRE and generating similarity-based
    embeddings
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:c189900124d95e62cbbc4f342e69ae01513b7c47b7f67bf07334e104343cb8c8
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# scrnaseq-similarity-matrix-visualization

## Summary

Reproduce the SSRE paper's Figure 3-style visualization by embedding cell-to-cell similarity matrices learned by competing methods (native t-SNE, Corr, SIMLR, MPSSC, SSR, SSRE) into 2D with a modified t-SNE for scRNA-seq data, then comparing how clearly developmental stage/cell-type groups separate across methods.

## When to use

Use when you need to visually compare cell-type or developmental-stage separation produced by different learned cell-to-cell similarity matrices on scRNA-seq data, e.g. benchmarking SSRE against Corr, SIMLR, MPSSC, SSR on the Goolam or Yan datasets.

## When NOT to use

- The goal is quantitative clustering accuracy (NMI/ARI/Purity) rather than visual separation comparison — use the SSRE clustering evaluation instead of the visualization extension
- Data are not scRNA-seq similarity matrices but already-embedded 2D coordinates or a feature table requiring only standard t-SNE
- Datasets are large-scale, where SSRE is known to be relatively time-consuming and parallel computing would be required

## Inputs

- scRNA-seq cell-to-cell similarity matrices learned by native t-SNE, Corr, SIMLR, MPSSC, SSR, and SSRE
- Goolam scRNA-seq expression matrix with cells from five developmental stages
- Yan scRNA-seq expression matrix with cells from seven developmental stages
- Developmental stage labels for each cell (e.g. oocyte, 2-cell) used to color embeddings

## Outputs

- 2D t-SNE coordinates for every cell under each similarity matrix
- Stage-colored 2D embedding panels arranged per dataset matching Figure 3 layout
- Assembled figure file figure3_tsne_panels.png

## How to apply

Load the Goolam (five developmental stages) and Yan (seven developmental stages) scRNA-seq expression matrices from the hemberg-lab/SC3 data repository. Obtain cell-to-cell similarity matrices from each of the six methods, using the SSRE implementation from CSUBioGroup/SSRE with the default combination of all three pairwise similarities. For each similarity matrix, run the paper's modified t-SNE embedding of scRNA-seq data to obtain 2D coordinates per cell, then plot stage-colored panels arranged per dataset to match the Figure 3 layout. Accept the output when the SSRE panels show oocyte and 2-cell stage groups visually separated from other stages on both the Goolam and Yan datasets, consistent with the reported finding that SSRE clustered Yan cells more accurately into oocyte, 2-cell, and other cell types than competing methods.

## Related tools

- **SSRE** (Provides the learned cell-to-cell similarity matrix (default: all three pairwise similarities) that is embedded with the modified t-SNE and compared against competing methods) — https://github.com/CSUBioGroup/SSRE
- **MPSSC** (Competing spectral clustering method whose learned similarity matrix is one of the embeddings visualized for comparison) — https://github.com/ishspsy/project
- **SC3** (Source of the Goolam and Yan scRNA-seq datasets used as visualization inputs) — https://github.com/hemberg-lab/SC3
- **MATLAB** (One of the two implementation languages for running SSRE and generating similarity-based embeddings) — https://github.com/CSUBioGroup/SSRE
- **Python** (One of the two implementation languages for running SSRE and generating similarity-based embeddings) — https://github.com/CSUBioGroup/SSRE

## Examples

```
load('Data_Yan.mat'); S = SSRE(in_X); Y = tsne_p(S, 2); scatter(Y(:,1), Y(:,2), 20, stage_labels); saveas(gcf, 'yan_ssre_panel.png')
```

## Evaluation signals

- SSRE panels on the Yan dataset show oocyte and 2-cell clusters visually separated from other stage groups more clearly than Corr, SIMLR, MPSSC, SSR, and native t-SNE panels
- SSRE panels on the Goolam dataset also show oocyte/2-cell stage groups visually separated from other stages
- Each method yields a complete 2D embedding for every cell, and panels are stage-colored and arranged per dataset to match the Figure 3 layout
- The assembled output figure3_tsne_panels.png contains one panel per method per dataset (six methods x two datasets)
- SSRE similarity is generated with the default combination of all three pairwise similarities, matching the paper's default setting

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is suggested but not implemented as an acceleration strategy
- The visualization result is a qualitative visual separation check, not a quantitative clustering metric
- Native t-SNE on raw expression is a weak baseline since traditional methods cannot work well on scRNA-seq data due to high noise rate and high dropouts

## Evidence

- [results] On the Yan dataset, SSRE clustered cells more accurately according to oocyte, 2-cell, and other cell types than the competing methods (Corr, SIMLR, MPSSC, SSR, native t-SNE).: [evidence span withheld — anchor status: not_found]
- [abstract] SSRE can be extended to visualization of scRNA-seq data and identification of differentially expressed genes.: "In addition, SSRE can be extended to visualization of scRNA-seq data and identification of differentially expressed genes."
- [supplementary] The matlab and python implementations of SSRE are available at the CSUBioGroup/SSRE repository.: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [discussion] All three pairwise similarities are used by default in SSRE.: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
- [discussion] One limitation of SSRE is that it is relatively time-consuming on large-scale datasets.: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework"
