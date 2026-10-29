---
name: ligand-receptor-interaction-detection-in-spatial-transcriptomics
description: Use when seq-based ST data (e.g. 10x Visium) have been resolved to single-cell
  resolution via SpatialScope decomposition and you need to identify significant ligand-receptor
  pairs between spatially co-located cell types, such as EC–SMC signaling in a vascular
  region of interest.
when_to_use_negative:
- The ST data are still at spot level (Visium ~55 μm spots containing multiple cells)
  without prior single-cell decomposition, since communication analysis on mixed spots
  is unreliable
- You only need spot-level cell-type proportions (deconvolution); RCTD/Cell2location/CARD
  suffice and no ligand-receptor testing is required
- Image-based ST data already at single-cell resolution but with only a few hundred
  genes, where the whole-transcriptome ligand-receptor database cannot be evaluated
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SpatialScope
  role: Decomposes spot-level Visium data to single-cell resolution via score-based
    generative model and annealed Langevin dynamics, producing the single-cell expression
    input for communication analysis
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: Giotto
  role: Runs spatCellCellcom ligand-receptor detection on the decomposed cells with
    a Delaunay-triangulation spatial network and provides the 876 ligand/receptor
    training genes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:b32f405ef73c45773678056a9ddf614755650ab6377b16254ce6596deda9d06e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - Giotto
  - RCTD
  - SpatialScope
  - StarDist
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ligand-receptor-interaction-detection-in-spatial-transcriptomics

## Summary

Detect significant ligand-receptor signaling pairs between cell types located in a spatial transcriptomics dataset by running Giotto's spatCellCellcom on SpatialScope single-cell-resolution output, using a Delaunay-triangulation spatial network and strict significance/expression filters. This makes spot-level Visium data usable for cell-cell communication analysis that the original ~55 μm spot resolution cannot support.

## When to use

Use when seq-based ST data (e.g. 10x Visium) have been resolved to single-cell resolution via SpatialScope decomposition and you need to identify significant ligand-receptor pairs between spatially co-located cell types, such as EC–SMC signaling in a vascular region of interest.

## When NOT to use

- The ST data are still at spot level (Visium ~55 μm spots containing multiple cells) without prior single-cell decomposition, since communication analysis on mixed spots is unreliable
- You only need spot-level cell-type proportions (deconvolution); RCTD/Cell2location/CARD suffice and no ligand-receptor testing is required
- Image-based ST data already at single-cell resolution but with only a few hundred genes, where the whole-transcriptome ligand-receptor database cannot be evaluated

## Inputs

- SpatialScope Decomposition output with posterior-mean single-cell expression (generated_cells.h5ad)
- Cell type labels per decomposed cell (CellTypeLabel_nu10.csv / sp_adata.h5ad)
- Giotto ligand-receptor database (876 ligands/receptors) added to training genes

## Outputs

- Filtered table of significant ligand-receptor pairs per cell-type pair (p.adj < 0.25, abs(log2fc) > 0.1, lig_nr > 10, rec_nr > 10, lig_expr > 0.5, rec_expr > 0.5)
- Spatially resolved cell-cell communication map over the Delaunay spatial network

## How to apply

First obtain posterior-mean single-cell expression (e.g. generated_cells.h5ad) from the SpatialScope Decomposition step, since Visium's 55 μm spots mix multiple cells and limit communication analysis. Load the decomposed cells into Giotto, add the 876 Giotto ligands/receptors to the training genes, and build a Delaunay-triangulation spatial network to define neighboring cells. Run spatCellCellcom on this network, then filter candidate pairs to reliable signals with p.adj < 0.25, abs(log2fc) > 0.1, lig_nr > 10, rec_nr > 10, lig_expr > 0.5 and rec_expr > 0.5. On the human heart Visium data this procedure recovered the EC–SMC pairs JAG1-NOTCH3, DLL4-NOTCH3, SERPINE2-LRP1 and RTN4-LINGO1 in the 331-spot vascular ROI.

## Related tools

- **SpatialScope** (Decomposes spot-level Visium data to single-cell resolution via score-based generative model and annealed Langevin dynamics, producing the single-cell expression input for communication analysis) — https://github.com/YangLabHKUST/SpatialScope
- **Giotto** (Runs spatCellCellcom ligand-receptor detection on the decomposed cells with a Delaunay-triangulation spatial network and provides the 876 ligand/receptor training genes)

## Examples

```
R: spatCellCellcom(gobject, spatial_network_name = "Delaunay_network", cluster_column_used = "cell_types")  # then filter: p.adj < 0.25, abs(log2fc) > 0.1, lig_nr > 10, rec_nr > 10, lig_expr > 0.5 & rec_expr > 0.5
```

## Evaluation signals

- Filtered LR pair list is non-empty only after applying all six thresholds (p.adj < 0.25, abs(log2fc) > 0.1, lig_nr > 10, rec_nr > 10, lig_expr > 0.5, rec_expr > 0.5)
- Recovered pairs include the expected EC–SMC interactions JAG1-NOTCH3, DLL4-NOTCH3, SERPINE2-LRP1 and RTN4-LINGO1 in the 331-spot heart vascular ROI
- Interacting cell-type pairs are spatially adjacent in the Delaunay network (e.g. endothelial cells and SMCs at the central vascular structure, ~18 spots)
- Ligand and receptor expression values in the filtered results exceed 0.5 and each partner is supported by more than 10 cells (lig_nr, rec_nr > 10)

## Limitations

- Visium spot size (~55 μm) means a spot often contains multiple cells, so communication analysis requires prior single-cell resolution; platform effects must be corrected or downstream results are unsatisfactory
- SpatialScope decomposes only the genes present in the ST data, so LR detection is limited to transcriptome-wide genes measurable in the seq-based assay
- Giotto filtering with p.adj < 0.25 is a permissive threshold; significant pairs should be validated against known biology (e.g. arterial TAGLN-high/LGR6-low SMC signature)

## Evidence

- [methods] Giotto ligand-receptor signaling thresholding: "we selected top and reliable ligand-receptor signaling with the following threshold: p.adj < 0.25, abs(log2fc) > 0.1, lig_nr > 10, rec_nr > 10, lig_expr > 0.5 & rec_expr > 0.5"
- [intro] motivation for single-cell resolution: "considering the larger spot size of 55 μm, a Visium spot often contains multiple cells, which limits its usage in resolving detailed tissue structure"
- [abstract] downstream communication analysis capability: "facilitating downstream analysis, including detecting cellular communication through ligand-receptor interactions, localizing cellular subtypes"
- [readme] SpatialScope produces the single-cell-resolution input: "A unified approach for integrating spatial and single-cell transcriptomics data by leveraging deep generative models"
