---
name: self-mapping-benchmark-analysis
description: Use when evaluating how accurate and stable a reference-mapping classifier
  (e.g. scID) is in a controlled self-mapping setting, where published cluster labels
  exist and ARI against them is the metric of interest.
when_to_use_negative:
- Reference and target are different datasets with batch effect — instead use the
  cross-mapping setting where estimate_weights_from_target=TRUE is motivated by strong
  batch effect between two datasets
- Data are log-transformed — scID requires library-depth (TPM/CPM) normalized, not
  log-transformed, expression
- The reference data are not well clustered and lack good gene signatures — the article
  states scID is not a recommended approach in that case
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scID
  role: Main label-transfer method being benchmarked; find_markers extracts cluster-specific
    signatures and scid_multiclass assigns target cells to reference clusters
  repo: https://github.com/BatadaLab/scID
- name: R
  role: Implementation environment for the scID package, mclust ARI computation, and
    table writing
- name: mclust
  role: Provides adjustedRandIndex used to score agreement between scID labels and
    published labels
- name: Seurat
  role: Listed as an Imports dependency of the scID package
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:92a48f7715397370810ca663ecb782840c5e4ee952375b87ff2f873faec44cdb
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
  - Seurat
  - mclust
  - scID
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# self-mapping-benchmark-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark a cell-type label-transfer method such as scID by using the same published scRNA-seq dataset as both reference and target, then quantify agreement with the published cell labels via Adjusted Rand Index, as done for scID's Figure 1D across four datasets from different tissues and technologies.

## When to use

Use when evaluating how accurate and stable a reference-mapping classifier (e.g. scID) is in a controlled self-mapping setting, where published cluster labels exist and ARI against them is the metric of interest.

## When NOT to use

- Reference and target are different datasets with batch effect — instead use the cross-mapping setting where estimate_weights_from_target=TRUE is motivated by strong batch effect between two datasets
- Data are log-transformed — scID requires library-depth (TPM/CPM) normalized, not log-transformed, expression
- The reference data are not well clustered and lack good gene signatures — the article states scID is not a recommended approach in that case

## Inputs

- Raw scRNA-seq count matrices for published datasets with published cell labels (e.g. 10X neuron_9k and nuclei_900 E18 mouse brain data)
- Pre-computed TPM-normalized example data and markers.rds from github.com/BatadaLab/scID (ExampleData)
- Reference cluster label list (reference_clusters) per dataset

## Outputs

- scid_multiclass output per dataset, including scID_output$labels, $markers, $weights and $scores
- Per-dataset Adjusted Rand Index table (ari_self_mapping_table.csv) with one row per dataset

## How to apply

Download the published scRNA-seq count matrices together with their published cell labels (e.g. the 10X E18 mouse brain neuron_9k and nuclei_900 datasets from the 10x Genomics links, plus pre-computed TPM-normalized example data from github.com/BatadaLab/scID under ExampleData). Normalize raw counts with library-depth normalization (CPM/TPM via counts_to_cpm/normalize_gene), avoiding log-transformed data since scID requires non-log expression. For each dataset, use it as both reference and target: extract cluster-specific gene signatures with find_markers at a minimum logFC threshold (e.g. logFC = 0.6 as in the vignette) with only_pos = FALSE so that downregulated (negative) markers are retained, since scID v2.0.0 uses negative markers together with positive ones. Run scid_multiclass on each dataset with the extracted reference_clusters, setting estimate_weights_from_target to TRUE because estimation of gene signature weights from the target data appears more accurate under batch effect. Finally compute mclust::adjustedRandIndex between scID_output$labels and the published labels per dataset, compile the ARIs into a per-dataset table (e.g. ari_self_mapping_table.csv), and compare against the relatively high values reported in Figure 1D.

## Related tools

- **scID** (Main label-transfer method being benchmarked; find_markers extracts cluster-specific signatures and scid_multiclass assigns target cells to reference clusters) — https://github.com/BatadaLab/scID
- **R** (Implementation environment for the scID package, mclust ARI computation, and table writing)
- **mclust** (Provides adjustedRandIndex used to score agreement between scID labels and published labels)
- **Seurat** (Listed as an Imports dependency of the scID package)

## Examples

```
scID_output <- scid_multiclass(target_gem = target_gem, reference_gem = reference_gem, reference_clusters = reference_clusters, estimate_weights_from_target = TRUE); mclust::adjustedRandIndex(scID_output$labels, published_labels)
```

## Evaluation signals

- Each self-mapping ARI should be relatively high and close to 1, consistent with the relatively high values reported for scID in Figure 1D
- Output file exists and contains exactly one row per benchmarked dataset (four rows for the four published datasets)
- All ARI values fall in the expected high range (close to 1), not near 0
- scid_multiclass returns all four objects (labels, markers, weights, scores) with non-empty labels for every target cell
- Gene signatures were extracted with negative markers retained (only_pos = FALSE), matching scID v2.0.0 behavior

## Limitations

- Self-mapping inflates accuracy relative to true cross-dataset mapping because reference and target share batch structure; Figure 1D values should not be read as cross-dataset generalization
- scID relies on good gene signatures: 'If neither of the two data is well clustered and contains good gene signatures, scID is not a recommended approach for identification of equivalent cell types.'
- Imputation of gene expression for shallow target data is deferred to future work and not implemented or evaluated
- Weights estimated from the target are recommended when reference and target have strong batch effect; in self-mapping this setting approximates the paper's recommended configuration but does not test batch robustness

## Evidence

- [results] In self-mapping (same dataset as reference and target) of four published datasets spanning different tissues and technologies, scID's accuracy as assessed by Adjusted Rand Index was relatively high (Figure 1D).: [evidence span withheld — anchor status: not_found]
- [readme] negative markers: [evidence span withheld — anchor status: external_doc]
