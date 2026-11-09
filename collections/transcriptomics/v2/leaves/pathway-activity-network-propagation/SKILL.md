---
name: pathway-activity-network-propagation
description: Use when single-cell or spatial transcriptomics data must be scored per
  cell for pathway activity without clustering — e.g., to fetch cells active for a
  gene set (Hallmark G2M, GO BP terms) or to weight cells for spatial KDE testing.
when_to_use_negative:
- Data are targeted-panel spatial assays with limited numbers of genes — GSDensity
  is 'not suitable for targeted gene data having limited numbers of genes' unless
  unmeasured genes are imputed, which is not implemented
- The gene set failed the coordination test (compute.kld adjusted p >= 0.05) — binarization
  should only be performed on gene sets that pass the coordination test
- Only relative pathway ranking speed on small cell numbers matters — AUCell was faster
  than GSDensity when the number of cells was small
edam_operation: http://edamontology.org/operation_3929
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: GSDensity
  role: Provides compute.mca, compute.nn.edges, run.rwr/run.rwr.list, and compute.cell.label
    to co-embed cells/genes, build the cell-gene graph, and propagate pathway activity
    per cell
  repo: https://github.com/qingnanl/gsdensity
- name: CelliD
  role: Inspires the MCA co-embedding step used before graph construction; requires
    the updated Bioconductor version for installation
  repo: https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- name: Seurat
  role: Loads and normalizes the scRNA-seq/spatial matrices (e.g., NormalizeData on
    pbmc3k; SeuratData stxBrain Visium) and stores PALs in cell metadata for plotting
- name: msigdbr
  role: Supplies the gene sets (GO:BP, KEGG, BIOCARTA, Hallmarks) converted to a named
    list for propagation seeds
provenance:
  source_task_ids:
  - task_007
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44206-x
    title: Pathway centric analysis for single-cell RNA-seq and spatial transcriptomics
      data with GSDensity
  source_content_hash: sha256:0f8ec88097e050216cabba058f33b9b2c2971f0ec8ebdac6b4ae89abd6ccc0b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qingnanl/gsdensity
  tools:
  - CelliD
  - GSDensity
  - RANN
  - Seurat
  - SeuratData
  - igraph
  - monocle3
  - msigdbr
  edam_operation: http://edamontology.org/operation_3929
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# pathway-activity-network-propagation

## Summary

Compute per-cell pathway activity likelihood scores (PALs) with GSDensity by seeding a random-walk-with-restart on a nearest-neighbor cell-gene graph built from the MCA co-embedding, enabling cluster-free, pathway-centric classification of single cells (e.g., G2M-positive tumor cells) and spatial pathway relevance weighting.

## When to use

Use when single-cell or spatial transcriptomics data must be scored per cell for pathway activity without clustering — e.g., to fetch cells active for a gene set (Hallmark G2M, GO BP terms) or to weight cells for spatial KDE testing.

## When NOT to use

- Data are targeted-panel spatial assays with limited numbers of genes — GSDensity is 'not suitable for targeted gene data having limited numbers of genes' unless unmeasured genes are imputed, which is not implemented
- The gene set failed the coordination test (compute.kld adjusted p >= 0.05) — binarization should only be performed on gene sets that pass the coordination test
- Only relative pathway ranking speed on small cell numbers matters — AUCell was faster than GSDensity when the number of cells was small

## Inputs

- Normalized cell-by-gene matrix (e.g., Seurat object: pbmc3k, TNBC-1 from GSE148673, stxBrain anterior1 Visium)
- Named list of gene sets (MSigDB GO:BP, KEGG, BIOCARTA, Hallmarks via msigdbr)
- MCA co-embedding from compute.mca
- Vector of cell names for label propagation
- Optionally: spatial coordinates or cluster/annotation metadata for compute.spatial.kld.df or compute.spec

## Outputs

- Named vector of per-cell pathway activity likelihood scores (PALs) from run.rwr/run.rwr.list
- Binarized positive/negative cell labels from compute.cell.label (antimode-thresholded PALs)
- Spatial relevance statistics (permutation-based KLD p-values) when PALs weight spatial KDE
- Downstream test results (e.g., Wilcoxon p-value for MKI67 between G2M+/G2M− cells)

## How to apply

