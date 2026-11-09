---
name: dot-plot-visualization-of-annotation-results
description: Use when EasyCellType has produced annotation test results (from easyct
  with test='GSEA' or Fisher) and you need a dot plot plus top-five candidate cell-type
  rankings per cluster to judge annotation quality.
when_to_use_negative:
- Marker lists were assembled by simple gene-list overlap without ranking information
  — EasyCellType's GSEA/visualization assumes ranked scores and 'simply overlapping
  the two lists without considering gene ranking might lead to unreliable results'
- Input is not cluster-level marker output from a differential-expression step (e.g.
  already-annotated cell-type labels or raw counts) — this skill visualizes annotation
  test results, not raw data
- Comparing annotation tools at the algorithm level rather than inspecting one tool's
  outputs (e.g. benchmarking against scCATCH's findmarkergene/findcelltype)
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: EasyCellType
  role: Provides plot_dot() to render the overall dot plot of GSEA/Fisher annotation
    results and the easyct() results object that is visualized
  repo: https://github.com/rx-li/EasyCellType
- name: Seurat
  role: Upstream source of cluster markers (FindAllMarkers with only.pos=TRUE, min.pct=0.25,
    logfc.threshold=0.25) that feed the annotation results being visualized
- name: ggplot2
  role: Graphics backend used to generate the dot-plot visualizations in R
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad029
    title: 'EasyCellType: marker-based cell-type annotation by automatically querying
      multiple databases'
  source_content_hash: sha256:2584398f0a7563855d6b4123e9a5d7243f95384d412dd8a803e2dd11c4c8619a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rx-li/EasyCellType
  tools:
  - AnnotationDbi
  - CellMarker
  - EasyCellType
  - Seurat
  - clusterProfiler
  - ggplot2
  - org.Hs.eg.db
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# dot-plot-visualization-of-annotation-results

## Summary

Visualize EasyCellType cell-type annotation results as a dot plot summarizing GSEA or Fisher-exact-test scores per cluster/candidate cell type, so researchers can pick optimal annotations. This step turns ranked enrichment results (e.g. from querying Seurat markers against CellMarker restricted to brain) into a graphical ranking table of top candidate cell types.

## When to use

Use when EasyCellType has produced annotation test results (from easyct with test='GSEA' or Fisher) and you need a dot plot plus top-five candidate cell-type rankings per cluster to judge annotation quality.

## When NOT to use

- Marker lists were assembled by simple gene-list overlap without ranking information — EasyCellType's GSEA/visualization assumes ranked scores and 'simply overlapping the two lists without considering gene ranking might lead to unreliable results'
- Input is not cluster-level marker output from a differential-expression step (e.g. already-annotated cell-type labels or raw counts) — this skill visualizes annotation test results, not raw data
- Comparing annotation tools at the algorithm level rather than inspecting one tool's outputs (e.g. benchmarking against scCATCH's findmarkergene/findcelltype)

## Inputs

- EasyCellType results object from easyct() run with db='cellmarker', species='Human', tissue list (e.g. brain-restricted), and test='GSEA'
- Marker list data frame with columns Entrez ID, cluster and expression score (avg_log2FC), ranked by descending score within each cluster
- Expected/true cluster labels for validation (e.g. ASR-FB, AST-PP, Endothelial, Microglia, Oligodendrocytes, OPC)

## Outputs

- Overall dot plot of candidate cell types per cluster (plot_dot output)
- Ranking table of top-five candidate cell types per cluster

## How to apply

After running easyct on a marker list built from Seurat FindAllMarkers output (only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25, genes mapped to Entrez IDs via org.Hs.eg.db), pass the results object to plot_dot(test='GSEA', results) to generate the overall dot plot. The dot plot displays candidate cell types from the queried database (e.g. CellMarker, restricted to brain tissues) per input cluster, reflecting enrichment strength and gene-ranking information rather than simple list overlap. Extract the top-five ranked candidate cell types per cluster into a ranking table for explicit comparison against expected labels. Rationale: the package's visualization is designed to 'help the researchers to understand the results and pick the optimal annotations', and validation is done by checking that known clusters (e.g. ASR-FB, AST-PP, Endothelial, Microglia, Oligodendrocytes, OPC in the Velmeshev ASD dataset) are recovered as top-ranked candidates.

## Related tools

- **EasyCellType** (Provides plot_dot() to render the overall dot plot of GSEA/Fisher annotation results and the easyct() results object that is visualized) — https://github.com/rx-li/EasyCellType
- **Seurat** (Upstream source of cluster markers (FindAllMarkers with only.pos=TRUE, min.pct=0.25, logfc.threshold=0.25) that feed the annotation results being visualized)
- **ggplot2** (Graphics backend used to generate the dot-plot visualizations in R)

## Examples

```
plot_dot(test="GSEA", results)
```

## Evaluation signals

- The dot plot renders candidate cell types per input cluster for every cluster present in the results object, with no missing-cluster rows
- Extracted top-five ranking table contains exactly five candidate cell types per cluster where at least five candidates exist
- Validation: for the Velmeshev ASD dataset case, six clusters (ASR-FB, AST-PP, Endothelial, Microglia, Oligodendrocytes, OPC) are correctly annotated by the GSEA approach, and neuronal clusters receive candidates consistent with their true labels
- Ranking order in the table matches the enrichment scores shown in the dot plot (descending score within each cluster)
- Annotation is based on ranked gene scores (avg_log2FC), not unranked overlap, consistent with the modified Fisher's exact test / GSEA design

## Limitations

- Simply overlapping gene lists without considering gene ranking might lead to unreliable results, so the dot plot should only be interpreted for ranked marker-score inputs
- The tool provides annotation recommendations in graphical outcomes rather than definitive labels; researchers must pick optimal annotations themselves
- Only three public marker gene databases are referenced (the article does not fully enumerate the tissue database names), restricting candidate cell-type vocabularies

## Evidence

- [intro] Visualization of test results: "The package generates straight-forward visualization of the test results, which help the researchers to understand the results and pick the optimal annotations"
- [abstract] Two statistical tests underlying the plotted results: "The package provides two statistical tests, gene set enrichment analysis and a modified version of Fisher’s exact test"
- [abstract] Ranked marker input requirement: "Additionally, simply overlapping the two lists without considering gene ranking might lead to unreliable results."
- [methods] easyct invocation that produces the results object: "annot.GSEA <- easyct(input.d, db="cellmarker", species="Human","
- [readme] README restates visualization purpose: "The package generates straight-forward visualization of the test results, which help the researchers to understand the results and pick the optimal annotations."
