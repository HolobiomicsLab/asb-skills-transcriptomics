---
name: marker-enrichment-heatmap-visualization
description: Use when FlowSOM clusters (e.g., 17 clusters on COVID-19 IMC data with
  38 protein markers) must be visually validated against known marker biology to confirm
  cell-type identity after scMerge2 adjustment.
when_to_use_negative:
- Cells have not yet been clustered (no FlowSOM cluster assignments) — enrichment
  per cluster is undefined until clustering is done.
- The goal is quantitative clustering-accuracy assessment (ARI computation) rather
  than visual confirmation of cell-type identity.
- The data are scRNA-seq samples where the specific IMC protein markers (CD4, CD8a,
  CD20) and imaging mass cytometry panel do not apply.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scMerge2
  role: Produces the adjusted expression matrix whose per-cluster marker enrichment
    is visualized; removes dataset variability in imaging mass cytometry experiments
  repo: https://github.com/SydneyBioX/scMerge
- name: FlowSOM
  role: Provides the 17-cluster assignments over which marker enrichment is aggregated
    and plotted
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:d0320cb33b82bbc5410d0c5e17329386f2a891ade5b8868b23d1799a43365c5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - FlowSOM
  - scMerge2
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# marker-enrichment-heatmap-visualization

## Summary

Generates marker-enrichment heatmaps per cluster (plus marker-specificity scatter plots such as CD4/CD8a/CD20) to confirm the cell-type identity of FlowSOM clusters computed on scMerge2-adjusted imaging mass cytometry data.

## When to use

Use when FlowSOM clusters (e.g., 17 clusters on COVID-19 IMC data with 38 protein markers) must be visually validated against known marker biology to confirm cell-type identity after scMerge2 adjustment.

## When NOT to use

- Cells have not yet been clustered (no FlowSOM cluster assignments) — enrichment per cluster is undefined until clustering is done.
- The goal is quantitative clustering-accuracy assessment (ARI computation) rather than visual confirmation of cell-type identity.
- The data are scRNA-seq samples where the specific IMC protein markers (CD4, CD8a, CD20) and imaging mass cytometry panel do not apply.

## Inputs

- scMerge2-adjusted expression matrix (23 COVID-19 IMC samples, 38 protein markers)
- original (unadjusted) IMC expression matrix for comparison
- FlowSOM cluster assignments (17 clusters) for both matrices
- manual cell-type annotation for the IMC cells

## Outputs

- marker-enrichment heatmap per cluster (figure file) for original and adjusted clusterings
- CD4/CD8a/CD20 marker-specificity scatter plots (figure files)
- ari_comparison.csv tabulating original (0.13) vs adjusted (0.58) ARI values

## How to apply

After clustering the original and scMerge2-adjusted expression matrices with FlowSOM at 17 clusters, compute per-cluster enrichment of the 38 protein markers and render a heatmap per clustering so marker-expression patterns can be matched to the manual cell-type annotation. Complement the heatmap with marker-specificity scatter plots for canonical IMC markers (CD4 for helper T cells, CD8a for cytotoxic T cells, CD20 for B cells) to verify each cluster expresses its expected markers. Rationale: the scMerge2-adjusted matrix (which uses hierarchical integration, pseudo-bulk building, and pseudo-replication) removes dataset variability, so marker enrichment should be cleaner and clusters more consistent with the manual annotation, whose ARI improved from 0.13 to 0.58. Save the heatmap and scatter-plot figure files alongside the ari_comparison.csv so visual and quantitative evidence support each other.

## Related tools

- **scMerge2** (Produces the adjusted expression matrix whose per-cluster marker enrichment is visualized; removes dataset variability in imaging mass cytometry experiments) — https://github.com/SydneyBioX/scMerge
- **FlowSOM** (Provides the 17-cluster assignments over which marker enrichment is aggregated and plotted)

## Examples

```
BiocManager::install("scMerge"); data(segList)
```

## Evaluation signals

- CD4, CD8a, and CD20 scatter plots show the expected marker specificity: helper T-cell clusters are CD4-high, cytotoxic clusters CD8a-high, B-cell clusters CD20-high.
- Marker-enrichment heatmaps for the scMerge2-adjusted clustering show cleaner, more distinct per-cluster marker blocks than the original-data heatmaps.
- Adjusted-data clustering is substantially more concordant with the manual annotation (ARI 0.58 vs 0.13), corroborating the visual evidence.
- Number of FlowSOM clusters equals 17, matching the workflow specification.
- All figure files and ari_comparison.csv are saved as deliverables.

## Limitations

- Visual marker-confirmation is qualitative; it supplements but does not replace the quantitative ARI comparison against the manual annotation.
- Marker-specificity checks rely on the 38-protein IMC panel; clusters lacking canonical markers (CD4/CD8a/CD20) cannot be confidently assigned by this route.
- Interpretation depends on the quality of the scMerge2 adjustment, which in the paper is performed within a single modality; multi-modal joint integration is deferred to future work.

## Evidence

- [abstract] scMerge2 removes dataset variability in CyTOF, imaging mass cytometry and CITE-seq experiments: "we demonstrate that scMerge2 can remove dataset variability in CyTOF, imaging mass cytometry and CITE-seq experiments"
- [discussion] Three scMerge2 innovations justify the adjusted matrix used for enrichment: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [readme] scMerge2 R package installation path from README: "`scMerge` is a R package for merging and normalising single-cell RNA-Seq datasets."
- [full_text] ARI improvement motivating cluster-identity confirmation: [evidence span withheld — anchor status: not_found]
