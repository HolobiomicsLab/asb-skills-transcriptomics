---
name: marker-gene-expression-validation
description: Use when BUSseq has been fit to scRNA-seq count data (e.g., the mouse
  hematopoietic study) and unknown clusters must be assigned biological identity via
  lineage marker genes and reference cell-type atlas correlations.
when_to_use_negative:
- Cell-type labels are already known and the goal is only batch correction, not de
  novo cluster annotation
- The fitted model is confounded such that biological variability cannot be separated
  from batch effects (completely confounded designs admit no applicable method)
- The marker genes of interest are absent from the measured gene panel, making cluster
  identity testing impossible
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: BUSseq
  role: Fits the Bayesian hierarchical model, clusters cell types, and provides corrected_read_counts(),
    celltypes(), celltype_mean_expression(), and heatmap_data_BUSseq() used for marker
    validation
  repo: https://github.com/songfd2018/BUSseq
- name: gplots
  role: R visualization support for rendering marker-gene heatmaps
- name: R
  role: Environment for computing per-cluster mean expression and Pearson correlations
    against Haemopedia profiles
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:e81f602167ee61f22a4e24a13e63700e726eaa9a9ef545b08d01c7d332462a2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq
  tools:
  - BUSseq
  - R
  - gplots
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# marker-gene-expression-validation

## Summary

Validate that BUSseq-discovered clusters correspond to biologically meaningful hematopoietic cell types by testing expected marker-gene assignments (e.g., Apoe/Gata2 for a CMP-MEP intermediate, Ctse for MEP, Ebf1/Vpreb1/Vpreb3/Igll1 for lymphoid progenitors) on corrected read counts and by correlating cluster profiles against Haemopedia RNA-seq references.

## When to use

Use when BUSseq has been fit to scRNA-seq count data (e.g., the mouse hematopoietic study) and unknown clusters must be assigned biological identity via lineage marker genes and reference cell-type atlas correlations.

## When NOT to use

- Cell-type labels are already known and the goal is only batch correction, not de novo cluster annotation
- The fitted model is confounded such that biological variability cannot be separated from batch effects (completely confounded designs admit no applicable method)
- The marker genes of interest are absent from the measured gene panel, making cluster identity testing impossible

## Inputs

- Fitted BUSseq object (e.g., BUSseqfits_example) with per-cell cluster assignments
- Corrected read counts from corrected_read_counts()
- Marker gene list (Apoe, Gata2, Ctse, Ebf1, Vpreb1, Vpreb3, Igll1)
- Haemopedia RNA-seq cell-type reference profiles

## Outputs

- Marker-gene by cell-type heatmap image (heatmap_data_BUSseq with image_dir/project_name)
- cluster3_haemopedia_correlations.csv with Haemopedia cell_type and correlation columns
- Summary of the six-cell-type structure including the intermediate CMP-MEP stage

## How to apply

Extract per-cell cluster assignments and cell-type mean expression profiles from the fitted BUSseq object (e.g., BUSseqfits_example) with celltypes() and celltype_mean_expression(). Compute corrected_read_counts() — BUSseq's corrected values can be used downstream 'as if all cells were sequenced in a single batch' — then summarize per-cluster mean expression for the marker genes (Apoe, Gata2 for Cluster 4; Ctse for MEP; Ebf1, Vpreb1, Vpreb3, Igll1 for Cluster 3) to test expected marker-cluster assignments. Render a gene-by-cell-type heatmap with heatmap_data_BUSseq(), setting gene_set to the marker genes and supplying image_dir/project_name. Finally, compute Pearson correlations between Cluster 3's cell-type mean expression profile and each Haemopedia RNA-seq cell-type profile to identify the best-matching progenitor populations, expecting common lymphoid progenitors as the top match.

## Related tools

- **BUSseq** (Fits the Bayesian hierarchical model, clusters cell types, and provides corrected_read_counts(), celltypes(), celltype_mean_expression(), and heatmap_data_BUSseq() used for marker validation) — https://github.com/songfd2018/BUSseq
- **gplots** (R visualization support for rendering marker-gene heatmaps)
- **R** (Environment for computing per-cluster mean expression and Pearson correlations against Haemopedia profiles)

## Examples

```
R: hm <- corrected_read_counts(BUSseqfits_example); heatmap_data_BUSseq(BUSseqfits_example, data_type="corrected_read_counts", gene_set=c("Apoe","Gata2","Ctse","Ebf1","Vpreb1","Vpreb3","Igll1"), project_name="markers", image_dir="figures")
```

## Evaluation signals

- Both output files exist with correct tabular/image format
- The correlation table contains a Haemopedia cell_type column and a correlation column
- The top-ranked Haemopedia match for Cluster 3 is a common lymphoid progenitor population
- Cluster 4 shows high expression of Apoe and Gata2 (absent in CMP and MEP) while Ctse is expressed only in the MEP cluster, supporting the intermediate CMP-MEP interpretation
- Cluster 3 expresses the B-lineage markers Ebf1, Vpreb1, Vpreb3, and Igll1

## Limitations

- Cluster assignments may contain errors; multi-stage approaches that ignore uncertainties in previous stages can lead to false positives and false negatives
- MCMC inference is computationally heavy, although most steps are parallelizable
- The mouse hematopoietic raw count data must be downloaded separately (OneDrive, password BUSseq2019) and placed into the MouseHematopoietic/RawCountData folder

## Evidence

- [discussion] BUSseq simultaneously corrects batch effects, clusters cell types, imputes missing data, and identifies intrinsic genes that differentiate cell types.: "BUSseq simultaneously corrects batch effects, clusters cell types, imputes missing data, and identifies intrinsic genes that differentiate cell types."
- [methods] heatmap_data_BUSseq signature: "heatmap_data_BUSseq(sce_BUSseqfit, data_type, gene_set, project_name, image_dir, color_key_seq, image_width, image_height)"
- [methods] corrected read counts accessor: "- `corrected_read_counts(sce_BUSseqfit)`"
- [readme] README overview of BUSseq capabilities: "BUSseq is able to simultaneously correct batch effects, clusters cell types, and takes care of the count data nature, the overdispersion, the dropout events, and the cell-specific sequencing depth"
- [readme] README on downstream use of corrected values: "After correcting the batch effects with BUSseq, the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch."
