---
name: phylogenetic-tree-visualization
description: Use when a single-cell lineage-tracing dataset has a reconstructed phylogeny
  (e.g. CRISPR-Cas9 lineage trees from Cassiopeia) that must be interactively browsed,
  selected, and overlaid with UMAP/tSNE projections and metadata in a shareable web
  report.
when_to_use_negative:
- Only a static, publication-figure tree is needed — iTOL is a web-server tool for
  uploading tree structures and annotation files for interactive viewing without an
  R analysis session
- No lineage/phylogenetic data are available and only scRNA-seq expression interpretation
  is required — plain VISION handles signature enrichment without a tree
- The tree exceeds the tested scale (~4,000 leaves), where low-latency selections
  and subtree collapsing were validated
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0069
tools:
- name: PhyloVision
  role: R package whose viewResults call builds the interactive web report containing
    the radial/linear phylogeny viewer
  repo: https://github.com/YosefLab/VISION
- name: VISION
  role: Front-end framework on which the interactive web-based report (including the
    phylogeny display) is built
  repo: https://github.com/YosefLab/VISION
- name: Cassiopeia
  role: Upstream tool used to reconstruct the single-cell phylogenies that PhyloVision
    visualizes
- name: iTOL
  role: Alternative web-server tool for uploading tree structures and annotation files
    for interactive viewing (no joint expression analysis)
- name: CeLaVi
  role: Alternative tool generating interactive web-based reports for cell-lineage
    viewing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:9719fe9e9569a891a74f1b7b80ebd58cc9dafa10593e9321336e3f4e56430fa4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - Cassiopeia
  - CeLaVi
  - PhyloVision
  - R
  - VISION
  - iTOL
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0069
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# phylogenetic-tree-visualization

## Summary

Render single-cell lineage phylogenies as an interactive radial or linear tree display embedded in a shareable web-based report (via PhyloVision's PhyloPlot.js on the VISION front-end), supporting low-latency cell selections and subtree collapsing for exploratory lineage-expression analysis.

## When to use

Use when a single-cell lineage-tracing dataset has a reconstructed phylogeny (e.g. CRISPR-Cas9 lineage trees from Cassiopeia) that must be interactively browsed, selected, and overlaid with UMAP/tSNE projections and metadata in a shareable web report.

## When NOT to use

- Only a static, publication-figure tree is needed — iTOL is a web-server tool for uploading tree structures and annotation files for interactive viewing without an R analysis session
- No lineage/phylogenetic data are available and only scRNA-seq expression interpretation is required — plain VISION handles signature enrichment without a tree
- The tree exceeds the tested scale (~4,000 leaves), where low-latency selections and subtree collapsing were validated

## Inputs

- Analyzed VISION/PhyloVision R object containing a reconstructed single-cell phylogeny (e.g. Newick tree inferred with Cassiopeia)
- Cell metadata and two-dimensional projections (UMAP/tSNE) added via addProjection
- Optional saved report state JSON files (differential-expression cache and selections) for load_de_cache / load_selections

## Outputs

- phylovision_report.html web report with interactive radial/linear phylogeny (PhyloPlot.js)
- Accompanying JSON state files encoding selections and analysis results
- UMAP/tSNE overlay panels and collapsable signature/Hotspot module tables (FDR < 0.05)

## How to apply

Load the analyzed VISION/PhyloVision object in R (package from github.com/YosefLab/VISION), optionally restoring prior report state with load_de_cache and load_selections, then invoke viewResults to build and serve the dynamic report; when hosting on a remote server run it on a chosen port with host='0.0.0.0' and browser=FALSE. The tree renders through PhyloPlot.js as a radial or linear phylogeny with attached UMAP/tSNE overlay panels, and supports low-latency leaf selections and subtree collapsing — validated on trees up to ~4,000 leaves. Cells can be grouped via breadth-first search over internal nodes (Cluster-Phylogeny, target := 10) so subtrees correspond to phylogenetic clusters. Validate that the emitted report assets (phylovision_report.html plus JSON state) exist, parse as HTML, and contain the collapsable signature/Hotspot module tables (FDR < 0.05 entries), the phylogeny display, and the projection panels. Unlike static alternatives (iTOL, CeLaVi), the report is fully interactive and shareable for joint exploration of tree structure and expression data.

## Related tools

- **PhyloVision** (R package whose viewResults call builds the interactive web report containing the radial/linear phylogeny viewer) — https://github.com/YosefLab/VISION
- **VISION** (Front-end framework on which the interactive web-based report (including the phylogeny display) is built) — https://github.com/YosefLab/VISION
- **Cassiopeia** (Upstream tool used to reconstruct the single-cell phylogenies that PhyloVision visualizes)
- **iTOL** (Alternative web-server tool for uploading tree structures and annotation files for interactive viewing (no joint expression analysis))
- **CeLaVi** (Alternative tool generating interactive web-based reports for cell-lineage viewing)

## Examples

```
viewResults(phylo_object, port = 8123, host = "0.0.0.0", browser = FALSE)
```

## Evaluation signals

- phylovision_report.html and accompanying JSON files exist (file_exists check)
- Report assets parse as valid HTML (format_is check)
- Report contains the radial/linear phylogeny display (PhyloPlot.js) and UMAP/tSNE overlay panels (contains_substring / expert_review)
- Collapsable signature/Hotspot module tables present with FDR < 0.05 entries
- Tree interactions (selections, subtree collapsing) remain low-latency for trees up to ~4,000 leaves

## Limitations

- Dynamic phylogeny viewer was only tested on trees of up to approximately 4,000 leaves
- R implementation may pose challenges for interoperation with separate Python software such as Cassiopeia
- Serving viewResults on a remote server requires host='0.0.0.0' and browser=FALSE; local hosting otherwise assumed

## Evidence

- [results] PhyloVision's dynamic phylogeny viewer was tested on trees of up to approximately 4,000 leaves with low-latency selections and subtree collapsing: "allowing low-latency selections and subtree collapsing for large trees (we tested trees of up to ∼4,000 leaves)"
- [results] Web report built on VISION front-end: "Our interactive web-based report is built on our VISION front-end (DeTomaso et al., 2019)"
- [abstract] Interactive shareable web report: "PhyloVision provides a feature-rich, interactive, and shareable web-based report for investigating these modules"
- [methods] Clustering cells on the phylogeny for subtree display: "PhyloVision clusters cells on a phylogeny by performing a breadth-first search over internal nodes"
- [intro] iTOL as a contrasting static tree-viewing tool: "the interactive Tree of Life (iTOL; Letunic and Bork, 2021) is a scalable web-server-based tool that allows users to upload tree structures"
