---
name: single-cell-rnaseq-cluster-annotation
description: Use when an unsupervised clustering of scRNA-seq data (e.g., BUSseq with
  K chosen by BIC) has yielded clusters without labels and you must assign cell-type
  identities using marker-gene expression and reference datasets such as Haemopedia.
when_to_use_negative:
- Cell-type labels are already known a priori (e.g., FACS-sorted populations fully
  explain all clusters), so de novo annotation is unnecessary
- The number of cell types K is still unknown — run BUSseq MCMC across K and select
  the minimum BIC before annotating clusters
- The data are completely confounded by batch (e.g., batch 1 measures only some cell
  types that batch 2 does not), where no batch-correction/annotation method is applicable
  per the article
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: BUSseq
  role: Bayesian hierarchical model that fits clusters, provides celltype/celltype_mean_expression
    accessors, corrected read counts for annotation, and marker heatmap rendering
  repo: https://github.com/songfd2018/BUSseq
- name: gplots
  role: R visualization support for the marker-gene by cell-type heatmap
- name: R
  role: Environment for running the BUSseq R package accessors, computing Pearson
    correlations against Haemopedia profiles, and writing the correlation CSV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:ef0b1fb05e075a7e0cb6f8d7f384ddbc43e7cc43c31d3f9fe14d4743a5bb38ce
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
n_claims: 6
---

# single-cell-rnaseq-cluster-annotation

## Summary

Annotate BUSseq-derived cell clusters with biological identities using lineage marker genes and reference map correlations, as demonstrated in the mouse hematopoietic study where six clusters were resolved into FACS-known and newly identified subpopulations (e.g., an intermediate CMP-MEP stage and common lymphoid progenitors).

## When to use

Use when an unsupervised clustering of scRNA-seq data (e.g., BUSseq with K chosen by BIC) has yielded clusters without labels and you must assign cell-type identities using marker-gene expression and reference datasets such as Haemopedia.

## When NOT to use

- Cell-type labels are already known a priori (e.g., FACS-sorted populations fully explain all clusters), so de novo annotation is unnecessary
- The number of cell types K is still unknown — run BUSseq MCMC across K and select the minimum BIC before annotating clusters
- The data are completely confounded by batch (e.g., batch 1 measures only some cell types that batch 2 does not), where no batch-correction/annotation method is applicable per the article

## Inputs

- Fitted BUSseq object (e.g., BUSseqfits_example) containing per-cell cluster assignments and cell-type mean expression profiles
- Corrected read counts from corrected_read_counts() on the fitted object
- Marker gene list (e.g., Apoe, Gata2, Ctse, Ebf1, Vpreb1, Vpreb3, Igll1)
- Reference cell-type expression atlas (Haemopedia RNA-seq cell-type profiles)

## Outputs

- cluster3_haemopedia_correlations.csv with a Haemopedia cell_type column and a Pearson correlation column
- Marker-gene by cell-type heatmap image (via heatmap_data_BUSseq with image_dir/project_name)
- Summarized cell-type annotation of each cluster, including the intermediate CMP-MEP stage and lymphoid progenitor identities

## How to apply

Extract per-cell cluster assignments and cell-type mean expression profiles from the fitted BUSseq object with the celltypes() and celltype_mean_expression() accessors. For candidate marker genes (Apoe and Gata2 for the CMP-MEP intermediate Cluster 4; Ctse for the MEP cluster; Ebf1, Vpreb1, Vpreb3, Igll1 for the lymphoid Cluster 3), compute corrected_read_counts() and summarize per-cluster means to confirm the expected marker-cluster assignments, rendering the gene-by-cell-type heatmap with heatmap_data_BUSseq() using gene_set, image_dir, and project_name. For clusters without a clear FACS label, correlate the cluster's cell-type mean expression profile against reference cell-type profiles (Haemopedia RNA-seq) using Pearson correlation and take the top-ranked match (expected: common lymphoid progenitors for Cluster 3). Rationale: BUSseq's corrected values can be used for downstream analysis as if all cells were sequenced in a single batch, so cross-platform reference mapping is legitimate.

## Related tools

- **BUSseq** (Bayesian hierarchical model that fits clusters, provides celltype/celltype_mean_expression accessors, corrected read counts for annotation, and marker heatmap rendering) — https://github.com/songfd2018/BUSseq
- **gplots** (R visualization support for the marker-gene by cell-type heatmap)
- **R** (Environment for running the BUSseq R package accessors, computing Pearson correlations against Haemopedia profiles, and writing the correlation CSV)

## Examples

```
heatmap_data_BUSseq(sce_BUSseqfit, gene_set=c('Apoe','Gata2','Ctse','Ebf1','Vpreb1','Vpreb3','Igll1'), project_name='hematopoietic_markers', image_dir='Image/')
```

## Evaluation signals

- cluster3_haemopedia_correlations.csv exists, is valid tabular data, and contains a Haemopedia cell_type column and a correlation column
- The top-ranked Haemopedia match for Cluster 3 is a common lymphoid progenitor population
- Marker-cluster assignments hold in corrected counts: Apoe and Gata2 are high in Cluster 4 (absent in CMP and MEP), Ctse is expressed only in the MEP cluster, and Ebf1/Vpreb1/Vpreb3/Igll1 mark Cluster 3
- The marker-gene heatmap image file is produced in the specified image_dir with the expected image format
- The annotation recovers a six-cell-type structure consistent with the FACS labels plus the newly identified intermediate CMP-MEP stage

## Limitations

- MCMC-based clustering is computationally heavy, and multi-stage pipelines ignore clustering uncertainties, which can propagate errors into annotation
- Completely confounded designs (batches measuring disjoint cell-type sets) admit no applicable correction, undermining annotation
- MNN/Scanorama-style assumptions (batch effects nearly orthogonal to and smaller than biological variation) cannot be validated at design stage, so correction choice affects downstream annotation quality

## Evidence

- [abstract] BUSseq can simultaneously correct batch effects, cluster cell types, impute missing data, and detect differentially expressed genes without preliminary normalization: [evidence span withheld — anchor status: not_found]
- [methods] corrected_read_counts accessor: "- `corrected_read_counts(sce_BUSseqfit)`"
- [methods] heatmap_data_BUSseq signature: "heatmap_data_BUSseq(sce_BUSseqfit, data_type, gene_set, project_name, image_dir, color_key_seq, image_width, image_height)"
- [readme] corrected values usable downstream: "After correcting the batch effects with BUSseq, the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch."
- [discussion] multi-stage uncertainty caveat: "As the clustering results may be prone to errors in practice, this can lead to false positives and false negatives."
