---
name: adjusted-rand-index-clustering-evaluation
description: Use when a cell-classification or clustering output (e.g. scID target
  labels) must be quantitatively compared against published cell labels on the same
  cells, and a chance-corrected agreement score like ARI is needed.
when_to_use_negative:
- When reference and target datasets differ and labels for the same cells do not exist
  pairwise — ARI requires labels over the identical cell set
- 'When reference clusters are unreliable: the article notes that ''If neither of
  the two data is well clustered and contains good gene signatures, scID is not a
  recommended approach'', making the downstream ARI meaningless'
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scID
  role: Produces the predicted target cell labels (scid_multiclass output) whose agreement
    with published labels is scored by ARI
  repo: https://github.com/BatadaLab/scID
- name: mclust
  role: Provides adjustedRandIndex used to compute the Adjusted Rand Index between
    scID labels and published labels
- name: R
  role: Implementation environment for the scID pipeline and ARI computation
- name: Seurat
  role: Listed as an import of scID, used for handling scRNA-seq data in the workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:50cd8611495fe0909eb5ce61a1f9cf4999ab12d7877be33d10fea44894d45320
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BatadaLab/scID
  repo_url: https://github.com/BatadaLab/scID
  tools:
  - R
  - SC3
  - Seurat
  - mclust
  - scID
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# adjusted-rand-index-clustering-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates cell-labeling accuracy by computing the Adjusted Rand Index (ARI) between predicted cell labels (e.g. scID_output$labels from scid_multiclass) and published reference labels, as used to show scID's relatively high self-mapping accuracy on four published scRNA-seq datasets (Figure 1D).

## When to use

Use when a cell-classification or clustering output (e.g. scID target labels) must be quantitatively compared against published cell labels on the same cells, and a chance-corrected agreement score like ARI is needed.

## When NOT to use

- When reference and target datasets differ and labels for the same cells do not exist pairwise — ARI requires labels over the identical cell set
- When reference clusters are unreliable: the article notes that 'If neither of the two data is well clustered and contains good gene signatures, scID is not a recommended approach', making the downstream ARI meaningless

## Inputs

- Predicted cell labels (e.g. scID_output$labels, a named list of cluster labels for target cells)
- Published/reference cell labels for the same cells (e.g. cluster labels from the four published scRNA-seq datasets)
- Library-depth-normalized (CPM/TPM, non-log-transformed) gene expression matrices used to run scid_multiclass

## Outputs

- Per-dataset Adjusted Rand Index values (one ARI per dataset, expected to be relatively high/close to 1 in self-mapping)
- Compiled table (ari_self_mapping_table.csv) with one row per dataset

## How to apply

Run the classifier (e.g. scID's scid_multiclass) on each dataset to obtain predicted labels, then compute mclust::adjustedRandIndex between scID_output$labels and the published cell labels per dataset. In the scID self-mapping benchmark (Figure 1D), the same dataset serves as both reference and target: extract cluster-specific gene signatures with find_markers at a minimum logFC of 0.6 with only_pos = FALSE to retain downregulated cluster-specific genes, and set estimate_weights_from_target to TRUE since weights estimated from the target appear more accurate under strong batch effect. Compile the per-dataset ARI values into a table (e.g. ari_self_mapping_table.csv) with one row per dataset. ARI is chance-corrected, so values close to 1 indicate near-perfect label agreement; correct application is judged by relatively high ARI consistent with Figure 1D.

## Related tools

- **scID** (Produces the predicted target cell labels (scid_multiclass output) whose agreement with published labels is scored by ARI) — https://github.com/BatadaLab/scID
- **mclust** (Provides adjustedRandIndex used to compute the Adjusted Rand Index between scID labels and published labels)
- **R** (Implementation environment for the scID pipeline and ARI computation)
- **Seurat** (Listed as an import of scID, used for handling scRNA-seq data in the workflow)

## Examples

```
scID_output <- scID::scid_multiclass(target_gem, reference_gem, reference_clusters, estimate_weights_from_target = TRUE); ari <- mclust::adjustedRandIndex(unlist(scID_output$labels), published_labels)
```

## Evaluation signals

- Each self-mapping ARI should be relatively high and consistent with the values reported for scID in Figure 1D
- Output file (ari_self_mapping_table.csv) exists and contains exactly one row per dataset (four rows)
- All ARI values fall in the expected high range (close to 1) since reference and target are the same dataset
- ARI is computed with mclust::adjustedRandIndex over the same cell identifiers present in both scID_output$labels and the published labels
- Sanity check that scid_multiclass was run with estimate_weights_from_target = TRUE for batch-effected data, as recommended

## Limitations

- The workflow reports relative ARI values visualized in Figure 1D rather than exact numeric thresholds; 'relatively high' is the stated expectation
- ARI on self-mapping (same data as reference and target) overestimates performance on cross-dataset mapping with batch effect
- If neither dataset is well clustered with good gene signatures, scID is not recommended, invalidating the ARI comparison
- No software version of scID is stated in the discussion; results may vary with package version (v2.0.0 introduced negative markers)

## Evidence

- [other] In self-mapping (same dataset as reference and target) of four published datasets spanning different tissues and technologies, scID's accuracy as assessed by Adjusted Rand Index was relatively high (Figure 1D).: [evidence span withheld — anchor status: not_found]
- [methods] estimate_weights_from_target is set to TRUE in order to estimated gene weights from the target by selecting training target cells: "estimate_weights_from_target is set to TRUE in order to estimated gene weights from the target by selecting training target cells"
- [methods] logFC is defining minimum logFold-change for a gene to be seleced as cluster-specific.: "logFC is defining minimum logFold-change for a gene to be seleced as cluster-specific."
- [readme] uses the framework of Fisher's Linear Discriminant Analysis to identify transcriptionally related cell types between scRNA-seq datasets: "uses the framework of Fisher's Linear Discriminant Analysis to identify transcriptionally related cell types between scRNA-seq datasets"
- [discussion] If neither of the two data is well clustered and contains good gene signatures, scID is not a recommended approach for identification of equivalent cell types.: "If neither of the two data is well clustered and contains good gene signatures, scID is not a recommended approach for identification of equivalent cell types."