First co-embed cells and genes with GSDensity's compute.mca (MCA, inspired by CelliD), then run compute.kld to confirm the gene set is coordinated (adjusted p < 0.05) before propagation. Build a nearest-neighbor cell-gene graph with compute.nn.edges (nn.use = 300, neighborhood size being an important parameter) and run run.rwr (or run.rwr.list for many gene sets) with default restart = 0.75, using pathway genes as seeds for random walks. For binary cell classification, threshold the PAL distribution at its antimode via compute.cell.label (multimode-based detection) — but only for gene sets passing the coordination test, and visually inspect PAL distributions since some pathways show more than two modalities. For spatial data, use the resulting PALs as cell weights in compute.spatial.kld.df, comparing pathway-activity-weighted vs equal-weighted 2D KDE with permutation-based null testing. Downstream validation can use Wilcoxon tests on marker genes between positive and negative cells (e.g., MKI67 between G2M+ and G2M− TNBC-1 tumor cells).

## Related tools

- **GSDensity** (Provides compute.mca, compute.nn.edges, run.rwr/run.rwr.list, and compute.cell.label to co-embed cells/genes, build the cell-gene graph, and propagate pathway activity per cell) — https://github.com/qingnanl/gsdensity
- **CelliD** (Inspires the MCA co-embedding step used before graph construction; requires the updated Bioconductor version for installation) — https://bioconductor.org/packages/devel/bioc/html/CelliD.html
- **Seurat** (Loads and normalizes the scRNA-seq/spatial matrices (e.g., NormalizeData on pbmc3k; SeuratData stxBrain Visium) and stores PALs in cell metadata for plotting)
- **msigdbr** (Supplies the gene sets (GO:BP, KEGG, BIOCARTA, Hallmarks) converted to a named list for propagation seeds)

## Examples

```
el <- compute.nn.edges(coembed = ce, nn.use = 300); cv <- run.rwr(el = el, gene_set = gene.set.list[["GOBP_B_CELL_ACTIVATION"]], cells = cells); cl <- compute.cell.label(cv)
```

## Evaluation signals

- Coordination test passes before propagation: selected gene sets have adjusted p-value < 0.05 from compute.kld (e.g., 97 KEGG/BIOCARTA pathways; GO BP terms in the Visium analysis)
- Binarized label validity: G2M-positive tumor cells show higher MKI67 expression than G2M-negative cells with Wilcoxon p < 2.2e-16 in the TNBC-1 dataset
- Graph parameter sanity: cell-gene nearest-neighbor graph built with nn.use = 300 and default restart = 0.75 matches the published workflows
- Spatial validation: pathway-activity-weighted vs equal-weighted 2D KDE yields significant permutation-based KLD p-values (adjusted p < 0.05), and the spatial relevance vs compute.spec specificity scatter matches the published vignette figure
- PAL distributions of important pathways are visually inspected to confirm the automatic antimode binarization is sensible

## Limitations

- Neighborhood size (nn.use) is an important parameter when constructing the cell-gene graph and affects results
- MCA embedding is memory-heavy (~25 Gb for a 40,000-cell dataset like hcabm40k)
- Some pathways show more than two modalities, so automatic antimode-based binarization requires manual visual inspection and correction
- Installation fails on macOS with M1 chips due to a CelliD/scuttle dependency; the 'anticlust' dependency requires an older version (e.g., 0.6.1)

## Evidence

- [intro] Pathway activity for each cell can be calculated using network propagation in a nearest-neighbor cell-gene graph, with pathway genes used as seeds for random walks: "Pathway activity for each cell can be calculated using network propagation in a nearest-neighbor cell-gene graph, with pathway genes used as seeds for random walks"
- [methods] el <- compute.nn.edges(coembed = ce, nn.use = 300): "el <- compute.nn.edges(coembed = ce, nn.use = 300)"
- [readme] cv <- run.rwr(el = el, gene_set = gene.set.list[["GOBP_B_CELL_ACTIVATION"]], cells = cells): "cv <- run.rwr(el = el, gene_set = gene.set.list[["GOBP_B_CELL_ACTIVATION"]], cells = cells)"
- [discussion] GSDensity uses network propagation to estimate the pathway relevance at single-cell level, while CelliD performs hypergeometric test: "GSDensity uses network propagation to estimate the pathway relevance at single-cell level, while CelliD performs hypergeometric test"
- [discussion] we recommend performing binarization on only gene sets that pass the coordination test: "we recommend performing binarization on only gene sets that pass the coordination test"
