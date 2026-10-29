---
name: pal-distribution-antimode-binarization
description: Use when a pathway's per-cell activity scores (PAL from GSDensity run.rwr)
  form a multimodal distribution and cells must be split into positive/negative pathway
  states for downstream differential tests, without relying on clustering.
when_to_use_negative:
- The gene set failed the GSDensity coordination/KL-divergence test — the article
  recommends binarizing only gene sets that pass the coordination test
- Input is targeted gene data with limited numbers of genes (e.g. targeted panels)
  — GSDensity is designed for whole-transcriptomic data
- The PAL distribution is multimodal with more than two modalities and cannot be visually
  corrected, so a two-group split would misrepresent the pathway states
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GSDensity
  role: Provides run.rwr, compute.cell.label (antimode-based PAL binarization) and
    compute.kld coordination testing used in this skill
  repo: https://github.com/qingnanl/gsdensity
- name: Seurat
  role: Data loading/normalization and metadata storage of the binarized labels for
    downstream Wilcoxon comparisons
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:f68adbbceb5e06b9730eb43c631a98c967b3d166cebc12132759e6b79601131d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - GSDensity
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# pal-distribution-antimode-binarization

## Summary

Binarize single cells into pathway-positive and pathway-negative groups by thresholding the per-cell pathway activity level (PAL) distribution — computed via GSDensity random-walk propagation — at its antimode (multimode-based detection), as demonstrated on the Hallmark G2M checkpoint gene set in TNBC tumor cells.

## When to use

Use when a pathway's per-cell activity scores (PAL from GSDensity run.rwr) form a multimodal distribution and cells must be split into positive/negative pathway states for downstream differential tests, without relying on clustering.

## When NOT to use

- The gene set failed the GSDensity coordination/KL-divergence test — the article recommends binarizing only gene sets that pass the coordination test
- Input is targeted gene data with limited numbers of genes (e.g. targeted panels) — GSDensity is designed for whole-transcriptomic data
- The PAL distribution is multimodal with more than two modalities and cannot be visually corrected, so a two-group split would misrepresent the pathway states

## Inputs

- Per-cell pathway activity level (PAL) vector from run.rwr (label propagation probability per cell, e.g. cv)
- Seurat-normalized single-cell matrix (e.g. tumor-cell subset identified by copyKat from GSE148673)
- Gene set of interest (e.g. MSigDB Hallmark G2M checkpoint)
- Cell-gene nearest-neighbor edge list from compute.nn.edges

## Outputs

- Binary per-cell labels (pathway-positive / pathway-negative, e.g. G2M-positive vs G2M-negative cells)
- Label vector suitable for adding to Seurat metadata (e.g. meta.data$b_cell_activition_bin)
- Downstream differential expression comparison (e.g. Wilcoxon test of MKI67 between label groups)

## How to apply

First run the GSDensity coordination test (compute.kld) on the gene set and only binarize gene sets that pass it, since binarization is recommended solely for coordinated pathways. Compute per-cell pathway activity with run.rwr on the nearest-neighbor cell-gene graph (e.g. compute.nn.edges with nn.use = 300, restart = 0.75), then apply compute.cell.label to detect the antimode of the PAL distribution and assign each cell a positive or negative label. In the TNBC-1 application, this split G2M-positive from G2M-negative tumor cells, and MKI67 expression was then compared between groups with a Wilcoxon rank-sum test, reproducing p < 2.2e-16. Visually inspect the PAL distribution to validate the automatic antimode detection, because some pathways show more than two modalities and correction may be necessary.

## Related tools

- **GSDensity** (Provides run.rwr, compute.cell.label (antimode-based PAL binarization) and compute.kld coordination testing used in this skill) — https://github.com/qingnanl/gsdensity
- **Seurat** (Data loading/normalization and metadata storage of the binarized labels for downstream Wilcoxon comparisons)

## Examples

```
cv <- run.rwr(el = el, gene_set = gene.set.list[["HALLMARK_G2M_CHECKPOINT"]], cells = cells); cl <- compute.cell.label(cv)
```

## Evaluation signals

- Binarization is only performed for gene sets with compute.kld adjusted p-value < 0.05 (res$p.adj < 0.05)
- In the TNBC-1 case, the G2M-positive group shows significantly higher MKI67 than the G2M-negative group (Wilcoxon p < 2.2e-16)
- Visual inspection of the PAL distribution confirms the antimode coincides with the automatic split point
- Each cell receives exactly one of the two labels (positive/negative) added to Seurat metadata

## Limitations

- Manual visual inspection of PAL distributions is recommended because some pathways show more than two modalities and the automatic antimode can be wrong
- Binarization should only be applied to gene sets passing the coordination test; applying it to uncoordinated gene sets is not supported
- Over- or under-corrected batch effects may bias pathway activity scores and thus the binarization outcome

## Evidence

- [discussion] we also offer an option of PAL-based cell binarization for downstream analysis, as was demonstrated in the TNBC data analysis: "we also offer an option of PAL-based cell binarization for downstream analysis, as was demonstrated in the TNBC data analysis"
- [discussion] we recommend performing binarization on only gene sets that pass the coordination test: "we recommend performing binarization on only gene sets that pass the coordination test"
- [discussion] we recommend users to visually inspect the PAL distributions of important pathways to validate the automatic results: "we recommend users to visually inspect the PAL distributions of important pathways to validate the automatic results and perform correction as necessary."
- [methods] cl <- compute.cell.label(cv): "cl <- compute.cell.label(cv)"
- [readme] get a 'positive' or 'negative' label for each cell of this gene set: "# get a 'positive' or 'negative' label for each cell of this gene set"
