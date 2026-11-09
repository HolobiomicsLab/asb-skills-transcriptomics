---
name: spearman-correlation-cell-similarity
description: Use when mapping leukemia/carcinoma sub-cluster cells onto a reference
  scRNA-seq atlas (e.g., AML1_cluster1 onto normal BMMCs) and a rank-based, outlier-robust
  expression similarity term is needed as one component of a composite statistic.
when_to_use_negative:
- The goal is clustering or trajectory inference of the target cells themselves —
  CCI requires sub-clusters to be pre-computed before similarity is scored
- Only a single signature (e.g., one marker gene set) is available and no expression
  profiles exist — the article notes identifying counterparts based on a single signature
  is an unsolved challenge
- Input is single-cell epigenomic data rather than scRNA-seq expression profiles,
  which the authors defer to future work
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: CCI (counterpart composite index)
  role: Framework that combines the Spearman rank correlation with Euclidean distance,
    PCA-space distance, and gene-set enrichment statistics into a composite-likelihood
    BF probability
  repo: https://github.com/pengfeeei/cci
- name: bash
  role: Runs the released pipeline (run_cci.sh) that computes the CCI scores for a
    sub-cluster
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41421-020-00223-4
    title: Integrated decoding hematopoiesis and leukemogenesis using single-cell
      sequencing and its medical implication
  source_content_hash: sha256:0a4932a621516c498e4806d94c23325789fb4207078786a0af6efc7420b4d582
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pengfeeei/cci
  tools:
  - CCI (counterpart composite index)
  - bash
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# spearman-correlation-cell-similarity

## Summary

A rank-based similarity measurement used inside the CCI (counterpart composite index) framework: Spearman correlation of expression profiles quantifies how similar a leukemia cell (or sub-cluster) is to each reference hematopoietic cell, feeding into the composite-likelihood Bayes-factor score.

## When to use

Use when mapping leukemia/carcinoma sub-cluster cells onto a reference scRNA-seq atlas (e.g., AML1_cluster1 onto normal BMMCs) and a rank-based, outlier-robust expression similarity term is needed as one component of a composite statistic.

## When NOT to use

- The goal is clustering or trajectory inference of the target cells themselves — CCI requires sub-clusters to be pre-computed before similarity is scored
- Only a single signature (e.g., one marker gene set) is available and no expression profiles exist — the article notes identifying counterparts based on a single signature is an unsolved challenge
- Input is single-cell epigenomic data rather than scRNA-seq expression profiles, which the authors defer to future work

## Inputs

- Target sub-cluster expression matrix (cells x genes) from data_obj/, e.g. AML1_cluster1 (100 cells, 4000 genes)
- Reference hematopoietic atlas expression matrix from data_ref/ with gene list, cell types, and per-cell identity files

## Outputs

- Per-pair Spearman rank correlation values between leukemia and reference cell profiles
- Component statistic vector contributing to the composite CCI BF-probability score per cell/sub-cluster pair

## How to apply

Load the target sub-cluster expression matrix (e.g., the toy data in data_obj/ with 100 cells and 4000 genes) together with the reference atlas expression matrix (cells as rows, genes as columns) from data_ref/. For each leukemia-cell vs reference-cell (or sub-cluster vs reference sub-cluster) pair, compute the Spearman rank correlation of the two expression profiles over shared genes; because Spearman operates on ranks it tolerates the 'huge differences' between leukemia and healthy cells that defeat single-signature matching. Combine this correlation term with the other CCI component statistics — Euclidean distance of expression profiles, Euclidean distance in the space of the top 30 principal components, and the gene-set-enrichment AUC-difference term — into the composite CCI likelihood expressed as a Bayes-factor (BF) probability, mirroring the formula in the released cci package source. Rank-based agreement makes it robust to monotone scaling of expression values across sequencing depths.

## Related tools

- **CCI (counterpart composite index)** (Framework that combines the Spearman rank correlation with Euclidean distance, PCA-space distance, and gene-set enrichment statistics into a composite-likelihood BF probability) — https://github.com/pengfeeei/cci
- **bash** (Runs the released pipeline (run_cci.sh) that computes the CCI scores for a sub-cluster)

## Examples

```
bash run_cci.sh AML1_cluster1
```

## Evaluation signals

- Reimplemented Spearman-based CCI BF-probability scores match the released tool's scores in result/ (high rank agreement, near-zero maximum absolute deviation)
- All genes shared between target sub-cluster and reference atlas are used in rank computation; gene lists from data_ref/ align to the expression matrix columns
- Scores are computed per leukemia-cell/sub-cluster vs reference-cell pair, producing one BF probability per pair
- For the toy data (100 cells, 4K genes) computation completes in minutes on one normal CPU

## Limitations

- CCI projects cell clusters rather than single cells 'due to prevalent droplets in scRNA-seq', so target cells must be properly clustered before computing similarities
- Rank correlation alone is insufficient; a single-signature approach fails due to huge differences between leukemia and healthy cells, so the Spearman term must be combined with the other three statistics
- Large reference atlases may need downloading separately (the comprehensive normal BMMC atlas is provided in the repository)

## Evidence

- [methods] CCI combines multiple statistics — expression level, co-expression, and dimension-reduction embedding space — within a composite likelihood statistical framework: [evidence span withheld — anchor status: not_found]
- [discussion] We developed CCI that integrates multiple measurements for searching the healthy counterpart of each leukemia cell subpopulation in reference hematopoietic cells.: "We developed CCI that integrates multiple measurements for searching the healthy counterpart of each leukemia cell subpopulation in reference hematopoietic cells."
- [readme] CCI projects cell clusters to reference instead of single cell due to prevalent droplets in scRNA-seq.: "CCI projects cell clusters to reference instead of single cell due to prevalent droplets in scRNA-seq. Please properly cluster your target cells before running CCI."
- [discussion] it is a big challenge to identify the healthy counterparts of each leukemia cell subpopulation based on single signature due to the huge differences: "it is a big challenge to identify the healthy counterparts of each leukemia cell subpopulation based on single signature due to the huge differences"
- [readme] For the toy data which contains 100 cells and 4K genes, computating will be finished in several minutes by one normal CPU spread.: "For the toy data which contains 100 cells and 4K genes, computating will be finished in several minutes by one normal CPU spread."
