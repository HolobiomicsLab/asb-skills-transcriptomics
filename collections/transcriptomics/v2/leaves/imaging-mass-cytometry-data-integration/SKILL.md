---
name: imaging-mass-cytometry-data-integration
description: Use when integrating multi-sample CyTOF/imaging mass cytometry protein-expression
  data (e.g. 23-sample COVID-19 IMC with 38 protein markers) to remove sample/dataset
  variability before clustering against manual cell-type labels.
when_to_use_negative:
- Input is single-cell transcriptomics only (scRNA-seq) or multi-modal joint integration
  is required — the paper's integration is performed within each modality, with multi-modal
  joint integration deferred to future work
- Only an embedding output is needed for exploratory visualisation — scMerge2's value
  here is returning an adjusted expression matrix usable for sample-level analysis;
  embedding-only methods would not support downstream marker-specificity validation
- Data has no batch/sample structure or no reference annotation exists to assess clustering
  concordance (ARI cannot be computed)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMerge2
  role: Performs hierarchical integration of the 23 IMC samples via pseudo-bulk building
    and pseudo-replication to output the batch-adjusted 38-marker expression matrix
  repo: https://github.com/SydneyBioX/scMerge
- name: FlowSOM
  role: Clusters both original and scMerge2-adjusted IMC expression matrices into
    17 clusters for ARI evaluation against manual cell-type annotation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:878d314bad7d3c10ba57ef2a9e0e0e025a5e283f6c60d88563a408e2bf33aa15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - FlowSOM
  - scMerge2
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# imaging-mass-cytometry-data-integration

## Summary

Batch-effect correction of multi-sample imaging mass cytometry (IMC) data using scMerge2's hierarchical integration with pseudo-bulk building and pseudo-replication, followed by FlowSOM clustering, to produce an adjusted 38-marker expression matrix whose clusters align with manual cell-type annotations.

## When to use

Use when integrating multi-sample CyTOF/imaging mass cytometry protein-expression data (e.g. 23-sample COVID-19 IMC with 38 protein markers) to remove sample/dataset variability before clustering against manual cell-type labels.

## When NOT to use

- Input is single-cell transcriptomics only (scRNA-seq) or multi-modal joint integration is required — the paper's integration is performed within each modality, with multi-modal joint integration deferred to future work
- Only an embedding output is needed for exploratory visualisation — scMerge2's value here is returning an adjusted expression matrix usable for sample-level analysis; embedding-only methods would not support downstream marker-specificity validation
- Data has no batch/sample structure or no reference annotation exists to assess clustering concordance (ARI cannot be computed)

## Inputs

- COVID-19 imaging mass cytometry dataset: 23 samples, 38 protein markers (single-cell protein expression matrix)
- Manual cell-type annotation for IMC cells
- scMerge2 hierarchical integration specification (sample/condition batch structure)

## Outputs

- scMerge2-adjusted expression matrix (38 markers, batch-corrected across 23 samples)
- FlowSOM clustering of original and adjusted data (17 clusters each)
- ARI comparison table (ari_comparison.csv): original ARI 0.13 vs adjusted ARI 0.58
- Marker-enrichment heatmaps per cluster and CD4/CD8a/CD20 marker-specificity scatter plots (figure files)

## How to apply

Load the COVID-19 imaging mass cytometry dataset (23 samples, 38 protein markers) together with the manual cell-type annotation. Run scMerge2 to integrate samples into an adjusted expression matrix, exploiting its three innovations: hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication. Cluster both the original and adjusted matrices with FlowSOM, setting the number of clusters to 17. Compute the adjusted Rand index (ARI) of each clustering against the manual annotation — the paper's validation increases from ARI 0.13 (original) to 0.58 (scMerge2-adjusted). Confirm cluster identities with marker-enrichment heatmaps per cluster and CD4/CD8a/CD20 marker-specificity scatter plots, then save ari_comparison.csv and figures.

## Related tools

- **scMerge2** (Performs hierarchical integration of the 23 IMC samples via pseudo-bulk building and pseudo-replication to output the batch-adjusted 38-marker expression matrix) — https://github.com/SydneyBioX/scMerge
- **FlowSOM** (Clusters both original and scMerge2-adjusted IMC expression matrices into 17 clusters for ARI evaluation against manual cell-type annotation)

## Examples

```
BiocManager::install("scMerge")  # then run scMerge2 hierarchical integration and FlowSOM clustering per https://sydneybiox.github.io/scMerge/articles/scMerge2.html
```

## Evaluation signals

- ARI against manual cell-type annotation increases from 0.13 (original matrix) to 0.58 (scMerge2-adjusted matrix) with FlowSOM k=17
- Adjusted-data clustering shows substantially higher ARI than original-data clustering — a strict monotonic improvement is the paper's stated validation criterion
- Marker-enrichment heatmaps show expected per-cluster marker enrichment consistent with annotated cell types
- CD4/CD8a/CD20 scatter plots show marker-specific separation confirming cluster cell-type identity
- ari_comparison.csv is produced with both ARI values tabulated

## Limitations

- Integration demonstrated only within each modality; joint multi-modal integration (e.g. combining IMC with scRNA-seq signal) is not supported and deferred to future work
- Integration order is data-guided (batches within one study or studies of similar size integrated first); knowledge-guided integration ordering is described as an unused alternative
- Embedding-only outputs (as from many competing methods) hinder interpretation of downstream features such as cell-cell interactions or pathway information — adjusted expression matrix is preferred

## Evidence

- [abstract] scMerge2 removes dataset variability in CyTOF, imaging mass cytometry and CITE-seq experiments: "we demonstrate that scMerge2 can remove dataset variability in CyTOF, imaging mass cytometry and CITE-seq experiments"
- [discussion] Three key innovations of scMerge2 used in the workflow: "This was achieved via the use of three essential innovations with hierarchical integration, pseudo-bulk building to minimise processing demand, and pseudo-replication"
- [intro] Scalability across single-cell technologies including imaging mass cytometry: "integrate many millions of cells from single-cell studies generated from various single-cell technologies, including scRNA-seq, CyTOF, and imaging mass cytometry"
- [intro] Adjusted expression matrices matter for downstream analysis: "the lack of adjusted expression matrices restricts the utilisation of such integrative results and diminishes their potency and generalizability"
- [readme] scMerge package availability and vignettes: "`scMerge` is a R package for merging and normalising single-cell RNA-Seq datasets."
