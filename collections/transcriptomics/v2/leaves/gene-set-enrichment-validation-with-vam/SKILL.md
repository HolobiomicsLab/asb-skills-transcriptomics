---
name: gene-set-enrichment-validation-with-vam
description: Use when a rare, newly recovered single-cell subcluster must be confirmed
  against a published marker gene set — e.g., an apCAF subcluster (23 cells, 0.092%
  of the Elyada PDAC dataset) that was absent from the original analysis.
when_to_use_negative:
- The input is a broad, already-homogeneous cluster — VAM validation presupposes SCISSORS
  has first recovered candidate rare subclusters via silhouette scoring and reclustering
- No externally defined, biologically curated marker gene set exists for the candidate
  cell type — subcluster annotation would then rest only on subcluster-specific marker
  gene examination
- The goal is de novo marker discovery for an uncharacterized subcluster rather than
  validation against a published cell-type signature
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCISSORS
  role: Recovers the rare candidate subclusters (via silhouette scoring and iterative
    semi-supervised reclustering) whose identity the VAM enrichment test then validates
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Hosts the subclustered single-cell object; supplies DimPlot visualization
    of validated subcluster assignments on the t-SNE embedding
  repo: https://github.com/jr-leary7/SCISSORS
- name: ggplot2
  role: Saves the subcluster assignment figure (ggsave to SCISSORS_Clusters.pdf) after
    validation
  repo: https://github.com/jr-leary7/SCISSORS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:50a499a8317b9111176a35c7f0eda6e9597d1bab750721b802458e5056bedb69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - R
  - SCISSORS
  - Seurat
  - ggplot2
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Gene-set enrichment validation with VAM

## Summary

Validate that a computationally recovered rare subcluster (e.g., the 23-cell apCAF population in the Elyada human PDAC data) corresponds to a biologically defined cell type by testing significant enrichment of an externally defined marker gene set using variance-adjusted Mahalanobis (VAM) enrichment, rather than relying on subcluster size or ad hoc marker inspection alone.

## When to use

Use when a rare, newly recovered single-cell subcluster must be confirmed against a published marker gene set — e.g., an apCAF subcluster (23 cells, 0.092% of the Elyada PDAC dataset) that was absent from the original analysis.

## When NOT to use

- The input is a broad, already-homogeneous cluster — VAM validation presupposes SCISSORS has first recovered candidate rare subclusters via silhouette scoring and reclustering
- No externally defined, biologically curated marker gene set exists for the candidate cell type — subcluster annotation would then rest only on subcluster-specific marker gene examination
- The goal is de novo marker discovery for an uncharacterized subcluster rather than validation against a published cell-type signature

## Inputs

- Seurat object with SCISSORS-recovered subcluster assignments for a heterogeneous broad cluster (e.g., Cluster 8 of the Elyada human PDAC scRNA-seq dataset)
- Published marker gene set for the target cell type (e.g., the originally defined apCAF marker genes from Elyada et al.)

## Outputs

- VAM enrichment statistics/significance per subcluster for the marker gene set
- Validation call: confirmation that the rare subcluster (23 cells, 0.092% of the dataset) is the apCAF population
- Annotated t-SNE/DimPlot figure (e.g., SCISSORS_Clusters.pdf) showing subcluster assignments

## How to apply

After SCISSORS iterative reclustering (silhouette scoring plus multi-step semi-supervised reclustering around the Seurat pipeline) splits a heterogeneous broad cluster such as Cluster 8 of the Elyada human PDAC data into CAF, endothelial, and perivascular subclusters, take the originally published marker gene set for the cell type of interest (the apCAF markers defined in the original study). Score each recovered subcluster for enrichment of that gene set with VAM, which accounts for gene-gene correlation structure when computing enrichment statistics, making it sensitive enough to detect enrichment in very small subclusters (~23 cells, 0.092% of the dataset). Treat significant VAM enrichment of the published marker set as the validation criterion that the rare subcluster is the real, previously overlooked population — recovery of the correct cell count alone is not sufficient. Report the enriched subcluster's cell count and dataset fraction alongside the enrichment result, and visualize assignments on the t-SNE embedding with Seurat's DimPlot before saving (e.g., ggsave to SCISSORS_Clusters.pdf).

## Related tools

- **SCISSORS** (Recovers the rare candidate subclusters (via silhouette scoring and iterative semi-supervised reclustering) whose identity the VAM enrichment test then validates) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Hosts the subclustered single-cell object; supplies DimPlot visualization of validated subcluster assignments on the t-SNE embedding) — https://github.com/jr-leary7/SCISSORS
- **ggplot2** (Saves the subcluster assignment figure (ggsave to SCISSORS_Clusters.pdf) after validation) — https://github.com/jr-leary7/SCISSORS

## Examples

```
seu_cluster8 <- Recluster(seu_cluster8, clusters = "8"); DimPlot(seu_cluster8, group.by = "subcluster") + ggsave("SCISSORS_Clusters.pdf")
```

## Evaluation signals

- The rare subcluster is recovered at the reported size: 23 cells, 0.092% of the Elyada human PDAC dataset
- The recovered subcluster shows significant VAM enrichment of the originally defined apCAF marker gene set
- The subcluster is annotated as apCAF (an antigen-presenting CAF) after examination of subcluster-specific marker genes alongside the enrichment result
- Subcluster assignments render correctly on the t-SNE embedding via DimPlot and export cleanly with ggsave (SCISSORS_Clusters.pdf)
- The population is one that the original human analysis of the same data did not detect, so the enrichment-based confirmation is the discriminating check against a spurious split

## Limitations

- Validation depends on the quality and completeness of the published marker gene set — enrichment cannot confirm a subcluster whose true signature diverges from the reference markers
- Extremely low-abundance subclusters (23 cells here) carry subtle contributions to global clustering, so small differences in analysis and parameter choice can alter which cells land in the subcluster before enrichment is even tested
- VAM confirmation is specific to the tested gene set; a subcluster could pass for apCAF markers yet require additional marker-gene examination to distinguish it from adjacent CAF, endothelial, or perivascular subclusters

## Evidence

- [results] SCISSORS reanalysis of the same human PDAC scRNA-seq data identified the rare apCAF population (23 cells, 0.092% of the dataset), confirmed by significant VAM enrichment of the originally defined apCAF marker genes.: [evidence span withheld — anchor status: not_found]
- [results] VAM validation of the rare subcluster marker signature: [evidence span withheld — anchor status: not_found]
- [abstract] SCISSORS silhouette scoring and semi-supervised reclustering: "SCISSORS employs silhouette scoring for the estimation of heterogeneity of clusters and reveals rare cells in heterogenous clusters by a multi-step semi-supervised reclustering process"
- [abstract] SCISSORS as Seurat-integrated framework: "SCISSORS is wrapped around the popular Seurat R package and can be easily integrated into existing Seurat pipelines"
- [abstract] Small parameter differences alter clustering and thus subcluster recovery: "even small differences in analysis and parameter choosing can greatly alter clustering results and thus impose great influence on which cell types are identified"
