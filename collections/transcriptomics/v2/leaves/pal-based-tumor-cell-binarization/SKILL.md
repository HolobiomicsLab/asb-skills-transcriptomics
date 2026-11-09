---
name: pal-based-tumor-cell-binarization
description: Use when scRNA-seq tumor cells must be split into pathway-active vs pathway-inactive
  groups (e.g., G2M checkpoint-positive dividing TNBC cells) without clustering, so
  downstream per-group tests like a TYRO3 Wilcoxon rank-sum can be run.
when_to_use_negative:
- Data is targeted gene panel data with limited numbers of genes — GSDensity is explicitly
  not suitable for such data and no imputation is implemented
- The pathway gene set fails the coordination (KLD) test, since binarization is recommended
  only for coordinated gene sets
- PAL distribution of the pathway is visibly multi-modal (more than two modalities),
  so binary labeling would misrepresent pathway activity
tools:
- name: gsdensity
  role: Provides compute.mca, compute.nn.edges, run.rwr, and compute.cell.label for
    MCA co-embedding, graph construction, pathway-activity propagation, and PAL-based
    binarization
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: Supplies the MCA co-embedding approach that gsdensity's compute.mca is built
    on
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: Seurat
  role: Used for NormalizeData preprocessing of count matrices and for downstream
    FeaturePlot/DimPlot visualization of propagated scores and binarized labels
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:be3d5ee5925ce31159ab55d0400399a24cf8346c01186cb462cb3b17a22fd341
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - CelliD
  - GSDensity
  - Seurat
  - gsdensity
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# pal-based-tumor-cell-binarization

## Summary

Binarize single-cell pathway activity into 'positive'/'negative' labels using GSDensity's random-walk-with-restart propagation scores and PAL-based cell labeling, as demonstrated for classifying dividing TNBC tumor cells via the G2M checkpoint gene set in the GAS6-TYRO3 analysis.

## When to use

Use when scRNA-seq tumor cells must be split into pathway-active vs pathway-inactive groups (e.g., G2M checkpoint-positive dividing TNBC cells) without clustering, so downstream per-group tests like a TYRO3 Wilcoxon rank-sum can be run.

## When NOT to use

- Data is targeted gene panel data with limited numbers of genes — GSDensity is explicitly not suitable for such data and no imputation is implemented
- The pathway gene set fails the coordination (KLD) test, since binarization is recommended only for coordinated gene sets
- PAL distribution of the pathway is visibly multi-modal (more than two modalities), so binary labeling would misrepresent pathway activity

## Inputs

- Seurat-normalized scRNA-seq count matrix (e.g., GSE161529 TNBC-2/TNBC-5 samples)
- Curated pathway gene set as a character vector of gene symbols (e.g., G2M checkpoint genes)
- MCA co-embedding (coembed) and nearest-neighbor edge list (nn.use = 300)
- Cell type annotation restricting analysis to tumor cells

## Outputs

- Per-cell random-walk propagation probability vector (run.rwr scores)
- Per-cell binarized pathway-activity labels ('positive'/'negative') from compute.cell.label
- Sample-level tables such as tyro3_dividing_tumor_wilcoxon_results.csv with binarization labels and Wilcoxon p-values

## How to apply

Normalize the count matrix with Seurat NormalizeData, then co-embed cells and genes in the shared latent space with gsdensity's compute.mca (MCA embedding, inspired by CelliD). Build the cell-gene nearest-neighbor graph with compute.nn.edges using nn.use = 300, and run random walk with restart (run.rwr, restart = 0.75) using the pathway gene set (e.g., G2M checkpoint genes) as seeds restricted to tumor cells. Binarize the propagation probabilities with compute.cell.label (PAL-based binarization) to assign each cell a positive/negative pathway-activity label. Per the discussion, binarize only gene sets that pass the coordination test and visually inspect PAL distributions, since some pathways show more than two modalities. The resulting labels feed downstream per-group comparisons, e.g., TYRO3 Wilcoxon tests in TNBC-5 and the 8-sample TNBC cohort (reported p = 3.18e-8 and p = 0.039 for GSM4909284_TN-MH0114-T2).

## Related tools

- **gsdensity** (Provides compute.mca, compute.nn.edges, run.rwr, and compute.cell.label for MCA co-embedding, graph construction, pathway-activity propagation, and PAL-based binarization) — https://github.com/qingnanl/gsdensity
- **CelliD** (Supplies the MCA co-embedding approach that gsdensity's compute.mca is built on) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **Seurat** (Used for NormalizeData preprocessing of count matrices and for downstream FeaturePlot/DimPlot visualization of propagated scores and binarized labels)

## Examples

```
ce <- compute.mca(object = tnbc5); el <- compute.nn.edges(coembed = ce, nn.use = 300); cv <- run.rwr(el = el, gene_set = g2m_genes, cells = tumor_cells); cl <- compute.cell.label(cv)
```

## Evaluation signals

- Reproduced TYRO3 Wilcoxon p-value in TNBC-5 dividing vs other tumor cells matches the reported 3.18e-8 within numerical tolerance
- Per-sample G2M binarization in the 8-sample TNBC cohort reproduces p = 0.039 for GSM4909284_TN-MH0114-T2 (the sample with relatively high TYRO3, detected in 24% of tumor cells)
- All cells receive exactly one 'positive' or 'negative' label with no NAs, and labels align with the bimodal structure of the PAL distribution
- Nearest-neighbor graph built with nn.use = 300 and RWR restart = 0.75 as in the reported workflow; results recorded in tyro3_dividing_tumor_wilcoxon_results.csv

## Limitations

- MCA embedding is memory-intensive (~25 Gb for a 40,000-cell dataset)
- Some pathways show more than two modalities in PAL distributions; automatic binarization should be visually inspected and corrected as necessary
- Over- or under-correction of batch effects may introduce bias, requiring careful evaluation before pathway analysis
- Not applicable to targeted gene data with limited numbers of genes; gene imputation is suggested but not implemented

## Evidence

- [discussion] we also offer an option of PAL-based cell binarization for downstream analysis, as was demonstrated in the TNBC data analysis: "we also offer an option of PAL-based cell binarization for downstream analysis, as was demonstrated in the TNBC data analysis"
- [intro] GSDensity cluster-free, pathway-based classification of tumor cells and GAS6-TYRO3 association: "We demonstrate the usage of GSDensity in cluster-free, pathway-based classification of tumor cells and found an association between the GAS6-TYRO3 signaling and tumor cell division"
- [methods] nn.use = 300 graph construction parameter: "el <- compute.nn.edges(coembed = ce, nn.use = 300)"
- [readme] README description of the binarized label output: "# get a 'positive' or 'negative' label for each cell of this gene set"
- [discussion] recommendation to binarize only coordinated gene sets: "we recommend performing binarization on only gene sets that pass the coordination test"
