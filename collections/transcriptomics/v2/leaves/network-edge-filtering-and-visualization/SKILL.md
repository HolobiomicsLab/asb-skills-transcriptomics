---
name: network-edge-filtering-and-visualization
description: Use when a TF-gene correlation/eGRN edge table has been computed (e.g.
  via scMEGA GetTFGeneCorrelation) and needs threshold-based pruning and network visualization.
when_to_use_negative:
- The network was not derived from enhancer-based multi-omic data (motif matching
  + chromVAR + peak-to-gene links); a generic correlation-matrix heatmap tool would
  suffice.
- Edges have no correlation scores to threshold on (e.g. an already-binary adjacency
  matrix) — filtering by correlation > 0.5 is meaningless.
- The goal is upstream candidate selection rather than network presentation/recovery
  benchmarking; use SelectTFs (cor.cutoff = 0.4) and SelectGenes first.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: scMEGA
  role: Provides GetTFGeneCorrelation, GetGRN, GRNHeatmap, and GRNPlot used to filter
    and visualize the eGRN
  repo: https://github.com/CostaLab/scMEGA
- name: igraph
  role: Graph construction and layout underlying GRNPlot network rendering
- name: ggraph
  role: ggplot2-based graph visualization used by GRNPlot
- name: Seurat
  role: Holds the RNA and chromVAR assays and trajectory used to compute TF-gene correlations
  repo: https://github.com/satijalab/seurat
- name: Signac
  role: Single-cell ATAC-seq object infrastructure supporting motif and peak assays
    in the paired object
  repo: https://github.com/stuart-lab/signac
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:974bffe364a26eaeb9c999e2776109bdb8577321f3771432ee48b98ff331239d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/scMEGA
  tools:
  - ArchR
  - Seurat
  - Signac
  - ggraph
  - igraph
  - scMEGA
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# network-edge-filtering-and-visualization

## Summary

Filter inferred enhancer-based gene regulatory network (eGRN) edges by TF-gene correlation threshold and visualize the resulting network as heatmaps and graphs. In the scMEGA workflow this step turns raw TF-gene correlation tables into interpretable, comparable network objects (igraph/ggraph plots and heatmaps) for downstream recovery-rate benchmarks.

## When to use

Use when a TF-gene correlation/eGRN edge table has been computed (e.g. via scMEGA GetTFGeneCorrelation) and needs threshold-based pruning and network visualization.

## When NOT to use

- The network was not derived from enhancer-based multi-omic data (motif matching + chromVAR + peak-to-gene links); a generic correlation-matrix heatmap tool would suffice.
- Edges have no correlation scores to threshold on (e.g. an already-binary adjacency matrix) — filtering by correlation > 0.5 is meaningless.
- The goal is upstream candidate selection rather than network presentation/recovery benchmarking; use SelectTFs (cor.cutoff = 0.4) and SelectGenes first.

## Inputs

- TF-gene correlation table from GetTFGeneCorrelation (chromVAR TF assay vs RNA gene assay along a trajectory)
- eGRN object from GetGRN combining motif matching, TF-gene correlations, and peak-to-gene links
- Cell trajectory object (e.g. Seurat/Signac object with AddTrajectory result)

## Outputs

- Filtered edge table retaining only TF-gene pairs with correlation > 0.5
- Network heatmap (GRNHeatmap) of TF-gene regulatory relationships
- Graph plot (GRNPlot) of the eGRN built with igraph/ggraph
- Recovery-rate summary comparing predicted-pair vs true-pair networks (e.g. cd4_tcell_grn_recovery_rates.csv)

## How to apply

After assembling eGRNs from motif matching (AddMotifs/JASPAR2020), TF activity deviation scores (RunChromVAR), and peak-to-gene links, compute TF-gene correlations along pseudotime with GetTFGeneCorrelation (tf.assay = 'chromvar', gene.assay = 'RNA'). Prune the edge table by keeping only edges with correlation > 0.5, which removes weak TF-gene regulations before visualization. Build a graph from the filtered edges and render it with scMEGA's GRNHeatmap and GRNPlot, which wrap igraph and ggraph. When the same network is built from two pairing sources (e.g. OptMatch-predicted vs true cell pairs), run the identical filtering on both and compare edge sets to compute recovery rates for TFs, genes, and TF-gene regulations.

## Related tools

- **scMEGA** (Provides GetTFGeneCorrelation, GetGRN, GRNHeatmap, and GRNPlot used to filter and visualize the eGRN) — https://github.com/CostaLab/scMEGA
- **igraph** (Graph construction and layout underlying GRNPlot network rendering)
- **ggraph** (ggplot2-based graph visualization used by GRNPlot)
- **Seurat** (Holds the RNA and chromVAR assays and trajectory used to compute TF-gene correlations) — https://github.com/satijalab/seurat
- **Signac** (Single-cell ATAC-seq object infrastructure supporting motif and peak assays in the paired object) — https://github.com/stuart-lab/signac

## Examples

```
obj <- GetGRN(obj, tf.assay = 'chromvar', gene.assay = 'RNA', trajectory.name = 'Trajectory'); p <- GRNPlot(grn = obj@misc$GRN)
```

## Evaluation signals

- Filtered edge table contains only TF-gene pairs with correlation > 0.5; spot-check the maximum/minimum retained correlations.
- Graph from igraph has node counts consistent with the candidate TF and gene sets (SelectTFs cor.cutoff = 0.4 candidates).
- Recovery rates when benchmarking predicted vs true pairs meet reported thresholds: >75% of TFs, 83% of genes, and 60% of TF-gene regulations recovered.
- GRNHeatmap and GRNPlot render without errors and show the expected TF-gene structure after edge filtering.
- Identical filtering applied to predicted-pair and true-pair networks yields comparable edge tables saved with the recovery-rate summary (cd4_tcell_grn_recovery_rates.csv).

## Limitations

- Package versions are not pinned in the tutorials; results may vary across Seurat v5 / Signac / scMEGA versions (the paper only notes 'Check session information sessionInfo()').
- Recovery rates depend on the quality of upstream cell pairing; the reported 75%/83%/60% thresholds were validated for OptMatch-predicted CD4 T cell pairs on the 10x PBMC multiome dataset.
- The correlation > 0.5 edge cutoff is a single heuristic threshold; no sensitivity analysis of alternative cutoffs is documented.

## Evidence

- [methods] Filter network edges (correlation > 0.5) and visualize with GRNHeatmap/GRNPlot (igraph/ggraph): [evidence span withheld — anchor status: not_found]
- [methods] Compute TF-gene correlations along the pseudotime with GetTFGeneCorrelation: [evidence span withheld — anchor status: not_found]
- [abstract] enables an end-to-end analysis of multi-omics data for gene regulatory network inference including modalities integration, trajectory analysis, enhancer-to-promoter association, network analysis and: "enables an end-to-end analysis of multi-omics data for gene regulatory network inference including modalities integration, trajectory analysis, enhancer-to-promoter association, network analysis and"
- [readme] Tutorials are available from https://costalab.github.io/scMEGA: "Tutorials are available from https://costalab.github.io/scMEGA"
- [methods] we first estimate an acitivty score for each TF in each cell. This is done by first performing motif matching and then computing deviation scores: "we first estimate an acitivty score for each TF in each cell. This is done by first performing motif matching and then computing deviation scores"
