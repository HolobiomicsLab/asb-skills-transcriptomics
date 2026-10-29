---
name: pancreas-islet-cell-type-annotation
description: Use when a reference scRNA-seq dataset with published cell-type labels
  (e.g. Segerstolpe Smart-Seq2 pancreas) must be mapped onto a target dataset from
  a different protocol (e.g. Baron CEL-Seq1 or Muraro CEL-Seq2 pancreas) with strong
  batch effect and library-depth asymmetry.
when_to_use_negative:
- When neither the reference nor the target dataset is well clustered and contains
  good gene signatures — the article states scID is then 'not a recommended approach
  for identification of equivalent cell types'
- When the input expression matrices are log-transformed rather than library-depth
  (TPM/CPM) normalized
- When reference and target come from the same platform with no batch effect and balanced
  cell/library sizes — target-based weight estimation and this benchmark setting were
  motivated by strong batch effect and library-size asymmetry
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_3308
tools:
- name: scID
  role: 'Primary classifier: extract reference cluster signatures (find_markers),
    estimate weights from target, and assign target cells via scid_multiclass under
    a Fisher''s Linear Discriminant Analysis-like framework'
  repo: github.com/BatadaLab/scID
- name: scmap
  role: Competing baseline method producing cell-type assignments for both pancreas
    target datasets
- name: CaSTLe
  role: Competing baseline method producing cell-type assignments for both pancreas
    target datasets
- name: scVI
  role: Competing integration-based baseline producing cell-type assignments for both
    pancreas target datasets
- name: R
  role: Runtime environment for scID; devtools::install_github installs the package
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.100914
    title: scID Uses Discriminant Analysis to Identify Transcriptionally Equivalent
      Cell Types across Single-Cell RNA-Seq Data with Batch Effect
  source_content_hash: sha256:cee53d4f30e491744ff6a27ba430ce5791fecdab189a24115b2fcbca3570fc57
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BatadaLab/scID
  tools:
  - CaSTLe
  - R
  - scID
  - scVI
  - scmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# pancreas-islet-cell-type-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmark skill for cross-platform classification of pancreas islet cell types: mapping cells in CEL-Seq1/CEL-Seq2 pancreas scRNA-seq targets to Smart-Seq2 reference clusters with scID's Fisher's Linear Discriminant Analysis-like framework, and comparing accuracy against scmap, CaSTLe, and scVI.

## When to use

Use when a reference scRNA-seq dataset with published cell-type labels (e.g. Segerstolpe Smart-Seq2 pancreas) must be mapped onto a target dataset from a different protocol (e.g. Baron CEL-Seq1 or Muraro CEL-Seq2 pancreas) with strong batch effect and library-depth asymmetry.

## When NOT to use

- When neither the reference nor the target dataset is well clustered and contains good gene signatures — the article states scID is then 'not a recommended approach for identification of equivalent cell types'
- When the input expression matrices are log-transformed rather than library-depth (TPM/CPM) normalized
- When reference and target come from the same platform with no batch effect and balanced cell/library sizes — target-based weight estimation and this benchmark setting were motivated by strong batch effect and library-size asymmetry

## Inputs

- Reference pancreas scRNA-seq count matrix (Segerstolpe Smart-Seq2) with published cell-type cluster labels, as an NxM gene-by-cell data frame
- Target pancreas scRNA-seq count matrices (Baron CEL-Seq1 and Muraro CEL-Seq2), as nxm gene-by-cell data frames
- List of cluster labels for the reference cells (reference_clusters)
- Published cell-type labels for the target datasets, used only for computing mapping accuracy

## Outputs

- scID_output$labels: named list of predicted cell-type labels for each target cell
- scID_output$markers: data frame of signature genes extracted from the reference clusters
- scID_output$weights: list of estimated discriminative weights for all cluster-specific genes
- scID_output$scores: data frame of target-cell scores for each reference cluster-specific geneset
- Per-method accuracy table/figure across the two target datasets, mirroring Figure 1E

## How to apply

Obtain the Segerstolpe Smart-Seq2 pancreas dataset (reference, with published labels) and the Baron CEL-Seq1 / Muraro CEL-Seq2 pancreas datasets (targets), then normalize count matrices to library-depth units (TPM or CPM), keeping the data non-log-transformed since scID requires column-normalized expression. Run scID's scid_multiclass with reference_clusters supplied, using find_markers to extract cluster-specific gene signatures from the reference and setting estimate_weights_from_target = TRUE so discriminative gene weights are estimated from the target cells — the article reports that when reference and target have strong batch effect, estimation of weights from the target appears more accurate, and v2.0.0 uses both positive and negative markers to improve classification when very similar cell types are present. Run the competing methods scmap, CaSTLe, and scVI on the same reference-target pairs, then compute per-method mapping accuracy against the published cell-type labels for each target. Validate by checking that scID ranks within the top two methods in both accuracy and consistency across the two CEL-Seq targets, mirroring the Figure 1E comparison.

## Related tools

- **scID** (Primary classifier: extract reference cluster signatures (find_markers), estimate weights from target, and assign target cells via scid_multiclass under a Fisher's Linear Discriminant Analysis-like framework) — github.com/BatadaLab/scID
- **scmap** (Competing baseline method producing cell-type assignments for both pancreas target datasets)
- **CaSTLe** (Competing baseline method producing cell-type assignments for both pancreas target datasets)
- **scVI** (Competing integration-based baseline producing cell-type assignments for both pancreas target datasets)
- **R** (Runtime environment for scID; devtools::install_github installs the package)

## Examples

```
install.packages('devtools'); devtools::install_github("BatadaLab/scID"); scID_output <- scID::scid_multiclass(target_gem, reference_gem, reference_clusters, estimate_weights_from_target = TRUE)
```

## Evaluation signals

- scID must rank within the top two methods in both accuracy and consistency across the Baron CEL-Seq1 and Muraro CEL-Seq2 targets, matching the qualitative outcome of Figure 1E
- Input expression matrices must be library-depth (TPM/CPM) normalized and non-log-transformed; log-transformed data violate scID's input requirement
- estimate_weights_from_target must be TRUE for this cross-platform pancreas mapping, since reference and target differ strongly in library depth and protocol
- Every target cell in scID_output$labels must carry a valid reference cluster label; use make_heatmap of average signature expression per reference/target cluster to sanity-check label agreement
- Competing methods (scmap, CaSTLe, scVI) must be run on identical reference-target pairs with accuracy computed against the same published labels for a fair comparison

## Limitations

- scID is not recommended when neither dataset is well clustered or contains good gene signatures
- Imputation of gene expression for shallow target data was deferred to future work and is not implemented or evaluated in this method
- scID assumes no explicit corrections for technical and biological variation; over-correction of batch effect is avoided by design, but performance depends on discriminative gene signatures being extractable
- Under strong batch effect, weight estimation from the reference rather than the target may be less accurate

## Evidence

- [readme] Negative-marker improvement in v2.0.0: [evidence span withheld — anchor status: external_doc]
