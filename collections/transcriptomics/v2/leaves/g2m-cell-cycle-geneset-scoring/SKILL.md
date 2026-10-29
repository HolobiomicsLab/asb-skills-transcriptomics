---
name: g2m-cell-cycle-geneset-scoring
description: Use when scRNA-seq tumor cell data (e.g., GSE161529 TNBC count matrices)
  must be classified into dividing vs non-dividing cells via the G2M checkpoint gene
  set without clustering, to support pathway-associated comparisons such as TYRO3
  expression.
when_to_use_negative:
- Data is targeted gene panel with a limited number of genes — GSDensity is not suitable
  for targeted gene data since MCA co-embedding and density estimation require transcriptome-wide
  measurements
- Input is bulk RNA-seq or pre-computed feature tables rather than cell-by-gene scRNA-seq
  matrices, where cell-gene co-embedding is undefined
- A single sample/cell type has too few tumor cells for a meaningful nearest-neighbor
  graph or Wilcoxon comparison between dividing and other cells
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: GSDensity
  role: Provides compute.mca, compute.nn.edges, run.rwr, compute.cell.label for MCA
    co-embedding, graph propagation, and PAL-based pathway binarization
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: Supplies the MCA co-embedding approach underlying compute.mca; requires the
    Bioconductor devel version of CellID
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: Seurat
  role: Data container and normalization (NormalizeData) of TNBC count matrices before
    embedding
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:2571285d8a1cd08c25dd254bf165eed06dfc5dbd8388ae679eb0405d716b313a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - CelliD
  - GSDensity
  - Seurat
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# g2m-cell-cycle-geneset-scoring

## Summary

Scores and binarizes G2M checkpoint pathway activity per tumor cell using GSDensity's MCA co-embedding and random-walk-with-rest propagation, enabling cluster-free identification of actively dividing TNBC tumor cells for downstream differential tests (e.g., TYRO3).

## When to use

Use when scRNA-seq tumor cell data (e.g., GSE161529 TNBC count matrices) must be classified into dividing vs non-dividing cells via the G2M checkpoint gene set without clustering, to support pathway-associated comparisons such as TYRO3 expression.

## When NOT to use

- Data is targeted gene panel with a limited number of genes — GSDensity is not suitable for targeted gene data since MCA co-embedding and density estimation require transcriptome-wide measurements
- Input is bulk RNA-seq or pre-computed feature tables rather than cell-by-gene scRNA-seq matrices, where cell-gene co-embedding is undefined
- A single sample/cell type has too few tumor cells for a meaningful nearest-neighbor graph or Wilcoxon comparison between dividing and other cells

## Inputs

- Seurat-normalized scRNA-seq count matrices (e.g., GSE161529 TNBC-2 and TNBC-5, and the 8-sample published TNBC cohort)
- G2M checkpoint gene set (seed gene list, e.g., from MSigDB/Seurat cell cycle sets)
- Tumor cell barcodes/annotations per sample (e.g., GSM4909284_TN-MH0114-T2)

## Outputs

- Per-cell G2M pathway propagation probabilities (run.rwr cv vector)
- Binarized dividing vs other cell labels per sample (compute.cell.label cl)
- Per-sample TYRO3 Wilcoxon rank-sum p-values (e.g., p = 3.18e-8 in TNBC-5; p = 0.039 for GSM4909284_TN-MH0114-T2) written to tyro3_dividing_tumor_wilcoxon_results.csv

## How to apply

Normalize the Seurat object with NormalizeData, then co-embed cells and genes into a shared latent space using compute.mca (MCA embedding from CelliD, ~25 Gb memory for a 40k-cell dataset). Build a nearest-neighbor cell-gene graph with compute.nn.edges (nn.use = 300) and seed it with the G2M checkpoint gene set in run.rwr using the default restart probability of 0.75. Binarize the returned propagation probabilities with compute.cell.label (PAL-based binarization) to assign dividing vs other tumor cell labels; optionally inspect PAL distributions since some pathways show more than two modalities. Restrict to tumor cells (e.g., TNBC-5 or the 8-sample published cohort) and run a Wilcoxon rank-sum test of the target gene (TYRO3) between dividing and other cells, writing per-sample labels and p-values to tyro3_dividing_tumor_wilcoxon_results.csv.

