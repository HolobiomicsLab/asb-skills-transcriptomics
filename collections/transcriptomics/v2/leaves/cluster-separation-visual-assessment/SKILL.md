---
name: cluster-separation-visual-assessment
description: Use when you must visually compare whether one similarity-learning method
  (e.g., SSRE) separates known cell types/stages in 2D more clearly than competing
  methods, using the same embedding procedure across all methods.
when_to_use_negative:
- You only have a raw gene expression matrix and no learned cell-to-cell similarity
  — the embedding compares learned similarities, so a similarity step (e.g., SSRE)
  must precede this skill
- You need quantitative clustering metrics (NMI, Purity, ARI) rather than qualitative
  visual separation assessment — use the clustering performance workflow instead
- Input is a large-scale scRNA-seq dataset where SSRE's known runtime limitation makes
  similarity learning impractical
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SSRE
  role: Source of the SSRE similarity matrix (default combination of all three pairwise
    similarities) and the modified t-SNE visualization being assessed; MATLAB and
    Python implementations
  repo: https://github.com/CSUBioGroup/SSRE
- name: MPSSC
  role: Competing similarity-learning method whose learned affinity matrices are embedded
    for comparison; MATLAB implementation with example scRNA-seq data and t-SNE code
    bundled
  repo: https://github.com/ishspsy/project
- name: SC3
  role: Source of the Goolam and Yan scRNA-seq datasets used as inputs
  repo: https://github.com/hemberg-lab/SC3
- name: MATLAB
  role: Implementation language for SSRE and MPSSC similarity learning and modified
    t-SNE embedding
- name: Python
  role: Alternative implementation language for SSRE and for plotting the stage-colored
    embedding panels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:bbd4d19262f1ada79d1250dbd3cd6700e58955296a1f3f64aa55f762123a9c74
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

# Cluster separation visual assessment via modified t-SNE on learned cell-to-cell similarities

## Summary

A visualization skill from the SSRE paper (Genomics, Proteomics & Bioinformatics, 2021) that embeds cell-to-cell similarity matrices learned by competing scRNA-seq clustering methods (native t-SNE, Corr, SIMLR, MPSSC, SSR, SSRE) with a modified t-SNE, so that cluster separation of developmental-stage cell types can be compared qualitatively across methods on the Goolam and Yan datasets.

## When to use

Use when you must visually compare whether one similarity-learning method (e.g., SSRE) separates known cell types/stages in 2D more clearly than competing methods, using the same embedding procedure across all methods.

## When NOT to use

- You only have a raw gene expression matrix and no learned cell-to-cell similarity — the embedding compares learned similarities, so a similarity step (e.g., SSRE) must precede this skill
- You need quantitative clustering metrics (NMI, Purity, ARI) rather than qualitative visual separation assessment — use the clustering performance workflow instead
- Input is a large-scale scRNA-seq dataset where SSRE's known runtime limitation makes similarity learning impractical

## Inputs

- Goolam scRNA-seq expression matrix (five developmental stages) from hemberg-lab/SC3 data repository
- Yan scRNA-seq expression matrix (seven developmental stages) from hemberg-lab/SC3 data repository
- Cell-to-cell similarity matrices from six methods: native t-SNE, Corr, SIMLR, MPSSC, SSR, and SSRE (default: all three pairwise similarities combined)
- Ground-truth developmental stage labels for cell coloring

## Outputs

- 2D t-SNE coordinates for every cell, one embedding per similarity matrix per dataset
- Stage-colored 2D embedding panels arranged per dataset matching Figure 3 layout
- Assembled figure file figure3_tsne_panels.png

## How to apply

Load the Goolam and Yan scRNA-seq expression matrices (five and seven developmental stages, respectively) from the hemberg-lab/SC3 data repository, then compute or load the cell-to-cell similarity matrices from native t-SNE, Corr, SIMLR, MPSSC, SSR, and SSRE (SSRE from CSUBioGroup/SSRE, using the default combination of all three pairwise similarities). For each similarity matrix, run the paper's modified t-SNE embedding of scRNA-seq data to obtain 2D coordinates for every cell. Plot the embeddings as stage-colored panels arranged per dataset to match the layout of Figure 3. The rationale is that SSRE's sparse subspace representation plus similarity enhancement yields similarities whose block structure (reliable when combined with spectral clustering) should translate into visually separated stage groups. Acceptance requires the SSRE panels to show oocyte and 2-cell groups visually separated from other stages in both datasets, consistent with the paper's reported result.

## Related tools

- **SSRE** (Source of the SSRE similarity matrix (default combination of all three pairwise similarities) and the modified t-SNE visualization being assessed; MATLAB and Python implementations) — https://github.com/CSUBioGroup/SSRE
- **MPSSC** (Competing similarity-learning method whose learned affinity matrices are embedded for comparison; MATLAB implementation with example scRNA-seq data and t-SNE code bundled) — https://github.com/ishspsy/project
- **SC3** (Source of the Goolam and Yan scRNA-seq datasets used as inputs) — https://github.com/hemberg-lab/SC3
- **MATLAB** (Implementation language for SSRE and MPSSC similarity learning and modified t-SNE embedding)
- **Python** (Alternative implementation language for SSRE and for plotting the stage-colored embedding panels)

## Examples

```
P_ssre = run_ssre_similarity(in_X, 'sim', 'all'); Y = tsne_modified(P_ssre, 2); scatter(Y(:,1), Y(:,2), 20, stage_labs); % modified t-SNE embedding of the SSRE similarity matrix, colored by developmental stage
```

## Evaluation signals

- SSRE panels for both Goolam and Yan show oocyte and 2-cell stage groups visually separated from other stage groups
- Every similarity matrix (six methods × two datasets) yields a 2D embedding covering every cell exactly once
- Panel layout and stage coloring match the arrangement of Figure 3 in the SSRE paper
- All six method panels are generated under the same modified t-SNE procedure so the visual comparison is like-for-like
- Final artifact figure3_tsne_panels.png is assembled with all panels present

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is proposed but not implemented in the paper
- Visual separation is a qualitative acceptance criterion and may be judged subjectively by the inspector
- Only oocyte/2-cell separation is used as the validation signal; separation quality of other stage groups is not guaranteed by the paper's reported result
- Users may choose one or two of the three pairwise similarities, but the assessed configuration uses all three by default; deviating changes the SSRE similarity being visualized

## Evidence

- [abstract] SSRE visualization extension: "In addition, SSRE can be extended to visualization of scRNA-seq data and identification of differentially expressed genes."
- [discussion] Default similarity combination: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
- [readme] Dataset source tool: [evidence span withheld — anchor status: not_found]
- [intro] Similarity-based block structure rationale: "The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably."
