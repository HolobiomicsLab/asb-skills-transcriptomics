---
name: spatial-transcriptomics-regulome-scoring
description: Use when TF expression is too sparse in spatial transcriptomics spots
  (e.g. Kuppe et al. 2022 myocardial infarction data) to map TF activity directly,
  and an inferred GRN edge list of TF-target links is available to build target-gene
  regulome scores per spot.
when_to_use_negative:
- Direct TF expression in the spatial spots is sufficiently abundant to visualize
  without regulome aggregation
- No TF-target edge list (GRN) has been inferred — the scores depend entirely on scMEGA-derived
  target-gene sets
- The dataset is single-cell multiomics without spatial coordinates; use scMEGA's
  chromVAR-based TF activity estimation instead (AddMotifs + RunChromVAR)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scMEGA
  role: Provides GRNSpatialPlot to overlay per-TF regulome scores onto spatial tissue
    coordinates; the GRN itself was inferred with scMEGA's enhancer-based pipeline
  repo: https://github.com/CostaLab/scMEGA
- name: Seurat
  role: Loads and normalizes the myocardial infarction spatial transcriptomics data
    as a Seurat spatial object before regulome scoring
  repo: https://github.com/satijalab/seurat (branch seurat5)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:047676b4b6a9952ea40f750fc20ceeba021a2c50524f375540dbacd6b78cfcb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/scMEGA
  tools:
  - R
  - Seurat
  - scMEGA
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# spatial-transcriptomics-regulome-scoring

## Summary

Scores transcription factor (TF) regulomes across spatial transcriptomics spots by aggregating mean expression of each TF's GRN target genes (from a scMEGA-inferred fibroblast GRN) and overlaying the scores on tissue coordinates, enabling TF activity mapping when direct TF expression is too sparse to detect.

## When to use

Use when TF expression is too sparse in spatial transcriptomics spots (e.g. Kuppe et al. 2022 myocardial infarction data) to map TF activity directly, and an inferred GRN edge list of TF-target links is available to build target-gene regulome scores per spot.

## When NOT to use

- Direct TF expression in the spatial spots is sufficiently abundant to visualize without regulome aggregation
- No TF-target edge list (GRN) has been inferred — the scores depend entirely on scMEGA-derived target-gene sets
- The dataset is single-cell multiomics without spatial coordinates; use scMEGA's chromVAR-based TF activity estimation instead (AddMotifs + RunChromVAR)

## Inputs

- GRN edge list (df.grn2 / final_grn.Rds) filtered at correlation > 0.4 linking TFs to target genes
- scMEGA trajectory Seurat object with target assay
- Spatial transcriptomics dataset (Kuppe et al. 2022 human myocardial infarction) as a Seurat spatial object with normalized expression assay
- Fibrotic-region marker genes (COL1A1, POSTN, FN1) for region annotation

## Outputs

- Per-spot regulome scores for each TF (e.g. NR3C2, RUNX1)
- Spatial overlay plots (GRNSpatialPlot) of regulome scores on tissue coordinates
- Comparative assessment of regulome gradients / mutual exclusivity across the fibrotic zone

## How to apply

Start from the inferred fibroblast GRN edge list (e.g. df.grn2 filtered at correlation > 0.4 and saved as final_grn.Rds) and the scMEGA trajectory object holding the target assay. Load the spatial transcriptomics data (Kuppe et al. 2022 human myocardial infarction) into a Seurat spatial object and normalize its expression assay. For each TF of interest (e.g. NR3C2, RUNX1), aggregate the mean expression of its GRN target genes per spatial spot to compute a regulome score, because direct TF expression is too sparse in spatial spots. Overlay each TF regulome score onto tissue spatial coordinates with the scMEGA function GRNSpatialPlot(object, assay, df.grn, tf.use, min.cutoff, max.cutoff, vis.option). Compare regulome plots within fibrotic regions identified by marker positivity for COL1A1/POSTN/FN1 (terminally differentiated myofibroblasts) to detect spatial gradients and mutually exclusive regulome expression.

## Related tools

- **scMEGA** (Provides GRNSpatialPlot to overlay per-TF regulome scores onto spatial tissue coordinates; the GRN itself was inferred with scMEGA's enhancer-based pipeline) — https://github.com/CostaLab/scMEGA
- **Seurat** (Loads and normalizes the myocardial infarction spatial transcriptomics data as a Seurat spatial object before regulome scoring) — https://github.com/satijalab/seurat (branch seurat5)

## Examples

```
GRNSpatialPlot(object = spatial.obj, assay = "Spatial", df.grn = final_grn, tf.use = "NR3C2", vis.option = "plot1")
```

## Evaluation signals

- Rendered NR3C2 and RUNX1 regulome plots display spatially disjoint (mutually exclusive) gradients across the fibrotic zone, consistent with the reported finding
- Regulome scores are computed only from target genes present in the GRN edge list filtered at correlation > 0.4
- Regulome enrichment localizes to spots positive for fibrotic markers COL1A1, POSTN, and FN1
- Each TF's score distribution across spots spans a visible gradient after applying GRNSpatialPlot's min.cutoff/max.cutoff
- Spatial coordinates in the plot match the Seurat spatial object's spot positions (no misregistration of scores)

## Limitations

- Direct TF expression is too sparse in spatial spots to detect, which is why target-gene aggregation is required; TFs with few high-confidence targets will yield noisy regulome scores
- Regulome quality depends on the upstream GRN inference; the reported case study is limited to fibroblast/myofibroblast GRNs in human myocardial infarction
- Assessment of mutual exclusivity in the reference workflow was by visual inspection of rendered spatial plots, not a quantitative statistical test

## Evidence

- [methods] GRNSpatialPlot invocation from task workflow: [evidence span withheld — anchor status: not_found]
- [methods] Correlation filter on GRN edges: [evidence span withheld — anchor status: not_found]
- [methods] Fibrotic marker definition: "some cells highly express COL1A1, POSTN, and 
FN1 and they could be terminally differentiated myofibroblasts"
- [methods] TF activity estimation rationale: "we first estimate an acitivty score for each TF in each cell. This is done by first performing motif matching and then computing deviation scores"
- [other] Spatial regulome finding: [evidence span withheld — anchor status: not_found]
