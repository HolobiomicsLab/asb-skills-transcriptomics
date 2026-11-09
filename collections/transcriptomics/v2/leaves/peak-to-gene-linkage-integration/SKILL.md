---
name: peak-to-gene-linkage-integration
description: Use when building an enhancer-based gene regulatory network from single-cell
  multiome data, where candidate TF–gene pairs must be filtered by enhancer–promoter
  linkage plus TF motif binding in linked peaks.
when_to_use_negative:
- Inputs lack a peak-to-gene link table or motif occurrence matrix — correlation-only
  TF–gene inference cannot satisfy the enhancer-based retention rule
- The goal is cell-level TF activity scoring rather than network assembly — RunChromVAR
  deviation scores alone are the appropriate earlier step
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0749
tools:
- name: scMEGA
  role: Provides GetGRN and supporting functions to intersect motif matching, TF–gene
    correlations, and peak-to-gene links into the eGRN
  repo: github.com/CostaLab/scMEGA
- name: Seurat
  role: Hosts the multiomic Seurat object whose ATAC motifs assay supplies the motif
    occurrence matrix
  repo: github.com/satijalab/seurat (branch seurat5)
- name: Signac
  role: Chromatin data structures underlying the peak/motif assays in the paired object
  repo: github.com/stuart-lab/signac (branch seurat5)
- name: igraph
  role: Graph construction for rendering the directed, weighted TF-to-gene network
- name: ggraph
  role: Visualization backend for GRNPlot network rendering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:5dd86977376121769b9e61fd86f5badd20cc41949e12f5bcf3a9281708121599
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - Signac
  - ggraph
  - igraph
  - scMEGA
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# peak-to-gene-linkage-integration

## Summary

Assembles scMEGA's enhancer-based GRN (eGRN) by intersecting peak-to-gene links with a TF–gene correlation table and a motif-matching matrix: a TF–gene edge is retained only when the gene has at least one linked enhancer and that enhancer is bound by the TF.

## When to use

Use when building an enhancer-based gene regulatory network from single-cell multiome data, where candidate TF–gene pairs must be filtered by enhancer–promoter linkage plus TF motif binding in linked peaks.

## When NOT to use

- Inputs lack a peak-to-gene link table or motif occurrence matrix — correlation-only TF–gene inference cannot satisfy the enhancer-based retention rule
- The goal is cell-level TF activity scoring rather than network assembly — RunChromVAR deviation scores alone are the appropriate earlier step

## Inputs

- Trajectory Seurat/Signac object with ATAC motifs assay (obj@assays$ATAC@motifs@data)
- Peak-to-gene link data frame (df.p2g) with >=2 kb enhancer-to-gene links
- TF–gene correlation data frame (df.cor) from chromVAR deviation-score/correlation analysis

## Outputs

- final_grn.Rds: directed weighted TF-to-gene edge list with columns tf, gene, weights (all weights > 0.4)
- Directed, weighted GRN visualization via GRNPlot (igraph/ggraph) with TF nodes colored by trajectory time point

## How to apply

Extract the motif occurrence matrix from the ATAC motifs assay of the trajectory Seurat object (obj@assays$ATAC@motifs@data), subset it to peaks present in the peak-to-gene link table and to TFs retained in the TF–gene correlation table (df.cor). Apply scMEGA's GetGRN function with the motif matching matrix, df.cor, and df.p2g so a gene links to a TF only when an associated peak is bound by that TF and the TF–gene correlation supports the regulation. Filter the resulting edge list to correlations above 0.4, keeping tf/gene/correlation columns and renaming correlation to weights, then save the edge list as final_grn.Rds. Rationale: the enhancer-based rule anchors regulation to cis-regulatory evidence rather than correlation alone, combining chromVAR-derived TF activity, motif matching, and peak-to-gene association computed earlier in the scMEGA workflow.

## Related tools

- **scMEGA** (Provides GetGRN and supporting functions to intersect motif matching, TF–gene correlations, and peak-to-gene links into the eGRN) — github.com/CostaLab/scMEGA
- **Seurat** (Hosts the multiomic Seurat object whose ATAC motifs assay supplies the motif occurrence matrix) — github.com/satijalab/seurat (branch seurat5)
- **Signac** (Chromatin data structures underlying the peak/motif assays in the paired object) — github.com/stuart-lab/signac (branch seurat5)
- **igraph** (Graph construction for rendering the directed, weighted TF-to-gene network)
- **ggraph** (Visualization backend for GRNPlot network rendering)

## Examples

```
grn <- GetGRN(motif.mat, df.cor, df.p2g); grn.f <- subset(grn, correlation > 0.4)[, c('tf','gene','correlation')]; names(grn.f)[3] <- 'weights'; saveRDS(grn.f, 'final_grn.Rds')
```

## Evaluation signals

- final_grn.Rds exists and contains directed TF-to-gene edges with tf/gene/weights fields
- All retained edge weights exceed the 0.4 correlation threshold
- Every retained TF–gene edge has at least one associated enhancer bound by that TF (the enhancer-based criterion)
- Network renders as a directed graph with TF nodes colored by trajectory time point and TF labels shown

## Limitations

- Software/package versions were not specified in the tutorials (only sessionInfo() is referenced), so reproducibility depends on the developer branch versions installed via devtools
- The retention rule depends on quality of upstream peak-to-gene links and motif matching; no changelog or discussion of edge-case behavior was found in the article

## Evidence

- [methods] GetGRN retention rule: [evidence span withheld — anchor status: not_found]
- [abstract] end-to-end scope: "enables an end-to-end analysis of multi-omics data for gene regulatory network inference including modalities integration, trajectory analysis, enhancer-to-promoter association, network analysis and"
- [readme] installation workflow: "We provided the following tutorials to show how to use scMEGA to build GRN by using single-cell multiomics/multimodal data"
- [methods] TF activity estimation: "we first estimate an acitivty score for each TF in each cell. This is done by first performing motif matching and then computing deviation scores"
