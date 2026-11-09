---
name: cross-dataset-expression-correlation-mapping
description: Use when unsupervised BUSseq clusters need biological labels beyond FACS
  labels — e.g., verifying an intermediate CMP-MEP cluster or mapping a CLP/LMPP-like
  cluster to Haemopedia progenitor populations.
when_to_use_negative:
- Cells already have trusted FACS or reference-matched labels, so unsupervised cluster-to-reference
  correlation adds no annotation value
- The scRNA-seq data are not read counts (e.g., UMI-normalized or TPM matrices), since
  BUSseq models count data with overdispersion and dropout directly
- Batches are completely confounded with cell types (e.g., batch 1 has only cell types
  1–2, batch 2 only 3–4), where no batch-correction method is applicable
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: BUSseq
  role: Fits the Bayesian hierarchical model, supplies cluster assignments, cell-type
    mean expression, corrected read counts, and heatmap_data_BUSseq for marker visualization
  repo: https://github.com/songfd2018/BUSseq
- name: BUSseq-Rpackage
  role: R package providing the accessors (celltypes, corrected_read_counts) and heatmap_data_BUSseq
    used in the workflow
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: gplots
  role: R plotting support for rendering the marker-gene by cell-type heatmap
- name: R
  role: Environment for computing per-cluster mean expression, Pearson correlations
    against Haemopedia profiles, and writing CSV outputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:1acaa8638409802c955c1ab91ddffce614a508a670748b9c9f81fbbe989c78ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq
  tools:
  - BUSseq
  - BUSseq-Rpackage
  - R
  - gplots
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cross-dataset-expression-correlation-mapping

## Summary

Annotates clusters discovered by BUSseq in a target scRNA-seq dataset (e.g., the mouse hematopoietic study) by correlating cell-type mean expression profiles with reference cell-type profiles from an external resource (Haemopedia RNA-seq), combined with marker-gene heatmap validation.

## When to use

Use when unsupervised BUSseq clusters need biological labels beyond FACS labels — e.g., verifying an intermediate CMP-MEP cluster or mapping a CLP/LMPP-like cluster to Haemopedia progenitor populations.

## When NOT to use

- Cells already have trusted FACS or reference-matched labels, so unsupervised cluster-to-reference correlation adds no annotation value
- The scRNA-seq data are not read counts (e.g., UMI-normalized or TPM matrices), since BUSseq models count data with overdispersion and dropout directly
- Batches are completely confounded with cell types (e.g., batch 1 has only cell types 1–2, batch 2 only 3–4), where no batch-correction method is applicable

## Inputs

- Fitted BUSseq object (e.g., BUSseqfits_example) with per-cell cluster assignments
- Corrected read counts from corrected_read_counts(sce_BUSseqfit)
- Marker-gene sets (Apoe, Gata2, Ctse, Ebf1, Vpreb1, Vpreb3, Igll1)
- Haemopedia RNA-seq reference cell-type expression profiles

## Outputs

- cluster3_haemopedia_correlations.csv with Haemopedia cell_type and correlation columns
- Marker-gene by cell-type heatmap image written via image_dir/project_name
- Summary of the six-cell-type structure including the intermediate CMP-MEP stage

## How to apply

Load the fitted BUSseq object (BUSseqfits_example) and extract per-cell cluster assignments and cell-type mean expression profiles. Compute corrected read counts with corrected_read_counts(), summarize per-cluster mean expression for lineage markers (Apoe, Gata2 for the CMP-MEP intermediate cluster; Ctse for MEP; Ebf1, Vpreb1, Vpreb3, Igll1 for the lymphoid-primed cluster) to confirm marker-cluster assignments, and render the marker-by-cell-type heatmap with heatmap_data_BUSseq() using gene_set and the image_dir/project_name arguments. Compute Pearson correlations between the target cluster's cell-type mean expression profile and each Haemopedia RNA-seq cell-type profile; the top-ranked match assigns the cluster's identity (expected: common lymphoid progenitors). Validate that the correlation table has Haemopedia cell_type and correlation columns, the heatmap image is written, and the top match is biologically consistent with the marker evidence.

## Related tools

- **BUSseq** (Fits the Bayesian hierarchical model, supplies cluster assignments, cell-type mean expression, corrected read counts, and heatmap_data_BUSseq for marker visualization) — https://github.com/songfd2018/BUSseq
- **BUSseq-Rpackage** (R package providing the accessors (celltypes, corrected_read_counts) and heatmap_data_BUSseq used in the workflow) — https://github.com/songfd2018/BUSseq-Rpackage
- **gplots** (R plotting support for rendering the marker-gene by cell-type heatmap)
- **R** (Environment for computing per-cluster mean expression, Pearson correlations against Haemopedia profiles, and writing CSV outputs)

## Examples

```
Rscript -e 'corr <- cor(as.matrix(celltype_mean_expression(BUSseqfits_example)[3,]), haemopedia_profiles); write.csv(corr, "cluster3_haemopedia_correlations.csv"); heatmap_data_BUSseq(BUSseqfits_example, data_type="corrected", gene_set=markers, project_name="markers", image_dir=".")'
```

## Evaluation signals

- cluster3_haemopedia_correlations.csv exists, is valid tabular output, and contains a Haemopedia cell_type column plus a correlation column
- The top-ranked Haemopedia match for Cluster 3 is a common lymphoid progenitor population
- The marker-gene heatmap image file exists and is a valid image, generated with gene_set set to the marker genes
- Cluster 4 shows high expression of Apoe and Gata2 (absent in CMP and MEP) while Ctse is expressed only in the MEP cluster, supporting the intermediate CMP-MEP assignment
- The recovered structure is six clusters, with at least two subpopulations beyond the FACS labels

## Limitations

- Cluster assignments from any method can be prone to errors, and multi-stage workflows that ignore such uncertainty can yield false positives and false negatives
- MCMC inference for BUSseq is computationally heavy, although most steps are parallelizable
- Selecting the optimal number of cell types K requires comparing BIC values across candidate K (e.g., enumerating K from 3 to 6); a wrong K distorts the cluster-to-reference mapping

## Evidence

- [results] In the hematopoietic analysis, BUSseq's Cluster 4 shows high expression of marker genes Apoe and Gata2 (absent in CMP and MEP), and Ctse is expressed only in MEP, supporting Cluster 4 as an intermediate CMP-MEP cell stage.: [evidence span withheld — anchor status: not_found]
- [methods] Accessors for corrected values used in downstream analysis.: "corrected_read_counts(sce_BUSseqfit)"
- [methods] Heatmap rendering function signature with gene_set, image_dir and project_name arguments.: "heatmap_data_BUSseq(sce_BUSseqfit, data_type, gene_set, project_name, image_dir, color_key_seq, image_width, image_height)"
- [methods] Optimal cluster number selection by BIC enumeration.: "To select the optimal *K*, we need to compare the BIC values for different *K*s. In the vignettes, we enumerate *K* from 3 to 6."
- [readme] Corrected values can be used downstream as if all cells came from a single batch.: "After correcting the batch effects with BUSseq, the corrected value can be used for downstream analysis as if all cells were sequenced in a single batch."
