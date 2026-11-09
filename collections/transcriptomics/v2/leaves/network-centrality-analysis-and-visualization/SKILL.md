---
name: network-centrality-analysis-and-visualization
description: Use when a weighted TF-target network (edge table with correlation scores)
  has been inferred and you must identify the most important regulator(s) and visualize
  network modules, e.g. from scMEGA GetGRN output.
when_to_use_negative:
- The data are raw single-cell counts or a Seurat/Signac object without an inferred
  GRN — centrality analysis requires a TF-gene edge table first produced by GetGRN
- Edges lack correlation weights, so a correlation-based filter (e.g. > 0.4) and weighted
  centrality cannot be applied
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3170
tools:
- name: scMEGA
  role: Provides GetGRN to assemble the enhancer-based GRN, GRNPlot for network visualization,
    GRNHeatmap for module resolution, and PseudotimePlot for TF activity profiling
  repo: https://github.com/CostaLab/scMEGA
- name: igraph
  role: Graph representation and computation of betweenness and page-rank centrality
    scores on the filtered GRN
- name: ggraph
  role: Rendering of the GRN network visualization (GRNPlot with seed=42)
- name: Nebulosa
  role: Kernel-density visualization of gene expression features in the single-cell
    object supporting module/cluster interpretation
  repo: https://github.com/powellgenomicslab/Nebulosa
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad003
    title: 'scMEGA: single-cell multi-omic enhancer-based gene regulatory network
      inference'
  source_content_hash: sha256:b22f50687be4dac351a22e3c0e77c3948573b0373a73a8949c7a12d80baf19e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/scMEGA
  tools:
  - ArchR
  - JASPAR2020
  - Nebulosa
  - R
  - Signac
  - TFBSTools
  - destiny
  - devtools
  - ggraph
  - igraph
  - optmatch
  - scMEGA
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42105263157894735
n_claims: 19
---

# network-centrality-analysis-and-visualization

## Summary

Profile an inferred enhancer-based gene regulatory network (eGRN) as a graph: filter edges by TF-gene correlation, resolve regulatory modules, and rank transcription factors by betweenness and page-rank centrality to pinpoint key regulators such as RUNX1 in myofibroblast differentiation.

## When to use

Use when a weighted TF-target network (edge table with correlation scores) has been inferred and you must identify the most important regulator(s) and visualize network modules, e.g. from scMEGA GetGRN output.

## When NOT to use

- The data are raw single-cell counts or a Seurat/Signac object without an inferred GRN — centrality analysis requires a TF-gene edge table first produced by GetGRN
- Edges lack correlation weights, so a correlation-based filter (e.g. > 0.4) and weighted centrality cannot be applied

## Inputs

- GRN edge table from scMEGA GetGRN (motif.matching + TF-gene correlations + peak-to-gene links), stored as final_grn.Rds after filtering to correlation > 0.4
- chromVAR TF deviation scores and TF-gene correlation values along the pseudotime trajectory

## Outputs

- final_grn.Rds containing tf-gene-correlation edges
- GRNPlot network visualization (ggraph/igraph) of the filtered eGRN
- GRNHeatmap resolving regulatory modules
- Centrality ranking (betweenness and page-rank scores) identifying the top TF, e.g. RUNX1
- PseudotimePlot profiles of TF binding activity, TF expression, and target gene expression

## How to apply

Assemble the enhancer-based GRN with scMEGA GetGRN from motif.matching, TF-gene correlations, and peak-to-gene links, then filter edges to correlation > 0.4 and save the edge table as final_grn.Rds. Load the filtered edge table into igraph and visualize the network with GRNPlot (seed=42 for layout reproducibility, rendered via ggraph). Use GRNHeatmap to resolve regulatory modules — in the myocardial infarction fibroblast case this yields two modules matching the SCARA5+ progenitor and POSTN/COL1A1+ myofibroblast sub-clusters, with NR3C2 in module 1. Compute betweenness and page-rank centrality scores on the network (igraph) to rank TFs; the TF with the highest combined importance is called as the top regulator (RUNX1 in the case study). Complement centrality with trajectory profiling (PseudotimePlot) of the top TF's binding activity, expression, and target expression along pseudotime.

## Related tools

- **scMEGA** (Provides GetGRN to assemble the enhancer-based GRN, GRNPlot for network visualization, GRNHeatmap for module resolution, and PseudotimePlot for TF activity profiling) — https://github.com/CostaLab/scMEGA
- **igraph** (Graph representation and computation of betweenness and page-rank centrality scores on the filtered GRN)
- **ggraph** (Rendering of the GRN network visualization (GRNPlot with seed=42))
- **Nebulosa** (Kernel-density visualization of gene expression features in the single-cell object supporting module/cluster interpretation) — https://github.com/powellgenomicslab/Nebulosa

## Examples

```
scMEGA::GRNPlot(grn, seed = 42); igraph::page_rank(graph_from_data_frame(final_grn))
```

## Evaluation signals

- final_grn.Rds exists and contains tf-gene-correlation edges after the correlation > 0.4 filter
- GRNHeatmap resolves two regulatory modules matching the SCARA5+ progenitor and POSTN/COL1A1+ myofibroblast sub-clusters
- NR3C2 is assigned to module 1 and RUNX1 is ranked most important by betweenness/page-rank centrality
- GRNPlot layout is reproducible with seed=42
- PseudotimePlot shows NR3C2/RUNX1 binding activity and target expression changing along the progenitor-to-myofibroblast trajectory

## Limitations

- Software/package versions are not specified in the tutorials (only sessionInfo() is referenced), so reproducibility across versions is not guaranteed
- No changelog is available, so function/API drift in scMEGA cannot be tracked from the repository docs
- Module resolution and top-TF ranking depend on the correlation > 0.4 edge filter and on the pseudotime trajectory defined over clusters c(0, 2, 1); different thresholds or trajectories change the result

## Evidence

- [methods] RUNX1 ranked most important by betweenness/page-rank centrality: [evidence span withheld — anchor status: not_found]
- [abstract] network analysis and visualization in scMEGA: "enables an end-to-end analysis of multi-omics data for gene regulatory network inference including modalities integration, trajectory analysis, enhancer-to-promoter association, network analysis and"
- [abstract] case study on myofibroblast GRNs: "We provide a case study on gene regulatory networks controlling myofibroblast activation in human myocardial infarction."
- [readme] scMEGA installation from GitHub: "# Install scMEGA
devtools::install_github("CostaLab/scMEGA")"
