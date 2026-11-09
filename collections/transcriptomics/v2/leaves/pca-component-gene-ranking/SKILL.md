---
name: pca-component-gene-ranking
description: Use when benchmarking a periodicity- or trajectory-based gene ranking
  against a linear PCA baseline on the same scRNA-seq matrix, e.g., to test whether
  circular pseudo-time captures cell-cycle biology better than principal components.
when_to_use_negative:
- 'Input data is not variance-normalized: PCA loadings on raw or uncentered counts
  are dominated by highly expressed genes, invalidating the ranking as a fair baseline
  (the article specifies scaled, centered log2(x+1) data).'
- 'The biology of interest is not periodic/circular (e.g., simple linear differentiation
  trajectories): linear PCA may be an adequate representation and the comparison against
  a circular manifold is not informative.'
- The goal is cell-cycle phase scoring from marker genes (e.g., Cyclone-style phase
  prediction) rather than ab-initio gene ranking — Cyclum's README instead recommends
  refine_labels for integrating external phase labels.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Cyclum
  role: Provides the ab-initio circular-pseudo-time ranking and the GSEA data-transfer
    helpers (mat2txt, vec2cls, mat2cls) used to run the PCA-baseline comparison
  repo: github.com/KChen-lab/cyclum
- name: Seurat
  role: Cited in the article as an example of linear-representation-based cell-cycle
    regression that PCA ranking serves as a baseline against
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:6da3ca972fd0bf2aba62099f07b95a5694d9e871602ab97277bca503257f49b7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cyclum
  - Seurat
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pca-component-gene-ranking

## Summary

Rank genes by their loadings on principal components (PC1, PC2) computed on preprocessed scRNA-seq data, then use these ranked lists as linear-manifold baselines against which ab-initio circular rankings (e.g., Cyclum) are compared via GSEA enrichment of the GO:0007049 cell-cycle gene set.

## When to use

Use when benchmarking a periodicity- or trajectory-based gene ranking against a linear PCA baseline on the same scRNA-seq matrix, e.g., to test whether circular pseudo-time captures cell-cycle biology better than principal components.

## When NOT to use

- Input data is not variance-normalized: PCA loadings on raw or uncentered counts are dominated by highly expressed genes, invalidating the ranking as a fair baseline (the article specifies scaled, centered log2(x+1) data).
- The biology of interest is not periodic/circular (e.g., simple linear differentiation trajectories): linear PCA may be an adequate representation and the comparison against a circular manifold is not informative.
- The goal is cell-cycle phase scoring from marker genes (e.g., Cyclone-style phase prediction) rather than ab-initio gene ranking — Cyclum's README instead recommends refine_labels for integrating external phase labels.

## Inputs

- Preprocessed scRNA-seq gene expression matrix (log2(x+1)-transformed, scaled variance = 1, centered mean = 1; mESC dataset in the reference study)
- GSEA-compatible .txt expression matrix (produced by mat2txt)
- GSEA .cls phenotype/ranking files (PC scores as continuous phenotypes, produced by mat2cls)
- GO:0007049 cell-cycle biological-process gene set

## Outputs

- Gene ranked list ordered by PC1 loadings
- Gene ranked list ordered by PC1-PC2 loadings
- GSEA normalized enrichment scores (NES) for the cell-cycle gene set per ranked list (expected ~1.07 for PC1 and ~1.06 for PC1-2 on mESC)
- gsea_nes_results.csv collecting NES values for Cyclum, PC1, and PC1-2 rankings

## How to apply

Start from the same preprocessed expression matrix used for the primary method — for the mESC dataset this is log2(x+1)-transformed data scaled to variance = 1 and centered at mean = 1. Compute PCA on this matrix and rank genes separately by component 1 and by components 1-2 (loadings magnitude defines gene order). Run GSEA independently on each ranked list (PC1, PC1-2) against the GO:0007049 cell-cycle biological-process gene set and collect normalized enrichment scores (NES). Compare these NES values against the NES from the ab-initio (Cyclum circular-pseudo-time-based) ranking; in the published mESC comparison the PCA baselines yielded NES of 1.07 (PC1) and 1.06 (PC1-2), clearly below Cyclum's 1.57. The GSEA input side is supported by Cyclum's helper utilities: write the expression matrix to a GSEA-compatible .txt file and the pseudotime/PC phenotypes to a .cls file (vec2cls/mat2cls in R).

## Related tools

- **Cyclum** (Provides the ab-initio circular-pseudo-time ranking and the GSEA data-transfer helpers (mat2txt, vec2cls, mat2cls) used to run the PCA-baseline comparison) — github.com/KChen-lab/cyclum
- **Seurat** (Cited in the article as an example of linear-representation-based cell-cycle regression that PCA ranking serves as a baseline against)

## Examples

```
Rscript -e "source('cyclum/R/gsea_io.R'); mat2txt(expr, 'mESC_gsea.txt'); mat2cls(pcs, 'pcs.cls')"  # then run GSEA with GO:0007049 on each PC-ranked list
```

## Evaluation signals

- NES for the PC1-ranked list against the GO:0007049 cell-cycle gene set reproduces ~1.07 on the mESC dataset
- NES for the PC1-2-ranked list reproduces ~1.06 on mESC
- Both PCA NES values are strictly lower than the Cyclum-based NES of 1.57, confirming Cyclum infers cell-cycle genes better than principal components
- Preprocessing invariants hold before PCA: variance = 1 and mean = 1 on the log2(x+1)-transformed matrix, preserving the circular pattern
- GSEA input files pass format checks: expression matrix written as GSEA-compatible .txt and PC phenotypes as a continuous .cls file

## Limitations

- PCA provides only linear representations, which the article argues cannot accurately represent nonlinear circular periodicity such as the cell cycle — hence PCA baselines are expected to underperform circular-manifold rankings.
- The reference NES values (1.07, 1.06) are specific to the mESC dataset with the specified log2(x+1)/scale/center preprocessing; other datasets or preprocessing will yield different scores.
- Cyclum was tested on specific dependency versions (e.g., tensorflow 1.x, keras 2.2.4, python 3.7.4); newer versions may require the pinned versions listed in the README.

## Evidence

- [results] GSEA of ab-initio discovered genes yielded a normalized enrichment score (NES) for cell-cycle genes of 1.57 for Cyclum, compared favorably with NES scores of 1.07 and 1.06 for PC1 and PC2: [evidence span withheld — anchor status: not_found]
- [methods] scaled (variance = 1), centered (mean = 1) log-transformed data work well: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [readme] mat2txt, which writes a expression matrix to a GSEA compatible .txt file: [evidence span withheld — anchor status: external_doc]
- [intro] Existing trajectory/pseudo-time inference methods are not optimal for representing such nonlinear periodicity: "Existing trajectory/pseudo-time inference methods are not optimal for representing such nonlinear periodicity8."
