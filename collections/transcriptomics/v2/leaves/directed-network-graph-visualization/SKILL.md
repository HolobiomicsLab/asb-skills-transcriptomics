---
name: directed-network-graph-visualization
description: Use when a TF–gene edge list (tf/gene/weights columns) from scMEGA or
  similar eGRN inference must be visualized as a directed, weighted network with TF
  nodes annotated by trajectory position.
when_to_use_negative:
- The edge list has not been filtered to correlations above 0.4 — plotting unfiltered
  edges will show noise interactions
- The network is undirected or edges lack weights (no tf/gene/weights schema) — a
  directed weighted rendering would misrepresent the data
- No per-TF trajectory/time-point annotation exists to color nodes by, removing the
  main rationale for this visualization
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3170
tools:
- name: scMEGA
  role: Provides GRNPlot and the GetGRN-generated edge list that is visualized as
    a directed weighted network
  repo: github.com/CostaLab/scMEGA
- name: igraph
  role: Constructs the directed graph from tf/gene/weights edges for rendering
- name: ggraph
  role: ggplot2-based graph layout and drawing of the directed, weighted TF–gene network
- name: R
  role: Implementation language for the network construction and visualization
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:3aab9ab71a7d53895a8a457c84becf662bf02f679ff77ca550e23656d57211fc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Signac
  - ggraph
  - igraph
  - scMEGA
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# directed-network-graph-visualization

## Summary

Renders an inferred, directed, weighted gene regulatory network (eGRN) as a graph with igraph/ggraph, coloring TF nodes by trajectory time point and labeling TFs, so regulatory structure across a pseudo-time trajectory is inspectable.

## When to use

Use when a TF–gene edge list (tf/gene/weights columns) from scMEGA or similar eGRN inference must be visualized as a directed, weighted network with TF nodes annotated by trajectory position.

## When NOT to use

- The edge list has not been filtered to correlations above 0.4 — plotting unfiltered edges will show noise interactions
- The network is undirected or edges lack weights (no tf/gene/weights schema) — a directed weighted rendering would misrepresent the data
- No per-TF trajectory/time-point annotation exists to color nodes by, removing the main rationale for this visualization

## Inputs

- final_grn.Rds — filtered GRN edge list data frame with tf, gene, and weights columns (weights = TF–gene correlation > 0.4)
- Trajectory/time-point annotation per TF from the pseudo-time trajectory (e.g. SCARA5+ to myofibroblast trajectory in a Seurat object)

## Outputs

- Directed, weighted network plot (igraph/ggraph via GRNPlot) with TF nodes colored by trajectory time point and TF labels shown
- igraph graph object with directed TF-to-gene edges and weights attributes

## How to apply

Take the filtered GRN edge list saved as final_grn.Rds, which contains only the tf, gene, and correlation columns (correlation renamed to weights) with all weights above the 0.4 correlation threshold. Build a directed graph from these TF-to-gene edges using igraph, then render it with GRNPlot/ggraph so edge weights carry through as edge attributes. Color TF nodes by their trajectory time point (from the ArchR-style pseudo-time trajectory, e.g. SCARA5+ fibroblasts to myofibroblasts) and display TF labels so regulator identity and trajectory ordering are visible. Validate that the graph is directed, edges point TF-to-gene, and every edge weight exceeds 0.4.

## Related tools

- **scMEGA** (Provides GRNPlot and the GetGRN-generated edge list that is visualized as a directed weighted network) — github.com/CostaLab/scMEGA
- **igraph** (Constructs the directed graph from tf/gene/weights edges for rendering)
- **ggraph** (ggplot2-based graph layout and drawing of the directed, weighted TF–gene network)
- **R** (Implementation language for the network construction and visualization)

## Examples

```
grn <- readRDS("final_grn.Rds"); GRNPlot(grn, color.by = "timepoint", labels = TRUE)
```

## Evaluation signals

- final_grn.Rds exists and contains directed TF-to-gene edges with tf/gene/weights fields
- All retained edge weights exceed the 0.4 correlation threshold
- Rendered graph is directed with edges pointing from TF to target gene
- TF nodes are colored by trajectory time point and TF labels are displayed
- Edges in the plot match the rows of the filtered edge list (no additional or dropped edges)

## Limitations

- Software/package versions are not specified in the tutorials (only sessionInfo() is referenced), so reproducibility of the rendering may vary
- Visualization depends on upstream trajectory inference and motif matching quality; no changelog available to track changes to GRNPlot behavior

## Evidence

- [other] Render the directed, weighted network with GRNPlot (igraph/ggraph), coloring TF nodes by their trajectory time point and showing TF labels.: [evidence span withheld — anchor status: not_found]
- [other] Filter the resulting edge list to correlations above 0.4, keeping only the tf, gene, and correlation columns and renaming correlation to weights.: [evidence span withheld — anchor status: not_found]
- [abstract] scMEGA enables end-to-end analysis of multi-omics data for gene regulatory network inference including modalities integration, trajectory analysis, enhancer-to-promoter association, network analysis and visualization: [evidence span withheld — anchor status: not_found]
- [methods] we infer a pseudo-time trajectory from SCARA5+ fibroblasts to myofibroblast using the approach from [ArchR]: "we infer a pseudo-time trajectory from SCARA5+ fibroblasts to myofibroblast using the approach from [ArchR]"
