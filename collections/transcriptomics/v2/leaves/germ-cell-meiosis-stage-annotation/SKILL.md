---
name: germ-cell-meiosis-stage-annotation
description: Use when a germ-cell query dataset (e.g. in vitro induced meiosis, Ythdc2-KO
  or Zfp541-KO germ cells) must be assigned meiotic stages by mapping onto the mTCA
  reference, and day-resolved meiotic entry rates or developmental arrest must be
  quantified.
when_to_use_negative:
- Query cells contain novel states absent from the mTCA reference — fixed-model projections
  may be misleading; compare unsupervised clustering to mapping results to detect
  inconsistencies
- The task is cross-platform projection (e.g. Smart-seq2 query onto a 10x-built reference),
  where ProjectSVR performance declines due to differing UMI distributions and sequencing
  depth
- The reference atlas was built with a data-tailored integration strategy incompatible
  with ProjectSVR's integration-agnostic gene set activity score approach
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ProjectSVR
  role: Reference mapping of query germ cells onto the mTCA atlas via MapQuery, label
    transfer (LabelTransfer), and majority voting (MajorityVote)
  repo: https://github.com/JarningGau/ProjectSVR
- name: Seurat
  role: Seurat object handling for query datasets, subsetting on mapQ.p.adj < 0.1,
    and FeaturePlot/DimPlot visualization on ref.umap
  repo: https://github.com/satijalab/seurat
- name: AUCell
  role: Computes gene set activity scores (0–1) from rank expression that ProjectSVR's
    SVR models map to reference embeddings
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:a65bd827399c2233875a50c6dbaacdda5103b82438c15e12707545be1f8a77f4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jarninggau/ProjectSVR
  repo_url: https://github.com/JarningGau/ProjectSVR
  tools:
  - AUCell
  - ProjectSVR
  - Seurat
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# germ-cell-meiosis-stage-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates germ-cell meiosis stages (e.g. leptotene through pachytene, m11–m15) in a query scRNA-seq dataset by projecting cells onto the mouse testicular cell atlas (mTCA) with ProjectSVR, transferring reference labels, majority-voting them over k-means over-clustering, and assessing meiotic progression by treatment day.

## When to use

Use when a germ-cell query dataset (e.g. in vitro induced meiosis, Ythdc2-KO or Zfp541-KO germ cells) must be assigned meiotic stages by mapping onto the mTCA reference, and day-resolved meiotic entry rates or developmental arrest must be quantified.

## When NOT to use

- Query cells contain novel states absent from the mTCA reference — fixed-model projections may be misleading; compare unsupervised clustering to mapping results to detect inconsistencies
- The task is cross-platform projection (e.g. Smart-seq2 query onto a 10x-built reference), where ProjectSVR performance declines due to differing UMI distributions and sequencing depth
- The reference atlas was built with a data-tailored integration strategy incompatible with ProjectSVR's integration-agnostic gene set activity score approach

## Inputs

- Pre-built mTCA reference model file (model.mTCA.rds, Zenodo record 8350732)
- Query germ-cell dataset in Seurat slim .qs format (e.g. query_NRRA_induced_meiotic_cells.seurat.slim.qs, Zenodo record 8350748)
- Reference cell-type label column (cell_type) and treatment-day metadata on query cells

## Outputs

- Projected query cells on ref.umap with mapQ.p.adj mapping-quality scores and transferred cell-type labels
- Majority-voted meiosis stage annotations per cell (leptotene–pachytene stages m11–m15)
- PlotProjection UMAP figures split by treatment day and AlluviaPlot day-resolved meiosis-progression summaries
- Validation FeaturePlots of Vim and Ddx4 on ref.umap

## How to apply

Download the pre-built mTCA reference model (model.mTCA.rds from Zenodo record 8350732) and the query dataset (e.g. query_NRRA_induced_meiotic_cells.seurat.slim.qs from Zenodo record 8350748), then project query cells with ProjectSVR::MapQuery (add.map.qual = TRUE, ncores = 10) onto ref.umap. Filter poorly mapped cells with mapQ.p.adj < 0.1 (assessed via MapQCPlot at p.adj.cutoff = 0.1) before transferring reference cell-type labels with ProjectSVR::LabelTransfer (ref.label.col = 'cell_type'). Refine labels with MajorityVote (k = 100, min.prop = 0.3) applied over k-means over-clustering of the gene set score matrix, then visualize with PlotProjection (ref.color.by = 'cell_type', ref.size = 0.5, ref.alpha = 0.3) and summarize meiosis progression by treatment day with AlluviaPlot filled by the majority-voted predicted cell type. Validate assignments with FeaturePlot of the pan-somatic marker Vim and pan-germ-cell marker Ddx4 on ref.umap, flagging wrong assignments such as m9-PGC_Piwil4_arrest; the day-resolved alluvia restricted to meiotic stages (m11–m15) should reproduce the reported ~4–5% germ-cell meiotic entry after 3 days of NRRA treatment.

## Related tools

- **ProjectSVR** (Reference mapping of query germ cells onto the mTCA atlas via MapQuery, label transfer (LabelTransfer), and majority voting (MajorityVote)) — https://github.com/JarningGau/ProjectSVR
- **Seurat** (Seurat object handling for query datasets, subsetting on mapQ.p.adj < 0.1, and FeaturePlot/DimPlot visualization on ref.umap) — https://github.com/satijalab/seurat
- **AUCell** (Computes gene set activity scores (0–1) from rank expression that ProjectSVR's SVR models map to reference embeddings)

## Examples

```
seu.q <- ProjectSVR::MapQuery(seu.q, model.mTCA, ref.umap, add.map.qual = TRUE, ncores = 10); seu.q <- subset(seu.q, mapQ.p.adj < 0.1); seu.q <- ProjectSVR::LabelTransfer(seu.q, ref.label.col = "cell_type")
```

## Evaluation signals

- MapQCPlot at p.adj.cutoff = 0.1 shows a clear separation of well- vs badly-mapped cells; retained cells satisfy mapQ.p.adj < 0.1
- FeaturePlot of pan-somatic Vim and pan-germ-cell Ddx4 on ref.umap shows no misassignments such as m9-PGC_Piwil4_arrest receiving germ-cell labels
- Day-resolved AlluviaPlot restricted to meiotic stages (m11–m15) reproduces the reported ~4–5% of germ cells entering meiosis after 3 days of NRRA treatment
- For the perturbation benchmark, X-linked gene repression, pachytene-program activity, and leptotene-program status along pseudotime match the published outcome (e.g. NRRA-induced meiosis fails to reach pachytene)
- MajorityVote parameters (k = 100, min.prop = 0.3) over k-means over-clustering of the gene set score matrix yield stable, contiguous label blocks on ref.umap

## Limitations

- Independent SVR models per UMAP dimension neglect correlations among embedding dimensions
- Performance declines when projecting across platforms (e.g. Smart-seq2 to 10x) due to differing UMI distributions and sequencing depth
- Projections may be misleading for query cells containing novel states absent in the reference; users must compare unsupervised clustering to mapping results
- Reference building time is ~O(Nc^2), requiring subsampling to 4000–10,000 cells

## Evidence

- [readme] pre-built mTCA model download: [evidence span withheld — anchor status: external_doc]
