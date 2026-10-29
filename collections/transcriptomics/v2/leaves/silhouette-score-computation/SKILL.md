---
name: silhouette-score-computation
description: Use when comparing cross-platform scRNA-seq integration quality — e.g.
  after Seurat integration of platform-specific gene-by-barcode matrices (GBMs) with
  a reference GBM — by scoring cluster separation alongside kBET.
when_to_use_negative:
- Input is raw FASTQ or an unprocessed/unharmonised GBM — clusters must come from
  a completed Seurat integration of matched gene/barcode matrices before a Silhouette
  score is meaningful
- The question is upstream processing fidelity (e.g. FASTQ-to-count accuracy) rather
  than cross-platform integration quality — Silhouette alone does not measure read/UMI
  correctness
- Only one integration exists with no comparison candidate (e.g. no zUMIs counterpart),
  since the skill's value is the directional UniverSC-vs-platform-specific comparison
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: cluster (R package)
  role: Provides the silhouette() function used to compute the Silhouette score (version
    2.1.3) for each integrated dataset
- name: Seurat
  role: Runs the integration workflow that produces the integrated object and cluster
    assignments the Silhouette score is computed on
- name: kBET
  role: Complementary batch-effect metric (R package kBET, version 0.99.6) computed
    alongside the Silhouette score to evaluate integration quality
- name: UniverSC
  role: Upstream tool that processes the SmartSeq2 data with a single pipeline shared
    with the Chromium reference, yielding the integration expected to score higher
    (Silhouette ~0.43)
  repo: https://github.com/minoda-lab/universc
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34681-z
    title: A flexible cross-platform single-cell data processing pipeline
  source_content_hash: sha256:94a73e3525cb11c555dd67c60bf8ba95d6148ed32a46622c11b7c2575c73461e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/minoda-lab/universc
  tools:
  - Seurat
  - UniverSC
  - cluster
  - cluster (R package)
  - kBET
  - zUMIs
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# silhouette-score-computation

## Summary

Computes the Silhouette score (R package cluster, v2.1.3) to quantify the distinctness of clusters in a Seurat-integrated single-cell dataset, used here to judge whether SmartSeq2 mouse primary cell data processed by UniverSC integrates better with a Chromium reference than the same data processed by zUMIs.

## When to use

Use when comparing cross-platform scRNA-seq integration quality — e.g. after Seurat integration of platform-specific gene-by-barcode matrices (GBMs) with a reference GBM — by scoring cluster separation alongside kBET.

## When NOT to use

- Input is raw FASTQ or an unprocessed/unharmonised GBM — clusters must come from a completed Seurat integration of matched gene/barcode matrices before a Silhouette score is meaningful
- The question is upstream processing fidelity (e.g. FASTQ-to-count accuracy) rather than cross-platform integration quality — Silhouette alone does not measure read/UMI correctness
- Only one integration exists with no comparison candidate (e.g. no zUMIs counterpart), since the skill's value is the directional UniverSC-vs-platform-specific comparison

## Inputs

- Seurat-integrated object combining a candidate SmartSeq2 GBM (UniverSC- or zUMIs-processed) with the Chromium reference GBM of mouse primary cells
- GBMs harmonised to identical gene sets with intersecting barcodes and 0-UMI-filled missing genes
- Cluster/cell-type assignments for the integrated cells

## Outputs

- Mean Silhouette score per integration (e.g. 0.43 for UniverSC vs 0.36 for zUMIs)
- Rows appended to integration_scores.csv alongside kBET values for both integrations

## How to apply

First harmonise the GBMs being compared so they share identical gene sets: keep only barcodes found in both GBMs and add genes found in only one GBM with 0 UMIs assigned. Integrate each candidate GBM (e.g. UniverSC-processed and zUMIs-processed SmartSeq2, both generated against the full mouse genome GRCm39 with no downsampling) independently with the reference Chromium GBM using the Seurat integration workflow in R. On each integrated object, cluster the cells and compute the mean Silhouette score with the R package cluster (version 2.1.3), recording it alongside kBET for the same integration. Interpret higher Silhouette as more distinct clusters and better integration: in the paper, UniverSC-processed SmartSeq2 scored 0.43 versus 0.36 for zUMIs (Fig. 3B, C), mirroring the kBET direction (0.06 vs 0.11). Write both metrics for both integrations to integration_scores.csv so the UniverSC-vs-zUMIs direction can be validated against the published result.

## Related tools

- **cluster (R package)** (Provides the silhouette() function used to compute the Silhouette score (version 2.1.3) for each integrated dataset)
- **Seurat** (Runs the integration workflow that produces the integrated object and cluster assignments the Silhouette score is computed on)
- **kBET** (Complementary batch-effect metric (R package kBET, version 0.99.6) computed alongside the Silhouette score to evaluate integration quality)
- **UniverSC** (Upstream tool that processes the SmartSeq2 data with a single pipeline shared with the Chromium reference, yielding the integration expected to score higher (Silhouette ~0.43)) — https://github.com/minoda-lab/universc

## Examples

```
library(cluster); sil <- mean(silhouette(as.integer(Idents(integrated)), dist(Embeddings(integrated, reduction = "cca")))[, 3])
```

## Evaluation signals

- Directional check against the published result: UniverSC integration must show higher Silhouette (~0.43) than zUMIs (~0.36), matching the kBET direction (0.06 vs 0.11) in Fig. 3B, C
- Silhouette values fall in the [-1, 1] range, with higher values indicating more distinct clusters in the integrated data
- Both compared GBMs were harmonised to identical gene sets with intersecting barcodes and 0-UMI filling, so the two Silhouette scores are computed on comparable cell/gene spaces
- Scores are written to integration_scores.csv with both metrics for both integrations, allowing pairwise diff against the paper's reported values

## Limitations

- The Silhouette score is only one of two integration metrics used by the paper and must be interpreted jointly with kBET; a high Silhouette alone does not prove batch-effect removal
- The comparison is specific to 3' UMI-based processing — the paper notes that technologies for 5' scRNA-seq other than Chromium are not supported at the time of writing, limiting generalisation of the score comparisons
- Scores depend on reference choice and preprocessing: the paper used the full mouse genome GRCm39 with no downsampling, and harmonised GBMs (0-UMI gene filling, intersecting barcodes) — different settings will change the values

## Evidence

- [methods] Silhouette score calculated for integration evaluation: "To evaluate the quality of integration, kBET12 and Silhouette score13 were calculated for each case"
- [methods] Seurat integration of each SmartSeq2 GBM: "each version of SmartSeq2 GBM was integrated with the reference GBMs independently using Seurat"
- [methods] GBM harmonisation rule: "only barcodes found in both GBMs were kept, and genes only found in one GBM were added to the other with 0 UMIs assigned."
- [methods] Reference genome and no downsampling: "The full mouse genome (GRCm39) was used as the reference genome and no downsampling was performed for these datasets."
- [results] Expected UniverSC vs zUMIs direction: [evidence span withheld — anchor status: not_found]