## Related tools

- **GSDensity** (Provides compute.mca, compute.nn.edges, run.rwr, compute.cell.label for MCA co-embedding, graph propagation, and PAL-based pathway binarization) — https://github.com/qingnanl/gsdensity
- **CelliD** (Supplies the MCA co-embedding approach underlying compute.mca; requires the Bioconductor devel version of CellID) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **Seurat** (Data container and normalization (NormalizeData) of TNBC count matrices before embedding)

## Examples

```
ce <- compute.mca(object = tnbc5); el <- compute.nn.edges(coembed = ce, nn.use = 300); cv <- run.rwr(el = el, gene_set = g2m_genes, cells = tumor_cells); cl <- compute.cell.label(cv)
```

## Evaluation signals

- Wilcoxon p-value of TYRO3 between dividing and other tumor cells in TNBC-5 reproduces the reported 3.18e-8 within numerical tolerance
- Wilcoxon p-value for sample GSM4909284_TN-MH0114-T2 reproduces the reported 0.039 within tolerance, with TYRO3 detected in ~24% of its tumor cells
- compute.nn.edges graph built with nn.use = 300 over the compute.mca co-embedding; run.rwr run with default restart = 0.75 using the G2M gene set as seeds
- All per-sample binarization labels and TYRO3 Wilcoxon p-values present in tyro3_dividing_tumor_wilcoxon_results.csv
- PAL distributions of the G2M pathway visually inspected to validate automatic binarization

## Limitations

- MCA embedding is memory-heavy (~25 Gb for a 40,000-cell dataset)
- Neighborhood size (nn.use) is an important parameter when constructing the cell-gene graph
- PAL-based automatic binarization should be visually validated; some pathways show more than two modalities
- Over- or under-correction of batch effects may bias pathway analysis; gsdensity is not suitable for targeted gene data with limited genes

## Evidence

- [intro] Pathway activity for each cell can be calculated using network propagation in a nearest-neighbor cell-gene graph, with pathway genes used as seeds for random walks: "Pathway activity for each cell can be calculated using network propagation in a nearest-neighbor cell-gene graph, with pathway genes used as seeds for random walks"
- [discussion] we also offer an option of PAL-based cell binarization for downstream analysis, as was demonstrated in the TNBC data analysis: "we also offer an option of PAL-based cell binarization for downstream analysis, as was demonstrated in the TNBC data analysis"
- [methods] el <- compute.nn.edges(coembed = ce, nn.use = 300): "el <- compute.nn.edges(coembed = ce, nn.use = 300)"
- [intro] We demonstrate the usage of GSDensity in cluster-free, pathway-based classification of tumor cells and found an association between the GAS6-TYRO3 signaling and tumor cell division: "We demonstrate the usage of GSDensity in cluster-free, pathway-based classification of tumor cells and found an association between the GAS6-TYRO3 signaling and tumor cell division"
- [discussion] we recommend users to visually inspect the PAL distributions of important pathways to validate the automatic results and perform correction as necessary.: "we recommend users to visually inspect the PAL distributions of important pathways to validate the automatic results and perform correction as necessary."
- [discussion] The largest memory cost was in the MCA embedding step, with about 25 Gb for a medium sized dataset: "The largest memory cost was in the MCA embedding step, with about 25 Gb for a medium sized dataset"
- [readme] cv <- run.rwr(el = el, gene_set = gene.set.list[["GOBP_B_CELL_ACTIVATION"]], cells = cells): "cv <- run.rwr(el = el, gene_set = gene.set.list[["GOBP_B_CELL_ACTIVATION"]], cells = cells)
# get a 'positive' or 'negative' label for each cell of this gene set
cl <- compute.cell.label(cv)"
