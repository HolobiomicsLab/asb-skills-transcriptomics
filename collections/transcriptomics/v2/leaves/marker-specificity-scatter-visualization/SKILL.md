---
name: marker-specificity-scatter-visualization
description: Use when you have FlowSOM clusters (e.g. 17 clusters from the 23-sample
  COVID-19 IMC dataset with 38 protein markers) and must validate that clusters correspond
  to annotated cell types using marker-expression specificity plots.
when_to_use_negative:
- Input is raw unadjusted IMC data with known strong batch/sample variability — the
  plots may not separate cell types until scMerge2 adjustment is applied (original
  data yields ARI only 0.13).
- You need quantitative cluster-vs-annotation concordance (ARI) rather than qualitative
  identity confirmation — use the ARI comparison workflow instead.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMerge2
  role: Produces the adjusted expression matrix whose marker values are plotted to
    confirm cluster cell-type identity
  repo: https://github.com/SydneyBioX/scMerge
- name: FlowSOM
  role: Generates the 17 clusters whose marker specificity is being visualized
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:072d543a427bf69b207facaa3189c5fb60de82ca0f46ddeb2448fdff71aa8f6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - FlowSOM
  - scMerge2
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# marker-specificity-scatter-visualization

## Summary

Builds marker-specificity scatter plots (e.g. CD4 vs CD8a vs CD20 on the COVID-19 imaging mass cytometry panel) plus per-cluster marker-enrichment heatmaps, used after scMerge2-adjusted FlowSOM clustering to confirm that each of the 17 clusters corresponds to a manually annotated cell type.

## When to use

Use when you have FlowSOM clusters (e.g. 17 clusters from the 23-sample COVID-19 IMC dataset with 38 protein markers) and must validate that clusters correspond to annotated cell types using marker-expression specificity plots.

## When NOT to use

- Input is raw unadjusted IMC data with known strong batch/sample variability — the plots may not separate cell types until scMerge2 adjustment is applied (original data yields ARI only 0.13).
- You need quantitative cluster-vs-annotation concordance (ARI) rather than qualitative identity confirmation — use the ARI comparison workflow instead.

## Inputs

- scMerge2-adjusted expression matrix (23 COVID-19 IMC samples, 38 protein markers)
- FlowSOM cluster assignments (17 clusters) for each cell
- Manual cell-type annotation per cell
- Marker panel list including CD4, CD8a, CD20 and other protein markers

## Outputs

- Marker-specificity scatter plots over CD4/CD8a/CD20 per cluster (figure files)
- Marker-enrichment heatmap of mean marker expression per FlowSOM cluster (figure files)
- Visual confirmation of cell-type identity for each of the 17 clusters

## How to apply

After running scMerge2 on the imaging mass cytometry samples and clustering the adjusted expression matrix with FlowSOM into 17 clusters, compute per-cluster mean expression for each of the 38 protein markers. Plot marker-enrichment heatmaps with clusters as rows (or columns) and markers as the opposing axis, using the manual cell-type annotation to label cluster identities. Then build marker-specificity scatter plots over canonical IMC markers — CD4, CD8a and CD20 — positioning each cluster by its mean adjusted marker intensities so that T-helper, cytotoxic T-cell and B-cell populations separate along expected axes. The rationale, grounded in the scMerge2 paper, is that scMerge2 removes dataset variability in imaging mass cytometry experiments, so marker specificity in the adjusted matrix should cleanly discriminate cell types that were confounded in the unadjusted data (ARI 0.13 vs 0.58). A cluster is confirmed only if its dominant markers match the expected signature of its assigned cell type.

## Related tools

- **scMerge2** (Produces the adjusted expression matrix whose marker values are plotted to confirm cluster cell-type identity) — https://github.com/SydneyBioX/scMerge
- **FlowSOM** (Generates the 17 clusters whose marker specificity is being visualized)

## Evaluation signals

- Each cluster's dominant markers match the canonical cell-type signature (e.g. CD20-high clusters correspond to B cells, CD8a-high to cytotoxic T cells) in the scatter plots.
- Marker-separating axes (CD4 vs CD8a vs CD20) resolve distinct cluster populations in the adjusted matrix, unlike the unadjusted data.
- Clusters whose identity is visually confirmed correspond to the same labels used in the ARI computation that increases from 0.13 to 0.58.
- Figure files (scatter plots and per-cluster marker-enrichment heatmaps) are produced alongside the ari_comparison.csv output of the validation workflow.

## Limitations

- Marker-specificity plots are a qualitative check; they do not replace the ARI quantification against the manual annotation.
- In the paper, integration was done within each modality; joint use of multi-modal information is deferred to future work, so cross-modality marker interpretation is out of scope.
- Interpretation depends on the 38-marker IMC panel; markers absent from the panel cannot support identity assignments.

## Evidence

- [abstract] scMerge2 removes dataset variability in CyTOF, IMC and CITE-seq experiments: "we demonstrate that scMerge2 can remove dataset variability in CyTOF, imaging mass cytometry and CITE-seq experiments"
- [intro] scMerge2 applies to imaging mass cytometry data: "including scRNA-seq, CyTOF, and imaging mass cytometry"
- [discussion] Three scMerge2 innovations underpin the adjusted matrix used for plotting: "hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
