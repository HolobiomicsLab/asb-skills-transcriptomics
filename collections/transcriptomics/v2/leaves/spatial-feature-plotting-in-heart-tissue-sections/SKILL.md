---
name: spatial-feature-plotting-in-heart-tissue-sections
description: Use when TF expression in spatial transcriptomics spots is too sparse
  to visualize directly and you need to map a regulome-level activity (aggregated
  GRN target genes) onto heart tissue sections to localize fibrotic regulatory programs.
when_to_use_negative:
- Direct TF expression in the spatial object is sufficiently dense to plot without
  regulome aggregation — plotting individual genes would suffice.
- The spatial data is not myocardial/fibrotic tissue or lacks a normalized spatial
  assay and spatial coordinates in a Seurat object.
- No inferred GRN edge list (e.g. df.grn2/final_grn.Rds) exists, so no target-gene
  sets are available to aggregate into regulome scores.
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3173
tools:
- name: scMEGA
  role: Provides GRNSpatialPlot to overlay per-spot TF regulome scores on spatial
    tissue coordinates
  repo: https://github.com/CostaLab/scMEGA
- name: Seurat
  role: Holds and normalizes the myocardial infarction spatial transcriptomics object
    and renders spatial feature plots
  repo: github.com/satijalab/seurat (branch seurat5)
- name: R
  role: Implementation language for the whole spatial regulome plotting workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:56f4e090abbab43baa35d75000089237864be76f50c7c54fead3338413a72697
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/scMEGA
  tools:
  - R
  - Seurat
  - scMEGA
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Spatial feature plotting in heart tissue sections

## Summary

Overlay aggregated TF regulome scores (e.g. NR3C2, RUNX1 target-gene mean expression) onto the spatial coordinates of the Kuppe et al. 2022 human myocardial infarction spatial transcriptomics object using scMEGA's GRNSpatialPlot, since direct TF expression is too sparse in spatial spots.

## When to use

Use when TF expression in spatial transcriptomics spots is too sparse to visualize directly and you need to map a regulome-level activity (aggregated GRN target genes) onto heart tissue sections to localize fibrotic regulatory programs.

## When NOT to use

- Direct TF expression in the spatial object is sufficiently dense to plot without regulome aggregation — plotting individual genes would suffice.
- The spatial data is not myocardial/fibrotic tissue or lacks a normalized spatial assay and spatial coordinates in a Seurat object.
- No inferred GRN edge list (e.g. df.grn2/final_grn.Rds) exists, so no target-gene sets are available to aggregate into regulome scores.

## Inputs

- Seurat spatial object of human myocardial infarction spatial transcriptomics (Kuppe et al. 2022) with normalized expression assay
- Fibroblast GRN edge list df.grn2 / final_grn.Rds (edges filtered at correlation > 0.4)
- scMEGA trajectory object with the target assay
- TF regulon definitions (target-gene sets per TF, e.g. NR3C2, RUNX1) from the inferred GRN

## Outputs

- Spatial feature plots overlaying per-spot TF regulome scores on heart tissue sections (one per TF, e.g. NR3C2 and RUNX1)
- Comparative view of mutually exclusive regulome gradients within fibrotic zones (COL1A1/POSTN/FN1-positive regions)

## How to apply

Load the inferred fibroblast GRN edge list (df.grn2, filtered at correlation > 0.4 and stored as final_grn.Rds) together with the scMEGA trajectory object carrying the target assay. Load the Kuppe et al. 2022 myocardial infarction spatial transcriptomics data into a Seurat spatial object and normalize its expression assay. For each TF of interest (e.g. NR3C2, RUNX1), aggregate the mean expression of its GRN target genes per spatial spot to compute a regulome score, bypassing the sparse direct TF signal. Call scMEGA's GRNSpatialPlot(object, assay, df.grn, tf.use, min.cutoff, max.cutoff, vis.option) to render the regulome score over tissue spatial coordinates. Compare plots across TFs within fibrotic regions identified by COL1A1/POSTN/FN1 positivity to reveal mutually exclusive spatial gradients of regulome activity.

## Related tools

- **scMEGA** (Provides GRNSpatialPlot to overlay per-spot TF regulome scores on spatial tissue coordinates) — https://github.com/CostaLab/scMEGA
- **Seurat** (Holds and normalizes the myocardial infarction spatial transcriptomics object and renders spatial feature plots) — github.com/satijalab/seurat (branch seurat5)
- **R** (Implementation language for the whole spatial regulome plotting workflow)

## Examples

```
GRNSpatialPlot(object = obj.spatial, assay = "Spatial", df.grn = df.grn2, tf.use = "NR3C2", min.cutoff = 0, max.cutoff = 1, vis.option = "point")
```

## Evaluation signals

- Rendered NR3C2 and RUNX1 regulome plots show spatially disjoint (mutually exclusive) gradients across the fibrotic zone, as judged by visual inspection.
- Regulome signal localizes to spots positive for fibrosis markers COL1A1, POSTN, and FN1.
- Per-spot regulome scores fall within the min.cutoff–max.cutoff range supplied to GRNSpatialPlot and color scales are populated (not empty/degenerate).
- GRN edges used for aggregation satisfy the correlation > 0.4 filter from df.grn2/final_grn.Rds.

## Limitations

- Direct TF expression is too sparse in spatial spots to detect, so scores must rely on aggregated GRN target genes rather than the TF itself.
- The GRN must first be inferred from multi-omic data (scMEGA trajectory object with target assay); this skill does not infer the GRN itself.
- Software/package versions are not specified in the tutorials (only sessionInfo() is referenced), so exact reproducibility may vary.

## Evidence

- [other] regulome plotting workflow: [evidence span withheld — anchor status: not_found]
- [methods] fibrotic marker evidence: "some cells highly express COL1A1, POSTN, and FN1 and they could be terminally differentiated myofibroblasts"
- [abstract] case study scope: "We provide a case study on gene regulatory networks controlling myofibroblast activation in human myocardial infarction."
- [other] sparsity motivation: [evidence span withheld — anchor status: not_found]
- [readme] tutorials availability: "We provided the following tutorials to show how to use scMEGA to build GRN by using single-cell multiomics/multimodal data"
