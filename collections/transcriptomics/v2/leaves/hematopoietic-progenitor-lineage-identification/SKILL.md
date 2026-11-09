---
name: hematopoietic-progenitor-lineage-identification
description: Use when scRNA-seq data of hematopoietic progenitors (FACS-sorted LK,
  LSK, CMP, MEP, CLP/LMPP fractions) has been clustered by BUSseq and unknown clusters
  must be annotated as specific progenitor stages using marker genes and Haemopedia
  reference profiles.
when_to_use_negative:
- The dataset is not hematopoietic scRNA-seq data or no Haemopedia RNA-seq reference
  profiles are available for cluster annotation
- The BUSseq model has not been fitted yet (no cluster assignments or cell-type mean
  expression profiles exist to annotate)
- A completely confounded batch design is used (e.g., batch 1 has cell types 1-2,
  batch 2 has cell types 3-4), for which no batch-correction/clustering method is
  applicable
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: BUSseq
  role: Bayesian hierarchical model that simultaneously corrects batch effects and
    clusters hematopoietic cells into six types; provides celltypes(), celltype_mean_expression(),
    corrected_read_counts() and heatmap_data_BUSseq() accessors for marker testing
    and Haemopedia mapping
  repo: https://github.com/songfd2018/BUSseq
- name: gplots
  role: R visualization support used when rendering the marker-gene by cell-type heatmap
    image
- name: R
  role: Language in which the marker summarization, heatmap rendering, and Pearson
    correlation against Haemopedia profiles are implemented (tested on R 3.6.0)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:982e87675662191057cefd8c74e674869dbb16dcf335cb748f73fae4d65e1232
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq
  tools:
  - BUSseq
  - R
  - gplots
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# hematopoietic-progenitor-lineage-identification

## Summary

Recover the six-cluster hematopoietic cell-type structure from BUSseq clustering of mouse hematopoietic scRNA-seq data, including a newly identified intermediate CMP-MEP subpopulation (Cluster 4) and a lymphoid progenitor cluster (Cluster 3) assigned by Haemopedia RNA-seq correlation.

## When to use

Use when scRNA-seq data of hematopoietic progenitors (FACS-sorted LK, LSK, CMP, MEP, CLP/LMPP fractions) has been clustered by BUSseq and unknown clusters must be annotated as specific progenitor stages using marker genes and Haemopedia reference profiles.

## When NOT to use

- The dataset is not hematopoietic scRNA-seq data or no Haemopedia RNA-seq reference profiles are available for cluster annotation
- The BUSseq model has not been fitted yet (no cluster assignments or cell-type mean expression profiles exist to annotate)
- A completely confounded batch design is used (e.g., batch 1 has cell types 1-2, batch 2 has cell types 3-4), for which no batch-correction/clustering method is applicable

## Inputs

- Fitted BUSseq object (BUSseqfits_example) from mouse hematopoietic scRNA-seq read count data with K=6 cell types
- Per-cell cluster assignments from BUSseq MCMC inference
- Corrected read counts from corrected_read_counts(sce_BUSseqfit)
- Marker gene set (Apoe, Gata2, Ctse, Ebf1, Vpreb1, Vpreb3, Igll1)
- Haemopedia RNA-seq cell-type reference expression profiles

## Outputs

- cluster3_haemopedia_correlations.csv (Haemopedia cell_type column plus Pearson correlation column)
- Marker-gene by cell-type heatmap image (via heatmap_data_BUSseq image_dir/project_name)
- Summary of the six-cell-type structure recovered by BUSseq, including the intermediate CMP-MEP stage

## How to apply

Load the fitted BUSseq object (BUSseqfits_example) and extract per-cell cluster assignments and cell-type mean expression profiles. Test expected marker-cluster assignments on corrected_read_counts(): Apoe and Gata2 should be high in Cluster 4 (markers of CMP/MEP-adjacent identity), Ctse only in the MEP cluster, and Ebf1, Vpreb1, Vpreb3, Igll1 specific to Cluster 3. Render a marker-gene by cell-type heatmap with heatmap_data_BUSseq(), passing gene_set = marker genes and writing via image_dir/project_name. Annotate Cluster 3 by computing Pearson correlations between its cell-type mean expression profile and each Haemopedia RNA-seq cell-type profile, and accept the annotation only if the top-ranked match is a common lymphoid progenitor population. Write cluster3_haemopedia_correlations.csv and summarize the six-cell-type structure including the intermediate CMP-MEP stage.

## Related tools

- **BUSseq** (Bayesian hierarchical model that simultaneously corrects batch effects and clusters hematopoietic cells into six types; provides celltypes(), celltype_mean_expression(), corrected_read_counts() and heatmap_data_BUSseq() accessors for marker testing and Haemopedia mapping) — https://github.com/songfd2018/BUSseq
- **gplots** (R visualization support used when rendering the marker-gene by cell-type heatmap image)
- **R** (Language in which the marker summarization, heatmap rendering, and Pearson correlation against Haemopedia profiles are implemented (tested on R 3.6.0))

## Examples

```
R> corr <- cor(BUSseqfits_example@celltype_mean_expression[3,], t(haemopedia_profiles)); write.csv(data.frame(cell_type=names(corr), correlation=corr), 'cluster3_haemopedia_correlations.csv', row.names=FALSE)
```

## Evaluation signals

- Both output files exist with correct tabular/image format
- The correlation table contains a Haemopedia cell_type column and a correlation column
- The top-ranked Haemopedia match for Cluster 3 is a common lymphoid progenitor population
- Cluster 4 shows high expression of Apoe and Gata2 (absent in CMP and MEP) while Ctse is expressed only in the MEP cluster
- Lymphoid markers Ebf1, Vpreb1, Vpreb3, Igll1 are specific to Cluster 3 in the corrected read counts

## Limitations

- Cell-type annotation via marker expression is prone to clustering errors, which can lead to false positives and false negatives since multi-stage pipelines ignore uncertainties in previous stages
- The optimal number of cell types K must be selected by BIC comparison (enumerated K = 3 to 6 in the vignettes); an incorrect K breaks the six-cluster structure
- MCMC inference is computationally heavy, and the C++ source codes support Linux operating systems only (tested on Ubuntu 18.04 and CentOS 7)

## Evidence

- [results] In the hematopoietic analysis, BUSseq's Cluster 4 shows high expression of marker genes Apoe and Gata2 (absent in CMP and MEP), and Ctse is expressed only in MEP, supporting Cluster 4 as an intermediate CMP-MEP cell stage.: [evidence span withheld — anchor status: not_found]
- [methods] To select the optimal K, we need to compare the BIC values for different Ks. In the vignettes, we enumerate K from 3 to 6.: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [methods] heatmap_data_BUSseq accessor signature from the article methods section.: "heatmap_data_BUSseq(sce_BUSseqfit, data_type, gene_set, project_name, image_dir, color_key_seq, image_width, image_height)"
- [readme] After correcting the batch effects with BUSseq, the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch.: "After correcting the batch effects with BUSseq, the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch."
- [discussion] As the clustering results may be prone to errors in practice, this can lead to false positives and false negatives.: "As the clustering results may be prone to errors in practice, this can lead to false positives and false negatives."
